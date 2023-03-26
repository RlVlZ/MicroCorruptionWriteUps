# First Thoughts  

First thing we see is that the login function is pretty huge, so let's have a look at it with our most advanced tool : paint !

![Imgur](https://imgur.com/SwwwJNP.png)

Once the control flow and the function calls highligted the idea of the function is clear :  
1. getsn fills a buffer with our username
2. strcpy copies it into the stack
3. getsn fills a buffer with our password
4. strcpy copies it into the stack
5. a loop search for null bytes
6. a first test can lead to stop_ProgExec
7. a second test can lead to stop_ProgExec
8. a third test can give either "Password not correct" or unlock_door
9. a last test can lead to either a stop_ProgExec or a return

One again we spot that the **getsn** function gives us up to 0x63 (i.e. 99) bytes for the user name **AND** the password. This and the unfortunate use of **strcpy** is leading us toward some stack overflow exploitation... After testing some way too long usernames and passwor, it is clear the tests we saw before are trying to prevent us from overflowing the stack. So let's see what those test are really about.

# So much tests

## \#The while loop
Looking at that loop we see it is increasing the r14 register at every iteration. This r14 register is intially filled with the value from r4 register, before being added 0xffe8. A closer look to r4 shows us it is used somehow like a base_pointer, pointing to the return adress of the **loggin** function : 

![Imgur](https://imgur.com/WuNuMTS.png)  

On the previous image we see that after pushing 2 registers (r11 and r4) on the stack, the **login** function is  moving sp into r4 before adding 0x4 to the register. This register is then pointing to sp + 0x4, those 4 bytes matching the length of the 2 registers intially pushed : r4 is pointing toward the return adress. Then the stack frame is created by substracting 40 bytes to it (0xffff - 0xffd8 + 1) .

Great. Now we know that, in our loop, r14 is pointing to an adress calculted based on the lowest adress of the stack. This adress is being added 0xffe8, which is equivalent to a substraction of 0x18 (24 bytes). So in short : this loop is iterating from address [base_pointer - 24 bytes] and stops whenever it encounter a null-byte. This look like we are looking for the end of a string. What could be there ?  
The start adress of the loop is almost equal to the adress given to the **strcpy** function at instruction 0x45c2 ! This is the call related to the password.  
We can now be pretty confident that the loop is looking for the end of the password.

## \#First test : is the password too long ?

After the loop, we see its result being stored in r11, then whatever is stored in r15 is substracted to r11. A quick look at the previous instructions show us r15 is storing the start adress of our password in the stack. So after this substraction r11 is storing the length, in bytes, of our password. Finally some value from adress [r4 - 0x18] is stored in r15 and a comparision is made betweent r11 and r15. Then we have a **jnc** instruction. This is a conditionnal jump checking the carry flag. Short story-long : if our length is greater or equel to the value in r15, we fall into the **stop_ProgExec**. Ok so this is testing the max size of our password. Seems like the adress storing the max value is in the stack, so possibly overwritable.

## \#Second test : is the password too short ?

We've done all the work for the first tests : this one is making a comparision between the password size and a value stored in [r4 - 0x19]. The jump is now a **jc** : if our length is lower or equal to the value from this adress, we get a **stop_ProgExec**. We are testing the min size of our password.

## \#Third tests : is the password correct ?

This one can be understood by looking a few lines before the jump : we see a call to the 7d interrupt, which is supposed to check the password. This is confirmed by the code being executed if the jump is not made : a call to the **unlock_door** function. We shouldn't worry about this test as our idea is to use overflow to change the return pointer, not to find the correct password.

## \#Fifth tests : is the password too long, the come back !

Here we are testing if the value present 0x6 bytes before r4 (the base pointer) is zero. If so, the function will return normally, if not, we get a **stop_ProgExec**. This byte is after the adress of our password + 16bytes, so the check is preventing us from overwritting the return adress. The idea is that to overwrite the return adress we need a bunch of non-null bytes, in order for the **strcpy** function to not stop too soon. But then, this test will prevent the login function from returning, making our overwritting useless... Damn. Ok anyway, let's play with our two possible overflows and see what we can find.

# StackOverflows untill having an idea

After a few overflows performed on both the password and the username, here is what I get from the stack structure :
1. First there is the Username (in green in the next image)
2. Then there are two bytes for min and max password length (in purple)
3. The password (in orange)
4. The byte that must be set to 0 in order to get a return (in white)
5. The return (in red)

![Imgur](https://imgur.com/9XsqJe8.png)

It is possible to use a username overflow to overwrite the min and max values used for testing the password. It is then possible to use a password overflow to overwrite the return adress, but then the last check is preventing us from using the overwritten address as a return adress... This took me a bit too much time to figure out but here is the idea : the username overflow can also overwrite the return adress ! This allows us to use the password overflow to set the annoying byte to 0 ! I have noticed that the **strcpy** function is adding a null-byte at the end of our strings, so this will do the trick for us !

# Building the payload

The most important part of our payload is the username : it needs to overwrite the min and max value correctly, and then to goes up to the return adress. As usually we will return into the **unlock_door** function, at adress 0x444a. For the min and max value I've chosen the values 0x01 and 0xff to be comfortable ^^. This gives us a first padding of 17 bytes to reach min and max, then a second padding of 23 bytes untill the return value :

![Imgur](https://imgur.com/H7kjUAM.png)

And now the password, whose only purpose is to set the byte at adress [r4 - 0x6] to 0x00 :

![Imgur](https://imgur.com/WuTP3uL.png)

No need for the null-byte : the **strcpy** function will add it for us.

# Exploitation

After entering our username payload, here is the state of the stack :

![Imgur](https://imgur.com/qS4nQUz.png)

We can see the min and max value at 0x01 and 0xFF, and our return adress is now the adress of the **unlock_door** function. However the 6th byte of the stack -from the base) is not at 0x00.
Now let's see the state of the stack after entering our password payload :

![Imgur](https://imgur.com/PN29IoL.png)

To make it more visual I used a 'bb' filled password. I make visible that our password overwrite a part of our username, just enought to make the **strcpy** function add a null-byte where needed. And now all conditions are set : the password is neither too short nor too long (0xff : we had space !), the 6th byte of the stack is set to zero and the **loggin** function is returning to the **unlock_door** function !

Lock unlocked !



