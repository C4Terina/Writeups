# First thoughts and steps 
I downloaded all the necessary files from [here](https://app.hackthebox.com/challenges). The downloaded file was impossible_password.zip which I then extracted and got impossible_password.bin.
My first thought was "oh a binary file, let's google how to reverse engineer a binary file before doing anything" and that's how I came across this video by [LiveOverflow](https://www.youtube.com/watch?v=VroEiMOJPm8).
That video ended up being extremely useful because it allowed me to understand how the debugger works and a few basic instructions. 

# Checking what the binary file does 
The next step was to see what the binary file does without looking any deeper yet. So I made it an executable and run it. 

![image](https://user-images.githubusercontent.com/68371827/163551712-b76a579d-9cf6-46ea-8ac8-1aa32b132a03.png)

As we can see there is a function which gets our input and prints either the first 20 characters of what we entered inside brackets or prints everything we entered before hitting space. 

# Digging deeper 
I first thought of using either [IDA](https://hex-rays.com/ida-free/) or [Ghidra](https://ghidra-sre.org/). I tried solving the challenge using Ghidra but as a beginner I didn't understand much.
The only useful info I got from Ghidra was finding the main function. 

![image](https://user-images.githubusercontent.com/68371827/163552532-48542ef8-095d-475a-8096-2c4f7206b204.png)

I first read this about the [__libc_start_main](https://refspecs.linuxbase.org/LSB_3.1.1/LSB-Core-generic/LSB-Core-generic/baselib---libc-start-main-.html). And I quote:
> The __libc_start_main() function shall perform any necessary initialization of the execution environment, call the main function with appropriate arguments, and handle the return from main(). If the main() function returns, the return value shall be passed to the exit() function. 

So from my understanding that's where the main function is! Let's see what it actually does now...

# Radare2 and dissasembling the code
I pretty stuck after finding the main function. After a little search and looking into the [HTB forums](https://forum.hackthebox.com/t/impossible-password/173) I saw that many people have used [Radare2](https://rada.re/n/radare2.html) to solve
the challenge so I decided to give it a try. I searched up how to use radare2 and came across [this video](https://www.youtube.com/watch?v=oyIoL32Bby0). I decided to follow the video step by step and see where it leads me while understanding how 
r2 works. 

![image](https://user-images.githubusercontent.com/68371827/163554443-3f23898b-bcea-4447-92f2-429842a97dc9.png)

I first started by analyzing the binary using aaa. After that I used the [f command](https://book.rada.re/refcard/intro.html?highlight=f#flags). I saw a huge list of all the flags and tried to find the main function. 

![image](https://user-images.githubusercontent.com/68371827/163554900-f4476063-3e48-490b-94e6-c060ff95f8a2.png)

Now let's seek to it. 

![image](https://user-images.githubusercontent.com/68371827/163555203-dc496398-114a-4076-a4ae-9aec9ae7ef59.png)
![image](https://user-images.githubusercontent.com/68371827/163555267-ec8c0703-c44d-4b6b-8606-bc266f748452.png)

A lot of information here. First of all the pdf command I used stands for print dissasemble function. You can find all the r2 documentation [here](https://book.rada.re/index.html). 
As we can see from the screenshots there's a main function which basically takes two strings, compares them and if they are equal the program jumps to ```0x00400925``` address:

![image](https://user-images.githubusercontent.com/68371827/163555993-f6264a3b-e0cd-4fa2-985b-56a5772f4527.png)

From the dissasembly alone we can understand which two strings it is trying to compare. First is our input (s1) and the second is
> 0x0040086c      48c745f8700a.  mov qword [s2], str.SuperSeKretKey ; 0x400a70 ; "SuperSeKretKey"

which is s2 

So let's test our finding as we run the program:

![image](https://user-images.githubusercontent.com/68371827/163556237-bedf5a17-1f3f-471b-83ba-73ce28191ff9.png)

Great! That means we are one step closer to getting the flag. After that step we see that there's another string compare:

![image](https://user-images.githubusercontent.com/68371827/163556463-4ec7c415-3b65-4fdc-8f8e-f74dcfd336f1.png)

But this time if the strings are equal it just...exits. From my understand the other string is this long "password":

![image](https://user-images.githubusercontent.com/68371827/163556700-e42ea644-8bee-45a1-8662-906afe754c06.png)

Let's test it out: 

![image](https://user-images.githubusercontent.com/68371827/163556852-79f937b8-7416-467d-a156-a5585696b715.png)

The program exits as expected when we enter the string. 


# Modifying the ASM instruction 
As we can see from this screenshot 

![image](https://user-images.githubusercontent.com/68371827/163556463-4ec7c415-3b65-4fdc-8f8e-f74dcfd336f1.png)

We can see that there is another subroutine called: ```0x00400971      e802000000     call fcn.00400978```. I'm guessing this is the subroutine which generates the flag and prints it. So what if instead of making the program jump to exit it instead continues on to call the subroutine? After a bit of searching and some help from my
surroundings I was told that the easiest way to bypass the jne instruction is to write no operation instruction (nop). And so that is what we are going to do:
First we enter read-write mode and we seek the ```0x00400968``` memory address which the jne instruction we want to modify is in. 

![image](https://user-images.githubusercontent.com/68371827/163558992-dab8bc98-e3d4-442e-8251-fc2b7e0d9809.png)

After that we will change the conditional and check the result with pdf: 

![image](https://user-images.githubusercontent.com/68371827/163559644-3b400073-4714-4932-af52-35edfbc4bcf9.png)
![image](https://user-images.githubusercontent.com/68371827/163559900-2ce22388-5874-429f-9d77-0da147c32135.png)

Success! Now we will exit r2 by entering ```q``` because we will not need it anymore. Let's test our modifications: 

![image](https://user-images.githubusercontent.com/68371827/163560175-da8b3ad4-ed5c-41ae-9093-e2ef29ec207d.png)

And we got the flag which I will not show for obvious reasons >:)

# Extras
As a beginner I've probably left out a lot of valuable information but this is how I understood the challenge. I hope this is clear enough and with not many mistakes. 




