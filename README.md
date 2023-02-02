# System Information Tool for Linux. Made for CSCB09

## Documentation

---

### Compilation

To compile to an executable, run 

`$ gcc -o sysinfo system_info.c`

in a terminal in the same directory as system_info.c

---

### Arguments
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

---

###### Graphical Legend
':' represents a unit of relative decrease of the memory utilization since the last sample.
'#' represents a unit of relative increase of the memory utilization since the last sample.
A relative decrease graphical string will be suffixed by '@'.
A relative increase graphical string will be suffixed by '*'.  

---

To view the output of the program without the terminal refreshing after samples, run
`$ ./sysinfo --sequential`
Note: This is particularly useful when redirecting the output to a file.

To modify the amount of samples taken run,
`$ ./sysinfo --samples=N`
where N represents the amount of samples you want as a positive integer.

---

###### Example

`$ ./sysinfo --samples=5` will only take 5 samples.

---

To modify the time between samples, run,
`$ ./sysinfo --tdelay=N`
where N represents the time taken between samples in seconds as a positive integer.

---

###### Example

`$ ./sysinfo --tdelay=2` will take 2 seconds between samples.

---

By default, the program will take 10 samples with 1 second in between each sample aka
`$ ./sysinfo --samples=10 --tdelay=1`

---






