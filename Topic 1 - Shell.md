
<iframe src="https://www.youtube.com/embed/Z56Jmr9Z34Q" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

## Topic 1: The Shell

## What is the shell?

Computers these days have a variety of interfaces for giving them commands; fanciful graphical user interfaces, voice interfaces, and even AR/VR are everywhere. These are great for 80% of use-cases, but they are often fundamentally restricted in what they allow you to do — you cannot press a button that isn’t there or give a voice command that hasn’t been programmed. To take full advantage of the tools your computer provides, we have to go old-school and drop down to a textual interface: The Shell.

Nearly all platforms you can get your hands on have a shell in one form or another, and many of them have several shells for you to choose from. While they may vary in the details, at their core they are all roughly the same: they allow you to run programs, give them input, and inspect their output in a semi-structured way.

In this lecture, we will focus on the Bourne Again SHell, or “bash” for short. This is one of the most widely used shells, and its syntax is similar to what you will see in many other shells. To open a shell _prompt_ (where you can type commands), you first need a _terminal_. Your device probably shipped with one installed, or you can install one fairly easily.

## Using the shell

- When you launch your terminal, you will see a _prompt_ that often looks a little like this:

	```bash
	missing:~$ 
	```

 - It tells you that you are on the machine `missing` and that your “current working directory”, or where you currently are, is `~` (short for “home”)
 - The `$` tells you that you are not the root user.
 - The most basic command is to execute the program `echo` with the argument `hello`. The `echo` program simply prints out its arguments. The shell parses the command by splitting it by whitespace, and then runs the program indicated by the first word, supplying each subsequent word as an argument that the program can access. If you want to provide an argument that contains spaces or other special characters (e.g., a directory named “My Photos”), you can either quote the argument with `'` or `"` e.g. `"My Photos"`, or escape just the relevant characters with `\` i.e. `My\ Photos`.

```bash
missing:~$ echo hello
hello
```

- But how does the shell know how to find the `echo` program? Well, the shell is a programming environment, just like Python or Ruby, and so it has variables, conditionals, loops, and functions (next lecture!). 

- When you run commands in your shell, you are really writing a small bit of code that your shell interprets. If the shell is asked to execute a command that doesn’t match one of its programming keywords, it consults an _environment variable_ called `$PATH` that lists which directories the shell should search for programs when it is given a command:

```bash
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
```

- When we run the `echo` command, the shell sees that it should execute the program `echo`, and then searches through the `:`-separated list of directories in `$PATH` for a file by that name. When it finds it, it runs it (assuming the file is _executable_; more on that later). We can find out which file is executed for a given program name using the `which` program. We can also bypass `$PATH` entirely by giving the _path_ to the file we want to execute.

## Navigating in the shell

- A path on the shell is a delimited list of directories; separated by `/` on Linux and macOS and `\` on Windows. 

- On Linux and macOS, the path `/` is the “root” of the file system, under which all directories and files lie, whereas on Windows there is one root for each disk partition (e.g., `C:\`). We will generally assume that you are using a Linux filesystem in this class. 

- A path that starts with `/` is called an _absolute_ path. Any other path is a _relative_ path. Relative paths are relative to the current working directory, which we can see with the `pwd` command and change with the `cd` command. In a path, `.` refers to the current directory, and `..` to its parent directory:

```bash
missing:~$ pwd
/home/missing
missing:~$ cd /home
missing:/home$ pwd
/home
missing:/home$ cd ..
missing:/$ pwd
/
missing:/$ cd ./home
missing:/home$ pwd
/home
missing:/home$ cd missing
missing:~$ pwd
/home/missing
missing:~$ ../../bin/echo hello
hello
```

- To see what lives in a given directory, we use the `ls` command. Unless a directory is given as its first argument, `ls` will print the contents of the current directory.

```bash
missing:~$ ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
```

- The `-l` argument gives us metadata of each file or directory present. 
- The `d` at the beginning of the line tells us that `missing` is a directory. Then follow three groups of three characters (`rwx`). These indicate what permissions the owner of the file (`missing`), the owning group (`users`), and everyone else respectively have on the relevant item. 
- Read (`r`) permission is used to access the file/directory's contents. You can use a tool like `cat` or `less` on the file to display the file contents. You could also use a text editor like Vi or `view` on the file to display the contents of the file. Read permission is required to make copies of a file, because you need to access the file's contents to make a duplicate of it.
- Write (`w`) permission allows you to modify or change the contents of a file/directory.
- Execute (`x`) permission allows you to execute the contents of a file. Typically, executables would be things like commands or compiled binary applications. However, execute permission also allows someone to run Bash shell scripts, Python programs, and a variety of interpreted languages. Execute permission is also required for access to the directory itself.
- A `-` indicates that the given principal does not have the given permission. Above, only the owner is allowed to modify (`w`) the `missing` directory (i.e., add/remove files in it). 
- Nearly all the files in `/bin` have the `x` permission set for the last group, “everyone else”, so that anyone can execute those programs.
- There are other ways to execute the contents of a file without execute permission. For example, you could use an interpreter that has execute permission to read a file with instructions for the interpreter to execute. An example would be invoking a Bash shell script:

	```bash
	$ bash script.sh
	```

- On most Unix-like systems, one user is special: the “root” user. The root user is above (almost) all access restrictions, and can create, read, update, and delete any file in the system. To perform an operation as root, use the `sudo` command. As its name implies, it lets you “do” something “as su” (short for “super user”, or “root”)

Some other handy programs to know about at this point are `mv` (to rename/move a file), `cp` (to copy a file), and `mkdir` (to make a new directory).
## Connecting programs

- In the shell, programs have two primary “streams” associated with them: their input stream and their output stream. 
- When the program tries to read input, it reads from the input stream, and when it prints something, it prints to its output stream. Normally, a program’s input and output are both your terminal. However, we can also rewire those streams!

- The simplest form of redirection is `< file` and `> file`. These let you rewire the standard input and output streams of a program to a file respectively:

```bash
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

