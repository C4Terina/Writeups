# Intro
This is a writeup for the medium reverse challenge from [UniCTF 2022 organized by HackTheBox](https://ctftime.org/event/1825). As a beginner in reverse engineering I challenged myself to solve one of the challenges in the CTF and after 15 hours of work I did it.

# First thoughts 
I downloaded the binary given, ran it and did some basic static analysis you always perform when playing such challenges. 

![image](https://user-images.githubusercontent.com/68371827/205880352-c3b5af96-55f7-4424-a055-8d9cdc501640.png)

Running it gave me an error and ``strings`` showed me nothing. File however showed me with what kind of file I have to work with. What I usually focus on is whether the binary is stripped or not stripped and if it's dynamically linked or not linked. This one is dynamically linked and not stripped which basically means that when I dissasemble it with Ghidra/Ida/Binja it will be easier to understand. 
After doing this I immediately opened Ghidra to check what's going on. 

# Disassembling 
First thing you have to do when you import the file and analyze it is to find the main function

![image](https://user-images.githubusercontent.com/68371827/205882096-140787c7-5752-4251-946f-273aa35664c4.png)

For the sake of me understanding what is going on I renamed some variables and changed the data type of some. So we have a ``printf`` which prints out the message we see in the terminal, an ``fgets`` which reads our input, a function called ``install_filter()``, a for loop and a huge ``syscall``

# Checking install_filter()

![image](https://user-images.githubusercontent.com/68371827/205882934-dda68703-64f1-4b32-83cf-b14bda18c84e.png)

First thing I did when looking at this was reading what ``prctl()`` does. After A LOT of reading and googling I came across these two websites. First one is a writeup for a [seccomp ctf challenge from 2017](https://blukat.me/2017/11/hitcon-quals-2017-seccomp/) 
and the second one explains [what seccomp and a bunch of other stuff](https://medium.com/codex/understanding-the-linux-kernel-through-ctf-challenges-seccomp-be6ed553a97). After reading and understanding what seccomp does I guessed that 
``install_filter()`` is a custom filter which sets up seccomp. I also realised that it wouldn't help me solve the challenge as there was nothing else to check. 
So next logical step was to follow the steps from the writeup I found.

# Seccomp-tools installation and dump 
The person who wrote the writeup I linked said that they used [seccomp-tools](https://github.com/david942j/seccomp-tools) for analysing the filter and that's what I did. After installing Ruby and seccomp-tools 
I followed the instructions which gave me the following output

![image](https://user-images.githubusercontent.com/68371827/205885417-54fcbd58-cb78-470f-8c12-e8fb4fd7d39e.png)

The output was 144 lines long. I quickly realised that this was not what a normal seccomp dump should look like. I spent hours looking at it and after some confirmation from outside sources I realised that this dump contains the flag. So I started checking the dump line by line. 

```0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x01 0x00 0xc000003e  if (A == ARCH_X86_64) goto 0003
 0002: 0x06 0x00 0x00 0x00000000  return KILL
 0003: 0x20 0x00 0x00 0x00000000  A = sys_number
 0004: 0x15 0x01 0x00 0x00000258  if (A == 0x258) goto 0006
 0005: 0x06 0x00 0x00 0x7fff0000  return ALLOW
 0006: 0x20 0x00 0x00 0x00000010  A = args[0]
```

These few lines tell me that if my architecture was ARCH_X86_64 then the binary wouldn't even execute. After that it puts ``sys_number`` into A and check if A == 0x258 which in decimal is 600. I've seen 600 before! It's the first argument in the syscall from main. 
After that it puts args[0] into A and I guessed it's the second argument of syscall so the rest should probably look like this 

``syscall(sys_number, args[0], args[1], args[2], args[3], args[4], args[5])``

Awesome this means that we know what goes into each of these arguments. After that I did what any sane person would do and did not write a script but instead wrote each instruction of the dump on a piece of paper and "executed" it. 

![IMG_20221206_123510](https://user-images.githubusercontent.com/68371827/205888478-057873c3-a284-4597-95ac-7c6e39521451.jpg)

I realised that 72 in ASCII is H and my flag should start with ``HTB{``. What I basically had to do was ask myself "what number do I have to substract from 72 to give me 12? 84!" and 84 in ASCII is T. I did this for every single instruction till I reached this point 

![image](https://user-images.githubusercontent.com/68371827/205889253-132b738a-c06b-476d-a0d9-5b30a9ce059e.png)

So far I had ``HTB{abr4ca-secco``...I was so close....what do I substract from 4294967294  to give me 14? This number doesn't exist in the ASCII table. I knew that the 3 last letters
should be ``mp`` to complete the word seccomp but ``HTB{abr4ca-seccomp}`` was not the flag. I was going insane at this point because I was on the same challenge for 15 hours straight. So I asked for a sanity check in HTB discord server for this challenge. 
I told them that I had this part of the flag and asked how many characters I'm missing. The answer was 4. I knew 3 of them were ``mp}`` and I started thinking to myself "what if it had an ! at the end?" AND IT DID. So the flag was ``HTB{abr4ca-seccomp!}`` and got it simply by guessing :)

