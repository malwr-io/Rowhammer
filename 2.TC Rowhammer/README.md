# TC Rowhammer Tool

Tool for testing the rowhammer bug, without need of special permissions (root or CAP_SYS_ADMIN) and Transparent Huge Page (THP) support.
<br><br>
It's based on a [known timing channel](https://people.inf.ethz.ch/omutlu/pub/mph_usenix_security07.pdf) created when different rows with in the same bank are activated .
It is a more targetted approach to rowhammer when no physical mapping information are available. It achieves that by creating a set with Same Bank Different Rows (SBDR) addresses based on the described timing channel. 
<br>

## Command line arguments

<pre>
  -s SIZE               allocate SIZE MB buffer for testing (default is 128 MB)
  -o OUTPUT_FILE,       path to the output file, default is stdout
  -m THRESHOLD_MULT     the THRESHOLD_MULT creates the threshold for SBDR and non-SBDR pairs (default is 1.3)
  -i ITERATIONS         gather timings for SBDR pairs after ITERATIONS iterations (default is 5000)
  -b TEST_ITERATIONS    initially hammer the rows for TEST_ITERATIONS iterations (default is 550000)
  -B STRESS_ITERATIONS  after a bit flip is found, hammer again the targets for STRESS_ITERATIONS (default is 1700000)
  -q SAMPLE_SIZE        number of measurement to take from a given address pair (default is 8)
  -e SECS               stop the test after SECS seconds (by default stops when all the rows are tested)
</pre>

## Usage

First build the tool by running one of the following options:

- This will generate an executable that will exhaustively test all the identified rows within a bank (tcrh) and are within the the defined range:
<pre>
    make
</pre>

- Similar with first one, but with the added support of identifying pages within the same row (tcrh_ext):
<pre>
    make ext
</pre>
	
- Build both versions with increased verbosity, the executables will require root privileges (for pagemap access):
<pre>
    make debug
</pre>
Then run it by:

	./tcrh<_ext> [-s <buffer_size>] [-o <output_file>] [-m <threshold_mult>] [-i <trial_iterations>] [-b <test_iterations>] [-B <stress_iterations>] [-q <sample_size>] [-e <run_time>]
    
## Remarks
This tool was tested in both native and virtualized environment and in both cases over 200 bit flips were generated within 20 minutes. Both the host and guest (VMware Workstation) were running Ubuntu 16.04.2 with the latest kernel version in a 4-core Sandy Bridge CPU with a single DIMM all running with the default configurations and THP disabled.

## Notes
The debug information (when enabled) about the DRAM mapping are targetted towards Sandy Bridge microarchitecture CPUs. If there is need for stats on a different microarchitecture, the code can be easily extended by using the implementation here: [hammertime](https://github.com/vusec/hammertime/blob/master/ramses/addr.c) <br>
Regardless of the debugging information, the main functionality of the program should more or less remain unaffected regardless of the microarchitecture.<br><br>
It makes use of RDTSCP for timing and the CLFLUSH for the cache eviction. In case the RDTSCP is not available, it can be replaced with RDTSC for x86 architectures that support it.
<br>