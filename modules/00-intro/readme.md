# Intro
`Written by GuyInATuxedo`


So I just want to say a few things for the people who are super new to binary exploitation / reverse engineering. If you are already familiar with assembly code / binary exploitation and reverse engineering, and tools like ghidra / pwntools / gdb, feel free to skip this whole section (and any other content you already know). The purpose of this section is to give an introduction to the super new people.

## Binary Exploitation

First off what's a binary?

A binary is compiled code. When a programmer writes code in a language like C, the C code isn't what gets actually ran. It is compiled into a binary file and the binary is run. Binary exploitation is the process of actually exploiting a binary, but what does that mean?

In a lot of code, you will find bugs. Think of a bug as a mistake in code that will allow for unintended functionality. As an attacker we can leverage this bug to attack the binary, and actually force it to do what we want by getting code execution. That means we actually have the binary execute whatever code that we want, and can essentially hack the code.

## Reverse Engineering

What is reverse engineering?

Reverse engineering is the process of figuring out how something works. It is a critical part of binary exploitation, since most of the time you are just handed a binary without any clue as to what it does. You have to figure out how it works, so you can attack it.

## Objective

Most of the time, your objective is to obtain code execution on a box and pop a root shell. If you have a different objective, it will usually be stated on the top line of the writeup. In almost every instance where your objective isn't to pop a shell, it's to get a CTF flag associated with this challenge, from the binary.

## What should I know going into this course?

There are a few areas that will help. If you know how to code, that will help. If you know how to code somewhat low level languages like C, that will help more. Also an understanding of the basics of how to use Linux helps a lot. But realistically, the only thing you really need is the ability to google things, and find answers by yourself.

## Why CTF Challenges?

The reason why I went with CTF challenges for teaching binary exploitation / reverse engineering, is because most challenges only contain a small subset of exploitation knowledge. With this format I can split it up into different subjects like `buffer overflow into calling shellcode` and `fast bin exploitation`, so it can be covered like a somewhat normal course.

## Environment

For your environment to actually do this work, I would recommend having an Ubuntu VM. However don't let me tell you how to live your life.

## Why Should I do this?

First off, I find it fun (if you don't find this fun, I wouldn't recommend doing this). Plus there are a lot of jobs out there to do this work, with not a lot of people to do the work. Plus who doesn't want to drop that chrome 0-day and watch the world burn?

## Difficulty curve

One more thing I want to say is the difficulty curve, in my opinion, is like that of a roller coaster. There are certain parts that are easier, and certain parts that are harder. Granted, difficulty is relative to the person.

# C, Compilation, and Binaries
`Written By Roppers`
Very briefly, we are going to do a quick refresher on the background info needed to understand what is going on in this course. As mentioned above, you don't need to be an expert in C or assembly, just have enough context and you will be able to figure it out. 

1. Read this: <https://www.geeksforgeeks.org/compiling-a-c-program-behind-the-scenes/>. Compile step by step and understand what is going on. 

2. On Linux, after you compile a C program it will be an ELF. There are big ole reference docs on this file format like all file formats, but we will skip that for now. In fact, we won't even waste a bunch of time learning about what ELFs are... do things, don't read about them. However, if you are the type of person who need to know something about what they are working with and you don't trust me, these resources are pretty decent.
   * <https://www.opensourceforu.com/2020/02/understanding-elf-the-executable-and-linkable-format/>
   * <https://linuxhint.com/understanding_elf_file_format/>

3. Do this readelf tutorial: <https://www.geeksforgeeks.org/readelf-command-in-linux-with-examples/>

4. After that, work through [Aviv's Unit 01 notes](unit_01.md) on C programming and compilation. Do all of the work shown in the lecture.

This will not have made you an expert in C compilation but it is enough to get you started. 


# [NEXT: Chapter 2: Intro. to Assembly](https://github.com/hoppersroppers/nightmare/blob/master/modules/01-intro_assembly/readme.md)
