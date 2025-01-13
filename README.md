# Assignment 1: Sockets, Mininet, & Performance

### Due: January 29, 2025 (11:59 PM)

In this project, you will create your own simplified version of [iPerf](https://iperf.fr/), a widely used network measurement tool. This project has the following goals:
- Learn how to set up and use [Mininet](https://mininet.org/), a network emulation software that enables you to create custom network topologies and test your code all on a single machine. This will be used again in Project 4, and is a useful tool for Projects 2 and 3. 
- Become familiar with programming with Linux [sockets](https://man7.org/linux/man-pages/man2/socket.2.html), which are essential for creating applications that involve network communication and will be used in all four projects. 
- Develop an intuition for and understanding of key network performance characteristics (throughput and latency).
- Become familiar with setting up a VM and using a build system (CMake), which are essential tools for both this class and software development in general. 

This project is divided into the following parts:
* [Part 0](#part0): VM and Dev Environment Setup
* [Part 1](#part1): Mininet Tutorial
* [Part 2](#part2): Custom Mininet Topology
* [Part 3](#part3): `iPerfer`
* [Part 4](#part4): Mininet Measurements

You can find [submission instructions here](#submission-instr). The first two parts are ungraded; the last three have deliverable components. 

<a name="part0"></a>
## Part 0: Setting Up Your VM and Dev Environment

For Projects 1 and 4, you will be using a virtual machine (VM) running Ubuntu Server 24.04. This is because we will be using Mininet (more about this later) to simulate different network topologies. In order to use Mininet, we need certain kernel features, which makes running a virtual machine a necessity.

### Finding a VM Software

We have decided not to prescribe a certain virtual machine for you to use; you may use any VM of your choice. However, in our own experiences, we have found offerings by VMWare to work well. VMWare Workstation (for Windows) and VMWare Fusion Player (for both x86 and ARM Mac) are free for personal use and have worked well for us. However, you need not use them if you have other preferences. 

**At this point in the spec, follow the instructions in `VM_Setup_Guide.pdf` to set up your virtual machine.**

### Development Tips
1. The Autograder is using C++23. Feel free to use whichever C++ features you would like that are included within the standard.
2. We highly recommend that you set up some kind of remote development environment to interact with your virtual machine. This is covered in the setup guide; you can `ssh` into the VM to develop in there from an editor on your local machine. 
3. To get your project files into the filesystem of the virtual machine, there are several options:
    1. You could use shared folders, which are typically configurable through VM settings. These can work really well, but can also be difficult to set up sometimes and may not work relaibly on your machine. 
    2. You can manually `scp` files from your local machine to the VM, similarly to how you `ssh` into the VM. 
    3. [Recommended] You can maintain a **private** GitHub repository of all your code and set up a GitHub authentication token in the VM to clone your repository inside the VM. This is pretty foolproof, though occaisionally annoying if you forget to pull. 

> Note: We encourage you to use GitHub for code versioning and remote backups. Please make sure that any GitHub repository you create for your code in this class is **private**. Creating a public repository with your code, even if by accident, is considered a violation of the Honor Code. If you come across any public GitHub repositories with code for this project, please inform the instructors immediately. 

### CMake and Useful Libraries
One tool that is very common in the real world, but is seldom taught in classes are build systems (of which CMake is one). Build systems allow you to declaratively specify what programs can be built, and what dependencies each program has within a codebase.

For this project, you will be using CMake to build `iPerfer`. You will be using in external dependencies to both make your life easier, and to get used to working with other people's code. These dependencies are:
1. `cxxopts`: No one likes dealing with `getopt.h`. `cxxopts` allows you to easily define and use command line arguments for your program.
2. `spdlog`: Good logging is essential for real world programs. `spdlog` is a logging library that makes this easier. The library provides several levels of logging; the three most commonly used levels are `error`, `info`, and `debug`. Setting a certain logging level prints all messages at that level or lower (e.g. `error` is the lowest and `debug` is the highest here). 

To show the value of these tools, imagine accepting an server/client config and port as arguments, and then logging that you're listening/sending to that port. With these tools, it's as simple as:

```
    cxxopts::Options options("iPerfer", "A simple network performance measurement tool");
    options.add_options()
        ("s, server", "Enable server", cxxopts::value<bool>())
        ("p, port", "Port number to use", cxxopts::value<int>());

    auto result = options.parse(argc, argv);

    auto is_server = result["server"].as<bool>();
    auto port = result["port"].as<int>();

    spdlog::debug("About to check port number...")
    if (port < 1024 || port > 0xFFFF) {
      spdlog::error("Port number should be in interval [1024, 65535]; instead received {}", port); 
      return; 
    }

    spdlog::info("Setup complete! Server mode: {}. Listening/sending to port {}", is_server, port);
```
As you can see, you can easily and cleanly combine error, debugging, and informational messages in your program. You can set the global logging level using
```
spdlog::set_level(spdlog::level::debug); 
```
You may change the level depending on how much output you are interested in seeing. 

You are highly encouraged to use both `cxxopts` and `spdlog` to make your programs bug-free and easy to read. If you have other favorite tools that perform the same purpose, feel free to use those; however, the course staff will likely not be familiar with them and may not be able to help you. Please do not try to write your own command line input parser or use native `cout` statments for logging; we reserve the right to refuse to debug code that follows these [anti-patterns](https://en.wikipedia.org/wiki/Anti-pattern). 

<a name="part1"></a>
## Part 1: Mininet Tutorial (Not Graded)

Before you write or test `iPerfer`, you will learn how to use Mininet to create virtual networks and run simple experiments. According to the [Mininet website](http://mininet.org/), *Mininet creates a realistic virtual network, running real kernel, switch and application code, on a single machine (VM or native), in seconds, with a single command.* We will use Mininet in programming assignments throughout the semester. You should have installed Mininet as part of your VM setup in the previous part. 

### Mininet Walkthrough

Once you have access to Mininet, you should complete the following sections of the standard [Mininet walkthrough](http://mininet.org/walkthrough/):

* All of Part 1, except the section "Start Wireshark"
* The first four sections of Part 2—"Run a Regression Test", "Changing Topology Size and Type", "Link variations", and "Adjustable Verbosity"
* All of Part 3

At some points, the walkthrough will talk about software-defined networking (SDN) and OpenFlow. We will discuss these during the second half of the semester, so you do not need to understand what they mean right now; you just need to know how to run and interact with Mininet. We will review using Mininet in discussion as well.

> NOTE: You do not need to submit anything for this part of the assignment. This portion is meant to help your understanding for this and future assignments.

### Useful Mininet Commands
Below we've compiled a list of useful Mininet commands. Please refer to the official [Mininet documentation](mininet.org) for more information. These are also covered in the walkthrough. 

To launch Mininet with the standard topology, simply run
```
$ sudo mn
```
Note that Mininet must always run as root (i.e. using `sudo`). You can also launch Mininet using the Python API; an example of this is found in the `util/topology.py` file and more information can be found online. To launch Mininet with the Project 1 topology, you can run 
```
$ sudo python3 topology/topology.py
```
assuming you are currently in the root directory of the project. 

Once you have launched Mininet, you will be inside the Mininet CLI. Your command line prompt should look like
```
mininet>
```
Here are some useful commands inside the Mininet CLI:
```
$ nodes       // shows all current nodes in the network
$ dump        // shows all info about current topology
$ net         // shows all network interfaces
$ h1 ping h2  // run the [ping h2] command on h1
$ h1 bash     // enter a terminal inside h1
```
Once you run something like `$ h1 bash`, you will have a terminal inside of the emulated machine that is Host 1. You can then run any commands that Host 1 could run. In Mininet, all hosts have a shared filesystem. If you forget which host you are in, you can always run `ifconfig` to check your own IP address. 

### General Shell Tips with Mininet
The following shell tips may be useful over the course of this project:

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

### Running Multiple Hosts at Once in Mininet
Over the course of this assignment (and especially in Part 3) you may want to run 
programs in multiple hosts at once using Mininet (e.g. to run a client in one
host and a server in another). Normally, if you are running a Linux machine with an actual display, 
you can do something like `xterm h1 h2` to have separate terminals pop up for Host 1 and Host 2. Unfortunately, 
when using a VM, this is a bit tricker. As such, you have several approaches you can take to run programs on multiple hosts concurrently:

#### Option 1: Use mnbash
To solve this problem, we've developed an `mnbash` command that you can use to run commands on multiple hosts.
To use this, first run the following commands:
```
$ sudo cp util/mnbash /usr/bin/mnbash
$ sudo chmod +x /usr/bin/mnbash
```
This is a one-time setup; once you run these once on your VM, `mnbash` should always work. 

To use this functionality, first ensure a Mininet topology is running (e.g. by running `sudo mn`) in your Linux VM. 
Now, you can `ssh` into multiple terminals in the Linux VM and run commands on Mininet hosts by running something like
```
$ 
```


#### Option 2: Use Python API


#### Option 3: Use background processes to do it natively




<a name="part2"></a>
## Part 2: Write `iPerfer`

In this portion of the assignment, you will write your own version of `iPerf` to measure network bandwidth. Your tool, called `iPerfer`, will send and receive TCP packets between a pair of hosts using sockets.

> **NOTE:** You may refer to [Beej's Guide to Network Programming Using Internet Sockets](https://beej.us/guide/bgnet/html/) for socket programming. Discussion sections will also review the some of the basics.

When operating in client mode, `iPerfer` will send TCP packets to a specific host for a specified time window and track how much data was sent during that time frame; it will calculate and display the bandwidth based on how much data was sent in the elapsed time. When operating in server mode, `iPerfer` will receive TCP packets and track how much data was received during the lifetime of a connection; it will calculate and display the bandwidth based on how much data was received and how much time elapsed during the connection.

> **NOTE:** When measuring time, we highly recommend using `std::chrono::high_resolution_clock` for checking and computing passed time. From here, you can cast the time into milliseconds for more accurate time keeping.

### Setup

To setup the build system, you must complete the `CMakeLists.txt` files under the `cpp/` directory. We have already completed `cpp/CMakeLists.txt` for you. You should fill in `cpp/src/CMakeLists.txt` to compile a program called iPerfer. We have included the code to link the external dependencies as a comment in this file.

The basics section of "An Introduction to Modern CMake" should be more than sufficient to get you through this part, and can be found [here](https://cliutils.gitlab.io/modern-cmake/chapters/basics.html).

To build your CMake program, you *can* use the command line, but we recommend that you use IDE tools instead. For VSCode, this is pretty easy. Simply install the "CMake Tools" extension. Then, open the command pallette, and run "CMake: Configure" (select "Unspecified" if it asks you what kit to use). This will let you get Intellisense (autocomplete) on external dependencies. Then, to run the program, in the command pallette, hit "CMake: Build". This will build your project. Then, you can use a `launch.json` or your preferred way of running a binary to debug your program.

### Server Mode

To operate `iPerfer` in server mode, it should be invoked as follows:

`$ ./iPerfer -s -p <listen_port>`

* `-s` indicates this is the `iPerfer` server which should consume data
* `listen_port` is the port on which the host is waiting to consume data; the port should be in the range `1024 ≤ listen_port ≤ 65535`

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

`iPerfer` client must log a one-line summary using `spdlog::info` in the following format:

`Sent=X KB, Rate=Y Mbps`

where X stands for the total number of bytes sent (in kilobytes), and Y stands for the rate at which traffic could be read in megabits per second (Mbps).

**This should be the only info log your program prints.** You are free to use `spdlog::debug` to output debug logs.

Note X should be an integer and Y should be a decimal with three digits after the decimal mark. There are no characters after the `Mbps`, and there should be a newline. You can use `spdlog` formatting arguments (e.g. `spdlog::info("{:.3f}", my_num)`) to achieve this.

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
1. You will be submitting your projects using GitHub. In order to submit, create a **private** GitHub repo, and add eecs489staff@umich.edu as a collabarator. To the Autograder, only submit a `.aginfo` file that contains only a link to your GitHub repository. We will clone the **main** branch of your repository and use it in our grading.

1. Ensure that when you run CMake and build all of the targets, your CMake configuration outputs a single program named `iPerfer` to the `build/bin` directory.

1. For this project, submit your answers and explanations to GradeScope as PDF files containing all of the sections in `answers.txt`. Additionally, submit your topology picture and your program to GitHub as well.

## Autograder

The autograder tests the following aspects of `iPerfer`
1. Incorrect argument handling
2. Format of your iPerfer output
3. Correctness of iPerfer output (both the `Sent` and `Received` values as well as `Rate`).

Because of the guarantees of TCP, both Sent and Received should be the same. The `Rate` is tested by first running `iperf` over a link, then comparing your `iPerfer` output to the result given a reasonable margin of error.

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

## Acknowledgements
This programming assignment is based on Aditya Akella's Assignment 1 from Wisconsin CS 640: Computer Networks and has been modified by several years of previous EECS 489 staff. 
