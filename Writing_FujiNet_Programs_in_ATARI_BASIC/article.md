# Writing FujiNet Programs in ATARI BASIC

On ATARI systems, FujiNet programs which use the network device can be easily written in ATARI BASIC. This extends the ability of ATARI BASIC to create new networked programs, and to explore FujiNet's networking features in an interactive manner.

This is possible due to the N: device handler that is present on the FujiNet Tools disk that can be mounted from the _apps.irata.online_ TNFS server, as well as any of the DOS disks in the DOS/ folder on _apps.irata.online. This handler, allows any program which can accept a device specification to use the network in the same manner as any other ATARI device. This means that ATARI BASIC can access anything on the network in the same way that it can access a disk drive, a printer, or any other connected device.

Let's load a copy of ATARI DOS 2.0S from _apps.irata.online_ and use ATARI BASIC to talk to the network.

## Where is the N: handler?

The N: handler must be loaded from a disk. If you have a copy of the fnc-tools.atr disk at TNFS://apps.irata.online/Atari_8-bit/fnc-tools.atr, this handler is present as NDEV.COM, and can be renamed to AUTORUN on your favorite DOS disk. For convenience, this handler has been placed on every single disk in the TNFS://apps.irata.online/Atari_8-bit/DOS folder. You can pick any DOS in this folder, I'll select "Atari DOS 2.0S.atr" and boot with a BASIC cartridge inserted.

![The DOS disks in this folder have the N: handler on them.](the_dos_folder.png "The DOS disks in this folder have the N: handler on them.")

Booting this disk presents you with a READY prompt.

## Example 1: The world's simplest Terminal Program

The following BASIC program is a simple dumb terminal that uses the N: device to connect to a BBS. This program opens a network connection, and immediately enters into a loop where it displays any incoming network data, and relays any keypresses back to the network endpoint, until the endpoint disconnects. The program is only a few lines long:

```basic
100 OPEN #1,12,3,"N:TELNET://BBS.FOZZTEXX.NET/":OPEN #2,4,0,"K:"
101 TRAP 140
110 IF PEEK(764)<>255 THEN GET #2,K:PUT #1,K:XIO 15,#1,12,3,"N:"
120 STATUS #1,A:BW=PEEK(747)*256+PEEK(746):IF BW=0 THEN 110
130 FOR M=1 TO BW:GET #1,C:PUT #16,C:NEXT M:GOTO 110
140 CLOSE #1:? "DISCONNECTED.":END
```

![The Simplest Possible Terminal Program](simplest_terminal_program.png "The Simplest Possible Terminal Program")

Line 100 opens the network connection. We use IOCB #1 for the network connection. We use mode 12 to specify that we want the channel to be able to read and write simultaneously, and the second parameter of 3 specifies that the network endpoint uses the ASCII carriage return (CR) and line-feed (LF) characters to signify the end of a line, so that FujiNet can convert between them and the ATASCII end-of-line (EOL) character. Line 100 ends with opening a second channel to the keyboard (K:) device.

Line 101 sets an error trap to 140, which we use to close the connection in both the event that the connection is either disconnected, or there was a network error of some sort. While PEEK(195) can be used to determine which error happened, we don't handle it here, to keep things simple.

Line 110 handles the keyboard input and its subsequent output to the network endpoint. We check if a key is pressed with PEEK(764), and if so, we grab the key by issuing a GET to IOCB #2; issuing a PUT to IOCB #1 to send that keypress to the network, and issuing an XIO 15 to force the N: device to send pending network traffic out. As an aside, the N: device will send out network traffic in three cases: (1) when the transmit buffer of 127 bytes is full, (2) when an end of line (EOL) character is put into the buffer, or (3) when an XIO 15 is sent to flush the buffer.

Line 120 handles all of the network input. It starts by issuing a STATUS command to IOCB #1. The A variable coincidentally holds a result code of the STATUS operation, which on success is typically '1', but we don't use it. Instead, we use the two bytes starting at PEEK(746) to tell us how many bytes are waiting. Since this is a 16-bit value, we do a bit of math to set BW to a value between 0 and 65535, indicating how many bytes are waiting for us to read. If there are no bytes waiting for us to read, we simply loop back to line 110, otherwise we continue to line 130.

Line 130 enters a loop of up to the value set in BW, where we GET the next byte waiting in our receive buffer; placing it in C. We then use a trick in ATARI BASIC to PUT To IOCB #16, which will write to the (E:) screen editor device, printing the character to the screen. This avoids some screen messiness that can occur when using the PRINT command. This loop will occur for as many bytes as are waiting in the network receive buffer. Once done, the program loops back to 110.

