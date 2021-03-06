### If the tutorial is worthy enough, do share it.

> “Whosoever holds this hammer, if he be worthy, shall possess the power of Thor.”	- Stan Lee

> “Whosoever holds this script, if he be worthy, shall possess the power of Bash Scripting (for beginners).” - Vipul Sharma

#### What is a Shell Script?
Shell scripts are essentially bite-sized programs built using the commands available in your shell environment to automate specific tasks generally those tasks that no one enjoys doing by hand, like web scraping, tracking disk usage, downloading weather data, renaming files, and much more.
A shell is a command line interface to the library of commands availbale on a operating system.

#### Objective of this Tutorial?
To harness the power of the shell to automate some of our daily tasks:
	1. Currency conversion in real time with Google Exchange.
	2. Know the weather in any city in metric/U.S. units.
	3. An easily maintainable reminder utility to store information and retrieve them.
	4. Get the latest NEWS in terminal.
	5. Install .tar.gz softwares with just a call.
	6. Automate Git commit


#### Introduction
------------
Before we dig into how we achieve all the above mentioned task, lets go through some of the commands we will use in this tutorial.

The script takes arguments as paramters to run the various utilities in the script.

Since shell scripts don't need a special file extension, we aern't using one to run it as a command from the terminal. The script's path is added to the PATH variable for direct access.

We will go over soem basic commands used in the script before we explore individual scipts.

A **#!** is a convention (or Shebang) that indicates
 * this is a shell script
 * what kind of shell scipt it is.


**$ (dollar sign)**: is used to retrieve the value of a variable by prepending it to the variable name.


**$1, $2, ..** represent the poisitonal parameters (the arguments passed to the shell script)


**$#** is the number of positional paramters.



**$0** is the name of the shell or shell script.



**$(basename $0):** basename removes the leading path and leaves the filename intact.
Example: /home/strider/scripts/mjolnir -> mjolnir



**tput:** is a part of ncurses package that can be used to manipulate our terminal. In the script we have used it for the text effects: bold and sgr0 (turn of all the attributes). The commands are stored in variables for their easy use in the script.



**echo:** writes its aruments to standard output.
	 **Syntax:** echo [option(s)] [string(s)]
	 **Example:** echo Hello World!

	It is not necessary to surround the strings with quotes. Echo always exits with a 0 status and appends a new line at the end, which can be avoidided by passing -n option. -e option enables echo's interpretation of escape sequence characters such as \n, \t etc.



**printf:** allows for definition of a formatting string (providing more control over the output format) and gives a non-zero exit status code upon failure. It has the same format as C style printf().



**if/then:** construct tests whether the exit status of a list of commands is 0 (since 0 means "success" by UNIX convention), and if so, executes one or more commands.
	Syntax: if expression1 then statement1 else if expression2 then statement2 else statement3.



**case:** is also a conditional command in bash that declutters if..then mess for large number of comparisons.
	Syntax: case EXPRESSION in CASE1) COMMAND-LIST;; CASE2) COMMAND-LIST;; ... CASEN) COMMAND-LIST;; esac

	Each clause must be terminated with ";;". Each case statement is ended with the esac statement.
	*) is used as the default case, the actions to be performed when the input doesn't matches any criterion.



**grep:** searches for PATTERN in each FILE. A FILE of “-” stands for standard input. If no FILE is given, recursive searches examine the working directory, and nonrecursive searches read standard input. By default, grep prints the matching lines.

**Syntax:**
	grep [OPTIONS] PATTERN [FILE...]
  grep [OPTIONS] -e PATTERN ... [FILE...]
  grep [OPTIONS] -f FILE ... [FILE...]



**sed:** is a Stream EDitor which performs basic text transformations on an input stream (a file or input from a pipeline).
sed can be used in many different ways, such as:
 * Text substitution,
 * Selective printing of text files,
 * In-a-place editing of text files,
 * Non-interactive editing of text files, and many more.
**Sytax:**
	sed [-n] [-e] 'command(s)' files 
	sed [-n] -f scriptfile files



**curl:** is a tool to transfer data from or to a server, using one of the supported protocols. The command is designed to work without user interaction.
	**Sytanx:** curl [options] [URL...]



**cat:** concatenate files and print on the standard output. 
	**Syntax:** cat [OPTION]... [FILE]...


**more:** is a filter for paging through text one screenful at a time. This version is especially primitive. Users should realize that less provides more emulation plus extensive enhancements.
	**Syntax:** more [options] file...




1. Currency conversion in real-time with Google Exchange.
---------------------------------------------------------
For implementing this scipt snippet, we will tap into the poer of Google and used the Google Exchange to convert in real time.

We query the adress string https://finance.google.co.uk/bctzjpnsun/converter to obtain the result and display it to the user.

Google Currency Converter has three parameters that are passed via the URL itself: the amount, the original currency, and the currency we want.

**Example:** https://finance.google.co.uk/bctzjpnsun/?a=100&from=USD&to=INR

***Digging into the code***

We translate any lowercase letter into uppercase letter using the tr command at line #82 and #83. 

**lynx:** is a FOSS CLI (command line interface) browser. 
The -source options dumps the HTML output of the default document or those specified on the command line to standard output. The query lynx -source "$url?a=$amount&from=$basecurr&to=$targetcurr" returns the whole HTML page source code containing the result.

