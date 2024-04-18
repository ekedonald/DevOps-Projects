# Linux Practice Project
___

## Introduction to Linux and Basic Commands
Linux is a family of open-source Unix operating systems based on the Linux Kernel. They include Ubuntu, Fedora, Debian, openSUSE and Red Hat. Using Linux to manage a Virtual Private Server (VPS) is common practice.

When operating Linux, you need to use a shell (a program that gives you access to the operating system's services). Most Linux distributions use a graphical user interface (GUI), making them beginner-friendly.

## What Is a Linux Command?
A Linux command is a program or utility that runs on the CLI - a console that interacts with the system via texts and processes.

It's similar to the Command Prompt application in Windows. Linux commands are executed on Terminal by pressing Enter at the end of the line. You can run commands to perform various tasks, from package installation to user management and file manipulation.

Here's what a Linux command's general syntax looks like:

```bash
CommandName [options(s)] [parameter(s)]
```

A command may contain an option or a parameter. In some cases, it can still run without them. These are the three most common parts of a command:

CommandName is the rule that you want to perform. Option or flag modifies a command's operation. To invoke it, use hyphens (-) or double hyphens (-).

Parameter or argument specifies any necessary information for the command.

## TAKE NOTE
1. All Linux commands are case-sensitive.

## File Manipulation
## 1. sudo command:
The sudo command stands for "super user do" and it lets you perform tasks that require administrative or root user permissions. When using the sudo command, the system will prompt the user to aunthenticate themselves with a password.

Then, the Linux system will launch a timestap as a tracker. By default, every user can run sudo commands for 15 minutes/session.

Here's the general syntax:

```bash
sudo (command e.g apt upgrade)
```

so it becomes:
```bash
sudo apt upgrade
```

![sudo_command](./images/1.%20sudo%20command.png)

## 2. pwd command
The pwd command stands for "print working directory" and it is used to find the path of the current working directory.

The pwd command uses the following syntax:

```bash
pwd [option]
```

It has two acceptable options (flags):
* -L or logical prints environment variable content, including symbolic links.

* -P or physical prints the actual path of the current directory.

```bash
pwd
```

![pwd_command](./images/2.%20pwd%20command.png)

## 3. cd command:
The cd command is used to navigate through the Linux files and directories. Depending on your working directory, it requires either the full path or the directory name.

Running the command will take you to the home folder. Keep in mind that only users with sudo privileges can execute it.

Let's say you're in /home directory and want to go to a sub directory called ekeikenna. Enter the following command:

```bash
cd ekeikenna
```

![cd_command1](./images/3.%20cd%20command1.png)

If you want to switch to a completely new directory, for example: /home/ekeikenna/Learning_Linux, you have to enter the cd followed by the directory's absolute path:
 
 ```bash
cd /home/ekeikenna/Learning_Linux
 ```

![cd_command2](./images/3.%20cd%20command2.png)

To move to the prevous directory. You enter the following command:

```bash
cd ..
```

![cd_command3](./images/3.%20cd%20command3.png)

## 4. ls command:
The ls command is used to list files and directories running with a system. Running it without a flag will show the current working directory's content.

```bash
ls
```

![ls_command1](./images/4.%20ls%20command1.png)

To see other directory's content, type ls followed by the desired path. For example, to view files in the Learning_Linux folder, enter:

```bash
ls /home/ekeikenna/Learning_Linux
```

![ls_command2](./images/4.%20ls%20command2.png)

Here are some flags you can use with the ls command:

* lists all the files in the subdirectories:

```bash
ls -R
```

![ls_command3](./images/4.%20ls%20command3.png)

* shows hidden files in additon to the visible ones:

```bash
ls -a
```

![ls_command4](./images/4.%20ls%20command4.png)

* shows the file sizes in easily readable formats, suhc as MB, GB and TB:

```bash
ls -lh
```

![ls_command5](./images/4.%20ls%20command5.png)

## 5. cat command:
The concatenate "cat" command is the most frequently used Linux command. It lists, combines and writes file content to the standard output. To run the cat command, type cat followed by the file name and its extension. For instance:

```bash
cat sim1.txt
```

![cat_command1](./images/5.%20cat%20command1.png)

Here are other ways to use the cat command, you can try this on your own:

* Merge sim1.txt and sim2.txt and store the output in sim4.txt.

```bash
cat sim1.txt sim2.txt > sim4.txt
```

![cat_command2](./images/5.%20cat%20command2.png)

* Displays content in reverse order.

```bash
tac sim4.txt
```

![cat_command3](./images/5.%20cat%20command3.png)

## 6. cp command:
The command is used to copy the content of files or directories. Take a look at the following use cases:

* To copy a file from the current directory to another directory, enter cp followed by the file name and the destination directory. For example:

```bash
cp sim1.txt /home/ekeikenna/Learning_Linux/new_folder/
```

![cp_command1](./images/6.%20cp%20command1.png)

* To copy files to a directory, enter the file names followed by the destination directory as shown below:

```bash
cp sim1.txt sim2.txt sim3.txt /home/ekeikenna/Documents
```

![cp_command2](./images/6.%20cp%20command2.png)

* To copy the content of a file to a new file in the same directory, enter cp followed by the source file and the destination file as shown below:

```bash
cp sim1.txt sim4.txt
```

![cp_command3](./images/6.%20cp%20command3.png)

* To copy an entire directoy, pass the -R flag before typing the source directory, followed by the destination directory as shown below:

```bash
cp -R /home/ekeikenna/Documents /home/ekeikenna/Documents_backup
```

![cp_command4](./images/6.%20cp%20command4.png)

## 7. mv command:
The mv command is used to move and rename files and directories. Also, it doesn't produce an output upon execution. An example on how to use the mv command is shown below:

```bash
mv somefile /home/ekeikenna/Learning_Linux/new_folder
```

![mv_command1](./images/7.%20mv%20command1.png)

* You can also use the mv command to rename file:

```bash
mv somefile newfile
```

![mv_command2](./images/7.%20mv%20command2.png)

## 8. mkdir command:
The mkdir command is used to create one or multiple directories at once and set permissions for each them.  

Here's the basic syntax:

```bash
mkdir [option] directory_name
```

* For example, you want to create a directory called Music:

```bash
mkdir Music
```

![mkdir_command1](./images/8.%20mkdir%20command1.png)

* To make a new directory called Songs inside Music, use this command:

```bash
mkdir Music/Songs
```

![mkdir_command2](./images/8.%20mkdir%20command2.png)

* The mkdir command accepts many such as: 
-p or -parents create a directory between two existing folders. For example, mkdir -p Music/2020/Songs will make the new "2020" directory.

```bash
mkdir -p Music/2020/Songs
```
![mkdir_command3](./images/8.%20mkdir%20command3.png)

* To create a directory with full read, write and execute permissions for all users, enter the following command:

```bash
mkdir -m777 albums
```

![mkdir_command4](./images/8.%20mkdir%20command4.png)

To create a directory that prints a message for each created directory. Enter the following command:

```bash
mkdir -v artists
```
![mkdir_command5](./images/8.%20mkdir%20command5.png)

## 9. rmdir command:
To permanently delete an empty directory, use the rmdir command. Remember that the user should have sudo privileges in the parent directory.

* The command below shows how to remove an empty directory:

```bash
rmdir Music/2020
```

![rmdir_command](./images/9.%20rmdir%20command.png)

## 10. rm command:
The rm command is used to delete files within directory. The user performing the command must have write permissions. Remember the directory's location as this will remove file(s) and you can't undo it.

* Here's the general syntax:

```bash
rm html.txt
```

![rm_command1](./images/10.%20rm%20command1.png)

* To remove multiple files, enter the following command:

```bash
rm ufc1.txt ufc2.txt ufc3.txt
```

![rm_command2](./images/10.%20rm%20command2.png)

* To remove a file giving a system confirmation prompt before deletion:

```bash
rm -i ufc.txt
```

![rm_command3](./images/10.%20rm%20command3.png)

* To delete files without system confirmation is shown below:

```bash
rm -f  yaml.txt
```

![rm_command4](./images/10.%20rm%20command4.png)

* To delete files and directories recursively:

```bash
rm -r Songs
```

![rm_command5](./images/10.%20rm%20command5.png)

## 11. touch command:
The touch command allows you to create an empty file.

* For example, enter the following command to create an empty file:

```bash
touch html.index
```

![touch_command](./images/11.%20touch%20command.png)

## 12. locate command:
The locate command can find a file in the database system. However, you have to update the database before you use the locate command to fetch a file. To udpate the database is show below:

`sudo updatedb`

* Moreover, adding -i argument will turn off case sensitivity so you can search for a file even if you don't remember the exact name. The asterisk (*) wildcard can be used in combination to look for content that contains two or more words as shown below:

```bash
locate -i html*
```

![locate_command](./images/12.%20locate%20command.png)

## 13. find command:
The find command is used to search for files with a specific file directory. Here's the general syntax:

```bash
find [path] [option] [expression]
```

* For example, to look for a file called html.index within the home directory and its subfolders is shown below:

```bash
find /home -name index.html
```
![find_command](./images/13.%20find%20command.png)

# 14. grep command:
The grep "global regular expression print" command lets you find a word by filtering through texts in a specific file. Once the grep command finds a match, it prints all lines that contain the specific pattern. This command helps filter through large log files.

* An example of how to use grep is shown below:

```bash
ls -l | grep snap
```

![grep_command](./images/14.%20grep%20command.png)

## 15. df command:
The df command is used to report the sysem's disk space, shown in percentage and kilobyte (KB). Here's the general syntax:

```bash
df [options] [file]
```

* For example, enter the follwoing command if you want to see the current directory's system disk space usage in a human readable format:

```bash
df -h
```

![df_command1](./images/15.%20df%20command1.png)

* The df command in combination with -m displays file system usage in MBs as shown below:

![df_command2](./images/15.%20df%20command2.png)

* The df command in combination with -T displays the file system in a new column as shwown below:

```bash
df -T
```

![df_command3](./images/15.%20df%20command3.png)

* The df command in combination with -k displays the file system usage in KBs as shown below:

```bash
df -k
```

![df_command4](./images/15.%20df%20command4.png)

## 16. du command:
The du command can be used to used to check how much space a file or directory takes up. It can be used to identify which part of the system uses the storage excessively.

* Remember, you must specify the directory path when using the du command as shown below:

```bash
du /home/ekeikenna/Learning_Linux/new_folder/Music/Music
```

![du_command1](./images/16.%20du%20command1.png)

* The du command in combination with -s offers the total size of a specified folder as shown below:

```bash
du -s
```

![du_command2](./images/16.%20du%20command2.png)

* The du command in combination with -m provides folder and file information in MBs as shown below:

```bash
du -m
```

![du_command3](./images/16.%20du%20command3.png)

* The du command in combination with -k displays information in KBs as shown below:

```bash
du -k
```

![du_command4](./images/16.%20du%20command4.png)

* The du command in combination with -h displays information in human readble form in GBs as shown below:

```bash
du -h
```

![du_command5](./images/16.%20du%20command5.png)

## 17. head command:
The head command allows you to view the first 10 lines of a text. Adding an option lets you change the number of lines shown. The head command is also used to output piped date to CLI.

Here's the general syntax:

```bash
head [option] [file]
```

* For instance, you want to view the first ten line of document_list, located in the current directory:

```bash
head document_list
```

![head_command1](./images/17.%20head%20command1.png)

* The head combination with -n or -lines prints the first customized number of lines. For example, enter head -n 5 document_list to show the first five lines of documnet_list as shown below:  

![head_command2](./images/17.%20head%20command2.png)

* The head command in combination with -c or -bytes prints the first customized number of bytes of each file as shown below:

```bash
head -c 100 document_list
```

![head_command3](./images/17.%20head%20command3.png)

* The head command in combination with -q or -quiet will not print headers specifying the file name as shown below:

```bash
head -q document_list
```

![head_command4](./images/17.%20head%20command4.png)

## 18. tail command:
The tail command displays the last ten lines of a file. It allows users to check whether a file has a new data or read error messages. Here is the general format:

```bash
tail [option] [file]
```

* For example, you want to show the last ten lines of the document_list file:

```bash
tail -10 document_list
```

![tail_command](./images/18.%20tail%20command.png)

## 19. diff command:
Short for difference, the diff command compares two contents of a file line by line. After analyzing them, it will display the parts that do not match. Programmers often use the diff command to alter a program instead of rewriting the entire source code. Here's the general format:

```bash
diff [option] file1 file2
```

* For example, you want to compare two files - document_list and document_list2

```bash
diff document_list document_list2
```

![diff_command1](./images/19.%20diff%20command1.png)

* The diff command in combination with -c displays the difference between two files in a context form as shown below:

```bash
diff -c document_list document_list2
```

![diff_command2](./images/19.%20diff%20command2.png)

The diff command in combination with -u displays the output without redundant information as shown below:

```bash
diff -u document_list document_list2
```

![diff_command3](./images/19.%20diff%20command3.png)

## 20. tar command:
The tar command archives multiples files into a TAR file -a common Linux format similar to ZIP with optional compression. Here's the basic syntax:

```bash
tar [options] [archive_file] [file or directory to be archived]
```

* For instance, you want to create a new TAR archive named newarchive.tar in the current working directory:

```bash
tar -cvf newarchive.tar newfolder
```

![tar_command1](./images/20.%20tar%20command1.png)

* The tar command accepts many options such as -x extracts a file as shown below:

```bash
tar -xvf newarchive.tar
```

![tar_command2](./images/20.%20tar%20command2.png)

* The tar command in combination with -t lists the content of a compressed file as shown below:

```bash
tar -tf newarchive.tar
```

![tar_command3](./images/20.%20tar%20command3.png)

* The tar command in combination with -u archives and adds to an existing archive file as shown below:

```bash
tar -uf newarchive.tar
```

![tar_command4](./images/20.%20tar%20command4.png)

# File Permissions and Ownership
## 21. chmod command:
The chmod command modifies a file or directory's read, write and execute permissions. In Linux, each file is associated with 3 user classes - owner, group member and others. Here's the basic syntax:

```bash
chmod [option] [permission] [file_name]
```

* For example, the owner is currently the only one with full permissions to change the files deploy1.yml and deploy2.yml. To allow group members and others read, write and execute the file, chnage it to the -rwxrwxrwx permission type, whose numeric value is 777:

```bash
chmod 777 deploy1.yml deploy2.yml
```

![chmod_command1](./images/21.%20chmod%20command1.png)

* The chmod command in combination with -c or -changes displays information when a change is made as shown below:

```bash
chmod -c 776 deploy1.yml
```

![chmod_command2](./images/21.%20chmod%20command2.png)

* The chmod command in combination with -f or -silent suppressed the error messages as shown below:

```bash
chmod -f 775 deploy1.yml
```

![chmod_command3](./images/21.%20chmod%20command3.png)

* The chmod command in combination with -v or -verbose displays a diagnostic for each processed file as shown below:

```bash
chmod -v 774 deploy1.yml
```

![chmod_command4](./images/21.%20chmod%20command4.png)

## 22. chown command:
The chown command lets you change the ownership of a file, directory or symbolic link to a specified username. Here's the basic format:

```bash
chown [options] owner[:group] file(s)
```

* For exampe, you want to make fabian the owner of deploy1.yml:

```bash
sudo chown fabian deploy1.yml
```

![chown_command](./images/22.%20chown%20command.png)

## 23. jobs command:
A job is a process that the shell starts. The job commands will display all the running processes along with their statuses. Remember that this command is only available in csh, bash, tcsh and ksh shells. This is the basic syntax:

```bash
jobs [options] jobID
```

To check the status of jobs in the current shell, simply enter jobs to the CLI. Here are some options you can use -l list process IDs along with their information, -n lists jobs whose statuses have changed since the last notification and -p lists process IDs only.

![jobs_command](./images/23.%20jobs%20command.png)

## 24. kill command:
Use the kill command to terminate an unresponsive program manually. It will signal misbehaving applications and instruct them to kill their processes.

* To kill a program, you must know its process idenfication number (PID). If you don't know the PID, run the following command:

```bash
ps ux
```

![ps_ux_command](./images/24.%20ps%20ux%20command.png)

* After knowing what signal to use add the program's Process ID (PID), enter the following syntax:

```bash
kill [signal_option] pid
```

There are 64 signals that you can use, but these two are among the most commonly used:

![kill_command1](./images/24.%20kill%20command1.png)

* SIGTERM (15) requests a program to stop running and gives it some time to save all its progress. The system will use this by default if you don't specify the signal when entering the kill command. SIGKILL (9) forces program to stop and you will lose unsaved progress. For example, the program's PID is 63773 and you want to force it to stop:

```bash
kill SIGKILL 63773
```

![kill_command2](./images/24.%20kill%20command2.png)

## 25. ping command:
The ping command is one of the most used basic Linux commands for checking whether a network or a server is reachable. In addition, it is used to troubleshoot various connectivity issues. Here's the general format:

```bash
ping [option] [hostname_or_IP_address]
```

* For example, you want to know whether you can connect to Google and measure its response time:

```bash
ping google.com
```

![ping_command](./images/25.%20ping%20command.png)

## 26. wget command:
The Linux command line lets you download files from the internet using the wget command. It works in the background without hindering other running processes. To use it, enter the following command:

```bash
wget [option] [url]
```

* For example, enter the follwoing command to download the latest version of WordPress:

```bash
wget https://wordpress.org/latest.zip
```

![wget_command](./images/26.%20wget%20command.png)

## 27. uname command:
The uname or unix name command will print detailed information about your Linux system and hardware. This includes the machine name, operating system and kernel. To run this command, simply enter uname into your CLI. Here's the basic syntax:

```bash
uname [option]
```

* These are the acceptable options to use -a prints all the system information, -s prints the kernel name and -n prints the system's node hostname.

![uname_command](./images/27.%20uname%20command.png)

## 28. top command:
The top command in Linux Terminal will display all the running processes and dynamic real-time view of the current system. It sums up the resouce utilization from CPU to memory usage.

```bash
top
```

![top_command](./images/28.%20top%20command.png)

## 29. history:
With history, the system will display up to 500 previously executed commands, allowing you to reuse them without re-entering. Keep in mind that only users with sudo privileges can execute this command. How this utility runs also depends on Linux shell you use. To run it, enter the command below:

```bash
history [option]
```

* This command supports many options, such as -c clears the complete history list, -d offset deletes the history entry at the OFFSET position and -a appends histroy lines.

![history_command](./images/29.%20history%20command1.png)

## 30. man command:
The man command provides a user manual of any commands or utilities you can run in Terminal including the name, description and options. It cosists of nine sections:

* Executable programs or shell commands system calls Library calls Games Special files File formats and conventions System administration commands Kernel routines Miscellaneous to display the complete manual, enter:

```bash
man [command_name]
```

* For example, you want to access the manual for the ls command:

```bash
man ls
```

![man_command](./images/30.%20man%20command.png)

* Enter this command if you want to specify the displayed section:

```bash
man [option] [section_number] [command_name]
```

* For instance, you want to see section 2 of the ls command manual:

```bash
man 2 ls
```

## 31. echo command:
The echo command is a built-in utility that displays a line of text or string using the standard output. Here's the basic syntax:

```bash
echo [option] [string]
```

This command supports many options which are shown below:

* Using -e enables the interpreation of \c which produced after it's positon in the text.

```bash
echo -e "Lionel Messi \cin Miami"
```

![echo_command1](./images/31.%20echo%20command1.png)

*  Using -e enables the interpretation of \b which removes the space before its position in the text.

```bash
echo -e "The \bWorld \bIs \bNot \bEnough"
```
![echo_command2](./images/31.%20echo%20command2.png)

## 32. zip, unzip commands:
The zip command is used to compress your files into a ZIP file, a universal format commonly used on Linux. It can automatically choose the best compression ratio. The zip command is also useful for archiving files, directories and reducing disk usage. 

* To use it, enter the following syntax: 

```bash
zip [options] zipfile file1 file2
```

* For example, you have a file named new.txt that you want to compress into archive.zip in the current directory:

```bash
zip archive.zip new.txt
```

![zip_command](./images/32.%20zip%20command1.png)

* On the other hand, unzip command extracts the zipped files from an archive. Here's the general format:

```bash
unzip [option] file_name.zip
```

* So to unzip a file called archive.zip in the current directory, enter:

```bash
unzip archive.zip
```

![unzip_command](./images/32.%20unzip%20command1.png)

## 33. hostname command:
Run the hostname to know the system's hostname. You can execute it with or without an option. Here's the general syntax:

```bash
hostname [option]
```

* There are many optional flags to use including -a or -alias displays the hostname's alias, -A or -all-fqdns displays the machine's Fully Qualified Domain Name (FQDN), -i or -ip-address displays the machine's IP address. For example, enter the following command to kmow your computer's IP address:

```bash
hostname -i
```

![hostname_command](./images/33.%20hostname%20command.png)

## 34. adduser, deluser commands:
Linux is a multi-user system meaning more than one person can use it simultaneously, adduser is used to create a new account while the passwd command allows modify a password for a specific user. Only those with root privileges or sudo can run the adduser command.

When you use the adduser command, it performs some major changes. Edits the /etc/passwd, /etc/shadow, /etc/group and /etc/gshadow files for the newly created accounts. Creates and populates a home directory for the user. Sets file permissions and ownerships to the home directory. Here's the basic syntax:

```bash
adduser [option] username
```

* To set password:

```bash 
passwd the_password_combination
```

* For example, to add a new person named frank and modify the password, enter the following command simultaneously:

```bash
adduser frank
```

```bash
passwd frank
```

![adduser_passwd_command](./images/34.%20adduser,%20passwd%20command.png)

* To delete a user account, use the deluser commmand as shown below:

```bash
deluser frank
```

![deluser_command](./images/34.%20deluser%20command.png)

## 35. apt-get command:
apt-get is a command line for handling Advanced Package Tool (APT) libraries in Linux. It lets you retrieve information and bundles from auntheticated sources to mange, update, remove and install software and its dependencies. Running the apt-get command requires you to use sudo or root privileges.

* Here's the main syntax:

```bash
apt-get [options] (command)
```

* The apt-get: update synchronizes the package files from their sources, upgrade installs the lastes version of all installed packages, checks updates the package cache and checks broken dependencies.

```bash
apt-get update
```

![apt_get_command](./images/35.%20apt-get%20command.png)

## 36. nano, vi, jed commands:
Linux allows users to edit and manage files via text editor, such as nano, vi or jed. nano and vi come wiht the operating system, while jed has to be installed. 

* The nano command denotes keywords and can work with most languages. To use it, enter the following command:

```bash
nano filename
```

![nano_command](./images/36.%20nano%20command.png)

* vi uses the two operating modes to work - insert and command. insert is used to edit and create a text file. On the other hand, the command performs operations such as saving, opening, copying and pasting a file. To use vi on a file, enter:

```bash
vi filename
```

![vi_command](./images/36.%20vi%20command.png)

* jed has a drop-down menu interface that allows users to perform actions without entering keyboard combinations or commands. Like vi, it has modes to load modules or plugins to write specific texts. To open the program, simply enter jed to then command line.

![jed_command](./images/36.%20jed%20command.png)

## 37. alias, unalias commands:
alias allows you to create a shortcut with the same functionality as a command, file name or text. When executed, it instructs the shell to replace one string with another. To use the alias command, enter this syntax:

```bash
alias Name=String
```

* For example, you want to make k the alias for the kill command:

```bash
alias l='ls -ltr'
```

![alias_command](./images/37.%20alias%20command.png)

* On the other hand, the unalias commadn deletes an existing alias. Here's what the general syntax looks like:

```bash
unalias [alias_name]
```

![unalias_command](./images/37.%20unalias%20command.png)

## 38. su commad:
The switch user or su command allows you to run a program as a different user. It changes the adminstrative account in the current log-in session. This command is especially beneficial for accessing the system through SSH or using GUI display manager when the root user is unavailable. Here's the geberal syntax of the command:

```bash
su [options] [username [argument]]
```

When executed without any option or argument, the su command runs through root privileges. It will prompt you to aunthenticate and use the privileges temporarily.

* Here are some acceptable options to use -p or -preserve environment keeps the same shell environment, consisting HOME, SHELL, USER and LOGNAME; -s or -shell lets you specify a different shell environment to run; -l or -login runs a login script to switch to a different username. Executing it requires you to enter the user's password.

![su_command](./images/38.%20su%20command.png)

## 39. htop command:
The htop command is an interactive program that monitors system resources and server processes in real time. It is available on most Linux distributions and you can install it usimng the default package manager.

Compared to the top command, htop has improvements and additonal features such as mouse operation and visual indicators. To use it, run the following command:

```bash
htop [options]
```

* The -d or -delay flag shows the delay between updates in tenths of seconds as shown below:

```bash
htop  -d 10
```

![htop_command1](./images/39.%20htop%20command1.png)

* The -C or -no-color flag enables the monochrome mode as shown below:

```bash
htop -C
```

![htop_command2](./images/39.%20htop%20command2.png)

* The -h or -help flag displays the the help message as shown below:


```bash
htop -h
```

![htop_command3](./images/39.%20htop%20command3.png)

## 40. ps command:
The process status or ps command produces a snapshot of all running processes in your system.The static results are taken from the virtual files in the /proc file system.

Executing the ps command without an option or argument will list the running processes in the shell along with the unique process ID (PID), the type of the terminal (TTY), the running time (TIME) the command that launches the process (CMD).

* The ps command in combination with -T displays all processes associated witht he current shell session as shown below:

```bash
ps -T
```

![ps_command](./images/40.%20ps%20command.png)

* The ps command in cobination with -u username lists processes associated with a specific user as shown below: 

```bash
ps -u
```

![ps_command2](./images/40.%20ps%20command2.png)

* The ps command in combination with -A or -e flag shows all the running processes as shown below:

```bash
ps -A
```

![ps_command3](./images/40.%20ps%20command3.png)