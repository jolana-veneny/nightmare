How you get your exploits to the binary doesn't particularly matter, what does is how you encode those bytes. Everything else is just ease of development. 
You should probably always be using pwntools, but this is all useful if you have to live off the land.

Bash:
```
echo $'AAAAAAAA\x08\x04\88\72' | ./program
```
or
```
echo -ne 'AAAAAAAA\x08\x04\88\72' | ./program

```

Bash Command Substitution:
```
./program $(echo $'AAAAAAAA\x08\x04\88\72')

```
From a File:
```
cat file_with_payload | ./program
```

Python2:
```
exploit_string = "A"*8
exploitString += "\x08\x04\88\72"
print(exploit_string).
```
Or in short:
```
python -c 'print "A"*8+"\x08\x04\88\72"' | ./program

```

Python3:
```
exploit_string = b"A"*8
exploitString += b"\x08\x04\88\72"
# Print doesn't work anymore!!!
sys.stdout.buffer.write(exploit_string).
```
Or in short: 

```
python3 -c 'sys.stdout.buffer.write(b"A"*8+b"\x08\x04\88\72)' | ./program
```

As an important note, all of these close out after writing to stdin, which means that you might miss a flag that gets printed, or you won't be able to interact with your shell. 
The best way to solve that is to throw a command at the end of the command to keep things open.
```
(echo -ne 'AAAAAAAA\x08\x04\88\72'; cat) | ./program
```


GDB:
```
gdb ./vuln_prog
run < <("echo $'AAAAAAAA\x08\x04\88\72'")
```
```
gdb ./vuln_prog
run < <("python3 -c 'sys.stdout.buffer.write(b"A"*8+b"\x08\x04\88\72)'")
```
```
gdb ./vuln_prog
run < <(python3 ~/exploit.py)
```

