# First Thoughts

We see the usual structure : **main** function call **login** function, which display some strings and ask for a password.  
Once again we see that **login** function is saving 0xf bytes of memory for local variables, but **getsn** function will scan up to 0x30 bytes. Although the adress given to **getsn** si 0x2400, which is not in the **login** allocated memory, **login** will need this password to give it to **conditional_unlock_door**. I guess graping the password from 2400 adress is done by the call to **strcpy**...  
So at first glance it looks like our previous exploit might work... I want to give it a try.

# Trying the exploit from WhiteHorse

So let's run the program without breackpoint, it will just stop at the password prompt. Then I enter `00000000000000000000000000000000c64230127f00b0123245` as an hex encoded input. This is the exploit from WhiteHorse. Of course the returning adress will propably not make sens but this is a quick test. Now we can go step by step. When returning in **login** from the getsn function, we can see that our exploit is present at adress 0x2400 and that the next instructions seem to call the function **strcpy** with two parameters : 0x2400, and the stackpointer. So after this call, we souhld see our exploit copied at the adress pointed by SP. Lets run through the **strcpy** call.

![Imgur](https://imgur.com/mOXGEdr.png)  
*A call to strcpy with two adresses as arguments : from and to*

![Imgur](https://imgur.com/3cBmyoK.png)  
*We see the end of the exploit at adress 0x2410, and our stack pointer with 0xf bytes reserved before the return adress*

But after the **strcpy** nothing have changed on the stack ! Still 16 zero bytes before the return adress... After a few try it eventually made sens : the **strcpy** function does not takes a "lenght" argument, and it is working with string... So it stop copying while it does not encouter a null byte. And my exploit was starting with a null byte ! But... wait : I also have nullbytes in my "payload" part ! Like in the `push 0x7f` : it's converted into a 2bytes values !

![Imgur](https://imgur.com/t6XRUQ1.png)

Ok but let's keep it cool : this is preventing us from copying the exploit in the **login** memory area, but our exploit should still lay in aroud the 2400 adress. So we can use a non-zero padding to overwrite the return adress, making it point to the exploit at adress 2410 ! Let's thy this !

# Trying to execute code from adress 2410

So we need a 16 bytes non-nul bytes of padding : I'll go for `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa`
Then we want the adress where the **login** function will return, with bigendianess, so : `1224` (not 1024 : this is the adress of the adress, we want the adress of the exploit, so the one right after)
And finally our exploit, as usually the opcode to push 0x7f and then call the **INT** : `30127f00b0124c45`
We don't care about the 0x00 in this opcode, because the return will go directly to its adress, not to the copied one.

Now let's try it. At first everthing seems to work properly, and after the **strcpy** function we see the return adress pointing to our exploit !

![Imgur](https://imgur.com/GcZ33xg.png)

We keep going untill the return statement and then... PC is pointing to adress 0x2412 but... The adress seems empty and the program crashes. What ??

Oh no, a call to **memset** is erasing the area ! This memory chunk will always be erased when **login** will return. Meaning we cannot use this part of memory.  
Ok so if we want to execute malicious code, it must be from the area where the buffer is copied, and it must contains no null-byte... Arf.

# Making a payload without null-byte

So we need a payload without null-byte, and pushing 0x7f before calling INT. My idea is to *compute* the value 0x007F in a register, and then push this register. To do so we will use some binary logic. I'll go with a AND statement, because it is easy to manipulate in assembly (I already saw some AND in other challenges), and easy to understand. 
In binary, 0x00FF will look like that :  
 0    0    f    f
0000 0000 0111 1111
So I want two values, none of them containing a null-byte, that can be ANDed to get this. I'll use the following :
0101 0101 0111 1111
1010 1010 1111 1111

This gives us 557f and aaff. So I want to move one of this value into a register, let's say r10, and it with the other value, and push the result on the stack before calling our interrupt. Let assemble those instructions :

![Imgur](https://imgur.com/VllFSnx.png)

Great ! The result has no null byte in it ! And now the payload will be something like that :  
Padding : aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa  
Address : 0044  
Exploit : 3a407f553af0ffaa0a12b0124c45  

No wait ! The address to overwrite the return, it contains a 00 ! No problem, let's return a bit further and fill the difference : 
Padding : aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa  
Address : 0244  
Filling : aaaa  
Exploit : 3a407f553af0ffaa0a12b0124c45  

So here is our payload : `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa0244aaaa3a407f553af0ffaa0a12b0124c45`

Let's try it and... Success !

# A quick overview of how it's working
