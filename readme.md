
# Overview

This is my personal notes and walkthrough of OverTheWire's wargame "Bandit". Bandit was created to learn and practice command line and security concepts. I am using this to document my thoughts, methodologies, findings, and takeaways from each level of this game. It will not only strengthen my linux skills, but also my reconnaissance, security, and problem-solving skills as well.

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

- Issuing `ls` showed me another directory named *inhere*
- changing directory `cd` into *inhere* and issuing `ls` again revealed 20 directories, named *maybehere00* through *maybehere19*
- Issuing `ls` on a few of these directories I found that they all followed the same file pattern, containing 9 files:
	- -file1*
	- .file1*
	- -file2
	- .file2
	- -file3*
	- .file3*
	- spaces file1*
	- spaces file2
	- spaces file3*

- I did not want to have to manually go through all of these directories and files so I started researching a way to recursively search a directory, its subdirectories, and files.

- I ended up solving this level 2 different ways. The first way was of my own theory, and the second was after reading the level hints from OnTheWire and reading a different walkthrough.

#### Solution 1
- My theory for this solution was due to the fact that every password flag until this point were all a fixed length, 32 characters long. Using this I decided to research how to recursively search for strings of a fixed length.
- This led me to diving into the linux tools **find** and **awk**.
	- **find** is a way to search through the directory tree based on certain parameters given to it. I wanted to recursively print all files within all directories
	- **awk** is used for pattern scanning and processing. I wanted to pipe the output of the **find** command into **awk**, giving me the ability to print every file within each subdirectory and search for strings with a fixed length of 32 characters. 

	- `find . -type f -exec cat {} \; | awk 'length($0) == 32'`
		- **find .** - search starting from the current location
		- **-type f** - search through files
		- **-exec cat {}** - Executes the `cat` command on every file, addend filename to each

		- **awk 'length($0) == 32'** - searches each entry for a string of length 32

- This gave me only 1 output, a string of 32 characters that looked like the other passwords
- To find the subdirectory and filename that contained this string,  **grep** was used:
	- `grep -r "<password_string>" .`
		- **grep -r "string" .** - Search recursively for the string starting from current directory.

- This gave me only 1 output
	- ./maybehere07/.file2:<password_string>

#### Solution 2

- Once I solved the puzzle using Solution 1 I went to the official OverTheWire page for the level and realized that they gave me 3 criteria to use when searching for the file:
	- The file is human-readable
	- It is not an executable file
	- It has a size of 1033 bytes

- Reading the `man find` page, each of these requirements can be used as arguments using **find**
- `find -type f -readable ! -executable -size 1033c`
	- **find type f** - find files
	- **-readable** - file must be readable by the user
	- **! -executable** - '!' is used to signify NOT, not executable
	- **-size 1033c** - searches for files of size 1033, 'c' is used for bytes

- This gave me only one output:
	- ./inhere/maybehere07/.file2

- This is the same location I found during Solution 1. Just to double check, using `cat` on that file revealed the same password I found in Solution 1.


# Level 6 - Level 7

- This level was solved in a similar way to the last level, with a few more steps
- Issuing `ls` gives no files in the bandit6 home directory
- As per the OverTheWire documentation, the password is stored somewhere on the server and has the following properties:
	- owned by user bandit7
	- owned by group bandit6
	- 33 bytes in size

- These parameters led me to believe that using **find** was the appropriate method
- I knew that since the file was stored somewhere on the server I would be performing the **find** search from the top of the file tree in the root `/` directory. So I navigated there with:
	- `cd /`

- Once in the root directory, and after researching the **find** manual page, I was able to craft this command
	- `find . -size 33c -user bandit7 -group bandit6`

- This returned several dozen responses, all giving a "Permission Denied" warning, except for 1 file:
	- *./var/lib/dpkg/info/bandit7.password*

- Issuing `cat ./var/lib/dpkg/info/bandit7.password` I was able to extract the correct password