In examples above, `cat` is a program that con`cat`enates files. When given file names as arguments, it prints the contents of each of the files in sequence to its output stream. But when `cat` is not given any arguments, it prints contents from its input stream to its output stream (like in the third example above).

You can also use `>>` to append to a file. Where this kind of input/output redirection really shines is in the use of _pipes_. The `|` operator lets you “chain” programs such that the output of one is the input of another:

```bash
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

We will go into a lot more detail about how to take advantage of pipes in the lecture on data wrangling.

- Note that operations like `|`, `>`, and `<` are done _by the shell_, not by the individual program. For example, the brightness of your laptop’s screen is exposed through a file called `brightness` under `/sys/class/backlight` and can only be modified by the root user. 

```bash
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

- In the case above, the _shell_ (which is authenticated just as your user) tries to open the brightness file for writing, before setting that as `sudo echo`’s output, but is prevented from doing so since the shell does not run as root. Using this knowledge, we can work around this:

```bash
$ echo 3 | sudo tee brightness
```

Since the `tee` program is the one to open the `/sys` file for writing, and _it_ is running as `root`, the permissions all work out. 

## Exercises

1. For this course, you need to be using a Unix shell like Bash or ZSH. If you are on Linux or macOS, you don’t have to do anything special. If you are on Windows, you need to make sure you are not running cmd.exe or PowerShell; you can use [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) or a Linux virtual machine to use Unix-style command-line tools. To make sure you’re running an appropriate shell, you can try the command `echo $SHELL`. If it says something like `/bin/bash` or `/usr/bin/zsh`, that means you’re running the right program.
2.  Create a new directory called `missing` under `/tmp`.
3.  Look up the `touch` program. The `man` program is your friend.
4.  Use `touch` to create a new file called `semester` in `missing`.
5.  Write the following into that file, one line at a time:
    
    ```bash
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu
    ```
    
    The first line might be tricky to get working. It’s helpful to know that `#` starts a comment in Bash, and `!` has a special meaning even within double-quoted (`"`) strings. Bash treats single-quoted strings (`'`) differently: they will do the trick in this case. See the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page for more information.
    
6.  Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file). **No execute (x) permission on file owner**.
7.  Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t? **Execute permission present on sh program**.
8.  Look up the `chmod` program (e.g. use `man chmod`).
9.  Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. How does your shell know that the file is supposed to be interpreted using `sh`? See this page on the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line for more information. **1) chmod u+x semester**. **2) The shebang at the top of the file specifies the interpreter used to execute the script but is overwritten when explicitly stating the interpreter when running the script. If the shebang is omitted, it defaults to the login shell.**
10.  Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory. **./semester | grep last-modified > ~/last-modified.txt**
11.  Write a command that reads out your laptop battery’s power level or your desktop machine’s CPU temperature from `/sys`. Note: if you’re a macOS user, your OS doesn’t have sysfs, so you can skip this exercise.

___