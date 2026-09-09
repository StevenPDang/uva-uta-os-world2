# Quests for World 2 "Embedded"

<!-- convert Slide4.PNG -resize 600x Slide4-resized.png -->

![alt text](https://raw.githubusercontent.com/fxlin/uva-os-world2/student/docs/Slide4-resized.png)

This project implements new features in the OS to allow multiple tasks to run
concurrently and to preempt their execution, through: (1) a scheduler,
cooperative and preemptive; (2) a memory allocator; and (3) task management
(sleep, wait, exit, kill). Everything still runs at EL1.

The project is adapted from UVA Operating System course materials and other
existing materials. It has been modified for instructional purposes and to align
with the learning objectives of this course.

**NOTE**. In all writeup below, we will refer to C function names and assembly
labels. We will not always give out the file names. To quickly locate them, use
vscode `ctrl+t` for C functions and `ctrl+shift+f` for assembly labels.

**NOTE**. In all coding quests, see the comments in the code for hints
and instructions.

## Quest01: boot

Complete boot.S: set up the kernel stack, so that the kernel execution can reach
`kernel_main()`. To help coding, use GDB to debug the boot process. 

Once successful, you should see the familiar message: 

````
------ kernel boot ----- core 0
build time (kernel.c) ...
````
😀 DELIVERABLE: none. 

## Quest02: two cooperative printers

OVERVIEW: we will enable cooperative multitasking. To do that, we complete
task creation and context switch. 
Note that the context switch in this quest is for two tasks to yield to each other voluntarily via function calls. This means their execution is not interrupted at arbitrary instructions. This is fundamentally different from preemptive multitasking, which we will explore in the next quest.
Make sure you understand these concepts. 

- In unittest.c, understand `test_kern_tasks_print()` and the functions called by it.  
in the body of the init task (`kernel.c`), place a call to the former function to
  launch two tasks.

- Task creation. Understand and then complete `copy_process()` in sched.c. 

- Context switch. Understand `switch_to()` in sched.c and its relation to `cpu_switch_to()` in switch.S.
Complete  `switch_to()`. 

- Complete assembly: `cpu_switch_to()` in switch.S (with the help of AI). Place a call it in `switch_to()` in sched.c.

- Scheduler. Grasp schedule() in sched.c. Complete `schedule()` by placing calls to `switch_to()`. 

> Note that schedule() is invoked for both cooperating (via yield() which directly calls schedule()) 
and preemptive scheduling (next quest, via timer interrupt which calls schedule()).

- The birth of a new task. Complete `ret_from_fork()` in entry.S (assembly).

😀DELIVERABLE: you will see two tasks printing their own messages. Take a
photo. 

## Quest03: two preemptive printers

OVERVIEW: we will bring up preemptive multitasking. To do that, we will enable
periodic timer interrupts. Then we will invoke the scheduler in the timer
interrupt handler, which preempts the current task.

- Understand `generic_timer_init()`. Understand its difference from sys_timer.
Place a call to it in `kernel_main()` to enable timer interrupts.

- Place `el1_irq` in the irq vector table (`entry.S`)

- Turn on the CPU irq in `kernel_main()`.

- Understand the role of `timer_tick()` in `sched.c`. In
`handle_generic_timer_irq()`, call `timer_tick()`. 

- CHECKPOINT. If everything goes well, at this time you should see `handle_irq()` called
periodically, which calls `handle_generic_timer_irq()`, which calls
 `timer_tick()`, which calls `schedule()`. Add debug messages in these places to
 verify this. Turn off the messages after verification.

 > At this point, `kernel_entry/exit` are not complete. 
 If you let the kernel to continue executing, the kernel may crash
 after returning from an exception (eret). 

- Understand `kernel_entry` and `kernel_exit` in `entry.S`. Understand their
  differences from the same macros that we saw in WORLD1. Complete them. 

- Determine the value of `S_FRAME_SIZE` in `entry.h`.

<!-- - ~~Understand `schedule()`. Complete `schedule()` by placing calls to
  `switch_to()`.~~  -->

😀DELIVERABLE: you will see two tasks printing their own messages, and the
messages are interleaved. Take a screenshot.

## Quest04: two donuts

OVERVIEW: to visualize our tasks, we will launch two tasks, each rendering a
rotating donut on the screen. 

- make sure you undrestand `donut_pixel()` in donut.c, especially how it renders
  different donuts to differnet screen locations via the "idx" argument. 

- complete `test_kern_tasks_donut()` and complete the functions called by it: 
spawn multpile tasks, each rendering a donut by
  calling `donut_pixel()`.

- on qemu: if the two donuts seem to spin in sync and not take turns, it may be due to 
the emulated cpu is a bit slow as compared to the timer interval. 
Try to tune the value of "interval" in `timer.c` which allows a task to run longer before it gets descheduled. 

😀DELIVERABLE: you will see two donuts __take turns__ to rotate on the screen. Record one video showing the visual effect (5--10sec).

![Example](https://raw.githubusercontent.com/fxlin/uva-os-world2/student/docs/2donuts.gif)

## Quest05 (side): N donuts

Note: each donut shall still be rendered by a separate task. 

- change the number of concurrent donuts to 4, which shall be rendered to the
  screen in a 2x2 grid. `donut_pixel()` already supports this. 

![4donuts.gif](https://raw.githubusercontent.com/fxlin/uva-os-world2/student/docs/4donuts.gif)

- support more concurrent donuts, i.e., 9, which shall be rendered to the
  screen in a 3x3 grid. You need to hack donut.c to support this.

- Get as many donuts to run as you can.

😀DELIVERABLE: Record a video of the maximum number of donuts you can run (5--10sec).

## Quest06: fast/slow donuts

- Check `task_struct::credits` and `task_struct::priority` in sched.h.
Understand how credits and priority affect the scheduling, by checking all code
that references to these two fields, e.g. 
   - `sched_init()`, `schedule()`, and `timer_tick()` in sched.c 

- Add code to `test_kern_tasks_donut()` and/or `kern_task_donut()` to set
  different priorities for different donuts, to implement the following visual
  effect: 
  - some donuts make more turns than others.

![2donuts-diff-priorities.gif](https://raw.githubusercontent.com/fxlin/uva-os-world2/student/docs/2donuts-diff-priorities.gif)
    
😀DELIVERABLE: Record five videos, each with a unique set of priorities
(5--10sec) using your phone. Record the priorities used for each video in a
single text file.

## Quest07 (side): donuts in sync

The idea: for multiple donuts to render in sync, `donut_pixel` may call 'yield()' 
after it renders each frame, so other donuts can run (for a frame).

<!-- - ~~Complete the code of `yield()` in `sched.c`.~~ -->

Visual effects: 

1. Add call to `yield()` in `donut_pixel()` to implement frame yield. 

![2donuts-sync.gif](https://raw.githubusercontent.com/fxlin/uva-os-world2/student/docs/2donuts-sync.gif)

2. Change the code, so that the donuts turn simluatenously, but at different speeds.

😀DELIVERABLE: Record videos, one for each visual effect (5--10sec) using your phone.

## Quest08 (side): kill a donut

- Complete `exit_process()` in sched.c.

- Add code to `donut_pixel()`, so that one donut task exits, either after running 
for rougly 3 seconds, or in response to a key press (via UART irq), cf the "UART rx irq" task in Project Assignment 1.  

- Other donuts shall continue to run.

😀DELIVERABLE: Record one video showing the visual effect (5--10sec) using your
phone.

## Quest09: wordsmith

> This quest may take more time than others.

- Understand `test_kern_reader_writer()` and the two tasks it creates. 
Call it from `kernel_main()`. Note: don't forget to comment out prior testing calls there. 

- Understand `do_write()` and `do_read()`. Complete the functions with calls to
  `wakeup()` and `sleep()` to synchronize the two tasks.

- Understand the idea of `sleep()` and `wakeup()` in sched.c. 
Complete their code, including `wakeup_nolock()` which is called by `wakeup()`.
The code will be revisited after we make a multicore kernel. 

- you will see the reader task printing poem it received. 

😀DELIVERABLE: 
Record a video (5--10sec) using your phone.

Reference: 

![alt text](https://raw.githubusercontent.com/fxlin/uva-os-world2/student/docs/wordsworth.gif)

## Quest10: summary

Summarize your main takeaways from this project in bullet points, in a single
word/text file. You can list as many bullet points as you want.

😀DELIVERABLE: the summary file.
