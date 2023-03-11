# First Thoughts

At first glance this one seems pretty tough. The main function is quiet short, and calls some **enc** function before to make a call to a raw address : #0x2400.  
I guess we will first have a look at the **enc** function, but maybe we'll need to disassemble whatever lays at address 2400...

# The enc function
  
Ok this is a lot of assembly.
  
![Imgur](https://imgur.com/2NiqXVe.png)

Let's try to understand whats happening there, although I would like to do other things today...
  
  ---
Quick solution : in the code past 0x2400 there is the call to puts with "what's the password", the call to getsn also and then the first adress where what we tipped is stored is compared to a 2bytes value.
Solution : enter this 2byte value as hex input, carefull with the endianess !
