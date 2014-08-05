# Day 7 at Hacker School
## I've been writing C.

-----
I've been writing a server in C -- a suggestion from the Hacker School resident this week. Her talk on stack smashing was awesome. Now I want to be an actual hacker. I've been practicing on [Microcorruption](https://microcorruption.com/login), a learn-assembly-via-capture-the-flag hacker game (currently starting level 3: Reykjavik).  

In the meantime, I learned about the C functions for creating, binding, and making connections via sockets. It's helped to hammer home a lot of the concepts I learned about last week, while writing a basic http server in Python (except C is a lot more confusing). [Beej's Guide to Network Programming](http://www.beej.us/guide/bgnet/) has become my bible. When I learned that there is also a [Beej's Guide to C Programming](http://beej.us/guide/bgc/output/html/multipage/index.html) , I almost cried. Beej is my hero.

Anyways, on Monday, I started by going through this [simple example](http://blog.manula.org/2011/05/writing-simple-web-server-in-c.html) of a basic web server in C. Once I had found Beej's guide, I used it to also write a very simple client program. The client could connect to the server and send some data, the server would receive the data and send "Hello World" back to the client, and then the connection would close. I learned that this was a TCP server (Transmission Control Protocol -- as opposed to User Datagram Protocol). TCP is used when you know all data NEEDS to be delivered, and in a certain order. The connection between server and client must be open -- it relies on *stream* sockets. UDP, on the other hand, does not require the connection to remain open, and does not guarantee that all the data will arrive in any particular order -- it relies on *datagram* sockets. UDP, aka 'Fire and Forget', is used when it's not that important to have every piece of information arrive, and order can be sorted out later. I also learned a bit about IPv4 and IPv6 -- enough to know that I don't really want to deal with sorting out the different protocols, so my server only handles IPv4.

After that was running, I started sending messages to my server using telnet. When I went back to connecting via my client, I was mystified to see that the server was replying to *my* client with the most recent messages from telnet! What? Well, I hadn't thought to clear the buffer where received data was stored after each interaction. Telnet's messages were still there, to be delivered back to *any* client, ad infinitum.  

----------------------

Yesterday (Tuesday) I went through and refactored the simple server using the structs designed specifically for socket servers -- until I actually understood all the pieces. I spent way too much time dealing with a connection issue that turned out to be very simple: something was already connected on the port I wanted to use.   
Moral: if server or client refuses to connect, change port number before resorting to hack-y changes you don't understand, like:  

```
char str[INET_ADDRSTRLEN];
inet_ntop(AF_INET, &(address_in.sin_addr), str, INET_ADDRSTRLEN);
```
After sorting that out, I took my server in the direction of an FTP server. Unlike an HTTP server, which is stateless, the FTP server needs to keep the connection open between multiple exchanges of data. By the end of the day, I had a simple loop where the client could send a user's messages to the server, which would echo the messages right back. 

-----------

Today, my goal was to be able to serve multiple connections from a single server -- threading. I spent some time playing with fork (by which one process can create multiple 'child' processes -- with a different memory space and process id (pid)) and exec (by which one process can begin running a different program). [This tutorial](http://www.yolinux.com/TUTORIALS/ForkExecProcesses.html) was helpful. Unlike forking, if a program spawns multiple threads, they will all share the same PID, a lot of data, and work concurrently (I still need to explore the order in which forked child programs run -- I think they run to completion, and only then does the parent process continue?).

I wrote a program that created 7 threads, and had them all 'working' on counting to 10. This resulted in:

```
 Thread 0 created successfully
Thread 0 working...

 Thread 1 created successfully
Thread 1 working...
Thread 0: 0

 Thread 2 created successfully
Thread 1: 0
Thread 2 working...
Thread 0: 1
Thread 1: 1
Thread 2: 0
Thread 0: 2

 Thread 3 created successfully
Thread 1: 2
Thread 3 working...
Thread 2: 1
Thread 0: 3

 Thread 4 created successfully
Thread 4 working...
Thread 1: 3
Thread 3: 0
Thread 2: 2
Thread 0: 4
Thread 4: 0
Thread 1: 4
Thread 3: 1

 Thread 5 created successfully
Thread 2: 3
Thread 5 working...
Thread 0: 5
Thread 4: 1
Thread 1: 5
Thread 3: 2

 Thread 6 created successfully
Thread 2: 4
Thread 6 working...
Thread 5: 0
Thread 0: 6
Thread 4: 2
Thread 1: 6
Thread 3: 3
Thread 2: 5
Thread 6: 0
Thread 5: 1
Thread 0: 7
Thread 4: 3
Thread 1: 7
Thread 3: 4
```

As you can see, they're all working at the same time! Crazy.

Thankfully, as far as my server is concerned, the progress made on any one connection shouldn't affect the progress made on any other. So I added a new thread to accept every new incoming connection. Now, my server can send/receive data with my client and telnet simultaneously. Cool.

----------

Still haven't checked whether I have any memory leaks. I think I'm freeing all malloc'd memory, but the structs and functions involved in sockets are a bit wonky, so I'm not really sure.

Tomorrow I'm going to work on transmitting file data from server to client. I've been practicing a bit with dealing with files and a lot with working with strings, and this is going to be a bit tricky. 