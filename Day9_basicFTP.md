# Day 9 at Hacker School
## The FTP server continues, and I learn to be wary of carriage returns.

-----

So I keep saying "And now I'm going to work on the actual file-transfer part" ... but for some reason or another, I still haven't gotten around to it.

First, I spent more time figuring out threading. I tried to have each thread "join" the main thread when it exited, but my first attempt just broke the multi-threadedness all together. Since each thread in my program operates on its own and has nothing to contribute back to the main thread on exit, actually 'joining' threads back in to the main thread turned out to be unnecessary. I also added a global "number of threads" variable so I could better understand how the values assigned to global variables are fixed across all threads, but the values of other local variables can vary from thread to thread. For example, after I implemented a simple user sign-in, different threads could be logged in or not. I also learned that [it is safe to use malloc](http://www.cocoawithlove.com/2010/05/look-at-how-malloc-works-on-mac.html) in threads -- I had been worried about this being an issue.

I set off to implement other simple FTP commands (based on the requirements of [this lab](http://www.ugrad.cs.ubc.ca/~cs219/Labs/Threads/lab-threads.html)). This was slow going because it involves a fair bit of string manipulation in order to parse the commands, and string manipulation in C is tricky (read: tedious). Eventually, after signing in (as "anonymous"), the client could PWD, CWD (problem: nothing is stopping clients from CWD-ing happily whereever they want and stealing/planting malicious scripts all over my file system), NLST (list directory contents), and QUIT. I soon realized, however, that threads all share a working directory!!! So when one client changes working directory, the working directory for all threads is changed. NOOOOOOOOO. I plan on working around this by storing a local "working directory" string for each thread, and issuing all other commands relative to this path.

-----------------------

Today I spent forever refactoring messy string manipulation stuff -- but at least I removed some code like:  

```
if (blatant_memory_leak == 1) {  
	free(data);  
	blatant_memory_leak = 0;  
}
```
I also got my understanding of reading and writing to files in C up to the point where I was ACTUALLY ready to implement the file transfers, but then I realized I didn't know how to do it in terms of the File Transfer Protocol! And so I discovered that my mac has a command-line FTP client built-in, and I used it to connect to the Linux people's ftp server,
(`ftp metalab.unc.edu` on the command line), navigate around the file system (and download their README), using [this very helpful guide to FTP](http://tldp.org/HOWTO/FTP-3.html) to understand a bit of what was going on. I was curious -- could this tool connect to my server? `ftp localhost 5000` revealed that, indeed, it could! It took some futzing around to understand when my server needed to listen vs send data and -- in general -- what the client was expecting (no -'s after status codes in single-line responses, and that every response from the server ends with '\n'). Also interesting was that the commands that I typed to the client were not the same commands as the client then sent to the server. 

The big moral of the day is that you need to be concious of characters that are not usually printed -- in my case, '\r' -- the [carriage return](http://en.wikipedia.org/wiki/Carriage_return). The FTP client ends all of its commands with '\r\n' -- the <CRLF> mentioned in the spec for the lab I was working with ('doh!). I was correctly removing the newline from the end of all commands sent to my server, but I did not realize that the carriage return was still there. This resulted in some very strange output, because the carriage return effectively moves the cursor to the beginning of the same line -- and output begins to overwrite anything that was already there. So debugging statements like  

```
printf("%s (%zd), %s (%zd)\n", parsed[0], strlen(parsed[0]), parsed[1], strlen(parsed[1]));  
```
where I was hoping to see  
 
```
USER == USER (anonymous)? 0
```

resulted in  

```
)? 0 == USER (anonymous
```
SO WEIRD! I eventually debugged this with the help of Tom (a Hacker School facilitator) by printing out the hex of every character in each string.

**TANGENT** -- Fun with other unprintables:  
Turn up your volume a bit and enter `printf "\a"` in the command line.
HAHAHAHHAA  
Now do it again!

Anyways, after I sorted out login (and cheered myself up/revived myself with the "\a" stuff), the FTP client started making all sorts of weird requests to my server! Exciting! I'm working on sorting out something to do with a request for a port. This [list](http://www.nsftools.com/tips/RawFTP.htm#PWD) of raw FTP commands is very helpful. The (eventual) goal is still a file transfer.

