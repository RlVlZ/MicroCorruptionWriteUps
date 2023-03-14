# First Thoughts

The welcoming message says that the soft is no more vulnerable to overflows... Let's see if that's true.
First thing we see is that, event though the message still says that passwords can't be greater than 16 char, the **getsn** function is called with 0x30 as size argument. That's a potential 32 bytes overflow to play with !

![Imgur](https://imgur.com/vllYSUH.png)

Next we see that there is no more check made inside our "overflowable" addresses, so this will not be juste like the Hanoi challenge.

Well, let's see what is hapenning.

# Nominal test

The login function contains a call to **test_password_valid**, which seems of interest for us, so we will set a breakpoint there and proceed.

As usually we get prompted for a password. Let's try 0123456789abcdef, so we don't overflow anything.

The function **test_password_valid** seems OK, the call to the 0x7d interrupt is legit, I can't seem to see anything wrong here. Keep going.

After executing **test_password_valid** r15 is tested. Our password surely is wrong, so we get 0x0000 in the register, which lead to a jump at "that password is not correct", and then the program loops.

![Imgur](https://imgur.com/zYSF8za.png)

Ok ok, let's try to overwrite as much as possible using this badly parametered call to **getsn**.

# Stress test.

Now we restart the programm and give it something to fill those 0x30 bytes authorized for getsn.
It crashes, with the error "insn address unaligned". Smells good !

![Imgur](https://imgur.com/kABxDLg.png)

Let's restarts and try to understand what is happening. We will have a closer look at our stackpointer, being given that, at some point, a pop seems to get an overflowed area into the PC.

We noticed that before the problem of unalignement occured, we had been prompted with "That password is not correct", so let's go directly after that call to **puts**.

![Imgur](https://imgur.com/XolEPGO.png)

And from there we will go step by step, looking at SP... BINGO ! After our breakpoint, we see that SP is incremented by 0x10, going right into an overflowed area, and the next instruction is a return statement !

![Imgur](https://imgur.com/esWOu3p.png)

*Here in yellow you see the overflowed area, SP pointing right inside it*

Look at that ! One step more and boom ! PC contains 0x4242, leading the program to crash. Whatever will be in place of those 0x42 0x42 bytes will be interpreted as an adress and the program will try executing it.

# Exploitation

There is a function called **unlock_door** at address 4446. We will try to execute it. Our exploit will be :

`000102030405060708090a0b0c0d0e0f4644`

Don't forget to tick the option for hex input. Also, be aware that because of endianess the address's bytes must be switched. 
And BINGO ! Access Granted !
