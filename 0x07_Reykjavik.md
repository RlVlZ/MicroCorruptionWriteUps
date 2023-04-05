# First Thoughts

At first glance this one seems pretty tough. The main function is quiet short, and calls some **enc** function before to make a call to a raw address : #0x2400.  
I guess we will first have a look at the **enc** function, but maybe we'll need to disassemble whatever lays at address 2400...

# The enc function
  
Ok this is a lot of assembly.
  
![Imgur](https://imgur.com/2NiqXVe.png)

Nothing interesting here at first glance : no call to any interrupt or whatever. Yeurk, let's have a look at whatever lays at adress 0x2400.

# The 0x2400 memory area after enc

So let's firts copy those bytes and format it so we can feed it to the disassembler. It's quite quick using VIM, its "Visual Bloc" mode and some "J" instructions. After giving it to the dissassembler we get something like that :

![Imgur](https://imgur.com/ZB7f38u.png)

There is a lot going on here, we can easilly find ret instructions and calls going into the current area... I'm pretty sure some of those functions could be compared to what we are used to in other challenges (puts, getsn etc.) to get a better understanding of what is going on here. But lazzy as I am, I will first try something else. In the documentation we learn that the unclocking of the door is made through an interrupt called with the parameter "0x7f" pushed on the stack. Is there any "push #0x7f" instruction over there ? Yes !

![Imgur](https://imgur.com/xlTH7Im.png)

This seems promissing : 0x7f is pushed just before a call ! What are the conditions to get here ? Seems like something is compared to the raw value 0x86c... It could be the result of some computation made on our password but let's first try it as input...

BINGO ! We just need to think about the endianess and that was it : 0x6c08
