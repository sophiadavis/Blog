# Day 10 at Hacker School
## The FTP server continues, and I do a lot of refactoring.

-----
Today, I got frustratingly little done. I did a lot of refactoring in order to consolidate socket creation and loose error checking code each into a single function. I think the server may need to open another socket to deal with data transfer, so at least with a 'prepare_socket' function I have a more-or-less straightforwards way to do that.

I can't get the client to accept long responses. Client/Server interaction holds up enough to PWD, CWD, QUIT, and NLST (NLST only if the entire response is on one line), but the client keeps making the PORT request (in which it tells the server which port to listen on for data transfer). I think creating a new socket for another connection will open up a whole new can of worms, and I've been trying to "fake" the client out in various ways. For example, if I turn on passive mode (as a user), the client sends my server a PASV command, which "tells the server to prepare for a new socket connection by creating a new socket and then listening for a connection from the client." Then, I'm having my server issue a correctly formatted response, telling the client to connect over the **same** socket (sneaky!). But so far, no beans. I think it also has something to do with Ascii vs Binary mode, and potentially a way to format end-of-message that the client is expecting but I'm missing... 

Also, I found out about the client's debug mode -- very helpful. In case you're curious, here is an example interaction between the client and my server:

```
ftp> ftp localhost 5000
Trying 127.0.0.1...
Connected to localhost.
220 Sophia's FTP server (Version 0.0) ready.
ftp_login: user `<null>' pass `<null>' host `localhost'
Name (localhost:sophia): anonymous
---> USER anonymous
230 User signed in. Using binary mode to transfer files.
---> SYST
215 MACOS Sophia's Server
Remote system type is MACOS.
---> FEAT
211 end
features[FEAT_FEAT] = 1
features[FEAT_MDTM] = 0
features[FEAT_MLST] = 0
features[FEAT_REST_STREAM] = 0
features[FEAT_SIZE] = 0
features[FEAT_TVFS] = 0
got localcwd as `/Users/sophia'
---> PWD
257 "/private/var/folders/r6/mzb0s9jd1639123lkcsv4mf00000gn/T/server" 
got remotecwd as `/private/var/folders/r6/mzb0s9jd1639123lkcsv4mf00000gn/T/server'
ftp> pwd
Remote directory: /private/var/folders/r6/mzb0s9jd1639123lkcsv4mf00000gn/T/server
ftp> cd folder1
---> CWD folder1
250 CWD successful
---> PWD
257 "/private/var/folders/r6/mzb0s9jd1639123lkcsv4mf00000gn/T/server/folder1" 
got remotecwd as `/private/var/folders/r6/mzb0s9jd1639123lkcsv4mf00000gn/T/server/folder1'
ftp> system
---> SYST
215 MACOS Sophia's Server
ftp> type binary
---> TYPE I
200 Using binary mode to transfer files.
ftp> passive
Passive mode: on; fallback to active mode: on.
ftp> nlist
---> TYPE A
502 I only work with binary ??
---> EPSV
500 Syntax error, command unrecognized.
disabling epsv4 for this connection
---> PASV
227 Entering Passive Mode =127,0,0,1,19,136
---> NLST
150 Here comes the directory listing. 	. 	.. 	folder1.1 	folder1.2 	text.txt 	226-Directory send OK.
220 Sophia's FTP server (Version 0.0) ready.
```
All my input (as user) appears after 'ftp> ' prompts. The '---> UPPERCASE COMMAND' is what the client sends to my server (you can see that there is no 1-to-1 correspondance), and the '### message' is my server's response.  I'm stuck down in the NLST command -- it sends back the contents, but only if the entire response is one line. Earlier today it was at least returning to the client. Looks like somehow another connection is being opened. Bummer.