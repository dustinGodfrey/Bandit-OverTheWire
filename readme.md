
# Overview

This is my personal notes and walkthrough of OverTheWire's wargame "Bandit". I am using this to document my thoughts, methodologies, findings, and takeaways from each level of this game. It will not only strengthen my linux skills, but also my reconnaissance and problem-solving skills as well.

I will be updating the repo as I continue on through the levels.

>[!Important] As per the request of OverTheWire I will not be publishing credentials from any level. This is purely for a learning journey.


# Level 0

- Login to the bandit server over port 2220.

`ssh bandit0@bandit.labs.overthewire.org -p 2220`


# Level 0 - Level 1

- The first command I always run within a machine like this is `ls`. It gives a quick overview of what we are dealing with and what might be of interest to us.

`ls`

- This shows one file on the machine, a readme file.

`cat readme`

- This shows the password for the next level.


# Level 1 - Level 2

 - I start this level the same as the last by running the `ls` command to see what it is the current directory.

`ls`

- This showed me that there was only one file in the directory, a file with only a dash (-) as the name of the file. 
- Running all the usual commands hangs or throws errors:
	- `cat -`
	- `open -`
	- `file -`
	- `ls -`
	- `cd -` : this throws an error 'OLDPWD not set', when researching this means that linux interprets `cd -` as change to the previous directory, so this would not work in this case.
	- `ls -lah`: shows that the file can only be read and executed by bandit2, and only members of the group bandit1 can read the file. This means we should be able to view the file because running `whoami` confirms we are user "bandit1"

- Using `pwd` to get the directory path '/home/bandit1' I was able to run:
	- `file /home/bandit1/-` which finally gave me some output:
		- /home/bandit1/-: ASCII text

- Using this I decided to try the `cat` command again using the same logic, this led to the password flag being shown:
	- `cat /home/bandit1/-`


# Level 2 - Level 3

- This one solved the same way as the last one
- Issuing `ls` shows us a file with the name *--spaces in this filename--*
- Using `cat ` and the full path including escape characters for the spaces I was able to extract the password:
	- `cat /home/bandit2/--spaces\ in\ the\ filename--`


# Level 3 - Level 4

- Issuing `ls` shows a directory named *inhere*
- Moving into the directory with `cd` and issuing `ls` again shows nothing
- This led me to believe that there may be a hidden file, so I issued `ls -la` so show all files, including hidden files
- This showed one hidden file named *...Hiding-From-You*
- Using `cat` on this file I was able to extract the password:
	- `cat ./...Hiding-From-You`


# Level 4 - Level 5

- Issuing `ls` shows a directory named *inhere*
- Moving into the directory with `cd` and issuing `ls` again shows 10 files:
	- *-file00*
	- *-file01*
	- *-file02*
	- *-file03*
	- *-file04*
	- *-file05*
	- *-file06*
	- *-file07*
	- *-file08*
	- *-file09*

- My first thought was to cat out each file, but I wanted to see if I could do it more efficiently.
- I started out by issuing:
	- `cat /home/bandit4/inhere/-file*`
- This showed me a wall of gibberish but I was able to see the password string amongst the noise.
- I wanted to attempt to pull out just that file so I could know the name, but wanted to again do it more efficiently than using `cat`on each file. After some research I learned that I can run `tail -n +1 <filenames>` and it would print the name of each file with the contents of the file under it, going through ever file.
	- `tail` is used to show the last n lines of a file.
	- When paired with +1 it will print the entire file, starting at the beginning
	- Per the `man tail` page:
		- "If more than a single file is specified, or if the -v option is used, each file is preceded by a header consisting of the string ==> XXX <== where XXX is the name of the file."

- Issuing `tail -n +1 /home/bandit4/inhere/-file*` shows the extracted password, located in the file named *-file07*


# Level 5 - Level 6

- 