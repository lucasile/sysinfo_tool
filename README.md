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

To see how memory information is being calculated and converted, see [`displayMemoryUsage()`](#displayMemoryUsage).

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

In the [`main()`](#main) function, we define an array of 6 integers that represent the possible argument flags passed to the program. 0 represents off, and 1 represents on. For samples and time delay, we just put their respective default values, since they are always on.

Then we call a function [`setFlags(int*, int, char**)`](#setFlags) that will take in a reference to the flags array, argc value, and a reference to the argv array.  It will take the arguments provided from the user, parse them, and update the flags array accordingly. If [`setFlags()`](#setFlags) returns 0, there was an error and we return 0 in main to terminate execution of the program.

If there is no error, then we call a function [`composeStats(int*)`](#composeStats) that will take in a reference to the flags array, and accordingly compose the proper output to the terminal based on the flags specified. 

###### displayUserUsage

In the [`displayUserUsage()`](#displayUserUsage) function, we use **utmp.h** to get all the user entries.

Firstly, we reset the pointer to the beginning of the users using `setutent()`. Then we can declare a struct `struct utmp *userEntry = getutent();` which will store a process into the struct. Afterwards we loop until all the entries have been gone over. Inside the loop, we make sure that the process is a user process, then we get the user's information, print it out accordingly, then assign the next user using `getutent()`.

Note: After we are done sampling everything, we accordingly call `endutent()` at the end of [`composeStats(int*)`](#composeStats).

###### displaySystemInformation

In the [`displaySystemInformation()`](#displaySystemInformation) function, we create a buffer struct where we use `uname()` to populate it with system information. If `uname()` returns -1, we have an error and we return out of the function. Otherwise, we can simply access the buffer and print out the system information.

###### displayMemoryUsage

In the [`displayMemoryUsage(int, int sampleSize, int, char[sampleSize][256], double[sampleSize])`](#displayMemoryUsage) function, we create a buffer struct, then use `sysinfo()` to populate it with memory information. If `sysinfo()` returns -1, we have an error and we return out of the function. Otherwise, we calculate and convert the information into usable data as follows.

total_ram = total_bytes / 1000000000  
total_virtual_ram = total_ram + (total_swap / 1000000000)  
used_ram = total_ram - (free_ram / 1000000000)  
used_virtual_ram = total_virtual_ram - ((free_ram + free_swap) / 1000000000)  

Since the memory utilization part shows previous samples, we must store the previously outputted strings in some history array. This is the `char history[sampleSize][256]` parameter. To save the formatted string in the array, we also have an `int sampleCount` parameter that indicates the sample that's currently being rendered. Using these values, we can use `snprintf(history[sampleCount - 1], sizeof(history[sampleCount - 1]), formatted_string_here, values...)` to save the string to the proper location in the array.

If graphical implementation was specified, we will now create a new string called `graphicsLine` that we will concatenate using `strncat()` to our previous string at `history[sampleCount - 1]`. 

To compose `graphicsLine`, we first check what our maximum length can be based on how much ram exists in the system, and what the scale we set is. For example, if `RAM_GRAPHICS_SCALE = 0.1`, then for every 0.1 gb change of the memory utilization, we will add a single graphical character. Similarily to saving the strings in history, we also need to save the memory used in a double array to calculate relative usage. We do this by setting `historyRam[sampleCount - 1] = usedRam;`

We then use `strcpy()` to copy a single '|' to the string. We then check if this is the first sample. If it is, then we will just set the baseline key as `*`. Otherwise, we need to calculate the relative utilization.

To do this, we just get the last entry in the double array by using `historyRam[sampleCount - 2]`, and then find the delta with the current memory used by subtracting them. Afterwards, we can just loop until a double exceeds this delta, incrementing by the `RAM_GRAPHICS_SCALE` each time. However, if the delta is negative, our loop is incorrect. For purely the loop's functionality, we then use the absolute value of the delta instead.

Inside of the loop we check whether the delta is negative or positive. If it is negative, we concatenate the character ':' and if it is positive, we concatenate the character '#'. 

Similarily, outside of the loop we cap the string using the characters '@' and '\*' depending on whether the delta was negative or positive respectively.

Afterwards we also want to append the delta to the string, so we use `snprintf()` to format it to a string, then use `strncat()` to concatenate it to `graphicsLine`.






