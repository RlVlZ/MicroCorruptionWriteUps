# First Thoughts

We see the usual structure : **main** function call **login** function, which display some strings and ask for a password.  
Once again we see that **login** function is saving 0xf bytes of memory for local variables, but **getsn** function will scan up to 0x30 bytes. Although the adress given to **getsn** si 0x2400, which is not in the **login** allocated memory, **login** will need this password to give it to **conditional_unlock_door**. I guess graping the password from 2400 adress is done by the call to **strcpy**...  
So at first glance it looks like our previous exploit might work... I want to give it a try.
