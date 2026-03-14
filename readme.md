
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


# Level 7 - Level 8

- Issuing `ls` shows only one file, *data.txt*
- Issuing `cat data.txt | wc -l` shows that we have 98,567 lines in this file
- Per the OverTheWire documentation, this level's password is located next to the word "millionth"
- Knowing that **grep** can be used to locate lines based on a keyword, I issued:
	- `grep millionth data.txt` to search for any lines containing "millionth" in the file *data.txt*

- This returned only one line:
	- *millionth    <password_string>*


# Level 8 - Level 9

- Issuing `ls` shows one file, *data.txt*
- Issuing `cat data.txt | wc -l` shows that we have 1001 lines in this file
- Per the OverTheWire documentation, this level's password is the only line of text in the file that occurs only one time

- Researching finding unique lines in a file led me to the tool **uniq**, which will report or omit repeated lines
- At the bottom of the `man uniq` page there is a line that states:
	- "'uniq' does not detect repeated lines unless they are adjacent.
       You may want to sort the input first..."

- Reading this I researched how to sort lines in a file and was led to the tool **sort**

- I was able to sort the list, then pipe that into **uniq**:
	- `sort data.txt | uniq -c`
		- **-c** argument is used to prefix lines by the number of occurrences

- This gave me a list of strings, all prefixed by the number 10, except for one line that was prefixed with the number 1; this is our password string


# Level 9 - Level 10

- Issuing `ls` shows one file, *data.txt*
- Per the OverTheWire documentation, the password is stored in the file as one of the few human-readable strings, preceded by several '=' characters
- Issuing `cat data.txt | grep '=='` returns an error:
	- *grep: (standard input): binary file matches*

- Researching this led me to learn that **grep** does not automatically search binary files, but it will with the tag **-a**
- Issuing `cat data.txt | grep -a '=='` showed me less binary, but still a wall of binary. Only now I could see a few strings including one that looked like the password.

- Wanting to extract just these strings I researched "human-readable strings linux" which led me to the tool **strings**, which prints the sequences of printable characters in files
- Issuing `strings data.txt` returned a list of strings, mostly garbage data, with that same password string embedded into the list
- I decided to pipe this output into **grep** to see if I could pull out any more information

- Issuing `strings data.txt | grep '=='` returned 4 lines:
	- *========== the*
	- *========== password*
	- *========== is*
	- *========== <password_string>*


# Level 10 - Level 11

- Issuing `ls` shows one file, *data.txt*
- `cat data.txt` shows a long string of characters, longer than our password flag usually has
- According to OverTheWire documentation, this file contains base64 data. This can be confirmed with the above `cat` command, the output ends in `==`, which is padding for base64 encoded strings.

- `man base64` explains how to decode base64 strings with the tag **-d** 
- Issuing `base64 -d data.txt` reveals the string:
	- *The password is <password_string>*


# Level 11 - Level 12

- Issuing `ls` shows one file, *data.txt*
- According to OverTheWire documentation, every character in the string, both uppercase and lowercase, needs to shift 13 positions.
- A 13-position character shift is a cipher known as ROT13, a type of Caesar Cipher
- Researching how to shift characters in a string led me to the tool **tr**, which is a tool for translating or deleting characters based on specific arrays given as arguments.

- Per the `man tr` page and also the `info tr` page, each array string should be a range of characters. I assumed I could just shift everything over 13 easily with the following command:
	- `cat data.txt | tr [a-z] [n-a]`
		- This did not work as **tr** does not accept a range that wraps back around the alphabet.
		- I tried many combinations of this command, some getting closer than others, but none fully translating the string. The closest result I received was:
			- *G]] p]sswrd] ]s <password_string>*
				- Not the expected output.
		- No matter how many combinations I tried, nothing would work 100%.
		- This led me to search through more documentation, eventually leading me back to the `info tr` page, but at the bottom of the page, away from the explanation of ranges and substitutions, and was only a single line that gave away what I needed to do:
			- `tr -cs A-Za-z0-9 '\012'`
				- This showed me that each range array string could be a series of ranges in one, not just one range per string.
				- Using this, and more trial and error, I was able to finally craft a successful command.
					- `cat data.txt | tr [A-Za-z] [N-ZA-Mn-za-m]`
						- The first string `[A-Za-z]` means "replace every uppercase letter A-Z and every lowercase letter a-z"
						- The second string is what to replace string 1 with [N-ZA-Mn-za-m]
							- Since **tr** does not take wrapping, you need to manually set the wrap yourself, so N-Z then adding A-M will get back around to the original place in the alphabet; A-Z = 26, N-Z(13) + A-M(13) = 26
						- The result of the above command was:
							- *The password is <password_string>*
 

