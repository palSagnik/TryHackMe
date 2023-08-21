## Description
Fall down the rabbit hole and enter wonderland.

## 1. Reconnaissance


<img width="1240" alt="Screenshot 2023-08-19 at 3 07 30 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/8017d99a-fc98-4f82-8635-84c86c5e88b3">






I tried wget and looking through some images on the website but it gave me no results.

<img width="1226" alt="Screenshot 2023-08-19 at 3 08 17 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/97732241-93c5-4fa7-856f-06bfb384d36c">



Now, I thought running gobuster was the best option.


<img width="1254" alt="Screenshot 2023-08-19 at 3 07 44 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/e5c0ffb5-e69c-407c-a92c-240cd3b03ad2">


We find that this website has hidden directories like, /img and /r. 
By some stroke of luck I thought to run gobuster in the /r directory, this was after thinking that as it Alice there must be 
some rabbit:) and somehow it worked.





<img width="1252" alt="Screenshot 2023-08-19 at 3 08 40 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/c752a93b-2f35-4ccd-b417-19d775bb14c3">






<img width="1262" alt="Screenshot 2023-08-19 at 3 09 07 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/0b3e5c01-d6cb-40a4-a57a-1e32b28bb573">






Still after running gobuster I find the /a directory and running gobuster on it again I find the /b directory. After this 
mental gymnastics I got to know that a tool named feroxbuster would have made my work easier:/.




<img width="1224" alt="Screenshot 2023-08-19 at 3 09 22 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/c456fe13-62f8-4904-b19e-567d4331ddd5">





## 2. SSH? ALICE? 

Now, after I reached the /r/a/b/b/i/t directory I directly check the source code which gives me the creds of alice for 
SSH login.

<img width="1242" alt="Screenshot 2023-08-19 at 3 12 59 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/43d5d07b-ef7f-4c0b-9d50-b2947c769efd">



Now that we are alice, we find 2 files in the /home/alice directory one of them is a python script while the other is 
a root.txt file which can only be read by root.

Running the python file we find that it imports the random module and then randomly extracts 10 lines from a poem 
given in the text itself and prints it.

<img width="1239" alt="Screenshot 2023-08-19 at 3 13 25 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/5ea48f59-3fe8-4cf1-b620-b3d9ceb50564">

Now I also check the sudo privileges which alice has


<img width="1257" alt="Screenshot 2023-08-19 at 3 29 31 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/7d5d9100-0ad1-4698-b701-07cbb8ef4a69">


The sudo privileges is a bit unique which leads to the idea of inheritance of sudo privileges by child processes. We find the random module being imported and we also 
know that while import modules which are essentially python files, python first checks the current directory and then checks in other places.

Then this would be my attack vector. Only because we have no absolute path, so we make a fake "random" library in the pwd.


I make a random.py file and write the following code in it.

```
import os
os.system("/bin/bash")
```
And voila we have a bash shell under rabbit.

Now moving to the /home/rabbit directory we find that there is binary with SUID bit set on it. But on running it we find that whatver we type we get a segmentation
fault. Decompiling the code with ghidra and opening it in VS code, we find this


<img width="576" alt="Screenshot 2023-08-21 at 6 46 33 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/dcba804e-388e-4dcd-bb76-4037e5698b84">


The binary sets the suid and gid of 1003, then the most interesting line is where it uses the date binary without any absolute PATH. This could be a good exploitation
point.

Hence we proceed somewhat like this

<img width="1259" alt="Screenshot 2023-08-19 at 4 58 02 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/53c9f291-0521-4035-9e01-2626e8205f83">

The reason we put our path at the leftmost end of the PATH variable is because the PATH variable searches for the path of said binary left to right in the path
variable.

And the script in the fake "date" binary is 

```
#!/bin/bash
/bin/bash
```

This should open a shell of uid and gid 1003.

we find that this is hatter, after going to /home/hatter, we see that there is a password.txt file and this mostly is hatter's ssh creds.


<img width="1196" alt="Screenshot 2023-08-19 at 5 33 55 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/fb0f05a3-a686-49aa-9e0d-d596b36a6367">


and we log in again using hatter's creds.

## 3. What else is left?


Now that we cannot use sudo, so we use a different command `getcap` that list the capabilities a certain file has.
We now find that perl has the unusual capabality od setting uids, the next step is just to fall on GTFObin's feet and pray it has something related.

And we are root.


## Conclusion

This box was a different ball game altogether, so many things to learn and so many points of exploitation.
