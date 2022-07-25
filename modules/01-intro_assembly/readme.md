This is a brief overview of resources to understand how computers work at the CPU and process level. 

# Basic Architecture

## Learn about the fetch, decode, execute cycle. 

<https://www.futurelearn.com/info/courses/how-computers-work/0/steps/49284>

## Program Counter and Instruction Pointer

These two are the exact same thing. Don't increment your Instruction Pointer until this article (and this joke) makes sense.

<https://embeddedartistry.com/fieldmanual-terms/program-counter>

If things don't make sense, keep trying to understand fetch decode execute.

I don't hate this video: <https://www.youtube.com/watch?v=ytKKCMHHxys>

## Registers and RAM

CPU has to save data somewhere, right?

#### Registers: 

Check out [this lesson on Registers](assembly/registers.md):

Then read this: <https://www.geeksforgeeks.org/different-classes-of-cpu-registers/>

#### RAM

Read [this article](https://www.techwalla.com/articles/how-does-ram-work-with-the-cpu) to get a more practical understanding of how RAM and CPUs work together.


## Programs in Memory

Programs in Memory

As discussed in the previous section, when programs are executed, the operating systems load them into RAM, aka memory. From here they are executed as the assembly instructions order them to.

First, read this over to see the general layout of programs. You only need to worry about the Stack and the address layout right now.

<https://www.geeksforgeeks.org/memory-layout-of-c-program/>

We will spend a lot more time on this later, so don't worry about any of it. I'm just giving you context.


# Assembly 

## Architectures 

Architecture is the specific design of hardware that an assembly language is written for. As an assembly language is a 1 to 1 mapping to machine language spoken by the processor, the architecture defines what the hardware does. It provides the instructions, the data types, the formats, the resources, all in a giant specification. The people who build CPUs implement this architecture specification. From there, the assembly language writers build an assembly language on this specification. This allows the language to be run on any CPU that matches the architecture.

Virtual machines implement their own architecture and are generally able to run other architectures on top of themselves.

The majority of desktops, laptops, and servers run on Intel x86 or its variants. A notable exception is the new Macbook Pro, which runs on custom ARM hardware.

Most mobile devices run on ARM. As mobile devices dominate the world, this means that ARM dominates the world. In the future we will see more ARM devices for everyday computing.

This section needs a bit more writing, but for now, read over the Wiki page, it's pretty decent: <https://en.wikipedia.org/wiki/Hardware_architecture>

## Assembly Tutorials

In this course you will mostly be learning about x86 assembly in the Intel syntax. If that means nothing to you right now that is fine, you'll be an expert soon enough.

[Click here to learn about x86 assembly.](assembly/readme.md)

Once you have finished that check out [this tutorial on reversing assembly.](reversing_assembly/)

Finally, work through [the Aviv Assembly notes](unit_02.md) and do the assignments in it.

## Wrap Up 

Watch this video from the great hacker LiveOverflow on the [CPU and Assembler](https://www.youtube.com/watch?v=6jSKldt7Eqs). You'll be seeing more of his content.

Read this and make sure it sort of makes sense: <https://revers.engineering/applied-re-basic-architecture/>. You absolutely should move on even if it doesn't, this whole course is meant to keep you moving forward before you understand exactly what is going on.

# [NEXT: Chapter 3: Intro. to Tooling](https://github.com/hoppersroppers/nightmare/blob/master/modules/02-intro_tooling/readme.md)
