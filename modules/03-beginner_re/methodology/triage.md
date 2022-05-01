# Triage 
Using 'beleaf' in ./csaw19_beleaf, use some of your CLI tools to learn more about it.

The goal for triage is to move quickly and get the lay of the land as quickly as possible.

Run:

1. `file` and `checksec`
2. `strings` 

Now actually just run the program a few times to figure out what is going on, play around with inputs. 

3. `ltrace` and `strace` are excellent for figuring out what is going on behind the scenes. 

4. If you need to spend more time figuring out what libraries are being run, `readelf` and `ldd` are great. This isn't too important for smaller binaries, but are great for massive binaries that do a lot of things. 

5. `valgrind` is great for figuring out what heap writes and reads are going on, but definitely isn't something you always need. 

Don't really worry about understanding everything. Just play with it for a few minutes

Try to think through these questions:
1. What does file tell us about this?
2. Are there any interesting strings? 
3. What are the interesting library calls made?
4. What are the interesting system calls made?
5. Is there anything else interesting you noticed? (Note: there doesn't need to be.)