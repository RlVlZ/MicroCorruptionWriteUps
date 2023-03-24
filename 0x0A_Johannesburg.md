# Firt Thoughts

Starting with a quick look at the login function, it seems that we still have a buffer overflow available :

![Imgur](https://imgur.com/oUaCJ8P.png)

On this image we see, in 0x452c, that the **login** function is saving 18 bytes for its stack frame (to add 0xffee on a fixed-size 2 bytes adress will reduce it by 0x12 - kind of modulo 0xFFFF).  
However, the **getsn** function is called with a 0x3f size argument ! And after that a call to **strcpy** will copy the datas from the getsn buffer into the stack. As we already know, this copy will only stop because of a \x00 byte.  
Well let's try to input a 63 byte long password. This time let's be lazy and use python to generate this intput :

![Imgur](https://imgur.com/zOx9CAa.gif)

The program ends properly, no segmentation fault, meaning the overflow didn't worked. It would have been too easy, wouldn't it ?

# What is happening ?

First I'll place a breakpoint after the call to **strcpy** to see is the stack is smashed as expected or not. Well of course it is !

![Imgur](https://imgur.com/dxFoNxj.png)

We have clearly overwritten the return adress of the **login** function, so why don't we get an error ? Let's continue the execution and find out...   
Oh ok, after printing that the password was too long, the functions calls a **__stop_ProgExec__** without returning ! But looking at he program flow, we can see there is a possiblity to "jump over" this call (I like drawing arrows on assembly):

![Imgur](https://imgur.com/ziAmbkD.png)

Now if we look at what is making this jump happen, we see that whatever is written at adress [sp + 0x11] is compared to 0x48. If not equal : we fall into the **__stop__ProgExec__** call. Equals ? The function returns !  
Ok it make sens now : the password is supposed to be 16 bytes max. The stack frame is 18 bytes, and at the begining of the **login** function, it's "last" byte (sp + 0x11) is set to value 0x48. So if after the **strcpy** call the value 0x48 is no more, it means it have been overwritten by an overflowing password.  
Yeah, but while this last sentence is true, it's opposite is not : if the 0x48 value is still there, it does not imply that is has not been overwritten. Our payload might simply overwrite it with another 0x48 byte. Ok let's craft it.

# Exploitation

Once again, python is our friend. Let's create a payload containing 17 bytes of anything, then a 0x48, then the adress of the **unlock_door** function, so that the return statement will jump into it and open the lock :

![Imgur](https://imgur.com/SG5jBlL.gif)

And now let's copy-past this as an hexadecimal input aaaand... succes !

Easier than the previous level if you ask me.

