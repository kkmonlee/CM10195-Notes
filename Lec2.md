# Inter-Process Communication

## Shared Memory

- Speed of shared memory means that it's very good for IPC, as long as it's supported by further mechanisms such as...
 - signals
 - semaphores
- ... to flag when data is ready

---

Exercise
: Compare shared memory and pipes

---

## Signals
A signal is a software equivalent of a **hardware interrupt**: they can be sent (initiated) by the **kernel** or by a **process**.

- Like a hardware interrupt, when a process recveives a signal, it stops what it's doing and goes off to execute a *signal handler*, in direct analogy with an interrupt handler.
- Handled within the user program: the signal handler is just some code in the program, written by the programmer.

###### Not really what happens.

- When a signal is raised (which needs a syscall) the OS takes over and notes the signal in the receiving process' PCB
- When the OS next runs that process, it jumps to the signal handler code within the process, rather than to the pace where the process was preempted.

- A process can send a signal to another process (or even itself) that has the same owner (same `userid`).
- The normal restrictions on `userid` applies: only `root` can send a signal to another user's process
- But remember that all signals are delivered to processes **via the kernel**
- The kernel can also itself initiate a signal, *e.g., a signal to indicate activity of a peripheral*.
- **Kernel can send signals to any process**

- Use the POSIX function `kill()` to send a signal in a user program
- And functions like `signal()`, `sigaction()`, `sigaddset()` and more to manage signals

A signal is a **single bit**, but there are different types of signals, e.g., HUP, INT, SEGV, KILL, PIPE, etc., to indicate different kinds of events.

A process can, to some extend, choose how to react to a signal
- **ignore it**: that is, inform the kernel that is does not want to receive a particular signal, so the kernel will not note delivery in the PCB
- **accept it and act on it**: run the signal handler code
- **suspend (voluntarily relinquish)**
- **terminate**

Some signals cannot be ignored or otherwise acted upon, in particular a KILL signal (`SIGKILL`), which always terminates the process (i.e., that process will be moved to the exit state).

This is why signals are regulated by the kernel; a user can kill their own processes, but not others'

- Signals are *asynchronous*, that is they might arrive at any point during the running of the program
- Programs that use signals must be written accordingly
- In the real world, they will arrive at the most inconvenient times

There exists default signal actions for each type of signal, but a program must include its own handler functions if it wants to do something other than the default action when it receives a signal.

- When a process receives a signal, it stops what it's doing, saves its state and calls the signal handler
- So this is just like an OS interrupt

### Signal examples

- `SIGINT` -- a general interrupt
- `SIGILL` -- sent by the kernel to a process when it has tried to use a privileged or non-exstent machine instruction
- `SIGKILL` -- non-ignorable terminate
- `SIGSEGV` -- sent by the kernel to a process when it has tried to access memory it shouldn't (*segmentation fault in C*)
- `SIGALRM` -- a timer signal (*not* the preemption timer!)
- `SIGUSR1` and `SIGUSR2` -- signals for the user of user programs
- `SIGRT` -- a large number of signals are provided for real-time processes

---

Each signal has a default action

- `SIGINT`, `SIGILL`, `SIGALRM`, `SIGSEGV`, `SIGUSR1`, `SIGUSR2` -- exit the program
- `SIGTSTP` -- suspend
- `SIGCONT` -- continue after a `SIGTSTP`
- `SIGCHLD` -- ignore

Most default actons can be overridden by the program, some, notably `SIGKILL`, **cannot**.

Signals are

- Fast and efficient
- Asynchronous
- Used a very great deal
- Only transmit a small amount of information
- So often are used in concert with **other IPC mechanisms**
- Are a bit fiddly to program correctly

## Semaphores

The action of a process A waiting for process B to finish something before A can continue is very common.

E.g., waiting for data to be written to an area of shared memory

- It is a very simple form of IPC
- Signals can be used but an alternative is to use a semaphore
- A signal is appropriate when you want to continue computing on something else; ...

- *Invented by Dijkstra*, a semaphore also allows *mutual exclusion*, where we can be sure only one process is accessing a resource at once.
- In this case, the synchronisation is the second process waiting for the first to finish with the resource
- A semaphone is a variable whose value can be accessed and altered by two operations *V* and *P* (Dijkstra is Dutch)
- Alternative names are: signal and wait; post and wait; raise and lower; up and down; lock and unlock; and more...

- Let *S* be a semaphore variable, usually residing in a chunk of shared memory

```
Start with S = 1
P(S):
if S = 1 then set S = 0
else block on S
V(S):
if one or more processes are blocking on S then allow one to proceed
else set S = 1
```



The important point being these operations are *atomic*, namely there is no game (or scheduling) between these tests and the actions.

For synchronisation:

    P(S)
    ... modify a resource...
    V(S)

    P(S) # wait for resource
    ...use resource...
    V(S)

