Description
====
In this project, a kernel module and userland application were created in other to filter and log network packets (ipv4) by the source/destination ip address.

Required Packages
====
Before building the solution, you must install the linux-headers-generic package. The can be done by running the including install_packages.sh shell script.

How to Build
====
To compile on the command line follow these steps:

1. cmake .
2. make

To compile in Clion, open the project with Clion and click the build button.

Run the program by entering ./bin/main into a terminal. Note that the kernel module is actually included in the main program (main), so if the executable is moved to a different folder the compiled .ko kernel module does not need to moved with it for it to run.

Note: you may receive an "operation not permitted" error; this is ok - just enter some text and it will automatically attempt to relaunch the program as sudo.

Also, running two instances of the program simultaneously will cause the userland code to segfault (due to issues involving shared access to the proc file). So try to avoid that.

Usage
====
Once the module is loaded, you will notice that log information from /var/log/syslog is automatically streamed to the console window. This can be stopped by typing "close". There are a variety of other commands that the program supports, detailed below:

1. "term" launches a new terminal window in which regular commands can be entered
2. "fname logfile.txt" specifies that packets should be logged to logfile.txt
3. "fsize 1000" specifies that the log file should be cleared whenever is exceeds 1000 bytes
4. "refresh" restarts the daemon responsible for reading from the proc file and appending to the log file. This command should never need to be used (except after RESET_ALL) but is included for the sake of completeness
5. "help" describes the format of valid commands
6. "exit" unloads the kernel module and exits
7. "open log.txt" streams new entries added to log.txt to the terminal window in the background (after closing whatever file is currently being streamed to the terminal window). Note that /var/log/syslog is open by default.
8. "close" closes the file currently being streamed to the terminal window
9. "show" displays logged packets in the terminal window as they are received from the kernel module (this is enabled by default). Note that the format of logged packets is "second.nanoseconds: source_ip -> dest_ip [inbound/outbound] {blocked}" where blocked is present only if the packet was blocked.
10. "hide" does not display logged packets as they are received from the kernel module
11. "DISABLE_LOG_BLOCK" disables the automatic logging of blocked packets
12. "ENABLE_LOG_BLOCK" enables the automatic logging of blocked packets (this is the default setting)
13. "LOG BLACK" logs all packets that are not given in the loglist (it is treated as a blacklist)
14. "LOG WHITE" (the default) logs all packets that are given in the log list (it is treated as a whitelist)
15. "LOG ADD_ADDR 204.79.197.200" adds 204.79.197.200 to the log list
16. "LOG REMOVE_ADDR 204.79.197.200" removes 204.79.197.200 from the log list
17. "LOG REMOVE_ALL" removes all addresses from the log list
18. "LOG RESET" resets all settings changed by the various LOG commands to default values
19.  "BLOCK $PARAM $IP" has exactly the same syntax as the LOG commands given above (where $IP is not required for all the commands), just for the block list instead of the log list. Note that the block list is in mode BLACK by default and blocks all packets on the block list, while in mode WHITE it blocks all packets not on the block list.
20. "RESET_ALL refresh" resets all settings, both LOG ones, BLOCK ones, and global settings (such as fname, fsize, show/hide, and LOG_BLOCK). Additionally, the file which is open is reset to /var/log/syslog. The reason why refresh needs to be specified with RESET_ALL is so that commands such as "RESET_ALL fname log2.txt" will work properly and log data to log2.txt instead of logging some data to the default logfile (log.txt) while the fname command is being executed.

As per convention, note that a status code of 0 means the command sent to the kernel module succeeded and 1 means that it failed. Information about the failure is usually logged to /var/log/syslog.

Test Cases
====
The first simple test case (I) involves logging packets to/from bing.com

0. Enter "RESET_ALL refresh" to reset all settings to default values
1. First, enter term to launch a new terminal window. In this window, enter "ping bing.com" to get the IP address of bing. The one I received is 204.79.197.200, although the address you get may be different depending on which server is contacted
2. Next enter "LOG ADD_ADDR 204.79.197.200" into the program to log packets to/from this addr. you should notice that logged packets start showing up on the screen. if not, enter "ping 204.79.197.200" into a terminal window.
3. Finally make sure the data is actually being written to the log file by entering "hide" followed by "open log.txt"

Another simple test case (II) involves blocking packets to/from bing.com

0. Enter "RESET_ALL refresh" to reset all settings to default values
1. First, enter term to launch a new terminal window. In this window, enter "ping bing.com" to get the IP address of bing. The one I received is 204.79.197.200, although the address you get may be different depending on which server is contacted
2. Next enter "BLOCK ADD_ADDR 204.79.197.200" to block packets to/from this addr. you should notice that the blocked packets start showing up on the screen. Also the ping program should return an error (sendmsg: operation not permitted). You can also try accessing bing in a web browser, which should fail (unless it tries to connect to a different server). Try accessing google and it should succeed

A variety of other tests can also be performed, such as using ps -aux to confirm that no processes are left over after the program exits and lsmod (|grep netmod) to confirm that the module is unloaded once the program is done executing.

Additionally, whitelist support for packet filtering can be tested via the BLOCK WHITE command. Also, allowed packets can be logged instead of blocked ones by entering in the following commands: DISABLE_LOG_BLOCK and LOG BLACK. DISABLE_LOG_BLOCK disables the logging of blocked packets (unless the source/dest ip is explicitly included in the log list and this list is a whitelist) and LOG BLACK ensures that the log list is used to blacklist instead of whitelist ip addresses for the purposes of logging packets.

Finally, that the log file is recreated after it exceeds a certain size limit (5,000,000 bytes, or 5MB by default) can be tested by lowering the limit to a couple of kilobytes via "fsize 5,000" and then checking that the file size does not significantly exceed 5 kB (it is only recreated after it exceeds the size limit, so it may use a couple bytes over the size limit before being recreated).

Actually, one more test that can be performed is entering invalid information in (invalid commands, invalid ip addresses, etc) and seeing what happens. Specifically, if one tries to add the same ip address to the block list twice, remove an ip address that isn't in the list, or overfill the block/log lists with ip addresses, the kernel module is smart enough to log such an attempt and return an appropriate error code (1) instead of crashing the system. The same applies if invalid messages are sent to the proc file used by the kernel module.
