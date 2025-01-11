+++
draft = false
date = "2024-08-04"
title = "SSL/TLS Chain of Trust"
description = "What is SSL Chain of Trust and why should we care? No really... why?"
slug = "tls"
authors = ["Jukelyn"]
categories = ["TLS", "SSL", "webserver", "Website"]
+++

[Jump to Updates]({{< ref "#updates" >}})
### What is TLS/SSL?

TLS (Transport Layer Security) and SSL (Secure Sockets Layer) are cryptographic protocols designed to provide secure communication over a network. TLS was developed by the Internet Engineering Task Force  (IETF) in 1999 to be an upgrade/replacement for SSL which is was developed by Netscape in 1994.

### Why should anybody care?

Now that you know what TLS and SSL are, why should we care? Well first of all, it helps to understand the purpose SSL/TLS serves when it comes to modern webservers and network interfacing. There are three main functions of TLS which include the following:

1. Encryption: Encrypts the data being transmitted, ensuring that it cannot be read by unauthorized parties.
2. Authentication: Verifies the identity of the parties involved in the communication.
3. Integrity: Ensures that the data has not been tampered with during transmission.

### The TLS/SSL handshake process[^1]
1. Each TLS certificate consists of a key pair made of a public key and private key. These keys are important because they interact behind the scenes during website transactions.

2. Every time you visit a website, the client server and web browser communicate to ensure there is a secure TLS/SSL encrypted connection.

3. When a web browser (or client) directs to a secured website, the website server shares its TLS/SSL certificate and its public key with the client to establish a secure connection and a unique session key.

4. The browser confirms that it recognizes and trusts the issuer, or Certificate Authority, of the SSL certificate. The browser also checks to ensure the TLS/SSL certificate is unexpired, unrevoked, and that it can be trusted.

5. The browser sends back a symmetric session key and the server decrypts the symmetric session key using its private key. The server then sends back an acknowledgement encrypted with the session key to start the encrypted session.

6. Server and browser now encrypt all transmitted data with the session key. They begin a secure session that protects message privacy, message integrity, and server security.

### Updates
Any updates being made will be below. Here is a table with jumps to various dates with a short description of changes made:

> Date: Short description
>
> TBD: TBA

[^1]: Info is from [DigiCert](https://www.digicert.com/how-tls-ssl-certificates-work)
<!-- 
[^1]: Learn more about `ssh-keygen` [here](https://linux.die.net/man/1/ssh-keygen)
[^2]: Learn more about SSH key encryption algorithms [here](https://goteleport.com/blog/comparing-ssh-keys/)
[^3]: Learn more about flags [here](https://www.ibm.com/docs/pt/aix/7.3?topic=names-command-flags)
[^4]: Learn more about `ssh-copy-id` [here](https://linux.die.net/man/1/ssh-copy-id) -->