# Level 12 - Level 13

- Instructions from OverTheWire
> The password for the next level is stored in the file *data.txt*, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use `mkdir` with a hard to guess directory name. Or better, use the command `mktemp -d`. Then copy the datafile using cp, and rename it using mv (read the manpages!)

- Issuing `ls` I can confirm there is a file titled *data.txt*
- Issuing `cat data.txt` I can confirm that this is a hexdump
- Per the instructions I created a directory in the */tmp* directory with `mktemp -d` which prints the directory name. I copy the file and move into the directory
	- `cp data.txt /tmp/tmp.NxbNDs4Duq`
	- `cd /tmp/tmp.NxbNDs4Duq`

- Researching how to reverse a hexdump I found the tool **xxd**
- Per the `man xxd` page I can revert a hexdump with a `-r` flag.
- `xxd -r data.txt data2.txt`
	- This will revert the hexdump of *data.txt* and output the results to *data2.txt*

- Issuing `file data2.txt` reveals that this is a gzip compressed file

- Per the `man gzip` page I can see that you unzip a gzip file and send output to new file with:
	- `gunzip -c data2.txt > data3.txt`

- Issuing `file data3.txt` reveals that this is a bzip2 compressed file

- Per the `man bzip2` page, we can decompress this file in a similar manner to the last:
	- `bunzip2 -c data3.txt > data4.txt`

- Issuing `file data4.txt` reveals that this is another gzip compressed file

- Again we will run the gunzip command:
	- `gunzip -c data4.txt > data5.txt`

- Issuing `file data5.txt` reveals this is a POSIX tar archive

- Per the `man tar` page, we can extract the tar archive but it needs to have a *.tar* file extension
	- `cp data5.txt data5.tar`

- Now we can perform the extraction on the new tar archive *data5.tar*
	- `tar -xf data5.tar`
		- `-x` will extract
		- `-f` tells tar that it is a file

	- This will create a new file named *data5.bin*

- Issuing `file data5.bin` reveals it is still a tar archive, just now in binary format

- Again we will issue the same **tar** command, but now on the binary file:
	- `tar -xf data5.bin`

	- This will create a new file named *data6.bin*

- Issuing `file data6.bin` reveals that it is another bzip2 compressed file

- Like earlier we will unzip this file:
	- `bunzip2 -c data6.bin > data7.txt`

- Issuing `file data7.txt` reveals it is another tar archive

- Again we will follow the same steps as earlier:
	- `cp data7.txt data7.tar`
	- `tar -xf data7.tar`

	- This created a new file *data8.bin*

- Issuing `file data8.bin` reveals it to be a gzip compressed file

