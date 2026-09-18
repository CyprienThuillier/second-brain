[THM course](https://tryhackme.com/room/linuxmodules)
### ```du```

```du``` is a command which helps you identify what files/directory are consuming how much space. 

Useful flags :
```bash
-a
	list files as well with the folder
-h
	list the file size in homan readable format(B,MB,KB,GB)
-c
	print the total size at the end
-d <number>
	specify the depth-ness of a directory you want to view the results for
--time
	get the results 
```

Examples :

```bash
du -a /home/
```

You can also use grep :

```bash
du -a /home/ | grep user
```

Finally, you can use ```du``` as an alternative to ```ls``` with the following tags :

```bash
du --time -d 1 .
```

### ```grep, egrep, fgrep```

```bash
grep "PATERN" file.txt
```

You can find a *PATERN* written in [[Regex]] to find what you want in the file.

```grep -E``` is same as ```egrep```
```grep -F``` is same as ```fgrep```

Useful flags :
```bash
-R
	does a recursive grep search for the files inside the folders (if found)
-h
	didables the prefixing of filenames in the results (if grepping recursively)
-c
	only list an interger value that how many times the pattern was found in the
	file/folder
-i
	search for the pattern while ignoring the case
-l
	only list the filename istead of pattern found in it
-n
	list the lines with their line nuber in the file containing the pattern
-v
	print all the lines that are not containing the pattern
-E
	consider the pattern as a regex (BRE/ERE)
-P
	consider the pattern as a regex (PCRE)
-e
	pecify multiple patterns and if any string matches with the pattern(s) it will list it
```

**Example :**

```bash
grep -i "user" login.txt
uSeR:my_username
```

### Strops (String Operations)

For strops, we have the following tools :
- tr
- awk
- sed
- xargs

Other commands to be familiar with :
- sort
- uniq

###### ```tr```

**Syntax :** ```tr [flags] [source]/[find]/[select] [destination]/[replace]/[change]```

Useful flags :

```bash
-d
	delete a given set of characters
-t
	contact source set with destination set
-s
	replace the source set with the destination set
-c
	reverse the command (e.g. -d -c deletes thee rest of the characters leaving
	the source set which we specified)
--help
	print all the available commands
```

**Example :**

```bash
cat file.txt | tr -s '[:lower:]' '[:upper:]'
```

![[tr_eg.png]]

###### ```awk```

**Syntax :** ```awk [flags] [select pattern/find(sort)/commands] [input file]```

If the commands you wrote are in a script, you can execute the script commands by using the ```-f``` flag and specifying the name of the script file (```awk -f script.awk input.txt```). 

**Built-in variables in AWK :**

Built-in variables include field variables ($1, $2. $3, ..., $n). They are used to specify a piece of data. If I run ```awk '{print $1 $3}' file.txt``` it will list me the words that are at 1st and 3rd fields.

![[awk_eg.png]]

And if I run ```awk '{print $1,$3}' file.txt```, it will add a space between the 1st and the 3rd fields.

Here are others useful variables :

```bash
NR: number the lines only (add ',$0' after to print the rest of the line)

FS: Field Separator is the variable to set in case you want to define the field for input stream

BEGIN: exeute the following action to the patern (e.g. "BEGIN {FS='o'} {print $1,$3} END{print 'Total Rows=',NR}")

RS: Record Separator that separates rows with '\n' or the specified char

OFS: Output Field Separator that specifiy a delimeter while outputing

ORS: Output Record Separator that apply the specified sequence as the lines separator
```

Useful flags :
```bash
-F
	specify FS and thus do not need to use the BEGIN rule
-v
	specify variables
-D
	debug the .awk scripts specifying this flags (awk -D script.awk)
-o
	specify the output file
```

###### ```sed```

**Syntax :** ```sed [flags] [pattern/script] [intput file]```

Useful flags :

```bash
-e
	add a script/command that needs to be executed with the pattern/script
-f
	specify the file containing string pattern
-E
	use extended regular expressions
-n
	suppress the automatic printing or pattern spacing
```

Mods/Commands

```bash
s
	Subsitute mode (find and replace)
y
	works same as substitution; the only difference is, it works on individual
	bytes in the string provided
```

Args

```bash
/g
	globally (any pattern change will be affected globally)
/i
	to make the pattern search case-insensitive
/d
	to delete the pattern found
/p
	prints the matching pattern
/1,/2,/3../n
	to perform an operation on an nth occurrence in a line
```

**Examples :**

The following command will replace the 3rd occurrence of the word "hack" by the work "hack" in the file "file.txt" ;

```bash
sed 's/hack/back/3g' file.txt
```

The following command will do the same but on the 3rd and 4th occurrences :

```bash
sed '3,4 s/hack/back/3g' file.txt
```

The following command will reformat the file "file.txt" replacing all the spaces by a simple space :

```bash
sed 's/  */ /g' file.txt
```

###### ```xargs```

Useful flags
```bash
-0
	terminate the arguments with null character
-a file
	allows xargs to read item from a file
-d delimiter
	specify the delimiter to be used when differentiating arguments in stdin
-L int
	specifies max number non-bloank inputs per command line
-s int
	sets the max-chars for the command, which includes it is initial arguments 
	and terminate nulls as well
-x
	exite the command execution if the size specified is exceeded
-E str
	specify the end-of-file string
-I str
	replace str occurrence in arguments with the one passed via stdin
-p
	prompt the user before running any command as a token of confimation
-r
	will not run the commant if the standard input is blank
-n int
	specifies the limi of max-args to be taken from command input at once. Afer
	the max-args limit is reached, it will pass the rest arguments into a new
	command line with the same flags issued to the previously ran command
-t
	verbose
--
	escape command line flags to positional arguments
```

**Examples :**

The following command will define a variable *argVar* to use later :

```bash
echo "file1 file2 file3" | xargs -t -I argVar sh -c '{ touch argVar; ls -l argVar; }'
```

 The following command will create all the files using the filenames in the file "file" and set the privileges to 400 for each file created :

```bash
cat file | xargs -I files -t sh -c "touch files; chmod 400 files"
```

The following command will, for each folder, write the name in the file "file" and delete the folder :

```bash
ls | xargs -I word -n 1 -t sh -c 'echo word >> file; rm word'
```

### ```sort``` and ```uniq```

The ```uniq``` command filters the output to remove any duplicates (LINES ONLY).

The ```sort``` command sorts the lines alphabetically and numerically, automatically. 

**Syntax :**

- ```<command> | sort```
- ```sort <file>```
- ```<command> | uniq```

Useful flags :

```uniq``` :
```bash
-c
	count the occurences of every line in file or stdin
-d
	only print the lines that are repeated
-u
	only print the lines that are already uniq
-i
	ignores case
```

```sort``` :
```bash
-r
	sorts in reverse order
-c
	check whether the file is alreasy sorted or not
-u
	sort and removes the duplicate lines
-o <file>
	save the output into file
```

**Examples :**

```bash
user@post:~$ cat file.txt
xxd is a great tool
A pen and paper
a pen and paper
123 is not a good password
88xxx is the code

user@post:~$ cat file.txt | sort
123 is not a good password
88xxx is the code
a pen and paper
A pen and paper
xxd is a greate tool
```

The following command will removes any duplicate lines ;

```bash
sort file.txt | uniq
```

### ```cURL```

**Syntax :** ```curl <url>```

Useful flags :

```bash
-#
	display a progress meter for the download progress
-o
	saves the file downloaded with the name giver following the flag
-O
	saves the file with the name it was saved on the server
-C-
	resume the broken download without specifying an offset
--limit-rate
	limits the download/upload rate to somewhere near the specified range
-u
	provides user authentication (format: -u user:password)
-T
	helps in uploading the file to some server
-x
	to view a page through proxy (specify the proxy server: -x .server.com -u user:password(Authentication for server)
-I
	queries the header and not the webpage
-A
	specify user agent
-L
	follow redirects
-b
	specify cookies while making a curl request
-d
	POST datta to the server
-X
	specify the HTTP method on the URL
```

### ```wget```

**Syntax :** ```wget protocol://url.com/```

Useful flags :

```bash
-b
	background the downloading process
-c
	continue to the partially downloaded file
-i int
	specify retries to the URL
-O <file>
	specify the output name of the downloaded file
-o <file>
	overwrite the logs into another file
-a <file>
	append the logs into already existing file without deleting previeous contents
-i <file>
	read the list of URLs from a file
--user=username
	give a login username
--password=passowrd
	give a login password
--ask-password
	ask for a password promt if login is necessary
--limit-rate=10k
	limits the download rate
-w=<int>
	specify the waiting time before the retreival from a URL
-T=<int>
	timeout the retreval after a specified amount of time
-N
	enables timestamping
-U
	specify the user-agent while downloading the file
```

### ```xxd```

**Syntax :** 

- ```<command> | xxd```
- ```xxd <flags>```

Useful flags :

```bash
-b
	give a binary representation instead of hexdump
-E
	change the character encoding in the right hand column from ASCII to EBCDIC
-c <int>
	sets the number of bytes to be represented in one row
-g
	set how many bytes/octets should be in a group separated by a whitespace
-i
	output the hexdump in C include format ('0xff' integers)
-l
	specify the lenght of output
-p
	converts the string passed into plain hexdump style
-r
	revert the hexdump to binary
-u
	uppercase hex letters
-s
	seek at offset
```

 There is a difference between ```-s +offset``` and ```-s offset``` while seeking through stdin.

**Examples :**

This command will seek at 10th byte (in hex) in file.txt and display only 50 bytes :

```bash
xxd -s 0xa -l 50 -b file.txt 
```

### Launch a server using python

```bash
python3 -m http.server
```

To access this server from another machine :

Using ```wget```

```bash
wget http://<TARGET_IP>:<PORT>/<FILENAME>
```

