# Day 1 at Hacker School
## I wrote an http server in Python.  

-----

First off, I opened the Python [Socket Programming HOWTO](https://docs.python.org/2/howto/sockets.html), and learned a little bit about sockets.

[The way I understand it](http://en.wikipedia.org/wiki/Network_socket), sockets are the links between clients and servers (and other things?) in computer networks.

I started out in a Python prompt (using bpython, which I learned about this morning from Tom):

```
>>> s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
>>> s.connect(("www.google.com", 80))  
>>> s.send('GET /\r\n\r\n') # http request to GET '/'
9  # returns number of bytes sent
>>> s.recv(1024)
'HTTP/1.0 200 OK\r\nDate: ...'
```
That opens a socket for a web **client** and connects it to Google on port 80. The client sends a GET http request, and when I call `.recv(num_bytes)` on the socket object, I can see the next `num_bytes` bytes of the response that is served back to the client. Each time I execute `s.recv(1024)`, I get the next 1024 bytes of the text of the webpage. Cool.

-----------------------

But I wanted to write a **server**. [Apparently](http://ilab.cs.byu.edu/python/socketmodule.html), here are the things a server needs to do:

1. Make a socket
2. Bind the socket to an address (host and port -- I chose 'localhost' on port 8000)
3. Listen for connections
4. Accept clients
5. Send/Receive data (in my case, http stuff)

In [round 1](https://github.com/sophiadavis/http-server/commit/154a1646d314072d724e2eb46130118a55e30800), I implemented steps 1-3, then 4 and 5 inside an infinite while loop -- my server continually accepted clients, received data from them, and served the same data right back at them.

At this point, I could run my script, go to localhost:8000 in the browser, and see this delivered:

```
GET / HTTP/1.1  
Host: localhost:8000  
Connection: keep-alive  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8  
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36  
Accept-Encoding: gzip,deflate,sdch  
Accept-Language: en-US,en;q=0.8,de;q=0.6,es;q=0.4,fr;q=0.2,ru;q=0.2  
```
...which I guess was the exact request that my browser client(?) sent to the server.

I could also run my script, open a client socket to connect to my server in a separate Python terminal, and bounce messages back and forth. 

Then I learned about telnet -- which let me easily connect multiple clients to my server and bounce messages back and forth. I added [some code](https://github.com/sophiadavis/http-server/blob/e099c1ec813e5f7ad73acde2eacbfb2d480ed83a/httpServer.py#L22) to print the address (a tuple: (host, port)) of each client as it connected to the server. They all had the same host (my computer), but the ports were different -- I guess that makes sense.

Cool.

Next step was to add in the serving http part. Instead of just sending the client's original request back, I had my server respond with [Wikipedia's demo http 200 OK server response](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Server_response). That's basically where I am [right now](https://github.com/sophiadavis/http-server/blob/master/httpServer.py), except it took me way too long to get here... It turns out http response syntax needs a really specific number of new line characters in certain places, and also if you include a header for Content-Length, then it better actually be the length of the content... and other silly stuff like that.


Another cool thing I learned was that, by default, sockets can't be reused immediately -- so I was spending a lot of time being frustrated, waiting to be able to reuse port 8000 again after CTRL-C'ing my server script... Tom tipped me off on how to [set an option on an unbound socket](https://github.com/sophiadavis/http-server/blob/master/httpServer.py#L10) so that the same socket can be reused immediately. Yay.

Tomorrow, I plan on parsing the http request (although parsing might be a strong word), and hopefully rendering a couple static pages.