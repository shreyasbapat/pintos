# pintos

Assignment for :

**CS310 - Introduction to Communicating Distributed Processes**

### Steps to install:

```
Instructions to install PintOS
1)Install qemu:
sudo apt-get install qemu

2) Install other utils:
sudo apt-get install binutils pkg-config zlib1g-dev libglib2.0-dev libc6-dev autoconf libtool libsdl1.2-dev libx11-dev  libxrandr-dev libxi-dev

3) Download pintos*.tar.gz from moodle and extract it in your home directory. 
finally you have ~/pintos folder that contains the src.

4) Add utils in the pintos source code to your path.
a) open .bashrc
$ gedit ~/.bashrc

b) append the following line to .bashrc, save and close the file
export PATH=~/pintos/src/utils:$PATH 

c) source the .bashrc
$ source ~/.bashrc

5) Edit ~/pintos/src/utils/pintos-gdb to change the definition of GDBMACROS to
point to where you installed gdb-macros. That is, open the pintos-gdb in gedit and change the path to GDBMACROS as shown below:
a) gedit ~/pintos/src/utils/pintos-gdb

b) GDBMACROS=~/pintos/src/misc/gdb-macros

c) save and close the file. Test pintos-gdb as follows:
$ pintos-gdb

6) Compile the Utilities

$ cd ~/pintos/src/utils
$ make

7) Edit Make.vars in threads directory to specify the emulator as qemu
Open the file “~/pintos/src/threads/Make.vars” and change the last line to:

SIMULATOR = --qemu

8) Compile Pintos Kernel 
$ cd ~/os-pg/pintos/src/threads/
$ make

9). Edit utility files to make qemu as the default emulator
Changes in “~/pintos/src/utils/pintos” :
    a. Change line no. 103 to “$sim = “qemu” if !defined $sim;” to enforce qemu as      simulator

10) Specify the proper kernel path 
Changes in “~/pintos/src/utils/pintos” :
a). On line no. 257, replace “kernel.bin” to path where we have built the kernel “/home/sriramk/pintos/src/threads/build/kernel.bin”.
 
b) Edit ~/src/utils/Pintos.pm by replacing loader.bin in line number 362
with the absolute path pointing to it: /home/sriramk/pintos/src/threads/build/loader.bin

11) Run the kernel
pintos run alarm-multiple

Should run the kernel in qemu and show the output of 5 thread creation.
```

### Tasks to be accomplished:

#### First Assignment: 

