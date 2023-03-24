# First Thoughts  

First thing we see is that the login function is pretty huge, so let's have a look at it with our most advanced tool : paint !

![Imgur](https://imgur.com/SwwwJNP.png)

Once the control flow and the function calls highligted the idea of the function is clear :  
1. getsn fill a buffer with our username
2. strcpy copies it into the stack
3. getsn fill a buffer with our password
4. strcpy copies it into the stack
5. a loop search for null bytes
6. a first test can lead to stop_ProgExec
7. a second test can lead to stop_ProgExec
8. a third test can give either "Password not correct" or unlock_door
9. a last test can lead to either a stop_ProgExec or a return

