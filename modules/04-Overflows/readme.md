Time for the good stuff, aka, the reason you are doing this course!

Before we jump into this, I need to make sure you understand the stack. 

# The Stack

The Stack is the fundamental underpinning of how memory works. It is complicated but you must understand it to move forward. We are going to watch a few videos until it makes sense. 

This explains the stack as a data structure: <https://www.youtube.com/watch?v=CgFVgp_VCN8>

This explains stack for memory: <https://www.youtube.com/watch?v=IWQ74f2ot7E>

And this finally explains what the hell it all means in regards to how programs work: <https://www.youtube.com/watch?v=Q2sFmqvpBe0&t=318s>

### Function Call Basics:

Read this, understand this, just keep going back over this until it all makes sense. This page is the single best reference for how programs actually work on the stack in existence.

<https://zhu45.org/posts/2017/Jul/30/understanding-how-function-call-works/>


# Overflows

Now that we understand the stack, let's learn how overflows work.

1. For an overview on overflows, watch this video: <https://www.youtube.com/watch?v=1S0aBV-Waeo&t=6s>
2. Check out these slide decks on overflows from RPISec's MBE on overflows [Lecture 04](04_lecture.pdf) and [Lecture 04 Extras](04_lecture_extras.pdf). 
3. Watch [this video from LiveOverflow](https://www.youtube.com/watch?v=T03idxny9jE) that should help you understand the mindset required for overflows.
4. Work through [Aviv's Overflow Notes](unit_03.md). Really take the time to get your head wrapped around these problems. 
5. Read this document ["Smashing the Stack for Fun and Profit"](http://www.phrack.org/issues/49/14.html#article) because it is famous. At the time this paper came out, the knowledge of the existence of buffer overflows was kept secret among a very small group of people. This is the first known document describing the dark magic that was released to the public. Now you are learning about it with a ton of examples on Github. Pretty dope.

If you are still not feeling 100% sure what you are supposed to do, don't worry, that is perfectly normal. Just move on and work through these writeups.

### Crush Some Variables 

3. Do your first overflow! [Challenge 1](04-bof_variable/tamu19_pwn1/)
4. Next one up! [Challenge 2](04-bof_variable/tw17_justdoit/)
5. This one is a bit harder[Challenge 3](04-bof_variable/csaw18_boi/)

### Call some Functions

6. Calling functions is pretty cool. [Challenge 4](05-bof_callfunction/csaw16_warmup/)
7. More CSAW challenges! [Challenge 5](05-bof_callfunction/csaw18_getit/)
8. Vulnchats are classic problems. [Challenge 6](05-bof_callfunction/tu17_vulnchat/)


# Shellcode 

Calling functions is nice and all, but you know what's better? Calling our own code! Let's learn about shellcode! 

0. Watch this video from LiveOverflow: <https://www.youtube.com/watch?v=HSlhY4Uy8SA>
1. Read [MBE's shellcode lecture](05_lecture.pdf).
2. Work through [Aviv's Shellcode Notes](unit_04.md). Take serious time on this. 
3. Do your first shellcode exploit. [Challenge 7](06-bof_shellcode/csaw17_pilot/)
4. Do another. REPS AND SETS. [Challenge 8](06-bof_shellcode/tamu19_pwn3/)
5. Alright this one is a bit harder. [Challenge 9](06-bof_shellcode/tu18_shellaeasy/)

### Mitigations 

Let's talk mitigations! The world has decided it doesn't want overflows to be too easy since Aleph1 put out Smashing the Stack so plenty of things have been added to make our lives slightly harder. 

To figure out what mitigations are in place, use the "checksec" program. Once you figure out what is there, start thinking through how to defeat the specific mitigations. 

8. Read this [overview on a variety of mitigations](https://resources.infosecinstitute.com/topic/how-to-mitigate-buffer-overflow-vulnerabilities/)
9. Read this too, because why not? <https://ctf101.org/binary-exploitation/what-is-binary-security/>
10. Read this [writeup on canaries](7.1-mitigation_canary/readme.md).
11. Read this [writeup on ASLR and PIE](5.1-mitigation_aslr_pie/readme.md)
12. Read this [writeup on NX](6.1-mitigation_nx/readme.md)
13. Learn how to defeat some mitigations with [Aviv's Unit 5 Notes](unit_05.md)

Mostly, we are going to need to do some ROP, but that's for a later section.

