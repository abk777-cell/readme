# System Monitoring Tool

**Program File Name:** `B09A1.c`

**Compiling Command:**

```sh
gcc -Wall -Werror -std=c99 B09A1.c -o B09A1 -lm
```

It could use any permutation of -Wall -Werror -std=c99, as long as -lm is a flag since `<math.h>` is a header.

## Overview

This program reports different metrics related to system utilization in real-time. It runs from the command line and outputs system metrics dynamically, with graphical representation of data where applicable. The program is designed to work on Linux-based systems, such as the IA-3170 lab workstations. Code that does not compile and/or run on these lab machines will receive a zero.

## Command Line Arguments

The program accepts the following command-line arguments:

- `--memory`

  - Displays only memory usage information.

- `--cpu`

  - Displays only CPU usage information.

- `--cores`

  - Displays only core-related information.

- `--samples=N`

  - Specifies the number of times system statistics are collected. If not provided, the default is **20**.

- `--tdelay=T`

  - Specifies the delay in microseconds between each sample. If not provided, the default is **500000 microseconds (0.5 seconds)**.

### Positional Arguments

If no flags are provided, the program allows positional arguments in the following order:

```sh
./B09A1 [samples] [tdelay] [--memory] [--cpu] [--cores] [--samples=N] [--tdelay=T]
```

- The first positional argument is interpreted as `samples`.
- The second positional argument is interpreted as `tdelay`.
- The flagged arguments can show up in any permutation as long as they do not appear before a positional argument.

## Default Behavior

If no arguments are provided, the program will display:

- **Memory Utilization**
- **CPU Utilization**
- **Cores Information**
- **Default settings:**
  - `samples=20`
  - `tdelay=500000` microseconds

## Expected Output

The program prints system utilization metrics to **STDOUT**, including:

- **Running Parameters:**

  - Number of samples
  - Sample frequency (delay time)

- **Memory Utilization:**

  - Memory usage of the monitoring tool itself
  - System memory usage (in GB)
  - A real-time graph of memory usage
  - Display axes with memory limits

- **CPU Utilization:**

  - Current CPU utilization percentage
  - A graph plotting CPU usage over time
  - Display corresponding axes

- **Cores Information:**

  - Number of CPU cores
  - Maximum core frequency
  - A diagram representing detected cores

## How to Run the Program

After compiling the program, here are some of the ways you can run the code:

Valid examples:

```sh
./B09A1 --memory
./B09A1 --cpu
./B09A1 --cores
./B09A1 30 100000 --cpu --memory
./B09A1 --samples=50 --tdelay=200000
./B09A1 --cores --samples=15
```

## Assumptions made about the command line arguments:

The values for samples and tdelay are positive integers.

## Function Descriptions

### `int is_numeric(const char *str)`

Determines whether a given string contains only numeric characters.

### `void parse_args(int argc, char **argv, tConfig *config)`

Parses command line arguments to configure runtime settings.

### `void collect_memory(double *memTotal_gb, double *memUsed_gb)`

Collects memory usage statistics using the system’s sysinfo structure.

### `void get_cpu_val(Cpu_times *cpu)`

Reads CPU time information from `/proc/stat`.

### `double get_cpu_usage(Cpu_times cput1, Cpu_times cput2)`

Calculates CPU usage percentage over an interval.

### `int collect_cores()`

Determines the number of CPU cores available by parsing `/proc/cpuinfo`.

### `double get_max_possible_freq()`

Retrieves the maximum possible CPU frequency.

### `void display_memory_graph(tConfig config, double memTotal_gb, double memory_samples[], int curr_sample)`

Displays a graphical representation of memory usage over time.

### `void display_cpu_graph(tConfig config, double cpu_samples[], int curr_sample)`

Displays a graphical representation of CPU usage over time.

### `void display_cores_diagram(int num_cores, double max_freq_mhz)`

Prints a diagram representing the CPU cores.

### `int main(int argc, char **argv)`

The main entry point of the program. It sets up the configuration, collects data samples, and displays dynamic graphs for memory, CPU usage, and optionally CPU cores.

**Workflow:**

- **Initialization:** Parses arguments and allocates memory.
- **Sampling Loop:** Collects data, updates graphs, and waits between samples.
- **Post-Processing:** Displays CPU core information.
- **Cleanup:** Frees allocated memory before exiting.

## Problem-Solving

### Command line parsing:

This assignment has a lot of moving parts to it, the two biggest being: command line parsing and dynamic graph printing.