The output of this query is then pipelined (thourgh the pipeline operator |) to grep command which searches for the pattern 'id=currency_converter_result'. It extracts the information and leaves us with a <div> tag containing the unformatted result (eg: <div id=currency_converter_result>100 USD = <span class=bld>6777.5000 INR</span>).

The output is then pipelined to sed command, which uses the substitution (s/.../.../) to remove all the html content from the input stream it got to a formatted output. The flag /g is used to tell sed to do a global replacement, as sed does the substitiuion for only the first occurance.


***How to use?*** <br/>
**Syntax :** mjolnir -c AMOUNT BASE_CURRENCY to TARGET_CURRENCY <br/>
**Example:** mjolnir -c 100 USD to INR




2. Know the weather in any city in metric/U.S. units.
-----------------------------------------------------
This script snippet utilizes curl command to fetch data from 'wttr.in'. It accepts any city's name and and returns beautiful ASCII output.
By default, the city is set to Boston and the units to metrics.

To check if no arguments are passed we have done a -z test. The -z option checks if a string is null, that is, has zero length.

How to use?
-----------
**Syntax :** mjolnir -w CITY UNITS

The url(wttr.in/CITY?UNITS) accepts the city and the units of measurements (u->U.S. and m->metric) .

**Example:** mjolnir -w Delhi m
		 mjolnir -w 			[will output Boston's weather in metrics]
		 mjolnir -w Delhi u 	[will output Delhi's weather in degree F]




3. Reminder Utility
-------------------
A terminal replacement of the favourite Sticky Notes application. It is a small scipt divided into two parts: 

The first, called with the option -r saves snippets of information into a single 'keep' file in our home directory. If invoked without any arguments, it reads the standard input until the end-of-file sequence (^D) or Ctrl+D. If invoked with arguemnts, it just saves those arguments directly into the data file.

Second, called with the option -f, which either displays the contents of the whole keep file when no argumetns are given or displays the results of searching through it using the arguments as a pattern.

***Digging into the code*** <br/>

The input stream from the terminal is stored in the keepfile using the cat command. The redirection operator ->> is used to redirect all the leading characters into the keep file, put using the cat command. This is done in the scenario when the reminder is directly passed as arguments during the call.

**Example:**
$ mjolnir -k
 --- Reminders! ---
Enter your note, [end it with ^D]: 
Assignmnet submission deadline is Wednesday.

**Example:**
$ mjolnir -k Wash the dishes.

It uses the @:2 positional parameter to store all the arguments after argument number 2 in the keep file.

For fetching the reminder from our script, use the -f argument.
It first checks if the .keep file exists in the system.

If no arguments are passed to search within the exisiting reminders, the whole file is displayed using more, yet another new command.

If an argument is passed to search in the reminder list (.keep file), we search using the grep command. 

grep -i -- "${@:2}" $keepfile | ${PAGER:-more}
 
 -i argument specifies to ignore case of the input

	The double dash (--) is a bash built-in commands and many other commands to signify the end of command options, after which only positional parameters are accepted.

	The output is then invoked to system pager utility more to display the output.

**Example:**
$ mjolnir -f wash
 --- Your Reminders! ---
Wash the dishes.




4. Get the latest NEWS in terminal.
------------------------------------
Another utility we need in our daily life is to see the news, and yes we can see it without leaving our beloved terminal.
For getting the news in real-time, we are using Yahoo's RSS feed.

The output are top 15 news, with links to the stories that can be accessed by simply clicking and following them.


Digging into the code
---------------------
***curl -s "https://www.yahoo.com/news/rss/" 2> /dev/null***

intiate a curl request to fetch data from yahoo news' rss feed. The -s option is for silent mode, ie, in case the fetch takes a long time we don't need to see the progress bar, just the result.

***2> /tmp/error redirects stderr to the /temp/error file***

The output of this command is an XML document containing the news feed from Yahoo.

grep -Eio "<item>.*</item>" |
grep -Eio "<(title|link)>[^<>]*</(title|link)>"
	
The output is then processed using the hierarchial order of the xml documents by digging into the item tag, and then segrigatting title and link to the stories.

The unformatted output of the above command is then fed to a series of sed commands binded together using the -e option.
The title is seperated from the <title> tag, link to the stories from the <link> tags. I have used different colors to display links and headlines, formatted using printf.

Since the number of news stories can be very huge, we can restrict it by using the head -n 15 to limit it to top 15 news.



5. Install .tar.gz softwares with just a call.
----------------------------------------------
While working on linux, we come accross a wide variety of packages that we need to install, and not all are present in software libraries or avialable as rmp packages. This utility helps us install compressed packages in a jiffy, no need to make and install just use mjolnir.

It's a simple utility, check the extension of the compressed file, and unzip it into the folder specified by the user.
The go into the extracted directory, execute make to make file and then install it. Tada!




6. Automate Git commit
----------------------
A small 4 line script to help you be productive. Just pass the comment and it will sync your project automatically.

***git pull***
syncs the existing project with any changes made by other contributors.

***git add.***
adds your file to the git pipeline to be commit

***git commit -am "$2"***
supply the commit message for all the files to be comitted in the current stash.

***git push origin master***
finally push the files to master branch.