If multiple processes attempt a *P(S)* simultaneously only one will succeed and continue; the others will be blocked
So if we code like:

    P(S)
    some code
    V(S)

being run by multiple processes using the shared semaphore *S*, only one process can execute the code at a time; the other will be blocked and get their turn later

More suggestively, using name signals and wait (**not** the same signal as in signals, earlier!)

Generally, the code would be to access some shared resource (often shared memory, e.g., B shouldn't read until A has finished writing), so the semaphone makes sure only one process can access the resource at a time.

---
This is a *binary semaphore*, as it takes just two values, 0 and 1
There is a simple generalisation to a *counting semaphore*.

    Start with S = n

    P(S):
    if S > 0 then set S = S - 1 else
    block on S

    V(S):
    if one or more processes are blocking on S then allow one to proceed
    else set S = S + 1

This allows no more than *n* processes into the region at once.

Semaphores were first used within OS kernels to protect shared resources but can be used in user programs to protect resources there, too: for example, a chunk of shared memory (e.g., shared memory IPC)

Correct implementation of user mode semaphores is very hard.
We have to ensure that it works even if

 1. the process is rescheduled in the middle between the test and the decrement of the count
 2. there are multiple parallel processors accessing the semaphore simultaneously

---
Suppose *S = 1* and we have processes A and B running, either being scheduled alternately or running simultaneously on two CPUs

    A
    read S
    get 1           read S
                    get 1
    Set S = 0
                    Set S = 0
    runs            runs

Or

    A               B
    read S          
    get 1           read S
                    get 1
                    Set S = 0
                    runs
    Set S = 0
    runs

---

So we can't just use the code as written: the gap between the test and decrement makes this not an implementation of a semaphore.

- There is a problem with *V*: find it
- We have to be a lot more clever
- Solutions come in hardware and software forms

---
### Hardware solutions
#### `test-and-set` instructions
Some CPUs have *test-and-set* instruction which acts like

```
if (i = 0) {
    i = 1;
    return true;
} else {
    return false;
}
```
on a register or memory location, but does it all *atomically*.

*P(S)* could be
```
while (!test_and_set(S.tas)); // do nothing
if (S.count > 0) {
    S.count = S.count - 1;
    S.tas = 0;
} else {
    S.tas = 0;
    wait(S);
}
```
This **protects** the critical region of the test, so eliminating the gap.

*V(S)* could be
```
while (!test_and_set(S.tas)); // do nothing
if (waiting_processes()) {
    start_one();
} else {
    S.count = S.count + 1;
}
S.tas = 0;
```
These solutions are not so good as they employ *busy wait* on the test-and-set (a *spinlock*).

We can use test-and-set itself to protect critical regions
```
while (!test_and_set(tas)); // do nothing
critical region
tas = 0;
non-critical
```
But not all makes of hardware support test-and-set.
>It is better to use semaphores that can be implementented in other ways than test-and-set.

#### Atomic instructions
Another hardware solution is an atomic instruction to exchance the values of two variables *v* and *w*.
```
temp = v;
v = w;
w = temp;
```
This too can be used to build a semaphore.
Similarly for other instructions such as `compare-and-swap`.

### Software solutions
But what if you are using a machine that does not have such hardware instruction?

There are several algorithms that build an atomic semaphore out of non-atomic operations.

These can be *very* subtle and hard to prove correct.

Exercise
: Look at the algorithms by

 - Dekker
 - Peterson
 - Lamport's Bakery

---
Semaphores are widely used

- each semaphore only needs a few bytes of shared memory
- they are small and fast given hardware support
- and OK in software
- used both in OSs and user programs to protect critical resources
- and are widely available in POSIX libraries

>*The simpler the code, the better it is.*

On the otherhand, semaphores are very low-level mechanism and it is easy to cause deadlock.

- Suppose we have semaphore S1 protecting file F1 and semaphore S2 protecting file F2.
- Process A wants to read from F1 and write to F2, while process B wants to read from F2 and write to F1

To make things consistent in the read/writes, both processes must grab both semaphores

- Process A grabs semaphore S1
- Process B grabs semaphore S2
- A tries to grab S2 and blocks
- B tries to grab S1 blocks

***... and we have a deadlock!***

Some regard semaphores are too lowlevel and too easy to use incorrectly, so there have been many other operations devices to provide synchronisation and mutual exclusion

- **Condition Variables**: more for synchronisation that mutaul exclusion; allows processes to agree on when to proceed
- **Barriers**: to synchronise a number of proceses at one point
- **Monitors**: more "object-oriented" as blocks of critical region code are implicitly protected (as provided by Java).
- and more where the construct tries to hide the complexity of synchronisation from the programmer (tuple spaces, etc.)

Exercise
: Use a counting semaphore to solve the Dining Philosopher's problem
