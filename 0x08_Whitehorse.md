# First Thoughts


Once again we see that function getsn is called with a 0x30 size, however the login function has reserved only 0xf bytes on the stack for its local variable.  
This means that the return of getsn will allow us to overwrite the return adress of the login function. However we have no more access to the **unlock_door** function.  
The question is now : what to do with this overflow ?

# Making our own unlock_door

Ok, so we don't have access to the **unlock_door** function. But what if we could code a similar function, and jump in it ? We are able to write *past* the return address !
Ultimately, the function is calling a #0x7F interrupt to open the door, right ? So here is the plan :  
1. Overwrite the return adress so it points to the following address
2. Populate the following address with opcode calling a 0x7f interrupt
3. let the magic happen

# Building the payload

We know the the **login** function has 0x0f bytes allocated on the stack, because of the following statement :

![Imgur](https://imgur.com/kigedXh.png)

And we already said that the getsn function will allow us to return, *in the stack*, up to 0x30 bytes ! So let's find the return address and the value we want to overwrite it to.  
When breaking in the login function, we see the stackpointer it set at address 0x42c4, containing the adress 0x443c. This adress is the adress following the end of the **main** function, So indeed the return adress for **login**.
Ok so we want to overwrite the stack up untill this adress, and give the value 42c6. Why ? Because this value is the one of the following adress, which we are able to overwrite as well (we olny used 0x0f bytes out of 0x30)!

So far our payload is as follow : 16 bytes of padding + c642 (endianess !). Now lets craft a call to the 0x7f interrupt, using the disassembly tool :

![Imgur](https://imgur.com/F2SHYg3.png)

Now we add this opt code to our payload and get : `00000000000000000000000000000000c64230127f00b0123245`. We enter it with the option for hex data ticked and tada !

Just to see what happen, here is the state of the stack before we enter our corrupted password (right before the call to getsn):

![Imgur](https://imgur.com/nZUUMFd.png)

We clearly see the 16 bytes allocated between the adress pointed by the StackPointer and the return address (0x443c). And now here is the stack after the getsn returned :

![Imgur](https://imgur.com/hhJ91Ta.png)

Now we see the return adress is no more 443c but 426c, followed by our malicious opcode. Lets step a bit. We go through **conditional_unlock_door**, through the **puts** call informing us that "that password is not correct", and then we hit the return condition.  
Have a look at the stack pointer now : 

![Imgur](https://imgur.com/oKGsFRe.png)

The stackpointer is on the new return adress... One more step and tada !

![Imgur](https://imgur.com/z5KQE9r.png)

Note that it is the Program Counter we are seeing now, pointing at our malicious opcode ! 

And this is how we get that poor program to unlock the door for us.
