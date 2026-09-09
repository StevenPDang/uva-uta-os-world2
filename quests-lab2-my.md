1. Introduction
This project aims to implement new features in the OS to allow multiple tasks to run concurrently and preempt their execution through (1) scheduler: cooperative and preemptive, (2) memory allocator, and (3) task management (sleep, wait, exit, kill). However, everything still runs at EL1. 

Note that the project is adapted from UVA Operating System CourseLinks to an external site. and other existing materials. It has been modified for instructional purposes and to align with the learning objectives of this course.

2. Assignments (Q1-Q10)
Q1: Boot
Download and extract the zip file [uva-os-world2.zipDownload uva-os-world2.zip].

Complete boot.S: set up the kernel stack, so that the kernel execution can reach kernel_main(). To help coding, use GDB to debug the boot process.

Once successful, you should see the familiar message:

------ kernel boot ----- core 0
build time (kernel.c) ...
Q2: Two cooperative printers
OVERVIEW: We will enable cooperative multitasking. To do that, we complete task creation and context switch. Note that the context switch in this quest is for two tasks to yield to each other voluntarily via function calls. This means their execution is not interrupted at arbitrary instructions. This is fundamentally different from preemptive multitasking, which we will explore in the next quest. Make sure you understand these concepts.

In unittest.c, understand test_kern_tasks_print() and the functions called by it.
in the body of the init task (kernel.c), place a call to the former function to launch two tasks.
Task creation. Understand and then complete copy_process() in sched.c.
Context switch. Understand switch_to() in sched.c and its relation to cpu_switch_to() in switch.S. Complete switch_to().
Complete assembly: cpu_switch_to() in switch.S (with the help of AI). Place a call it in switch_to() in sched.c.
Scheduler. Grasp schedule() in sched.c. Complete schedule() by placing calls to switch_to().
Note that schedule() is invoked for both cooperating (via yield() which directly calls schedule()) and preemptive scheduling (next quest, via timer interrupt which calls schedule()).
The birth of a new task. Complete ret_from_fork() in entry.S (assembly).
DELIVERABLE. You will see two tasks printing their own messages. Take a photo.

Q3: Two preemptive printers
OVERVIEW: we will bring up preemptive multitasking. To do that, we will enable periodic timer interrupts. Then we will invoke the scheduler in the timer interrupt handler, which preempts the current task.

Understand generic_timer_init(). Understand its difference from sys_timer. Place a call to it in kernel_main() to enable timer interrupts.
Place el1_irq in the irq vector table (entry.S).
Turn on the CPU irq in kernel_main().
Understand the role of timer_tick() in sched.c. In handle_generic_timer_irq(), call timer_tick().
CHECKPOINT. If everything goes well, at this time you should see handle_irq() called periodically, which calls handle_generic_timer_irq(), which calls timer_tick(), which calls schedule(). Add debug messages in these places to verify this. Turn off the messages after verification.
At this point, kernel_entry/exit are not complete. If you let the kernel to continue executing, the kernel may crash after returning from an exception (eret).
Understand kernel_entry and kernel_exit in entry.S. Understand their differences from the same macros that we saw in WORLD1. Complete them.
Determine the value of S_FRAME_SIZE in entry.h.
DELIVERABLE. You will see two tasks printing their own messages, and the messages are interleaved. Take a screenshot.

Q4: Two donuts
OVERVIEW: to visualize our tasks, we will launch two tasks, each rendering a rotating donut on the screen.

Make sure you undrestand donut_pixel() in donut.c, especially how it renders different donuts to differnet screen locations via the "idx" argument.
Complete test_kern_tasks_donut() and complete the functions called by it: spawn multpile tasks, each rendering a donut by calling donut_pixel().
On qemu: if the two donuts seem to spin in sync and not take turns, it may be due to the emulated cpu is a bit slow as compared to the timer interval. Try to tune the value of "interval" in timer.c which allows a task to run longer before it gets descheduled.
DELIVERABLE. You will see two donuts take turns to rotate on the screen. Record one video showing the visual effect (5--10sec).

Q5: N donuts
Note: each donut shall still be rendered by a separate task.

Change the number of concurrent donuts to 4, which shall be rendered to the screen in a 2x2 grid. donut_pixel() already supports this.
Support more concurrent donuts, i.e., 9, which shall be rendered to the screen in a 3x3 grid. You need to hack donut.c to support this.
Get as many donuts to run as you can.
DELIVERABLE. Record a video of maximum number of donuts that you can run (5--10sec).

Q6: Fast/slow donuts
Check task_struct::credits and task_struct::priority in sched.h. Understand how credits and priority affect the scheduling, by checking all code that references to these two fields, e.g.
sched_init(), schedule(), and timer_tick() in sched.c
Add code to test_kern_tasks_donut() and/or kern_task_donut() to set different priorities for different donuts, to implement the following visual effects:
Some donuts make more turns than others.
DELIVERABLE. Record five videos, one for each visual effect with a unique set of priorities  (5--10sec). Record the priorities in a single text file for the videos.

Q7: Donuts in sync
The idea: for multiple donuts to render in sync, donut_pixel may call 'yield()' after it renders each frame, so other donuts can run (for a frame).

Visual effects:

Add call to yield() in donut_pixel() to implement frame yield.
Change the code, so that the donuts turn simluatenously, but at different speeds.
Q8: Kill a donut
Complete exit_process() in sched.c.
Add code to donut_pixel(), so that one donut task exits, either after running for roughly 3 seconds, or in response to a key press (via UART irq), cf the "UART rx irq" task in Project Assignment 1.
Other donuts shall continue to run.
DELIVERABLE. Record one video showing the visual effect (5--10sec).

Q9: Wordsmith
Understand test_kern_reader_writer() and the two tasks it creates. Call it from kernel_main(). Note: don't forget to comment out prior testing calls there.
Understand do_write() and do_read(). Complete the functions with calls to wakeup() and sleep() to synchronize the two tasks.
Understand the idea of sleep() and wakeup() in sched.c. Complete their code, including wakeup_nolock() which is called by wakeup(). The code will be revisited after we make a multicore kernel.
You will see the reader task printing poem it received.
Q10: Summary
DELIVERABLE. Summarize your main takeaways from this project in bullet points in a single word/text file. You can list as many bullet points as you want. 