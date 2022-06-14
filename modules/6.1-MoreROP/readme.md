Now it's time for more ROP, but doing things a bit differently.


# SROP
SROP, or Sigreturn Oriented Programming, is a variant of ROP which opens up new doors for us. 

Read this: <https://hackmd.io/@imth/SROP>

Then read the whitepaper that created SROP [here](https://www.cs.vu.nl/~herbertb/papers/srop_sp14.pdf)

1. backdoorctf_funsignals
2. inctf17_stupiddrop
3. swamp19_syscaller
4. csaw19_smallboi

# Stack Pivot / Partial Overwrite

Sometimes, instead of controlling EIP, we find ourselves with the ability to control the stack frame. In Stack Pivoting, we create a false stack somewhere else (often the heap) and begin execution there.

1. defconquals19_speedrun4
2. insomnihack18_onewrite
3. xctf16_b0verfl0w

# Ret2Csu / Ret2dl

1. ropemporium_ret2csu
2. 0ctf 2018 babystack
