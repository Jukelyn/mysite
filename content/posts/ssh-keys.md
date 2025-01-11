+++
draft = false
date = "2024-03-15"
title = "All About SSH Keys"
description = "How to create and use SSH keys to remote login into machines."
slug = "ssh-keys"
authors = ["Jukelyn"]
categories = ["SSH"]
+++

[Jump to Updates]({{< ref "#updates" >}})
### What is SSH?

SSH, or Secure Shell, is a cryptographic network protocol used for secure communication between two computers over an unsecured network. It provides a secure channel over an insecure network by encrypting the data transmitted between the client and the server.

SSH keys offer a more secure and convenient way to authenticate to remote servers compared to traditional password-based authentication. They are widely used in various scenarios, including server administration, file transfers, and remote command execution.

### Creating an SSH key

SSH keys can be generated using the ssh-keygen[^1] command on Linux, MacOS, and Windows. Whenever I generate SSH keys, I like to use the *rsa*[^2] encryption algorithm, there other flags[^3] we can use as well. Another flag that I like to use is the *b* flag to specify the amount of bits in the key to create, I like 4096 bits. Finally, I like to specify the name of the key at this point using the *f* flag. So my complete command looks like this: 
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/example
```
Once we run this command, we will be asked to add a passphrase and then confirm it. I choose to not use passphrases as I feel that the SSH key is strong enough. If you feel like, for whatever reason, somebody will get their hands on your key, feel free to add the passphrase as an extra layer of security.

### Copying SSH public key to a server

Now that we have created an SSH key, say *~/.ssh/example* and *~/.ssh/example.pub*, we will want to copy the *public* key to the server we will use the SSH key with. For this example, the server we will be connecting to is at the private IP `192.168.1.1` and has a user named `juke` that I have the password to. To copy the public key to the server we will be using the ssh-copy-id[^4] command. We will use the *i* flag to specify which key will be copying over.
```bash
ssh-copy-id -i ~/.ssh/example juke@192.168.1.1
```
You will be prompted to enter the password of the `juke` account and upon completion, the key will be copied onto the server.

### Logging In Using SSH Keys + Next Steps

Now that we have an SSH public key on the server and we have the private key on our system, we can remotely connect to the server using SSH.
> Note: We have not done any kind of Port Forwarding so, at the moment, a connection is only possible when connected to the same LAN (same network). Port Forwarding is mentioned a bit later but I will not be going over how to set that up as it depends on the hardware that you have.

To connect to the server, we can just do the following command:
```bash
ssh -i ~/.ssh/example juke@192.168.1.1
```
Once we are connected to the machine, if this is a machine that you are an admin of, it is a good idea to turn off password authentication and also change the default SSH port from 22 to something else. Once you change the port, be sure to setup Port Forwarding on your router to match so that you are able to connect to the machine from outside the network.

### Changing the Default ssh Port
Changing the default SSH port (typically port 22) is a common security practice aimed at reducing the risk of automated attacks. It's important for a few reasons that include:

1. Reduction of Brute Force Attacks
    - Many automated bots and attackers scan for open ports, especially port 22, to target systems running SSH. By changing the port, you're making it harder for these automated bots to find your SSH service. This doesn't prevent targeted attacks but adds a layer of obscurity.
2. Reduce Log Spam
    - Since most attacks focus on port 22, changing the port reduces the number of brute-force login attempts and failed login attempts that clog up your logs. This makes it easier to spot real security issues.
3. Security by Obscurity
    - While changing the default SSH port doesn't technically enhance the security of your system by itself (it’s more about obscurity), it adds a basic layer of defense. If attackers don't know which port SSH is running on, they have to scan for it, which takes more time and may deter less sophisticated attacks.
4. Limit Exposure
    - Changing the SSH port to a non-standard port can reduce the exposure of your SSH service to opportunistic attacks that rely on default configurations. Attackers tend to target widely known ports, so a custom port might avoid some attention.
5. Integration with Firewalls
    - By changing the SSH port and locking down other ports through firewalls, you make your server more secure. This way, attackers would need to know the port you're using to even begin attacking the service.

While this is a useful security measure, it's still important to implement additional security practices like:

    - Using strong passwords or key-based authentication
    - Disabling password-based authentication entirely and ONLY using key-based auth
    - Enabling firewall rules to limit access to the SSH port

That all being said, here's how to change the default ssh port. Firstly, make sure you are logged in the server and use a text editor of your choice to edit the SSH config file. I am, and most others are, using sshd so that would be `sudo nano /etc/ssh/sshd_config`. In this file you will see something similar to below:

```bash
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
...
```

Simply uncomment and change the line with "Port 22" to something like "Port 44291". Once this is done, save and exit the file. Now all you have to do is restart the sshd service to finalize the changes by running `sudo systemctl restart sshd` or `sudo systemctl restart ssh`. You should also change your firewall rules to allow tcp traffic on this port and then disable port 22. Using ufw this is achieved by:
```bash
sudo ufw allow 44291/tcp && sudo ufw deny 22/tcp
```

You may also benefit from rebooting the server using `sudo reboot`. Now when you connect via ssh, make sure you add the `p` flag and specify the port that you set. For example, with port 44291 set and the above server IP, the login would now look like:

```bash
ssh -i ~/.ssh/example -p 44291 juke@192.168.1.1
```

Make sure to update your Port Forwarding on your router to use this new internal port that you have set, if you have not already. You can even configure the Port Forwarding to use a different external port that is forward to this obfuscated port, for example, forwarding 15683 to 44291. Then if you are going to ssh from outside your network you would need to do:

```bash
ssh -i ~/.ssh/example -p 15683 juke@[your public IP]
```


### Updates
Any updates being made will be below. Here is a table with jumps to various dates with a short description of changes made:

> Date: Short description
>
> [7 September 2024]({{< ref "#7-September-2024" >}}): Changing default ssh port

#### 7 September 2024
Added information on changing the default ssh port.

[^1]: Learn more about `ssh-keygen` [here](https://linux.die.net/man/1/ssh-keygen)
[^2]: Learn more about SSH key encryption algorithms [here](https://goteleport.com/blog/comparing-ssh-keys/)
[^3]: Learn more about flags [here](https://www.ibm.com/docs/pt/aix/7.3?topic=names-command-flags)
[^4]: Learn more about `ssh-copy-id` [here](https://linux.die.net/man/1/ssh-copy-id)