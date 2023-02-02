# System Information Tool for Linux. Made for CSCB09

Author: Lucas Ilea

Table of contents
=================

<!--ts-->
   * [Documentation](#documentation)
      * [Compilation](#compilation)
	  * [Arguments](#arguments)
	  * [Code](#code)
<!--te-->

Documentation
---

Compilation
---

To compile to an executable, run 

`$ gcc -o sysinfo system_info.c`

in a terminal in the same directory as system_info.c

Arguments
---
Running the program with `$ ./sysinfo --help` you will see the possible arguments:

```bash
./sysinfo --help (show this help page)
./sysinfo --user (default argument, report users connected and sessions)
./sysinfo --system (default argument, report system usage)
./sysinfo --graphics (include graphical output where possible)
./sysinfo --sequential (information output sequentially, no refreshing of screen)
./sysinfo --samples=N (take N samples over the specified time)
./sysinfo --tdelay=T (take N samples previously over T time in seconds)
```

By default, running `$ ./sysinfo` will run the program with the user and system arguments, aka 
`$ ./sysinfo --user --system`

To see users connected and their sessions, run
`$ ./sysinfo --user`

To see system information including CPU and memory utilization, run
`$ ./sysinfo --system`
Note: The program will pause for 500ms at the beginning when running this argument to provide accurate data by gathering a baseline sample.

To have a graphical output of the CPU and memory utilization, run
`$ ./sysinfo --graphics`
Note: This will have no effect on user statistics.

***

###### Graphical Legend
':' represents a unit of relative decrease of the memory utilization since the last sample.
'#' represents a unit of relative increase of the memory utilization since the last sample.
A relative decrease graphical string will be suffixed by '@'.
A relative increase graphical string will be suffixed by '*'.  

***

To view the output of the program without the terminal refreshing after samples, run
`$ ./sysinfo --sequential`
Note: This is particularly useful when redirecting the output to a file.

To modify the amount of samples taken run,
`$ ./sysinfo --samples=N`
where N represents the amount of samples you want as a positive integer.

***

###### Example

`$ ./sysinfo --samples=5` will only take 5 samples.

***

To modify the time between samples, run,
`$ ./sysinfo --tdelay=N`
where N represents the time taken between samples in seconds as a positive integer.

***

###### Example

`$ ./sysinfo --tdelay=2` will take 2 seconds between samples.

***

By default, the program will take 10 samples with 1 second in between each sample aka
`$ ./sysinfo --samples=10 --tdelay=1`

***

Code
---

Near the start of the file, you will find two constant string arrays (2D char arrays), that define the help command messages and error messages. This is to keep them in one place and organized so I can easily change them if I need to in the future.

There are also other constants to define how the `/proc/stat` file is formatted and the graphics scale for both CPU and memory utilization.

Underneath are a collection of the function prototypes.

###### main

In the [`main`](#main) function, we define an array of 6 integers that represent the possible argument flags passed to the program. 0 represents off, and 1 represents on. For samples and time delay, we just put their respective default values, since they are always on.

Then we call a function [`setFlags(int*, int, char**)`](#setFlags) that will take in a reference to the flags array, argc value, and a reference to the argv array.  It will take the arguments provided from the user, parse them, and update the flags array accordingly. If [`setFlags()`](#setFlags) returns 0, there was an error and we return 0 in main to terminate execution of the program.

If there is no error, then we call a function [`composeStats(int*)`](#composeStats) that will take in a reference to the flags array, and accordingly compose the proper output to the terminal based on the flags specified. 

###### displayUserUsage

###### setFlags





