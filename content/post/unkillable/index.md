---
title: "Demystifying the *Unkillable* Process in Linux —with a Little Help from Python"
description: >
  Why does `kill ‑9` occasionally fail?  
  This post walks through three kernel pathways that neutralise `SIGKILL`, shows
  how to flip the `SIGNAL_UNKILLABLE` flag from a tiny kernel module plus a
  Python driver, and demonstrates how to investigate stubborn `D`‑state or
  namespace‑shielded PIDs with `psutil`.
date: 2025-04-25
weight: 1
categories: ["sigkill", "linux","bash"]
tags: ["sigkill", "signal", "sigstop", "c"]
draft: false
---

# Demystifying the *Unkillable* Process in Linux —with a Little Help from Python
> **TL;DR**  
> `SIGKILL` and `SIGSTOP` are meant to be final, but three kernel pathways let a task survive:  
> 1. The process’s `SIGNAL_UNKILLABLE` flag is set (PID 1 has this by default).  
> 2. The thread is stuck in an uninterruptible sleep (`D` state) while inside the kernel.  
> 3. A tracer, cgroup freezer, or rogue kernel module silently drops or delays the signal.  
>   
> Below we’ll **reproduce the first case** with a 25‑line kernel module driven by Python, then learn to spot the other two cases with pure Python.

---

## 1  Why `SIGKILL` *ought* to win

* In the Linux `signal(7)` manual, **`SIGKILL (9)`** and **`SIGSTOP (19)`** are marked “cannot be caught, blocked, or ignored.”  
* Delivery can fail only when  
  * the caller lacks permission (`kill()` returns `EPERM`), **or**  
  * the target’s `signal_struct→flags` carries the bit **`SIGNAL_UNKILLABLE`**.

The kernel grants that flag to the very first userspace task (PID 1) so the system can’t kill its own *init*.

---

## 2  Flipping `SIGNAL_UNKILLABLE`: build an “immortal” process

> ⚠️ **Run in a throw‑away VM or container.**  
> Once a PID is flagged unkillable, only a reboot or voluntary exit clears it.

### 2.1  The 25‑line kernel module `unkillable.c`

```c
// compile with: make && sudo insmod unkillable.ko
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/pid.h>
#include <linux/sched/signal.h>

#define DEV "unkillable"

static ssize_t flip(struct file *f, char __user *u, size_t pid, loff_t *o)
{
    struct pid *p = find_get_pid(pid);
    if (p) {
        struct task_struct *t = pid_task(p, PIDTYPE_PID);
        if (t && t->signal)
            t->signal->flags |= SIGNAL_UNKILLABLE;   /* 🔑 magic */
        put_pid(p);
    }
    return 0;                    /* “read” zero bytes — side‑effect only */
}

static const struct file_operations fops = { .read = flip };

static int  __init init(void) { return register_chrdev(117, DEV, &fops); }
static void __exit exit(void) { unregister_chrdev(117, DEV); }
module_init(init); module_exit(exit);
MODULE_LICENSE("GPL");
```

**Makefile**

```makefile
obj-m += unkillable.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Build & load:

```bash
$ make
$ sudo insmod unkillable.ko
$ sudo mknod /dev/unkillable c 117 0
$ sudo chmod 666 /dev/unkillable
```

### 2.2  Python driver `immortal.py`

```python
#!/usr/bin/env python3
import os, time, ctypes

pid = os.getpid()
print(f"My PID is {pid}")

fd = os.open("/dev/unkillable", os.O_RDONLY)
# read()’s *count* parameter is treated as the target PID by the driver
ctypes.CDLL(None).read(fd, ctypes.c_char_p(0), pid)
print(f"SIGNAL_UNKILLABLE flag set — hit me with `sudo kill -9 {pid}`")

while True:
    time.sleep(1)
```

Run it:

```bash
$ python3 immortal.py
My PID is 44201
SIGNAL_UNKILLABLE flag set — hit me with `sudo kill -9 44201`
```

### 2.3  Shell test: `kill -9` that *fails*

```bash
# second terminal
$ sudo kill -9 44201        # exit‑status 0, signal accepted…
$ ps -p 44201 -o pid,stat,cmd
  PID STAT CMD
44201 S    python3 immortal.py   # …but the process is still alive
```

The kernel discarded the signal before delivery, so the task keeps running.

---

## 3  Diagnosing stubborn PIDs with pure Python

Most production “unkillables” aren’t flagged; they’re **stuck inside the kernel**.

```python
import psutil, signal

for p in psutil.process_iter(['pid', 'name', 'status']):
    if p.status() == psutil.STATUS_DISK_SLEEP:      # ’D’ state
        print("Blocked:", p.pid, p.name())
        try:
            p.send_signal(signal.SIGKILL)
        except psutil.AccessDenied:
            print("  EPERM — different UID or namespace?")
```

* `STATUS_DISK_SLEEP` corresponds to kernel `TASK_UNINTERRUPTIBLE`.  
  `SIGKILL` is queued but won’t run until the I/O finishes.  
* `psutil.AccessDenied` (or `kill -0 PID` → `EPERM`) means you’re outside the target’s UID or PID namespace.

---

## 4  PID 1 quirks (host & containers)

* **Global PID 1** is born with `SIGNAL_UNKILLABLE`; `kill -9 1` returns `EPERM`.  
* In containers, the entry‑point becomes PID 1 *in that namespace* and inherits the same immunity.  
  **Fix:** run your app under a mini‑init such as `tini` or `dumb‑init` so signals are forwarded and zombies reaped:

```dockerfile
ENTRYPOINT ["tini","--","python","app.py"]
```

---

## 5  Better ways to pause or protect workloads

| Need                               | Tool to use           | Why it’s better                              |
| ---------------------------------- | --------------------- | -------------------------------------------- |
| Pause/resume an entire workload    | **cgroup freezer**    | Stops tasks without abusing signals.         |
| Prevent accidental kills           | Supervisors (`systemd`, `supervisord`, Kubernetes) | Let crashes happen, then auto‑restart.      |
| Faster memory cleanup post‑kill    | `process_mrelease()` (newer kernels) | OOM reaper frees pages even if task is stuck.|

---

## 6  Key take‑aways

* **`SIGKILL` is absolute—unless the kernel never delivers it.**  
* Flipping `SIGNAL_UNKILLABLE` (or running as PID 1) is the *only* in‑kernel way to ignore `kill -9`.  
* The vast majority of “unkillable” sightings are really uninterruptible I/O or permission/namespace issues—no dark magic required.

Happy hacking — and remember: with great `CAP_SYS_MODULE` comes great responsibility!