```
Pintos Assignment: Busy wait, Thread states, Timer Alarms
Acknowledgement: derived from the original assignment posted by Ben Pfaff in PintOS document, Stanford University and Professors Vishv Malhotra, Gautam Barua, and Rashmi Dutta Baruah from IIT Guwahati.

Assume a single processor computer; only one thread can be executing at any given time. The thread is said to be in state THREAD_RUNNING. The other threads must wait for their turns. A threads waiting for its turn to use the processor is a ready thread. And, its state is THREAD_READY. The threads that are not seeking use the processor at a point in time are blocked threads in state THREAD_BLOCKED. These threads may be waiting for an event. In the exercise for this week, the events of interest to the waiting/blocked threads are the timer events. These are also called timer alarms or just alarms.

We plan to implement section 2.2.2 Alarm Clock mentioned in the PintOS document and few additional things as listed in this document.
"Reimplement timer_sleep(), defined in ‘devices/timer.c’. Although a working implementation is provided, it “busy waits,” that is, it spins in a loop checking the current time and calling thread_yield() until enough time has gone by. Reimplement it to avoid busy waiting."

The current implementation wastes the time by calling thread_yield() – the thread lets some other unknown thread use the processor time. This is bad because the scheduler must keep asking every thread frequently and/or periodically if the thread wants to use the processor. If the requested thread is not interested, then it will pass the option to yet another thread. The chain of threads unwilling to use the available processor time slot may be arbitrarily long. The blind thread_yield() does not direct the request to make use of the free processor time to the threads willing to use the time. 

A better method is to make the threads waiting for alarms blocked till the required number of ticks have been counted. The better algorithm is outlined as the changed code of function timer_sleep() in file devices/timer.c:

/* Sleeps for approximately TICKS timer ticks.
Interrupts must be turned on. */
void timer_sleep (int64_t ticks)
{
	int64_t start = timer_ticks ();
	int64_t wakeup_at = start + ticks;
	ASSERT (intr_get_level () == INTR_ON);
	/* Put the thread to sleep in timer sleep queue */

	thread_priority_temporarily_up ();
	thread_block_till (wakeup_at, before);
	
	/* original code -- to be decommissioned */
	while (timer_elapsed (start) < ticks)
		thread_yield (); */
	/* Thread must quit sleep and also free its successor
	if that thead needs to wakeup at the same time. */

	thread_set_next_wakeup ();
	thread_priority_restore ();
}

A thread planning to sleep must arrange to be woken up at a suitable time. Once the arrangement is made, the thread can block itself. The OS scheduler will not consider the blocked threads for allocation of time slot for execution on the processor.

The OS, however, must keep track of the time at which a blocked thread needs unblocking. Obviously, the thread of interest for this purpose is the one that has the earliest wakeup time among those who are blocked/sleeping for their timer alarms. The OS only needs to watch one (wakeup) time: the time for the next wakeup.

The time for the next alarm that the OS tracks can change only under two conditions:

* A new thread joins the set of sleeping threads and has wakeup time before the current earliest/next wakeup time. This change can be made by the thread planning to sleep before it actually blocks. Function thread_block_till (wakeup_at, before) implements this requirement. Or

* A sleeping thread is woken at the end of its sleep time. The thread must look
at all the remaining threads that are sleeping to find the nearest time for the
next wakeup. Set the system to perform next wakeup at this new time. This is
among the jobs for function thread_set_next_wakeup ()

We can let function thread_tick() do one unblock of one sleeping thread at each wakeup time. A simple way to impose this restriction is by changing next-wake-time to the end of time horizon while unblocking the waiting thread. This unblocked thread will correctly set next-wakeup-time before resuming its normal task.

The role of the other functions are described below:
a) thread_priority_temporarily_up() and
b) thread_priority_restore()

The function a) is called when a thread is about to block, so its temporary higher priority is not too restrictive to the other threads. Note there are several sleeping threads. So we need a list of sleeping (blocked) threads. This list is a shared list accessible by all threads. A shared resource such as this list is only usable serially on one-thread-at-a-time basis (mutual exclusion). To keep the wall-clock time for the use-duration short we may wish to run the threads using the list at the highest priority levels. The thread that was blocked was good to the other threads as other threads could use the freed processor time. When a thread wakes, the benefitted threads may be nice to it in-turn. We delay the restoration of the priority levels to the last step in the proposed algorithm.

Task 1:
Please work on these two priority-changing functions first to gain familiarity with the kernel code. See how a ready_list is used to record all ready threads in the system. It will take about a dozen lines of new code and changes to implement functions thread_priority_temporarily_up() and thread_priority_restore(). You will require a new member in struct thread to save priority.

When you run make check to test unmodified PintOS implementation under directory pintos/src/thread you have only 19 failed tests. On completion of the priority changing functions (we are still using the original while-loop with thread_yield() to wait for the specified number of ticks in function timer_sleep()), your success count for command make check should remain unchanged.

Also, note that on completion of this stage, ready_list is a sorted list sorted by thread priority. And, you would have written a function of type list_less_func to compare threads in ready_list for their correct position in the list.


Task 2:
Now we shall introduce another sorted list to hold threads that are blocked on their timer alarms. For convenience, we refer to it as list sleepers. Remember that this list should only be accessed by one thread at a time. Only after a thread has satisfied its needs is list sleepers allowed to be accessed by another thread.

Let us now describe an algorithm for other two functions: thread_block_till(wakeup_at, before) and thread_set_next_wakeup(). Function parameter before is a function to compare the threads in list sleepers and its prototype is bool before (const struct list_elem *a, const struct list_elem *b, void *aux UNUSED) also aliased as list_less_func.

As list sleepers may be a long list, some list functions described in kernel/list.h may take multiple ticks to complete. This may cause other threads to be unblocked and they too may access list sleepers before a previous thread has finished its use of the list. This is not safe and we do not want to let it happen.

We must control access to the list by a proper synchronization tool from synch.h.
The tool will deny access to the list by other threads while one thread is changing the list. To keep these “busy” durations short, we have already arranged for temporarily increase of the scheduling priority of the relevant threads around these two functions.

The thread must release the synch object before it enters the block state. If access to list sleepers is not free when the “controlling” thread is blocked, other threads will remain unable to access list sleepers. Even a tiny time (just a few instructions) between the release of a synchronization control and the actual block of the thread is a potential trouble spot. You must manage it. 

Task 3:
When a sleeper is woken (unblocked), the situation is trickier. On any given single timer tick, there may be several sleeping threads seeking to wake up. How do you attend to all of them within a single tick by calling thread_unblock() for each of them?

We suggest, you write code for this in function thread_set_next_wakeup () which was primarily designed to remove thread thread_current() from list sleepers when the unblocked thread is scheduled after it is woken/unblocked from its sleep by function thread_ticks().

The function must also make sure that if there are other sleeping threads that have the same wakeup time as the thread just woken up, then they are not left blocked. If a sleeping thread with the matching wakeup time is left waiting, then the waiting thread can only wakeup on a later timer tick. On the other hand, we cannot spend too much time waking all such threads because we do not want to hold the timer-tick in disabled state for too long. A missed tick would drift the system clock. We solved the problem by limiting to one thread unblock in function thread_ticks().

The task of unblocking other simultaneous wakeups is easily shared among thevmultiple threads. Each unblocked thread is required to unblock one thread that has the same wakeup time as itself. This action will recursively unblock all threads which have the matching wakeup times. All except the first call to thread_unblock() will be from function thread_set_next_wakeup () . An unblocked thread that does not unblock another thread sets the next-wake-up time. Suffice to remind that this time was set far into the future to prevent multiple unblocks from function thread_tick().

DONOT unblock a sleeping thread if there is another blocked thread with an earlier wakeup time.


```
