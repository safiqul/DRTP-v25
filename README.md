# Individual Assignment: DATA2410 Reliable Transport Protocol (DRTP) 

This individual assignment will count towards your final grade (40%). Your submission will be marked with respect to how well you have fulfilled the requirements listed below. You must pass all the mandatory assignments in order to take part in this exam.

# Prerequisites

We have already covered the required concepts in our lectures, labs, and mandatory assignments which you need for this assignment. Should you be missing lectures and lab sessions, it is your own responsibility to catch up to the competence level that you are supposed to get it from the lectures and lab sessions. 

## Overview

You will be writing the sending and receiving transport-level code for implementing a simple reliable data transfer protocol. You will use your simple transport protocol - DATA2410 Reliable Transport Protocol (DRTP) that provides reliable data delivery on top of UDP. Your protocol will ensure that data is reliably delivered in-order without missing data or duplicates.

You will write code that will reliably transfer file between two nodes in the network. You MUST implement one program: A file transfer application which will run as a server or a client.

> NOTE You must use UDP as the transport protocol and add reliability on top of that. You are not supposed to use any other transport protocol which already provides reliability (e.g., TCP). You must construct the packets and acknowledgements. You must establish the connection and gracefully close when the transfer is finished.


## A file transfer application

You will implement a file transfer application (application.py) where you will use -c to invoke a client
and -s to invoke a server.

### A simple file transfer server

A simple file transfer server will receive a file from a client through its DRTP/UDP socket and write it to the file system. The file name and the port numbers on which the server listens are given as command line arguments. A server will also track how much data was received from the connected clients; it will calculate and display the throughput based on how much data was received and how much time elapsed during the connection.

> NOTE: the port number of the server must also be the port number used by the client.

The server/receiver can be invoked with:

`python3 application.py -s -i <ip_address_of_the_server> -p <port>`

The output at the server side will look like this:

```console
SYN packet is received
SYN-ACK packet is sent
ACK packet is received
Connection established
21:20:58.675875 -- packet 1 is received
21:20:58.675940 -- sending ack for the received 1 
21:20:58.676036 -- packet 2 is received
21:20:58.676200 -- sending ack for the received 2 
21:20:58.676234 -- packet 3 is received
21:20:58.676366 -- sending ack for the received 3 
21:20:58.676414 -- packet 4 is received
21:20:58.676496 -- sending ack for the received 4 
21:20:58.676529 -- packet 5 is received
21:20:58.676591 -- sending ack for the received 5 
21:20:58.776558 -- packet 6 is received
21:20:58.776994 -- sending ack for the received 6 
21:20:58.777072 -- packet 7 is received
....

FIN packet is received
FIN ACK packet is sent

The throughput is 0.39 Mbps
Connection Closes

```

### A simple file transfer client

A client reads a file from the computer and sends it over DRTP/UDP. The file name, server address and port number are given as command line arguments. 

It is possible to transfer two source files at the same time. In this assignment,  we'll only allow one transfer. 

The sender can be invoked with:

`python3 application.py -c  -f Photo.jpg -i <ip_address_of_the_server> -p <server_port>`

The output at the client side will look like this:

```console
Connection Establishment Phase:

SYN packet is sent
SYN-ACK packet is received
ACK packet is sent
Connection established

Data Transfer:

21:05:04.715480 -- packet with seq = 1 is sent, sliding window = {1} 
21:05:04.715532 -- packet with seq = 2 is sent, sliding window = {1, 2} 
21:05:04.715553 -- packet with seq = 3 is sent, sliding window = {1, 2, 3} 
21:05:04.715570 -- packet with seq = 4 is sent, sliding window = {1, 2, 3, 4} 
21:05:04.715599 -- packet with seq = 5 is sent, sliding window = {1, 2, 3, 4, 5} 
21:05:04.815773 -- ACK for packet = 1 is received
21:05:04.815856 -- packet with seq = 6 is sent, sliding window = {2, 3, 4, 5, 6} 
21:05:04.816043 -- ACK for packet = 2 is received
21:05:04.816075 -- packet with seq = 7 is sent, sliding window = {3, 4, 5, 6, 7} 
21:05:04.816200 -- ACK for packet = 3 is received
21:05:04.816226 -- packet with seq = 8 is sent, sliding window = {4, 5, 6, 7, 8} 
21:05:04.816333 -- ACK for packet = 4 is received
21:05:04.816357 -- packet with seq = 9 is sent, sliding window = {5, 6, 7, 8, 9} 
21:05:04.816432 -- ACK for packet = 5 is received
21:05:04.816453 -- packet with seq = 10 is sent, sliding window = {6, 7, 8, 9, 10} 
....
DATA Finished



Connection Teardown:

FIN packet packet is sent
FIN ACK packet is received
Connection Closes


```


