# Assignment 1: Sockets, Mininet, & Performance

### Due: Sep 18, 2024, 11:59 PM

## AWS Setup 
Follow [this guide](https://www.eecs489.org/Project%201%20-%20Getting%20Started.pdf) to set up your EC2 instance for P1. Make sure you select the correct AMI (`EECS489P1`) instead of Ubuntu.   

## Development Tips
There are a few choices for how to develop and test code for this project:
1. You may write code on your local machine and use `rsync` to upload files to your EC2 instance. 
2. You may SSH into your EC2 instance and develop using your favorite terminal-based editor. 
3. You can SSH into your EC2 instance using VSCode, developing in a remote VSCode session. [Here](https://code.visualstudio.com/docs/remote/ssh) is a helpful guide on SSH-ing through VSCode. Your VSCode SSH config
file will probably look something like this:
```
Host [ec2-instance].compute-1.amazonaws.com
  HostName [ec2-instance].compute-1.amazonaws.com
  User ubuntu
  IdentityFile "[rooted path to your .pem file]"
```

## Glossary  
- File Descriptor(fd): An fd is an integer that represents an opened "file", except that the "file" here refers to the generic interface to IO(read/write data). 
- Socket: Abstraction of a connection between 2 machines. This is an fd you use in your code to refer to the connection. You pass this fd to various functions to work with the socket.
- Mininet: A simulated network with hosts, routers(switches), links, controllers. You will always work inside of the Mininet. 
- Topology: The graph structure of the network.
- Node: The nodes in the topology.
- Link: The edges in the topology.
- Path: The path consists of links to go from 1 node to another. 
- Link bandwidth: A link's bandwidth means how many maximum data(bits) it can transfer per time period(s). This is a fixed number given in the topology. 
- Link latency: A link's latency means how long does it take for 1 bit to go from 1 node to another. This is a fixed number given in the topology. 
- Throughput: A measured value representing how many data you transferred over a fixed time period. This is not a fixed number and is measured.
- Host(Mininet): Think of this like a computer with its own IP address, ethernet address, etc. All the mininet hosts share the same file system with the base machine, meaning you don't have to worry about copying files to the hosts.
- Server: In the context of this project, the server is a mininet host you run iPerfer in server mode and listen to any connections.
- Client: In the context of this project, the client is a mininet host you run iPerfer in client mode and connects to the server. 


## Objective
- Learn about socket programming: Learning how to send/receive data to/from other machines with sockets. You will do this for all later projects.
- Get used to AWS: You will be developing and testing your code on remote machines for later projects. 
- Get used to Mininet: Learning basic things like how to enter a host, how to start the mininet with topology, etc.
- Learn to measure basic properties of a network: Throughput and latency are important concepts you will need in measuring performance. Also learn to read a topology. 

## Overview

[`iPerf`](https://iperf.fr/) is a common tool used to measure network bandwidth. It functions as a speed test tool for TCP, UDP, and SCTP. In this assignment, you will write your own version of this tool in C/C++ using sockets. You will then use your tools to measure the performance of virtual networks in Mininet and explain how link characteristics and multiplexing impact performance.

* [Part 1](#part1): Mininet Tutorial (Not Graded)
* [Part 2](#part2): Write `iPerfer`
* [Part 3](#part3): Measurements in Mininet
* [Part 4](#part4): Create a Custom Topology
* [Submission Instructions](#submission-instr)
* [Autograder](#autograder)

<!--- Before you start doing anything with this project, however, please [register your github username with us](https://docs.google.com/forms/d/e/1FAIpQLSeQXMmYr_m7A9GPhSra4yguaS7PR3fw1QE7UIhsC0_KwwTdmg/viewform?usp=sharing) before 5p.m on Tuesday, Sep.5th. This is so that we can create a private repository for you to store your code and answers for this project. --->

## Learning Outcomes

After completing this programming assignment, students should be able to:

* Write applications that use sockets to transmit and receive data across a network
* Explain how latency and throughput are impacted by link characteristics and multiplexing

<a name="part1"></a>
## Part 1: Mininet Tutorial (Not Graded)`

Before you write or test `iPerfer`, you will learn how to use Mininet to create virtual networks and run simple experiments. According to the [Mininet website](http://mininet.org/), *Mininet creates a realistic virtual network, running real kernel, switch and application code, on a single machine (VM or native), in seconds, with a single command.* We will use Mininet in programming assignments throughout the semester.

### Running Mininet

It is best advised to run Mininet in a virtual machine (VM) or on your own Linux machine. If you don't have access to a linux machine, we will be using EC2 on AWS that will give you a linux environment with images that have Mininet pre-installed. Please following this <a href"link">tutorial </a> to start using AWS.

<!---For the former, we will be using [VirtualBox](https://www.virtualbox.org/), which is a free and open-source hypervisor. Please download and install the latest version of VirtualBox.

You will be using our VM image ([link here](https://drive.google.com/file/d/1G_VOCKQlMsEfzo0xkAtwNJtWNEKA3Wfr/view?usp=drive_link)) with Mininet 2.3 pre-installed. Please download and import the VM image into VirtualBox. To transfer files to/from your VM you can use the Shared Folder feature provided in VirtualBox. We will go over this in more detail in discussion.
--->
You are welcome to try to set up your own testing environment using the methods outlined in options 2 and 3 [here](http://mininet.org/download/#option-2-native-installation-from-source), however we will only officially be supporting the working with AWS.


<!--- >> ***Hints:*** Here is a [video tutorial](https://youtube.com/watch?v=apx88YDqgO4&si=fqbbjTgg2jv6jP24) on how to install UTM and import image for Mac M1/M2 chip. 


> Here are some possible trouble shooting methods for using shared folder. 
> 1.  Click on the “Device” tab, then select “Insert Guest Additions CD image”
>     Also in the “Device” tab, add the target host folder to “Shared folders”.
>     Restart the VM.
>     After that you may observe an external disk mounted on the guest os (named sf_{your shared folder name} in my case). 
> 2. https://askubuntu.com/questions/1181438/virtualbox-6-0-14-shared-folder-doesnt-appear-in-media
> 3. https://gist.github.com/estorgio/1d679f962e8209f8a9232f7593683265
> 4. https://www.youtube.com/watch?v=N4C5CeYfntE. --->

### Mininet Walkthrough

Once you have access to Mininet, you should complete the following sections of the standard [Mininet walkthrough](http://mininet.org/walkthrough/):

* All of Part 1, except the section "Start Wireshark"
* The first four sections of Part 2—"Run a Regression Test", "Changing Topology Size and Type", "Link variations", and "Adjustable Verbosity"
* All of Part 3

At some points, the walkthrough will talk about software-defined networking (SDN) and OpenFlow. We will discuss these during the second half of the semester, so you do not need to understand what they mean right now; you just need to know how to run and interact with Mininet. We will review using Mininet in discussion as well.

### Please check out [this post](https://edstem.org/us/courses/61627/discussion/5197510) on tips of using mininet. 

> **NOTE:** You do not need to submit anything for this part of the assignment. This portion is meant to help your understanding for this and future assignments.

### Note: Running Multiple Hosts at Once in Mininet

Over the course of this assignment (and especially in Part 3) you may want to run 
programs in multiple hosts at once using mininet (e.g. to run a client in one
host and a server in another). Typically, if you had access to a display for the
EC2 instance, you could simply start different terminals for each host in Mininet
(through something like `xterm h1`). However, SSH-ing into EC2 creates some 
limitations. The following shell tips may be useful:

- Use `>` to redirect output to a file. The following code will print the results
  of pinging h2 from h1 into `ping_h1_h2.txt`. 
  ```
  $ h1 ping -c 5 10.0.0.2 > ping_h1_h2.txt
  ```
- Use `&` to run a process in a non-blocking way by spawning a new thread in the
  background. The following code will do the exact same thing as the code above, 
  except you will be able to run another command while the first one is running.
  ```
  $ h1 ping -c 5 10.0.0.2 > ping_h1_h2.txt &
  ``` 
- Using `&` will typically cause the PID (process ID) of the background process
  to be printed out. You can kill the process using this PID to make it stop 
  using the following command:
  ```
  kill -9 [PID]
  ```

Through these tips, you can run code in multiple hosts at once by sending
processes to the background and redirecting output to appropriate files. 

<a name="part2"></a>
## Part 2: Write `iPerfer`

In this portion of the assignment, you will write your own version of `iPerf` to measure network bandwidth. Your tool, called `iPerfer`, will send and receive TCP packets between a pair of hosts using sockets.

> **NOTE:** You may refer to [Beej's Guide to Network Programming Using Internet Sockets](https://beej.us/guide/bgnet/html/) for socket programming. Discussion sections will also review the some of the basics.

When operating in client mode, `iPerfer` will send TCP packets to a specific host for a specified time window and track how much data was sent during that time frame; it will calculate and display the bandwidth based on how much data was sent in the elapsed time. When operating in server mode, `iPerfer` will receive TCP packets and track how much data was received during the lifetime of a connection; it will calculate and display the bandwidth based on how much data was received and how much time elapsed during the connection.

> **NOTE:** When measuring time, we highly recommend using `std::chrono::high_resolution_clock` for checking and computing passed time. From here, you can cast the time into milliseconds for more accurate time keeping.

### Server Mode

To operate `iPerfer` in server mode, it should be invoked as follows:

`$ ./iPerfer -s -p <listen_port>`

* `-s` indicates this is the `iPerfer` server which should consume data
* `listen_port` is the port on which the host is waiting to consume data; the port should be in the range `1024 ≤ listen_port ≤ 65535`

> For simplicity, you can assume these arguments will appear exactly in the order listed above.

You can use the presence of the `-s` option to determine `iPerfer` should operate in server mode.

If arguments are missing or extra arguments are provided, you should print the following as exactly specified (with a newline after it) and exit with status code 1:

`Error: missing or extra arguments`

If the listen port argument is less than 1024 or greater than 65535, you should print the following as exactly specified (with a newline after it) and exit with status code 1:

`Error: port number must be in the range of [1024, 65535]`

When running as a server, `iPerfer` must listen for TCP connections from a client and receive data as quickly as possible. It should then wait for some kind of message from the client indicating it is done sending data (we will call this a **FIN** message). The server should then send the client an **acknowledgement** to this FIN message. It is up to you to decide the format of these FIN and acknowledgement messages.

Data should be read in chunks of 1000 bytes. Keep a running total of the number of bytes received.

After the client has closed the connection, `iPerfer` server must print a one-line summary in the following format:

`Received=X KB, Rate=Y Mbps`

where X stands for the total number of bytes received (in kilobytes), and Y stands for the rate at which traffic could be read in megabits per second (Mbps).
Note X should be an integer and Y should be a decimal with **three digits** after the decimal mark. There are no characters after the `Mbps`, and there should be a newline.

For example:
`Received=6543 KB, Rate=5.234 Mbps`

The `iPerfer` server should shut down gracefully after it handles one connection from a client.

> **Note:** Please use `setsockopt` to allow reuse of the port number, this will make your life easier for testing and will allow you to pass the autograder, which runs the `iPerfer` server with the same port number each time. 

### Client Mode

To operate `iPerfer` in client mode, it should be invoked as follows:

`$ ./iPerfer -c -h <server_hostname> -p <server_port> -t <time>`

* `-c` indicates this is the `iPerfer` client which should generate data
* `server_hostname` is the hostname or IP address of the `iPerfer` server which will consume data
* `server_port` is the port on which the remote host is waiting to consume data; the port should be in the range 1024 ≤ `server_port` ≤ 65535
* `time` is the duration in seconds for which data should be generated. We will only test this with an integer value (i.e feel free to use time.h)

> For simplicity, you can assume these arguments will appear exactly in the order listed above.

You can use the presence of the `-c` option to determine that `iPerfer` should operate in the client mode.

If any arguments are missing or extra arguments are provided, you should print the following as exactly specified (with a newline after it) and exit with status code 1:

`Error: missing or extra arguments`

If the server port argument is less than 1024 or greater than 65535, you should print the following as exactly specified (with a newline after it) and exit with status code 1:

`Error: port number must be in the range of [1024, 65535]`

If the time argument ends up parsing to less than or equal to 0, you should print the following as exactly specified (with a newline after it) and exit with status code 1:

`Error: time argument must be greater than 0`

If both the port and time argument are invalid, print only the port error message.

When running as a client, `iPerfer` must establish a TCP connection with the server and send data as quickly as possible(just use a loop) for `time` seconds(hint: use `std::chrono::high_resolution_clock` or `time.h`). Data should be sent in chunks of 1000 bytes and the data should be all zeros(note: this is the char `'\0'`(0), not the char `'0'`(48)). Keep a running total of the number of bytes sent. After the client finishes sending its data, it should send a FIN message and wait for an acknowledgement before exiting the program. (hint: if your client/server is stuck here, try `shutdown(sockfd,SHUT_WR);`(no more transmission) after client finishes sending FIN) 

`iPerfer` client must print a one-line summary in the following format:

`Sent=X KB, Rate=Y Mbps`

where X stands for the total number of bytes sent (in kilobytes), and Y stands for the rate at which traffic could be read in megabits per second (Mbps).
Note X should be an integer and Y should be a decimal with three digits after the decimal mark. There are no characters after the `Mbps`, and there should be a newline.

For example:
`Sent=6543 KB, Rate=5.234 Mbps`

You should assume 1 kilobyte (KB) = 1000 bytes (B) and 1 megabyte (MB) = 1000 KB. As always, 1 byte (B) = 8 bits (b).

> **NOTE:** When calculating the rate, **do not** use the `time` argument, rather measure the time elapsed from when the client first starts sending data to when it receives its acknowledgement message. Additionally, note that the throughput is in Kilobytes (KB) whereas the rate is in Megabits per second. (Mbps) Make sure your units are accurate to avoid losing points on the autograder.

### Testing

You can test `iPerfer` on any machines you have access to. However, be aware the certain ports may be blocked by firewalls on end hosts or in the network, so you may not be able to test your program on all hosts or in all networks.

The primary mode for testing should be using Mininet. You should complete [Part 2](#part2) of this assignment before attempting that.

You should receive the same number of bytes on the server as you sent from the client. However, the timing on the server may not perfectly match the timing on the client. Hence, the bandwidth reported by client and server may be slightly different.

The autograder will be released about halfway through the assignment. Instructions for submission are [here](#submission-instr). It is not meant to be your primary source of testing/debugging, but is rather intended for you to see your overall progress.

<a name="part3"></a>
## Part 3: Measurements in Mininet

For the third part of the assignment you will use the tool you wrote (`iPerfer`) and the standard latency measurement tool `ping` (`ping` measures RTT), to measure the bandwidth and latency in a virtual network in Mininet. You must include the output from some of your experiments and the answers to the questions below in your submission. Your answers to the questions should be put in the file `answers.txt` **that we provide**. Please do **NOT** change the format of the `answers.txt` file (none of the answers to the questions should take more than one or two sentences).

Read the `ping` man page to learn how to use it.

A python script to run Mininet with the topology described below is provided [here](https://github.com/eecs489staff/a1-sockets-mininet/tree/main/starter_code) along with other files that you will find useful in completing this assignment.

To run Mininet with the provided topology, run the Python script `assignment1_topology.py` using sudo:

`sudo python3 assignment1_topology.py`

This will create a network with the following topology:

<img src="assignment1_topology.png" title="Assignment 1's topology" alt="Should be showing the topology described in assignment1_topology.py" width="350" height="220"/>

If you have trouble launching the script, a common fix is to first try running `sudo mn -c`, and then try launching the script again. This will clear anything on Mininet at that point.

Hosts (`h1` to `h10`) are represented by squares and switches (`s1` to `s6`) are represented by circles; the names in the diagram match the names of hosts and switches in Mininet. The hosts are assigned IP addresses 10.0.0.1 through 10.0.0.10; the last number in the IP address matches the host number.

> **NOTE:** When running ping and `iPerfer` in Mininet, you must use IP addresses, not hostnames. Also, if you are not confident your `iPerfer` is working correctly, feel free to use `iperf` for any throughput measurements noted below. Output using either program will be accepted.

### (Optional) Using the Mininet Python API instead of Command Line Interface (CLI) 
For the following questions, you may find it easier to use the Mininet Python
API to do measurements instead of typing commands in through the CLI, which can
be clunky and error-prone. This is certainly not required, but may make your
life easier. 

Here are a few links to supplement your understanding of the Mininet Python API:
- https://github.com/mininet/mininet/wiki/Introduction-to-Mininet#running
- https://mininet.org/api/classmininet_1_1net_1_1Mininet.html
- https://mininet.org/walkthrough/#part-4-python-api-examples 

A good starting point is to open up `assignment1_topology.py`. In the `main`
function, after `net.start()`, you can input the following lines of code:
```
h1 = net.get('h1')
h1.cmd('ping -c 5 10.0.0.2 > latency.txt')
```
Try it out and see what happens!

### Q1: Link Latency and Throughput
First, you should measure the RTT and bandwidth of each of the five individual links between switches (`L1` - `L5`). You should run ping with 20 packets and store the output of the measurement on each link in a file called `latency_L#.txt`, replacing # with the link number from the topology diagram above. You should run `iPerfer` for 20 seconds and store the output of the measurement on each link in a file called `throughput_L#.txt`, replacing # with the link number from the topology diagram above.

### Q2: Path Latency and Throughput
Now, assume `h1` wants to communicate with `h10`. What is the expected latency and throughput of the path between the hosts? Put your prediction in the `answers.txt` file under question 2.

Measure the latency and throughput between `h1` and `h10` using `ping` and `iPerfer`. It does not matter which host is the client and which is the server. Use the same parameters as above (20 packets / 20 seconds) and store the output in files called `latency_Q2.txt` and `throughput_Q2.txt`. Put the average RTT and measured throughput in the `answers.txt` file and explain the results. If your prediction was wrong, explain why.

### Q3: Effects of Multiplexing
Next, assume multiple hosts connected to `s1` want to simultaneously talk to hosts connected to `s6`. What is the expected latency and throughput when two pairs of hosts are communicating simultaneously? Put your predictions in your `answers.txt` file under question 3.1.

Use `ping` and `iPerfer` to measure the latency and throughput when there are two pairs of hosts communicating simultaneously; it does not matter which pairs of hosts are communicating as long as one is connected to `s1` and one is connected to `s6`. Use the same parameters as above. You do not need to submit the raw output, but you should put the average RTT and measured throughput for each pair in your `answers.txt` file under question 3.1 and explain the results. If your prediction was wrong, explain why.

Repeat for three pairs of hosts communicating simultaneously and put your answers in `answers.txt` under question 3.2. 

Do not worry too much about starting the clients at the exact same time. As long as the connections overlap significantly, you should achieve the correct results. You can achieve approximately simultaneous testing while using iPerfer by starting the servers first (in the background), and having client commands prepared that you can paste quickly into your terminal. 

### Q4: Effects of Latency
Lastly, assume `h1` wants to communicate with `h10` at the same time `h3` wants to communicate with `h8`. What is the expected latency and throughput for each pair? Put your prediction in your `answers.txt` file under question 4.

Use `ping` and `iPerfer` to conduct measurements, storing the output in files called `latency_h1-h10.txt`, `latency_h3-h8.txt`, `throughput_h1-h10.txt`, and `throughput_h3-h8.txt`. Put the average RTT and measured throughput in your `answers.txt` file and explain the results. If your prediction was wrong, explain why.

> **NOTE:** For the latency portions of `answers.txt`, make sure you are calculating the RTT for them. (The tools should report the RTT, as well)

<a name="part4"></a>
## Part 4: Create a Custom Topology
For the last part of this assignment, write a python script to create a custom network topology in Mininet that has at least 5 hosts and 5 switches. Save your python script as `<uniqname>_topology.py`. You might find looking into the source code for `assignment1_topology.py` particularly helpful. (Note: You will need a script that creates topologies for later homeworks) 

Finally, create a visualization of your custom topology (using circles to denote switches and squares to represent hosts) and save it as `<uniqname>_topology.png`. You may use any program or sketch to do this part. (We recommend [draw.io](https://app.diagrams.net/))

<a name="submission-instr"></a>
## Submission Instructions
Submission to the autograder will be done [here](https://g489.eecs.umich.edu/). 

To submit:
1. Make sure all the files you want to submit are in the same folder and you are able to compile your code with this flat file structure. **This includes the hand-graded files for Part 3 and Part 4 of the project.** 
2. Go to autograder website specified above and submit you files. Ignore any errors about missing .c files.
3. Please provide a Makefile that will produce an `iPerfer` executable when run with the command `make iperfer`.

<!---Your assigned repository must contain:

* The source code for `iPerfer`: all source files for `iPerfer` should be in a folder called `iPerfer`; the folder should include a `Makefile` to compile the sources. The autograder expects an `iPerfer` executable to be present after running `make` in this directory. The autograder will run `make clean` then `make` (must support both), if either do not work the submission will fail.
* Your measurement results and answers to the questions from Part 3: all results and a text file called `answers.txt` should be in a folder called `measurements`.
* Your custom network topology code and its visualization (`<uniqname>_topology.py` and `<uniqname>_topology.png`).

Example final structure of repository:
```
$ tree ./p1-joebb/
./p1-joebb/
├── README.md
├── assignment1_topology.png
├── assignment1_topology.py
├── iPerfer
│   ├── ** source files **
│   ├── Makefile <- supports "make clean" and "make"
│   └── iPerfer <- executable from running make
├── joebb_topology.png
├── joebb_topology.py
└── measurements
    ├── answers.txt
    ├── latency_L1.txt
    ├── latency_L2.txt
    ├── latency_L3.txt
    ├── latency_L4.txt
    ├── latency_L5.txt
    ├── latency_Q2.txt
    ├── latency_h1-h10.txt
    ├── latency_h3-h8.txt
    ├── throughput_L1.txt
    ├── throughput_L2.txt
    ├── throughput_L3.txt
    ├── throughput_L4.txt
    ├── throughput_L5.txt
    ├── throughput_Q2.txt
    ├── throughput_h3-h8.txt
    └── throughput_h1-h10.txt
```


When grading your assignment, we will **ONLY** pull from your assigned repository, and only look at commits before the deadline.
--->
<a name="autograder"></a>
## Autograder

The autograder tests the following aspects of `iPerfer`
1. Incorrect argument handling
2. Format of your iPerfer output
3. Correctness of iPerfer output (both the `Sent` and `Received` values as well as `Rate`).

Because of the guarantees of TCP, both Sent and Received should be the same. The `Rate` is tested by first running `iperf` over a link, then comparing your `iPerfer` output to the result given a reasonable margin of error.

<!---Our autograder runs the following versions of gcc/g++, please make sure your code is compatible
```
$ gcc --version
gcc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$ g++ --version
g++ (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
--->
## Acknowledgements
This programming assignment is based on Aditya Akella's Assignment 1 from Wisconsin CS 640: Computer Networks.
