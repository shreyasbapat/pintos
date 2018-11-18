## Pintos Assignment - B16145, B16024, B16065

#### Changed Files:

`~pintos/src/threads/thread.c`

Changed methods:

`thread_priority_temporarily_up(void)` :

Here, we create a pointer to the current thread and make the priority of the thread as 63, which in fact is the highest priority.

`thread_priority_restore(void)`

We revert back the priority by making the priority of the thread as stored priority. Stored priority is a new element in the thread structure.

`thread_block_till(int64_t wakeup)`

We first disable the interrupts, assert if the thread is running and then make the sleep_tics of the thread equal to wakeup value.
We insert the thread in the `sleeper` list, then we block the thread and resume the interrupts.