Starting off with the command line parsing. Since we were given that only the first two arguments could be positional, i started by going through the array of arguments first to check whether the first two would be numbers, using a helper function is\_numeric to confirm it. If the first argument (index 1) of argv was a number, we know it should be given to the sample variable as per the given instructions. Since we know that the two positional arguments will always be given at the front of the list (as per piazza, this guarantees that the numeric argument in the 3rd index will always be given to tdelay. Also, knowing that assumption, any numeric argument past the 3rd index should be a formatting error, causing the function to stop early with an error message. The first while loop will increment the index by the number of positional arguments there are leaving us to deal with only the flagged arguments. The unique challenge from this came from the nature of the flagged non-numeric arguments –core, –memory, and–cpu. The fact that not mentioning all them in the command makes them default to print, but mentioning even one of them makes it so that only the mentioned non-numeric flagged arguments are printed lead me to make a flag system for  –core, –memory, and–cpu. As consistent with the assignment nature, they have a default value of 1, which is confirmation that they should be printed, but as soon as a non-numeric flagged argument is detected after the first run through of all of the arguments, –core, –memory, and–cpu are set to zero and we loop around the argument array again and only give them 1 if they are explicitly mentioned as an argument. When it comes to the flagged numeric arguments –tdelay and samples, we use the first index counter arg\_index to go through the rest of the arguments and if they are present, Iuse atoi to assign their respective values. A few assumptions made is that the values for the flagged and non-flagged numeric arguments are positive integers (as per piazza). I also made the logical assumption that if someone already used the non-flagged version of a flagged numeric argument to give it some value, then used its flagged version to give it another value, it means they want the second value as their argument value for example:

./B09A1 20 –samples=40 would give samples a value of 40. If the argument fit none of the flags, it means there was a formatting error and an error message should be sent and the program should stop early. This was my thought process behind parse\_args(), it would be a function that would set the values for the arguments in the command line. I used a struct to contain these variables for readability and modularity purposes.

The function parseargs() takes the number of command line arguments, the argument array, along with a tconfig struct, which holds the configurations of our system tool. The function edits the tconfig struct, giving each of its fields the configuration values according to the command line arguments

&#x20;

### Collecting Memory

Once i gained and stored the info needed from the command line, I needed to look for the information of the system that i was required to print, that being the systems total memory, the used memory at different points in time (depending on the tdelay), utility of the cpu, max core frequency, and the number of cores.



collect \_memory(), the purpose of this function is to get the total and used memory when the function is being run. Using one of the headers that was provided to us in the assignment. The sys/sysinfo.h header provides the sysinfo struct, which, if you populate it with systeminfo, provides us with total and free memory in bytes, which we convert to gb since that the unit the assignment tells us to print

### Calculating CPU Utilization

When it comes to calculating CPU utilization for a moment in time, I learned that it essentially is the change in time spent not-idle/the change in total time between two almost-instantaneous snapshots of a CPU. So, essentially the time is the amount of time spent working in that instantaneous compared to the amount of time passed in the instantaneous interval. I used two functions to get the job done: get\_cpu\_val() to get the total amount of time the CPU spent idle and at work since boot, and get\_cpu\_usage(), which takes the amount of usage of the CPU in the interval between when two different CPU snapshots were taken. Using usleep(), which pauses the program for a small amount of time, I can then get the cpu\_usage of two CPU snapshots very close to each other for the instantaneous cpu utilization at a time.

### Calculating Cores and Max Frequency

When it comes to calculating cores, I used collect\_cores() to find them. The function opens /proc/cpuinfo and is able to check the amount of processors that are present by simply matching each line that starts with “processor” and gathering a tally, it returns an integer and has no parameters.



Similar is finding the max freq of cores, the function get\_max\_possible\_freq()uses system file "/sys/devices/system/cpu/cpu0/cpufreq/cpuinfo\_max\_freq"

And extracts the max freq and converts it to MHz and returns it as a double. It has no parameters.

### Printing Dynamic Graphs



The most challenging part of this assignment by far was trying to print the two graphs. In the demo video, the graphs appeared to be dynamically updating however, many times, there were samples. If both graphs were called, they updated simultaneously, which gave me the idea of splitting up the actual graph printing and the dynamic appearance into two parts. display\_cpu\_graph is a function that would print one “iteration” of the graph. An iteration creates a dynamic effect when the graph is printed and cleared, and then it gets its next iteration printed on the same spot by resetting the cursor until the final sample iteration graph is printed. What initially stumped me is that each iteration had the previous iteration’s information and just added another hashtag. This fact gave me the idea of having a “memory”, which would store the information of the previous samples that were being graphed. Due to the 2d nature of this print statement, i thought of storing the places for symbols in a 2d matrix, where the matrix would represent the graph, the empty parts of the graph being represented by ‘  ‘ being stored in the matrix. Choosing the vertical placing for the symbols were simple, using y axis value divided by a scale and rounded up to determine which fraction of the y axis the symbol would stay. Which column on the matrix the symbol would be in would be dependent on which sample it was. Using this thought process, I made display\_cpu\_graph and display\_memory graph. The actual “dynamic” part of it was done in the main function, where i looped over the number of chosen samples from the command line and then printed one iteration each of whichever graph had their flag value as 1 (meaning it was to be printed). After each loop ended, usleep was invoked for tdelay amount of time in accordance to assignment details. After that the loop starts again with clearing the terminal and resetting the cursor, making the print of the next sample iteration of graphs appear as though it was dynamic.

### Printing Core Diagram

My main concern with this part was the possibility that the diagram is not rectangular due to some remainder, i left the width as 4 boxes and was able to calculate how many boxes would be in the last row, as well as how many rows were there going to be with some simple math. I focused on making the diagram robust to any amount of cores. In the animation, if called, the cores diagram always appeared after the graph’s animations.



