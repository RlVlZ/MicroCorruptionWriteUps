# First thoughts
It seems like the password we type is given to the aes-ecb-decrypt function, along with the string "\x7fxuw\x0c^\x19     ".
Looks like our password will be decyphered and compared to the string "ACCESS GRANTED!\x00".
Now the idea would be to manage to cypher "ACCESS GRANTED!\x00" with the same algo and the same key (fortunately the algo is symetrical), and give the result as input when promted.