- Like earlier, we will uncompress this file with:
	- `gunzip -c data8.bin > data9.txt

- Issuing `file data9.txt` reveals it to be an ASCII text file

- Issuing `cat data9.txt` finally reveals our password:
	- *The password is <password_string>*


# Level 13 - Level 14

- Instructions from OverTheWire
> The password for the next level is stored in **/etc/bandit_pass/bandit14 and can only be read by user bandit14**. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Look at the commands that logged you into previous bandit levels, and find out how to use the key for this level.

- Logging into Level 13 and issuing `ls` returns:
	- *HINT*
		- Gives more information about the level. Recommends referring back to the website and states that you cannot log into bandit14 from inside of the bandit13 machine, you must exit back to your host machine and log in from there
	- *sshkey.private*
		- This is the ssh private key to log into bandit14

- Reading the `man ssh` page, you can use the flag `-i` to login with a file instead of a password, meaning that we will have to bring over the private key to our host machine in order to login.
- **scp** is a way to copy files from one machine to another, given you have the proper credentials
- Knowing this, we need to use **scp** to bring the bandit14 private key to our host machine via bandit13
- Since we can login to bandit13 with a password, bringing over the file becomes pretty straightforward:
	- `scp bandit13@bandit.labs.overthewire.org:~/sshkey.private <location_on_host_machine>`

- Issuing `ls` on our host machine in that location shows that we have the key on our system
- We can now attempt to login to bandit14 with this new key, using the `-i` flag:
	- `ssh bandit14@bandit.labs.overthewire.org -p 2220 sshkey.private`
		- But this throws an error:
			- " WARNING: UNPROTECTED PRIVATE KEY FILE!                                                   Permissions 0640 for 'sshkey.private' are too open.                                               It is required that your private key files are NOT accessible by others.                 This private key will be ignored.                                                                                 Load key "sshkey.private": bad permissions"

		- We want to reduce the permissions to the point of being protected:
			- `chmod 600 sshkey.private`
				- This explicitly sets the file so only the owner can read and write it, group and others have zero permissions

- We can now login to bandit14 with our key with new permissions:
	- `ssh bandit14@bandit.labs.overthewire.org -p 2220 sshkey.private`
		- This successfully logs us in to bandit14

- Once inside bandit14, you can issue the following command to display the password, you will need it for the next level:
	- `cat /etc/bandit_pass/bandit14`
		- <password_string>


# Level 14 - Level 15

- Instructions from OverTheWire:
> The password for the next level can be retrieved by submitting the password of the current level to **port 30000 on localhost**.

- Researching how to send data or strings to a port led me to the tool **netcat**
	- **Netcat** is a way to open connections to certain ports on a machine in order to send and receive data. It is usually set up with a listener and a sender

- Attempting to set up a listener, `netcat -l -p 30000` I get the following error message:
	- "netcat: Address already in use"

- This led me to believe that the listener is already setup on port 30000, so I just need to setup a sender command
- The syntax for this is:
	- netcat address port

- Using this I was able to send the command:
	- `netcat localhost 30000`

	- This opened a connection, awaiting my message
	- I sent over some fuzzed data and received the following message:
		- *Wrong! Please enter the correct current password.*
	- This confirmed that I set up the correct sender, so then I attempted to send the bandit14 password string
	- After pasting in the password for bandit14 and submitting, I received the following message:
		- *Correct!*
		- *<bandit15_password_string>*


# Level 15 - Level 16

- Instructions from OverTheWire
> The password for the next level can be retrieved by submitting the password of the current level to **port 30001 on localhost** using SSL/TLS encryption.

- Initially, I thought I needed to use **netcat** again, since the instructions were similar, but reading the `man netcat` page it did not list anything about SSL/TLS

- Under the "Helpful Reading Material" section of the OverTheWire page it links to an OpenSSL Cookbook

- Reading through `man openssl` and OpenSSL cookbook showed me to use of the **openssl** argument *s_client*
	- *s_client* implements a generic SSL/TLS client which can establish a transparent connection to a remote server speaking SSL/TLS. It is mainly used for testing services
	- Using this information, and referring to the OpenSSL Cookbook under the "Testing with OpenSSL" section, I found the following command structure:

``` bash

openssl s_client -crlf \
-connect www.feistyduck.com:443 \
-servername www.feistyduck.com