Because we set a trap for line 140, any error, such as a disconnection (which is the same as the end of file from a disk), will cause the program to jump to line 140, which dutifully closes the connection, informs the user of the disconnection, and ends the program.

### Other URLs?

Feel free to change the OPEN statement to connect to other endpoints. A wide range of protocols and potential endpoints can be used here, with no fundamental change to the rest of the program.

Take into account that many URL paths are on systems that are case-sensitive.

* N:HTTPS://www.gnu.org/licenses/gpl-3.0.txt
* N:SSH://username:password@yourlinuxbox/
* N:TCP://basementbbs.ddns.net:9000/

(Available protocols as of firmware version 1.5.1: FTP, HTTP/S, SD, SMB, SSH, TCP, TNFS, TELNET, UDP)

### Accepting a connection

Certain protocols, such as TCP also accept listening connections, so that other computers can connect to your FujiNet, and therefore your ATARI.

Make the following changes to the terminal program to change it to accept an incoming connection on TCP port 6502:

```basic
100 OPEN #1,12,3,"N:TCP://:6502/":OPEN #2,4,0,"K:"
102 PRINT "WAITING FOR CONNECTION...ANY KEY TO QUIT.";
103 POKE 65,0:REM QUIET THE SPEAKER
104 STATUS #1,A:IF PEEK(764)<>255 THEN 200:IF NOT PEEK(748) THEN 104
105 PRINT "ACCEPTING CONNECTION...";
106 XIO ASC("A"),#1,12,3,"N:"
107 PRINT "CONNECTED!"

140 XIO ASC('c'),#1,12,3,"N:":? "DISCONNECTED.":GOTO 102
200 CLOSE #1:? "NO LONGER LISTENING.":CLOSE #1:END
```

Feel free to run another copy of this terminal on another ATARI, or even running a PC using a program like 'nc' on Linux or Putty on Windows.

By removing the hostname from the URL in line 100, the connection becomes a listening connection. Because we need to specify a port, we specify port 6502, but it can be any number from 1 to 65535.

Line 102 announces that it is waiting for the next connection to port 6502.

Line 103 is optional, but it quiets the potential SIO noise that can come from polling for a connection.

Line 104 is a similar STATUS call to line 120, except that for listening connections, PEEK(748) indicates whether there is a connection waiting. If not, we loop, until there is, at which point we fall to line 105. It also checks if a key is pressed, and if so, we will close the listening connection and exit the program.

Line 105 announces that a connection is waiting, so we use line 106 to accept the connection. This turns the listening connection into a fully functional connection between the FujiNet and the waiting endpoint, so that communication can take place. At which point, the connection can proceed the same as before. Line 107 finally announces that we're connected.

A special note about accepting connections: Because the ATARI is a single-tasking system, the FujiNet firmware only allows for one connection to be waiting on a listening connection, anyone else attempting to connect while a listening connection is either accepted or waiting to be accepted will be dropped.

Line 140, as before is the destination of the TRAP set at the beginning of the program, except this time we do a special XIO to tell the FujiNet to close the client connection, and to convert the connection back into a listening connection so that another end-point can connect.

Line 200 closes the listening connection. This prevents any endpoint from connecting to the FujiNet at the listening port, and will close any client connections that may also be connected. We didn't do this for closing client connections, because the FujiNet's TCP/IP stack will wait for 120 seconds after a listening socket is closed to mark it safe for re-use again.

This approach will work as the basis for any peer to peer connection.

## Example 2: What about a web service?

There are many servers on the Internet which provide information in the form of dynamic web documents in structured languages such as JSON and XML. FujiNet can parse these itself, returning the relevant information, with a minimum of programming.

The example presented will fetch the most recent public post on the oldbytes.space Mastodon server, and display it. Mastodon is a federated social network allowing people to post small "toots" that can be read and replied by others. The users on these servers can subscribe to different users on the same or other servers in order to read their content. To facilitate this utility, these servers expose a complete interface of functions which can be used, for example, by the FujiNet to do things like read the posts on the server, to posting new "toots." and everything in between.

We'll use the following URL to fetch the most recent public post on the __oldbytes.com__ server:

```
HTTPS://OLDBYTES.SPACE/api/v1/timelines/public?limit=1
```

If we put this URL into a web browser, or into a utility such as 'curl' to see the resulting information, we get a very large JSON document, containing all sorts of structured data:

