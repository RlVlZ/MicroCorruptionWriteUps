# First Thoughts

Going through the main function, the binary seems quite classic : 
1. print "Enter the password to continue"
2. call to get_password
3. call to check_password
4. jump to Acces Granted if check_password have set r15 to non-zero value
5. else "Invalid password, try again"

So let's have a look at check_password

# check_password

Here it is pretty straight forward, password is checked by chunks of 2 bytes :

![image](https://user-images.githubusercontent.com/17447180/224369557-3d144a52-dddb-4ae6-8e55-85eddef0a763.png)

Each faild check will lead to erasing r14 before puting its value in r15 and returning, meaning the checkpassword will return 0.
Before the last check r14 is set to 1, and if the check succeed we go directly to the last 2 instructions : set r15 to r13 and ret. The function will return 1.

Now when trying to enter the password remember of the endianess : the password is checked for 7a24 3e63 6d30 3a4d.
However you'll have to enter 247a 633e 306d 4d3a

# Solution

Tick the check for hex data and enter : 
247a 633e 306d 4d3a
