That one was tough.

# First Thoughts

I started by having a look at the main function, where we can see that they *finally* resolved their **getsn** size issue. In the following screenshot we see the instruction at adresse 0x4454, giving a size of 19 bytes to the **getsn**.  
This is a bad news.

![Imgur](https://imgur.com/QkdB7BV.png)

I really try everythink here to overflow but could'nt find anything. No way to corrupt memory with this getsn function for what I know. OK. So there is another way. If it is not in the **main** function, it has to be in a function called by **main**. This is why I decided to dig into **printf**. And man, I dug.

# The printf function

It took me a while, half by static analysis, half testing dynamically what was going on, but I eventually reach a global understanding of the function. The hard part was doing all this on that big function not knowing if the security breach was in it. But I guess that's what a reverse engineer do : looking without certainty. I still had the advantage of knowing *there was* a security breach for sure !  

In the following text I will use the words "string" and "password" equally as I worked on the call of printf thats is supposed to print-back our password.

1. So first the function starts by looking for characters "%" in the string. For each character "%" that is not followed by another "%", a counter is increamented. Ok this is no surprise to anyone who started programming with C.  
2. Then if we have n "%" not followed by another "%" in the string, we will add 2*(n+1) bytes on the stack.
3. After what we copy n words from the string into the stack.
4. And now the BIG loop.

Here the function will go byte after byte through the password, checking first if the current byte is null, then if it is a "%", and if not the byte will be print using the **putchar** function.  
If the byte is a "%" (0x25 ASCII value) then the following char is tested :
* if it is another "%" then a "%" is printed and the function will continue by checking the byte after this "%%" pattern
* if it is one of the following set, there is customs actions taken : %s, %x, %n.
* if it is any other value, it is just not printed and we go to the following byte.  

I spent a lot of time looking at the custom actions taken for %s and %x (this last one was very instructive, using bitwise operations such as **rcc**, **rra**, playing with additions to switch bytes and so on). I quicly had a look at the %n and all that time I was trying to find a way of overflowing the stack. My main goal was to cheat somehow with the counter of "%". The idea was that this counter being used to aloocate space on the stack, maybe something could be leverage here. 
Nothing was showing up from those analysis and I was a bit tired. I decided to stop and have a look at the bigger picture : if an overflow was not the answer, was there a way to write, somehow, in memory ?  
This gave me the good lead. There were only two instructions moving values directly into memory : one at adress 0x461a, the other at adress 0x46ba. The first one was in my scope when I was tring to cheat on the "%" counter, and I knew it wasn't the answer. so I focused on the second one, marked by a red arrow in the following picture :

![Imgur](https://imgur.com/ZPKmDbP.png)

## The %n format

The second instruction that could write in memory was inside the %n case. The instruction is a simple `mov r10, 0x0(r15)`. I knew it was supposed to write in memory because of its syntax : 0x0(r15) indicates a place in memory, r15 being an adress and 0x0 and offset.  

![Imgur](https://imgur.com/YYW2l6n.png)

Now two question : what is in r10, and what is in r15 ?
1. r10 was easy : static analysis was enought to understand that is was keeping track of the number of char printed
2. r15 is set by the previous instruction : `mov @r9, r15` and I already had noticed r9 was initially pointing to the begining of the stack and then incremented after each putchar.

So, using a %n, whatever is pointed by r9 will be seen as an adress, and the number of characters printed before will be written at this adress... Interesting...  
I already knew that some words of our password are written to the stack at step 3 of the function, so maybe we can use this mechanism to put an adress of interest in the stack, and then overwrite wathever stands at this adress with the number of caracters printed by the function.  
So I tried the payload `9999256e` to see what happens and... Nothing. Weird. I was expecting some crash as the adress 9999 makes no sens. After having a look closely at what was hapening I saw that, at step "3" of the function, there is first a null word copied in the stack. Damn it ! The adress we try to write is written just after this null word but when we go through the "%n" code, r9 is pointing at this null word.  
However we can dump the memory at adress 0x0000 and see that we *did* write our counter there :

![Imgur](https://imgur.com/d63AX0a.png)

So now I need a way to make r9 pointing to the 2nd word of the stack when we execute the "%n" part of the code. Easy enought : after a look at the code we see that r9 will be increased after the detection of a "%" char followed by... whatever ! So let's try it :

`999925aa256e`

![Imgur](https://imgur.com/tqCsbeb.png)

BINGO !

So the 9999 is written in the stack after the null-word, then the "25aa" trigger the `incd r9` instruction at adress 46be, making r9 pointing to this "9999". Then the 256e (%n) will trigger the write of our counter (r10) at adress 9999. Which crash.
No we just need an interesting adress to overwrite...

# Exploitation

The last step was pretty easy : what adress do we want to overwrite ?

![Imgur](https://imgur.com/wpT8acY.png)

If the value pointed by the stack pointer is not 0, than we unlock the door ? Well let's overwrite that value then ! A quick breakpoint on this instruction gives us the adress pointed by SP at that time : 0x

![Imgur](https://imgur.com/dkNn6aU.png)

Ok well let's tick the "hex input" and enter `fa3925aa256e` ! That was it !