```json
[
  {
    "id": "115349957246460545",
    "created_at": "2025-10-10T09:52:58.000Z",
    "in_reply_to_id": null,
    "in_reply_to_account_id": null,
    "sensitive": false,
    "spoiler_text": "",
    "visibility": "public",
    "language": "en",
...some bits removed...
    "replies_count": 0,
    "reblogs_count": 0,
    "favourites_count": 0,
    "edited_at": null,
    "content": "<p>There's an enormous void of nothingness in our Universe. And scientists found it by accident<br>...",
    "reblog": null,
    "account": {
      "id": "113380806233811581",
      "username": "skyatnightmag",
      "acct": "skyatnightmag@flipboard.com",
      "display_name": "BBC Sky at Night Magazine",
      "locked": false,
      "bot": false,
      "discoverable": true,
      "indexable": true,
      "group": false,
      "created_at": "2024-10-23T00:00:00.000Z",
  }
...more bits removed...
]
```

While many of the different pieces of information returned by this JSON document have been edited out, it should be clear that there is a substantial amount of information returned in this one "toot" and only some parts of it are needed for a minimal display. Once you grab even more posts, these large documents would greatly overwhelm the modest amount of RAM available on an ATARI computer. The solution is to have the FujiNet parse the information, and send back the relevant pieces of information to the ATARI computer.

Given the information above, we'd like to show the /account/display name, the /account/created_at time stamp, and the /content parts of this document, and this is accomplished with the following ATARI BASIC program:

```basic
0 DIM A$(256)
1 TRAP 91
2 POKE 756,204
10 OPEN #1,12,0,"N:HTTPS://OLDBYTES.SPACE/api/v1/timelines/public?limit=1"
20 XIO 252,#1,12,1,"N:"
30 XIO ASC("P"),#1,12,0,"N:"
40 XIO ASC("Q"),#1,12,3,"N:/0/account/display_name"
50 INPUT #1,A$:? A$
60 XIO ASC("Q"),#1,12,3,"N:/0/created_at"
70 INPUT #1,A$:? A$
80 XIO ASC("Q"),#1,12,3,"N:/0/content"
90 GET #1,A:? CHR$(A);:GOTO 90
91 CLOSE #1:? :?
100 POKE 18,0:POKE 19,0:POKE 20,0
110 IF PEEK(19)&lt;30 THEN 110
120 GOTO 1
```
---

![A Mastodon Client in BASIC](mastodon.png "A Mastodon Client in BASIC")

Line 0 dimensions a 256 byte string variable A$.

Line 1 sets 91 as an error trap.

Line 2 changes the character set base to the International Character set. This line may be omitted on ATARI 400/800 systems, which do not have this character set.

Line 10 opens a connection to the __oldbytes.space__ Mastodon server using the HTTPS protocol. The path points to an API call that fetches posts from the public time line. The "?" Indicates a GET query parameter, and in this case limit=1 ensures that the server only sends a single post.

Line 20 uses XIO to send a SET CHANNEL MODE command. This changes the channel to allow the 'P'arse and 'Q'uery commands that will be used on the proceeding lines.

Line 30 uses XIO to send a PARSE command. This reads the network channel until there is no more data. The channel is subsequently passed through the JSON parser, with the results of the parse sitting in FujiNet memory, to be queried using the 'Q'uery command.

Line 40 uses XIO to send a 'Q'uery command. This particular JSON Query asks for the display name of a given poster. However, it is inside an array, so we have to ask for the first element of the array as part of the query, hence the "/0/"

Line 50 calls INPUT to read the queried result into variable A$, and subsequently display it on screen with the '?' statement.

Line 60 uses XIO to send a 'Q'uery command which asks for the created_at key for the very first returned post.

Line 70 calls INPUT to read the queried result into variable A$, and subsequently display it.

Line 80 uses XIO to send a final 'Q'uery command, which asks for the content for the very first returned post.

Line 90 Sets up an infinite loop which GETs and displays one byte at a time, because INPUT can only fetch single lines up to 127 bytes long. The end of the content will generate an end of file (EOF) error, so the TRAP will cause the loop to exit.

Line 91 is the destination of the TRAP statement, which closes the network connection, while printing two blank lines.

Line 100 zeroes the real time clock at addresses 18, 19, and 20.

Line 110 sets up a loop which exits when PEEK(19) is equal to 30, which is approximately 3 minutes, so we don't needlessly hammer the server.

Line 120 jumps back to the TRAP statement on line 1, which re-starts the program, while taking care not to re-dimension the A$ variable.

So you see, with the basic pattern of parsing, querying and reading, any piece of a JSON document can easily be returned.

There are many other programs that can be written in this manner, and the interactive nature of BASIC, and the careful integration of the network device as an operating system device ensures that any ATARI program you or someone else writes has the potential of using FujiNet features.

For those that want to study it, the source code to the N: handler is available on GitHub: https://github.com/FujiNetWIFI/fujinet-nhandler/tree/master/handler