Table below lists all the available options that you can use to invoke the server:
| flag        | long flag      | input       | type        | Description | 
|-------------|-----------------|-------------|-------------|-------------|
| `-s`         |  `--server`   |   X         | (boolean)   | enable the server mode |
| `-c`         |  `--client`   |   X         | (boolean)   | enable the client mode |
| `-i`         | `--ip`    | **ip address**  | string  | allows to bind the `ip address` at the server side. The client will use this flag to select server's ip for the connection - use a default value if it's not provided. It must be in the dotted decimal notation format, e.g. 10.0.1.2 |
|`-p`          | `--port`    | **port number**  | integer  |allows to use select `port number` on which the server should listen and at the client side, it allows to select the server's port number; the port must be an integer and in the range `[1024, 65535]`, default: 8088|
| `-f`  |`--file`      |  x   |  string |allows you to choose the jpg file| 
| `-w`  |`--window`    |  x   |  int |sliding window size, default: 3| 
| `-d`  |`--discard`      |  x   |  int |a custom test case to skip a seq to check for retransmission. If you pass `-d 11` on the server side, your server will discard packet with seq number 11 only for once. Make sure you change the value to an infinitely large number after your first check in order to avoid skipping seq=11 all the time.| 



# Details

A sender should read data in chunks of 992 bytes. For the sake of simplicity, assume 1 KB = 1000 Bytes, and 1 MB = 1000 KB. For each chunk (application data), a sender adds a header before it sends the data over UDP. The header length is 8 bytes and application data is 92 bytes.  A sender therefore sends 1000 bytes of data to a receiver, including the custom DRTP header. The header contains a sequence number (packet sequence number), an acknowledgment number (packet acknowledgment number), flags (only 4 bits are used for connection establishment, teardown, and acknowledgment packets), and a receiver window (for flow control). The DRTP header looks like this:


```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Sequence Number  |  Acknowledgment Number  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Flags            |  Receiver Window        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


```
> protocol "Sequence Number:16 bits, Acknowledgment Number: 16 bits, Flags: 16 bits Receiver Window: 16 bits".


Together with the header, your application data that you will send over DRTP/UDP:


```

+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Sequence Number  |  Acknowledgment Number  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Flags            |  Receiver Window        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               Your DATA                     |                 
|                  ...                        |                 
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```

If a DRTP packet contains no data, then it is only an acknowledgment packet ("ACK"). It confirms having received all packets up to the specified sequence number. An acknowledgement packet (only 6 bytes) is sent by the receiver with the acknowledgment number and set the ACK flag bit in the Flags field. 

Format of the flag fields:
```
                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                |                                                       |F|S|A|R|
                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                S = SYN flag, A= ACK flag, F=FIN flag, R= Reset Flag
```

> Note we're not going to use the R flag in our assignment, we'll set the value to 0


### Connection establishment and tear down

A sender initiates a three-way handshake with the receiver (similar to TCP) to establish a reliable connection. A sender first sends an empty packet with the syn flag ON. A server then responds with packet with SYN and ACK flags set, and a reciever window value of 15 and establishes the connection. Upon receiving syn-ack packet, a sender sends ack to the server and adjusts its sending window based on the receiver window value. 


### Reliability function

You will implement Go-Back-N (GBN()) reliability function. 

Go-Back-N (GBN()): sender implements the Go-Back-N strategy using a fixed window size of 3 (default case if not changed by `-w` flag) packets to transfer data. The sequence numbers represent packets, i.e. packet 1 is numbered 1, packet 2 is numbered 2 and so on. If no ACK packet is received within a given timeout (choose a default value: 400ms, use socket.settimeout() function), all packets that have not previously been acknowledged are assumed to be lost and they are retransmitted. A receiver passes on data in order and if packets arrive at the receiver in the wrong order, this indicates packet loss or reordering in the network. The DRTP receiver should in such cases not acknowledge anyting and may discard these packets.

See textbook and lecture slides for more details.

## Discussion:

Test your code in mininet using `simple-topo.py`

