# Operating Systems: Introduction

 - OS is just a program called the **kernel**.
 - Anything in the OS is done by a program.
 - A kernel's purpose is to
  - manage **resources** of the computer
  - provide the programmer with a useable **programming interface** to access those resources
 - The interface (GUI we see) is **not part of the OS**.

## Resources -- what are they?
Hardware
: CPU, memory, disk, network, sound, video, keyboard, etc.

Software
: anything that controls the above, though processing by use of the CPU is a primary focus

### Why manage resources?
 - For example, mobile phones have strict limitations on memory, CPU, power and energy consumption.
 - Most computers are small and very limited.
 - They need protection
  - preventing one program from corrupting another program or data on the same or another machine -- **security & integrity**
  - ensuring certain resouces are only available to those claims to have -- **authorisation**
  - protecting you from mistakes (eg. did you want to delete `/home/`?)
 - They need to meet certain criteria
  - **responsiveness**: making a program respond snappily or process network packets as soon as they arrive
  - **real-time**: certain events must be dealt within a fixed amount of time
  - **security**: prevention of accidental or malicious access or modification

## Programming Interface
 - Programmer who has to write applications for the machine does not want to know details of hardware
 - Think portability -- von Neumann's model
  - how do I get the best performance out of this disk, network, etc?
  - how do I prod this hardware to do what I want?
  - how should I deal with interrupts?
  - and so on...
 - We don't want to have to re-implement everything in every program.
 - Much better to let someone else use it (code reuse)
  - having an expert do this stuff once and provide a standard interface for us
