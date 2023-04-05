After what I endured on the Addis Ababa level, that one was reaaaaly easy. Solved in maybe 5 minutes. However I understood wome of the trouble I had in Addis Ababa so it was worth the shot.

# First Thoughts

I started by having a look at the **main** function. It begins just like in Addis Abbaba : a call to **printf** prompting `Enter your username below to authenticate.`, and another call to **printf** that displays a `>>`.  
Then someting interesting : the **getsn** function is called with 0x1f4 for the size argument ! I immediatly checked for the stack size : 

![Imgur](https://imgur.com/T3zIKit.png)

Erf. This is a huge stack. No overflow unless the **strcpy** is staring at a wrong adress... And it's not. When **strcpy** is called, r15 (containing its destination adress) and sp are equals :

![Imgur](https://imgur.com/WpRYCIz.png)

I also made sure there is nothing to overflow around the 0x2400 area where the **getsn** function will store our password. Anyway, there is, once again, a **printf** to diplay our password back. So, just like in Addis Ababa, we can use this function to modify a word in memory with the lenght of our password. And a huge amount of space for the password means a wider range of values writables !
At this point I alreay have an idea of what I could do : change the value used to call the **<INT>** from 0x007e to 0x007f. Let's try this !    
  
# First try

I decided to go just like in Addis Ababa except that I needed a precise size, meaning a padding. I knew (at least I thought I knew) from Addis Ababa that I needed a first "%" followed by a random char before my %n. So the idea was : [2 bytes of address + 125 bytes of padding + 25aa256e]. 2 + 125 = 127 = 0x7f.
The adress to overwrite is the one setting the interrupt in **conditional_unlock_door**: 
  
  
  
  
  
