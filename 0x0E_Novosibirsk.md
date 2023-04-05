After what I endured on the Addis Ababa level, that one was reaaaaly easy. Solved in maybe 5 minutes. However I understood some of the trouble I had in Addis Ababa so it was worth the shot.

# First Thoughts

I started by having a look at the **main** function. It begins just like in Addis Abbaba : a call to **printf** prompting `Enter your username below to authenticate.`, and another call to **printf** that displays a `>>`.  
Then someting interesting : the **getsn** function is called with 0x1f4 for the size argument ! I immediatly checked for the stack size : 

![Imgur](https://imgur.com/T3zIKit.png)

Erf. This is a huge stack. No overflow unless the **strcpy** is staring at a wrong adress... And it's not. When **strcpy** is called, r15 (containing its destination adress) and sp are equals :

![Imgur](https://imgur.com/WpRYCIz.png)

I also made sure there is nothing to overflow around the 0x2400 area where the **getsn** function will store our password. Anyway, there is, once again, a **printf** to diplay our password back. So, just like in Addis Ababa, we can use this function to modify a word in memory with the lenght of our password. And a huge amount of space for the password means a wider range of values writables !
At this point I alreay had an idea of what I could do : change the value used to call the **INT** from 0x007e to 0x007f. Let's try this !    
  
# First try

I decided to go just like in Addis Ababa except that I needed a precise size, meaning a padding. I knew (at least I thought I knew) from Addis Ababa that I needed a first "%" followed by a random char before my %n. So the idea was :  

`2 bytes of address + 125 bytes of padding + 25aa256e]`. 

2 + 125 = 127 = 0x7f. The adress to overwrite is the one setting the interrupt in **conditional_unlock_door**: 
  
![Imgur](https://imgur.com/vSPK1Bh.png)

We see the instruction at adress 0x44c6 : `3012 7e00`, corresponding to the push #0x7d. So we want to overwrite the adress 0x44c8, corresponding to the value 0x7e.  Let's go :

![Imgur](https://imgur.com/lVSdcy2.png)

This fails and the program ends without error. Damn. Let's try to une "99" as padding... Ah ! 
  
![Imgur](https://imgur.com/g32bjCJ.png)

Ok the overwritting *is* happening, but not at the adress we placed first in our payload. A quick debbug was enought to understand that, unlike in Addis Ababa, there was no null-word copied in the stack by **printf**, meaning no need for the first "25aa". 

# Second try
  
Ok so let's try it without this "25aa" in the payload !

![Imgur](https://imgur.com/AvNtYKK.png)
  
And... Succes !
  
# What about that null-word in Addis Ababa ?
  
Ok this level was very easy compared to the previous one, but I was a bit confused with this null word thing, so I went back in Addis Ababa to have a closer look. Turned out that this null-word was not a **printf** thing, but rather a **main** thing :

![Imgur](https://imgur.com/OH1Qvxn.png)

In this screen shot of Addis Ababa main function, we see that, before the **strcpy** is called to copy the password from buffer at adress 0x2400, SP is copied into r11, which is incd before being moved into r15 ! This is why our password is copied a word further than the stack pointer, and it was to correct this behavior that we had to add a "% + whatever" to our payload. As seen when looking for overflow, in the current level, when **strcpy** is called, sp and r15 are equals. That was not the case in Addis Ababa, as shown in the following screenshot :
  
![Imgur](https://imgur.com/hB7SORa.png)

Mystery solved !
  
