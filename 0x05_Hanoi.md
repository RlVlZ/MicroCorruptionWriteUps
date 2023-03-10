# First Thougths

Looking at the login function (which is directly called by main), we can quickly see something interesting. It is prompted that the pswd must have a size between 8 and 16 char, but we see that the getsn function is given 0x1c as size argument...
This mean we will be able to write 12 extra bytes ! Possible overflow here...

![Image1](https://imgur.com/jCmOqyO)

# Let's see if the overflow hypothesis is correct

Now what is doing the login function ?
After the getsn we see a call to test_password_valid, after what r15 (the return value) is evaluated. If 0, we go to 0x4552, displaying "Testing if the password is valid". Then...
Then... The value 0xf2 is compared with whatever is stored at @ 0x2410 ! If equals, we keep running untill a call to unlock_door  ! 0x2400 is in the range of our overflow ! Ok let's try this !

![Image2](https://imgur.com/dPlaBTS)

# Testing the overflow

So, we can write at @0x2400, and we want 0xf2 at 0x2410. Lets try the input 00000000000000000000000000000000f2...
BINGO !

That was an easy one.