1. Execute the file transfer application with window sizes of 3, 5, 10, 15, 20 and 25. Calculate the throughput values for each of these configurations and provide an explanation for your results. For instance, discuss why you observe an increase in throughput as you increase the window size and the effect of receiver window.
2. Modify the RTT to 50ms and 200ms. Run the file transfer application with window sizes of 3, 5, 10, 15, 20 and 25. Calculate throughput values for all these scenarios and provide an explanation for your results.
To change the RTT, replace the value of 100ms with 50ms or 200ms in line 43 of your simple-topo.py file: `net["r"].cmd("tc qdisc add dev r-eth1 root netem delay 100ms")` in your `simple-topo.py` 
3. Use the `--discard` or `-d` flag on the server side to drop a packet, which will make the client resend it. Show how the reliable transport protocol works and how it deals with resent packets.
4. To demonstrate how effective your code is, use tc-netem (refer to the [lab manual](https://github.com/safiqul/2410/blob/main/docs/netem/mininet-tc-delay.md)) to simulate packet loss. To do this, modify your simple-topo.py file by commenting out line#43 `net["r"].cmd("tc qdisc add dev r-eth1 root netem delay 100ms")` and uncommenting line#44: `net["r"].cmd("tc qdisc add dev r-eth1 root netem delay 100ms loss 2%")`. Test with a loss rate of 5% and 50% as well. Include the results in your discussion section and explain what you observe.
5. Explain what happens when a fin-ack packet is lost.

> **NOTE** you MUST run your server on h2


## Submission

You must submit **your_inspera_candidateNumber_homeexam.zip**  through the Inspera exam system. Do not include your name and ID. It's going to be anonymous. You will get access to inspera before the deadline.

Your zip file should contain:

1. Source code of Application.py (if you have other files, also include them)
    * where the code is well commented. Document all the variables and definitions. For each function in the program, you must document the following:
        * what are the arguments.
        * What the function does.
        * What input and output parameters mean and how they are used.
        * What the function returns.
        * Correctly handling exceptions.

Here is an example:

```
 # Description: 
 # checks if the port and addresses are in the correct format
 # Arguments: 
 # ip: holds the ip address of the server
 # port: port number of the server
 # Use of other input and output parameters in the function
 # checks dotted decimal notation and port ranges 
 # Returns: .... and why?
 #
```
2. A report with your Inspera candidateNumber and assignment title. Your document must be submitted in the pdf format. 
    * read the instructions (see below)
4. A README file: info on how to run application.py



Example final structure of your folder:

```
├── README
├── src
│   ├── code
├── your_inspera_id_documentation.pdf

```

> **NOTE** I strongly recommend that you confirm that the archive you uploaded contains what it should by downloading it and unpacking it again after submission. If you deliver the wrong files, there is nothing I can do about it when I check. We also recommend you upload draft versions as you work, to ensure you have a deliverable in the system should you experience last minute technical difficulties etc.


### Project report

You're required to submit a project report in PDF format. Please note that I won't accept Word or any other format. The report should contain the following sections:

1. Title Page: Your Inspera candidate number and project title
2. Introduction: Brief explanation of your project
3. Implementation: Code documentation; explanation of your application with code snippets (do not include full source code)
4. Discussion: Answer the provided questions and include any limitations of your project
5. References: Mention any sources used


The report should not exceed 15 pages, including the list of references. The page format must be A4 with 2 cm margins, single spacing, and a font such as Arial, Calibri, Times New Roman, or similar, set to 11-point size.

### Grading

* Implementation of `File transfer application with DRTP` -  `70%`

* Project report - `30%`

# Deadline
The deadline for submitting this exam is **May 19 2025 at 12:00 (NOON)** Oslo local time.

This is a HARD deadline, failure to deliver on time will result in fail in this part-exam.

I am not going to handle any enquiries regarding medical extensions and so forth, you must contact the study administration directly.

Follow the submission guidelines. Do not make your respository public, and *do not copy*.


## Suggestions

I have added some example outputs below for your convenience.

> **NOTE** The examples below use a timeout value of `5 seconds`. However, you must use `400 milliseconds` in your code.

### What if the server is not running or responding?

Here's an example scenario where you attempt to run a client while the server is not running:


```console

h1$ python3 application.py -c  -f Photo.jpg -i 10.0.1.2 -p 8080


Connection Establishment Phase:

SYN packet is sent

Connection failed

```


### Discussion#3  

Use the `--discard` or `-d` flag on the server side to drop a packet, which will make the client resend it. Show how the reliable transport protocol works and how it deals with resent packets.

Here are example outputs from both the client and server for a test case with the `-d` flag:

#### Client

```console

h1$ python3 application.py -c  -f Photo.jpg -i 10.0.1.2 -p 8080 -w 5

Connection Establishment Phase:

SYN packet is sent
SYN-ACK packet is received
ACK packet is sent
Connection established

Data Transfer:

14:05:32.336280 -- packet with seq = 1 is sent, sliding window = {1} 
14:05:32.336325 -- packet with seq = 2 is sent, sliding window = {1, 2} 
14:05:32.336342 -- packet with seq = 3 is sent, sliding window = {1, 2, 3} 
14:05:32.336355 -- packet with seq = 4 is sent, sliding window = {1, 2, 3, 4} 
14:05:32.336383 -- packet with seq = 5 is sent, sliding window = {1, 2, 3, 4, 5} 
14:05:32.439567 -- ACK for packet = 1 is received
14:05:32.439673 -- packet with seq = 6 is sent, sliding window = {2, 3, 4, 5, 6} 
14:05:32.439847 -- ACK for packet = 2 is received
14:05:32.439903 -- packet with seq = 7 is sent, sliding window = {3, 4, 5, 6, 7} 
14:05:32.439999 -- ACK for packet = 3 is received
14:05:32.440039 -- packet with seq = 8 is sent, sliding window = {4, 5, 6, 7, 8} 
14:05:32.440122 -- ACK for packet = 4 is received
14:05:32.440172 -- packet with seq = 9 is sent, sliding window = {5, 6, 7, 8, 9} 
14:05:32.440197 -- ACK for packet = 5 is received
14:05:32.440233 -- packet with seq = 10 is sent, sliding window = {6, 7, 8, 9, 10} 
14:05:32.541310 -- ACK for packet = 6 is received
14:05:32.541438 -- packet with seq = 11 is sent, sliding window = {7, 8, 9, 10, 11} 
14:05:32.541488 -- ACK for packet = 7 is received
14:05:32.541530 -- packet with seq = 12 is sent, sliding window = {8, 9, 10, 11, 12} 
14:05:37.547540 -- RTO occured
14:05:37.547697 -- retransmitting packet with seq =  8
14:05:37.547968 -- retransmitting packet with seq =  9
14:05:37.548021 -- retransmitting packet with seq =  10
14:05:37.548055 -- retransmitting packet with seq =  11
14:05:37.548088 -- retransmitting packet with seq =  12
14:05:37.649636 -- ACK for packet = 8 is received
14:05:37.650899 -- packet with seq = 13 is sent, sliding window = {9, 10, 11, 12, 13} 
14:05:37.651004 -- ACK for packet = 9 is received
14:05:37.651082 -- packet with seq = 14 is sent, sliding window = {10, 11, 12, 13, 14} 
14:05:37.651512 -- ACK for packet = 10 is received
14:05:37.651619 -- packet with seq = 15 is sent, sliding window = {11, 12, 13, 14, 15} 

...
...

```
#### Server

```console

h2$ python3 application.py -s -i 10.0.1.2 -p 8080 -d 8

SYN packet is received
SYN-ACK packet is sent
ACK packet is received
Connection established
14:05:32.439323 -- packet 1 is received
14:05:32.439399 -- sending ack for the received 1 
14:05:32.439501 -- packet 2 is received
14:05:32.439753 -- sending ack for the received 2 
14:05:32.439816 -- packet 3 is received
14:05:32.439933 -- sending ack for the received 3 
14:05:32.439971 -- packet 4 is received
14:05:32.440055 -- sending ack for the received 4 
14:05:32.440093 -- packet 5 is received
14:05:32.440166 -- sending ack for the received 5 
14:05:32.540440 -- packet 6 is received
14:05:32.541091 -- sending ack for the received 6 
14:05:32.541229 -- packet 7 is received
14:05:32.541384 -- sending ack for the received 7 
14:05:32.541472 -- out-of-order packet 9 is received
14:05:32.542064 -- out-of-order packet 10 is received
14:05:32.641781 -- out-of-order packet 11 is received
14:05:32.642411 -- out-of-order packet 12 is received
14:05:37.648725 -- packet 8 is received
14:05:37.649343 -- sending ack for the received 8 
...
...

The throughput is 0.17 Mbps
Connection Closes

```

### Discussion#4 

To demonstrate how effective your code is, use tc-netem (refer to the [lab manual](https://github.com/safiqul/2410/blob/main/docs/netem/mininet-tc-delay.md))) to simulate packet loss. To do this, modify your simple-topo.py file by commenting out line#43 `net["r"].cmd("tc qdisc add dev r-eth1 root netem delay 100ms")` and uncommenting line#44: `net["r"].cmd("tc qdisc add dev r-eth1 root netem delay 100ms loss 2%")`. Test with a loss rate of 5% as well. Include the results in your discussion section and explain what you observe.

Below are example outputs from both the client and server for the netem case with a `2%` loss rate.

#### client 

```console

h1$ `python3 application.py -c  -f Photo.jpg -i 10.0.1.2 -p 8080 -w 5`

Connection Establishment Phase:

SYN packet is sent
SYN-ACK packet is received
ACK packet is sent
Connection established

Data Transfer:

14:22:57.475156 -- packet with seq = 1 is sent, sliding window = {1} 
14:22:57.475191 -- packet with seq = 2 is sent, sliding window = {1, 2} 
14:22:57.475202 -- packet with seq = 3 is sent, sliding window = {1, 2, 3} 
14:22:57.475210 -- packet with seq = 4 is sent, sliding window = {1, 2, 3, 4} 
14:22:57.475232 -- packet with seq = 5 is sent, sliding window = {1, 2, 3, 4, 5} 
14:22:57.577384 -- ACK for packet = 1 is received
14:22:57.577482 -- packet with seq = 6 is sent, sliding window = {2, 3, 4, 5, 6} 
14:23:02.582289 -- RTO occured
14:23:02.582431 -- retransmitting packet with seq =  2
14:23:02.582679 -- retransmitting packet with seq =  3
14:23:02.582727 -- retransmitting packet with seq =  4
14:23:02.582762 -- retransmitting packet with seq =  5
14:23:02.582792 -- retransmitting packet with seq =  6
14:23:02.684768 -- ACK for packet = 2 is received
14:23:02.685949 -- packet with seq = 7 is sent, sliding window = {3, 4, 5, 6, 7} 
14:23:02.686033 -- ACK for packet = 3 is received
14:23:02.686100 -- packet with seq = 8 is sent, sliding window = {4, 5, 6, 7, 8} 
14:23:02.686144 -- ACK for packet = 4 is received
14:23:02.686196 -- packet with seq = 9 is sent, sliding window = {5, 6, 7, 8, 9} 
...
...
14:23:03.295070 -- packet with seq = 37 is sent, sliding window = {33, 34, 35, 36, 37} 
14:23:03.295134 -- ACK for packet = 33 is received
14:23:03.295192 -- packet with seq = 38 is sent, sliding window = {34, 35, 36, 37, 38} 
14:23:08.301512 -- RTO occured
14:23:08.301668 -- retransmitting packet with seq =  34
14:23:08.301888 -- retransmitting packet with seq =  35
14:23:08.301937 -- retransmitting packet with seq =  36
14:23:08.301971 -- retransmitting packet with seq =  37
14:23:08.302002 -- retransmitting packet with seq =  38
14:23:08.404595 -- ACK for packet = 34 is received
14:23:08.405668 -- packet with seq = 39 is sent, sliding window = {35, 36, 37, 38, 39} 
14:23:08.405719 -- ACK for packet = 35 is received
14:23:08.405751 -- packet with seq = 40 is sent, sliding window = {36, 37, 38, 39, 40} 
14:23:08.405771 -- ACK for packet = 36 is received
14:23:08.405797 -- packet with seq = 41 is sent, sliding window = {37, 38, 39, 40, 41} 
14:23:08.405842 -- ACK for packet = 37 is received
14:23:08.405868 -- packet

```

### Server

```console

h2$ python3 application.py -s -i 10.0.1.2 -p 8080
SYN packet is received
SYN-ACK packet is sent
ACK packet is received
Connection established
14:22:57.577169 -- packet 1 is received
14:22:57.577228 -- sending ack for the received 1 
14:22:57.577325 -- out-of-order packet 3 is received
14:22:57.577657 -- out-of-order packet 4 is received
14:22:57.577801 -- out-of-order packet 5 is received
14:22:57.678966 -- out-of-order packet 6 is received
14:23:02.683714 -- packet 2 is received
14:23:02.684470 -- sending ack for the received 2 
14:23:02.684661 -- packet 3 is received
14:23:02.684921 -- sending ack for the received 3 
...
...
14:23:03.194468 -- sending ack for the received 31 
14:23:03.294129 -- packet 32 is received
14:23:03.294673 -- sending ack for the received 32 
14:23:03.294828 -- packet 33 is received
14:23:03.295058 -- sending ack for the received 33 
14:23:03.295125 -- out-of-order packet 35 is received
14:23:03.295800 -- out-of-order packet 36 is received
14:23:03.395921 -- out-of-order packet 37 is received
14:23:03.396718 -- out-of-order packet 38 is received
14:23:08.403314 -- packet 34 is received
14:23:08.404282 -- sending ack for the received 34 
14:23:08.404497 -- packet 35 is received
14:23:08.404747 -- sending ack for the received 35 
...
...


```
