---
layout: post
title:  "Beginner’s guide to understand ssh and setting up ssh on ubuntu"
categories: [ Linux ]
image: assets/images/ssh-tunnel.gif
---

This article covers the SSH client on the Linux Operating System.

Note: This article is one of the top tutorials covering SSH on the Internet. It was originally written back in 1999.

There are a couple of ways that you can access a shell (command line) remotely on most Linux/Unix systems. One of the older ways is to use the telnet program, which is available on most network capable operating systems. Accessing a shell account through the telnet method though poses a danger in that everything that you send or receive over that telnet session is visible in plain text on your local network, and the local network of the machine you are connecting to. So anyone who can “sniff” the connection in-between can see your username, password, email that you read, and commands that you run. For these reasons you need a more sophisticated program than telnet to connect to a remote host.

SSH, which is an acronym for Secure SHell, was designed and created to provide the best security when accessing another computer remotely. Not only does it encrypt the session, it also provides better authentication facilities, as well as features like secure file transfer, X session forwarding, port forwarding and more so that you can increase the security of other protocols. It can use different forms of encryption ranging anywhere from 512 bit on up to as high as 32768 bits and includes ciphers like AES (Advanced Encryption Scheme), Triple DES, Blowfish, CAST128 or Arcfour. Of course, the higher the bits, the longer it will take to generate and use keys as well as the longer it will take to pass data over the connection.

![openssl_1]({{ site.baseurl }}/assets/images/slide11.jpg)
![openssl_2]({{ site.baseurl }}/assets/images/slide1.jpg)

These two diagrams on the left show how a telnet session can be viewed by anyone on the network by using a sniffing program like Ethereal (now called Wireshark) or tcpdump. It is really rather trivial to do this and so anyone on the network can steal your passwords and other information. The first diagram shows user jegadezz logging in to a remote server through a telnet connection. He types his username jegadezz, which are viewable by anyone who is using the same networks that he is using.

The second diagram shows how the data in an encrypted connection like SSH is encrypted on the network and so cannot be read by anyone who doesn’t have the session-negotiated keys, which is just a fancy way of saying the data is scrambled. The server still can read the information, but only after negotiating the encrypted session with the client. 

Client-Side Installation 

To install SSH client on Ubuntu by by executing the following command: 

sudo apt-get install openssh-client

Getting SSH installed is really easy, and only takes a few other bits of information to get going. On the computer which you’d like to use to connect to other computers, you’ll need to install the OpenSSH client if it isn’t already. On Ubuntu systems this can be done with sudo apt-get install openssh-client. Once that installation completes, you’re already good to go with one computer. 

To install SSH server on Ubuntu by by executing the following command: 

sudo apt-get install openssh-server

On every computer that you want to connect to, you’ll need to install the server-side part of the software if it isn’t already. You can do so on Ubuntu systems with the command sudo apt-get install openssh-server. Once this is installed, all of the needed software is installed. 