Write up:

I saw crazy casino and wanted to attempt reverse engineering it.

**Looking at assembly/disassembly in Ghidra:**

I started by downloading the chal file and popping it into ghidra. Noticed a few functions that caught my eye:
1) vip_lounge (THE END GAME)
2) entrance_desk (The line outside the club I'm not allowed in. AKA Where I spent most of my time)
3) main (kind of useless in this instance)

Let's start off by looking at entrance_desk first.

- I noticed that there was a variable I named "Signature" that was 64 bytes (this is the buffer)
- I noticed the read function that reads from a file descriptor of 0 (stdin) into the signature variable (the buffer) which is 64 bytes but the count allows for 200 bytes of input to be sent.

This is dangerous because read will write as many as 200 bytes, but if signature (buffer) is only a 64-byte array, then any input longer than 64 bytes will overflow past the end of that 64-byte buffer, corrupting adjacent stack/heap data. 

AKA a classic buffer overflow.


**GDB STUFF:**

So I started the attack by using GDB and trying to identify the number of bytes it takes to crash the program.

AKA Fuzzing.

I found that at the 76 character point for input was when I started getting sigsegv errors 




**Looking at assembly/disassembly in Ghidra:**

At around this time I started looking at the end game. I won't know what my payload is if I don't look at my goal.

vip_lounge takes in one parameter. COINS!!!

vip_lounge only checks 1 thing. If you have the right amount of coins.

If you have more or less than 1,000,000 coins. You are SOL.

So that's my goal. 1,000,000 coins or bust.




**Fuzzing:**

The final step was the longest Creating the payload:

I knew things broke at 76 characters. But I didn't know where the return value was stored on the stack.

So I started adding new characters after my 76 A's. I added in 4 byte intervals becuase it seemed like this program was using mostly DWORDS.

I saw the BBBB and CCCC go into the ebx and ebp registers. And finally with DDDD I saw the following error message in GDB:

[!] Cannot access memory at address 0x44444444

This is when I knew where the destination memory location should go.

1 problem was how the coins were stored in vip_lounge. Which is the "EBP" Register


**The python creation part:**

This is where I went off course a bit. I tried doing a ROP Chain to change the EBP register before returning to vip_lounge.

Problem was that it was EBP+4 bytes. So even if I put 1,000,000 coins in ebp, it didn't matter.

After struggling to figure out why it wasn't working for a while, I finally figured out that adding a dword little-endian 1,000,000 twice after rop-jumping into vip_lounge would overwrite the correct register and allow me to get the flag!!!

My python is supplied in this repository. I'm sure this code is not the most efficient way to solve this flag, but I got to implement buffer overflows, I did rop-chaining and used the ROPgadget python tool to help me find some ROP Gadgets for my solution, I played with pwntools again. Overall it was a good exercise for me.
