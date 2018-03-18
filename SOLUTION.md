# Challenge Solution

We have the following environment:

 1. Client A: from a local network that can access any service
	 1.  This client can run any distribution with OpenSSH client. But, with this example, CentOS 7 is used.
 2. Server A: Is where Client A can connect. And connect only via SSH.
	 1. It runs CentOS 7
	 2. Is a router and firewall with INPUT DROP policy and only allows connections to port 22(ssh).
	 3. It has an apache httpd server serving the port 8000. But you cannot access from Client A due to the firewall rules.
	 4. It has two network interfaces. One for the Internet and the other for the internal network. We can access Server B from this internal network interface.
 3. Server B: Is like Server A. But it can be connected only from Server A.
	 1. Is like Server A. Runs CentOS 7, has the same firewall rules, except the masquereade part. It has only one network interface. It has an apache httpd server using the port 8000. But you cannot access from Client A because it is isolated by Server A. And you cannot access from Server A due to the firewall rules.

We have to consider that all three machines have the same user. So we don't need to use user@server when connecting with SSH.

So how can we access a HTTP service running on port 8000 from Server B if, even Server A, cannot access it???

There is a trick using ssh that can make it possible.

First, we have to connect, from Client A to Server A using this command

> ssh  server-a -L 2222:server-b:22

What I have done is connect to Server A, using the -L switch. So I created a ssh tunnel that gets the port 22 from server-b, and make locally accessible from Client A. The port 2222 has chosen. Now you can connect directly to Server B using this command:

> ssh localhost -p 2222

And, them, we connect to Server B directly, bypassing, Server A with a ssh tunnel.

If we can do it from a remote ssh. We can do it with a remote port. Let's try this command:

> ssh localhost -p 2222 -L 8000:localhost:8000

To not get confused, I will explain what each part of this command means:

 - 'ssh localhost -p 2222' connects to the port 2222 of the localhost. Remember that this port 2222 was forwarded from Server A to port 22 of Server B
 - '-L 8000:localhost:8000: This part works only when the connection is established. And this localhost is not from Client A. Is from Server B. So I'm telling ssh to get the port 8000 from localhost:2222 server, and make it available from at port 8000 at the localhost of the Client A. We have to remember that this port 2222 is Server B port 22

Now open a web browser and access http://localhost:8000. If it works it will open the server start page. To make it easy I created two index.html. One for Server A and the second for Server B. And you are accessing the second one without changing any firewall rules, proxy or any other trick. Only using ssh, like this picture below.

picture

Everything can be done, at once, using a shell script and using private/public keys. The ssh keys method can be explained [here](http://www.linuxproblem.org/art_9.html) . So the possible script to bring port 8000 from Server B to Client A is shown below.

    #!/bin/bash
    #Brings Server B port 22 to Client A localhost port 2222
    
    ssh -fN -L 2222:server-b:22 server-a
    #The -fN switch put this connection in background, without a tty
    
    #Gets the port 8000 from Server B's localhost to port 8000 localhost from Client A
    #But it is only locahost:8000 from Client A.
    
    ssh -fN localhost -p 2222 -L 8000:localhost:8000
    echo "Now connect to http://localhost:8000 to access Server B"

To reproduce all commands I will provide both firewall rules from Server A and Server B and a httpd config file.