```

- The `-crlf` flags just mean that it will send multiple lines of commands, but we can skip that and craft our own command
	- `openssl s_client -connect localhost:30001`

	- This successfully connects and returns info about the SSL/TLS connection, featuring server certs, verification data, information about the handshake and ciphers, a hexdump of the TLS session ticket, plus much more.
	- At the end of the data, the service prints "read R BLOCK" and awaits input like in the previous level

- Issuing `<bandit15_password_string>` gets a return:
	- *Correct!*
	- *<bandit16_password_string>*


# Level 16 - Level 17

- Instructions from OverTheWire
> The credentials for the next level can be retrieved by submitting the password of the current level to **a port on localhost in the range 31000 to 32000**. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it. **Helpful note: Getting “DONE”, “RENEGOTIATING” or “KEYUPDATE”? Read the “CONNECTED COMMANDS” section in the manpage.*

- In the **Port Scanning** section of the `man nc` page, it details a command used to scan through multiple ports to check and see if they are open and running a service:
	- `nc -z localhost 31000-32000`
		- `-z` flag can be used to tell **nc** to report open ports, rather than initiate a connection

	- This returns 5 ports and information about each:
		- Connection to localhost (127.0.0.1) 31046 port [tcp/*] succeeded!
		- Connection to localhost (127.0.0.1) 31518 port [tcp/*] succeeded!
		- Connection to localhost (127.0.0.1) 31691 port [tcp/*] succeeded!
		- Connection to localhost (127.0.0.1) 31790 port [tcp/*] succeeded!
		- Connection to localhost (127.0.0.1) 31960 port [tcp/*] succeeded!

- Using this we can use the same logic and process at the last level, using *openssl s_client*
	- `openssl s_client -connect localhost:<port>`

	- Issuing this command on each port returns important information on how to move forward:
		- Ports 31046, 31691, and 31960 returned information confirming that these ports are open, but are not SSL/TLS connections
		- Ports 31518 and 31790 return data like the last level, information about the SSL/TLS server and certificates. This confirms that these are the only two ports hosting SSL/TLS connections.

- Since we now know the 2 ports to work with, we can try and submit the bandit16 password to each. At the bottom of each openssl s_client return there is another "read R BLOCK" prompt, awaiting our password.
- On both of these ports, when entering the password you are met with *KEYUPDATE* and are not able to enter any credentials.

- According to the "Connected Commands" section of the `man openssl` page, using the `-quiet` flag uses the command non-interactively.
- Using this we can issue a new command to the ports:
	- `echo "bandit16_password" | openssl s_client -connect localhost:31518 -quiet`
		- This returns some server info, plus the bandit16 password. We know from OverTheWire instructions that the server not holding bandit17 password will just return what we typed.

	- `echo "bandit16_password" | openssl s_client -connect localhost:31790 -quiet`
		- This returns some server info, plus a flag that says "Correct!" followed by a RSA Private Key

- Just like in the last level:
	- copy the key
	- exit back to host machine in order to ssh from outside the bandit server
	- `echo "<copied_ssh_key>" > bandit17.key`
	- `chmod 600 bandit17.key`

- ssh into bandit17
	- `ssh bandit17@bandit.labs.overthewire.org -p 2220 -i bandit17.key`

	- Once inside you can find the bandit17 password at */etc/bandit_pass/bandit17* for easier login
		- `cat /etc/bandit_pass/bandit17`


# Level 17 - Level 18

- Instructions from OverTheWire
> There are 2 files in the homedirectory: **passwords.old and passwords.new**. The password for the next level is in **passwords.new** and is the only line that has been changed between **passwords.old and passwords.new** **NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19**

- Issuing `ls` in the home directory I can confirm that there are two files:
	- *passwords.old*
	- *passwords.new*

- **diff** is a linux tool to find the difference between two files. Since there is only one password different this will be an easy search
	- `diff passwords.new passwords.old`

	- This will show two blocks of text. 
	- *< string
	- *---*
	- *> string
	
	- The first string is what is different in *passwords.new* and the second will show what is different in *passwords.old*. We do not care what is different about the old set, so we pay attention to the first string that is displayed

- Using this we can ssh into bandit18:
	- `ssh bandit18@bandit.labs.overthewire.org -p 2220`

	- When attempting to login, it will connect, then display "Byebye !". This is normal behavior per the OverTheWire instructions









