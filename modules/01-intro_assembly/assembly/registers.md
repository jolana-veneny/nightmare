# Registers

Registers are in RAM and are built into the CPUs. As volatile memory they are a fast (the fastest, in fact) way to pass data around and are where instructions are loaded when programs execute. 

You can put anything in a register, as long as it fits. 32 bit  architectures have 4 byte long registers, 64 bit have 8 byte. This means you can put an 4byte pointer in a x86 register, or a 4byte string, or a 4 byte int. You can also put a 1byte int into an 4byte register, but there are some things you have to do to make that work right. 

32 Bit Platforms have 8 registers, 64 bit have 16. 32 bit architecture uses a naming convention that starts with E, 64 starts with R.

We are going to be playing around with x86 and its 32 bit architecture to start and will make the leap into x64 later. 

The most notable registers are as follows:  (Named in 32 bit convention)

* EAX - multipurpose, often to return values
* EBX - often base pointer for data section
* ECX  - often a counter
* EDX  - often an I/O pointer
* ESI - "source index"
* EDI - "destination index"
* EBP - base pointer
* ESP - stack pointer

In addition to General Purpose registers we have  EIP (32 bit) and RIP (64 bit) instruction pointers. 
There are also registers of registers named EFLAGs, but we can ignore them for now. 

Don't worry too much about what happened, just a taste for ya. I'll explain more of  what these all mean in the next few modules. 

As an interesting note, some registers (EAX, EBX, ECX, EDX) are split into multiple registers. 
For example, EAX is split into AX, which is split into AH and AL. 
```
31 bits_________________16 __15____________________ 0 bit
|     EAX                    |              AX          |              
----------------------------------------------------------
|                            |    AH     |      AL    |
_____________________________________________________
```

This means that we can set AH, AL, or AX, independently of EAX, in case we only need one or two bytes... of course, that will clobber the low bytes of EAX so you have to watch out for it! We can access the lower byte of EAX with AL. Here is an example.
```
First, EAX = 1337 
EAX is equal to  binary(10100111001).
AL would be equal to binary(00001001) or shorter, binary(1001).
If we set AL = 0x4 , the new value of AL would be binary(0100).
EAX would then be equal to binary(10100000100).
```

There is a lot going on here. Smile and wave, smile and wave. 

