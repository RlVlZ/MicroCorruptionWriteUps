# First thoughts

The **main** function is kind of screaming "no overflow here !!" by saving 0xff9c bytes for its stack before calling the **login** function. Message received. Let's have a look at the login function then.
Ok I'm happy I started coding using C for I know what to expect from **malloc()** and **free()** functions.

![Imgur](https://imgur.com/JI49ZX4.png)

The **login** function flow is the following :

1. call **malloc** to get an adress with 10 bytes usable. The adress is stored in r10.
2. call **malloc** to get another adress with 10 bytes usable. That one is stored in r11.
3. Prompt the user, asking for a username and password, then wait for the username
4. store the username at address saved in r10, with a possible buffer overflow (0x30 vs 0x10)
5. Prompt the user for a username (looks like a dev. failed a copy-past here!)
6. store the password (...) at adress saved in r11, with another possible buffer overflow (0x30 vs 0x10)
7. Call the **test_password_valid** on the string saved in r11
8. If the function returned anything else than 0 in r15 : hurray ! Else : Nope.
9. No matter the result of step 8, call **free** on adress stored in r11, then r10, returns

Of course the overflows are interesting, but I feel like digging in the code. Still I'm a lazy ass, so I will not start by **malloc** but by **free**, which is much smaller !

# The free() function

