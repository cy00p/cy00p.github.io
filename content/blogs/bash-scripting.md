---
title: Bash Scripting
slug: bash-scripting-primer
description: Writing bash scripts is fun, and awesome
longDescription: Bash scripts are fun to write but can very annoying as well, we'll go through the fun part 
cardImage: "https://cy00p.github.io/cy00p.webp"
tags: [misc','bash','scripting','linux']
rating: 'medium'
readTime: 10
featured: true
timestamp: 2023-01-04T10:10:03+00:00
---

##### Preamble
You probably know about Linux, and have used/heard of Git on platforms such as Windows or Mac OS. So, at some point, you have realized there's something called **bash**, and you know there's something called a **terminal**.

> If, up to this point, you feel like things are already complicated, or you have no idea what I'm talking about, don't worry, we were all there. So, take a step back and learn a bit about this stuff; this article isn't going anywhere!

**Scripting**, whether you're doing it in **Python**, **JavaScript**, **Bash**, **Ruby**, **PowerShell** etc., is quite essential and you probably have various reasons for writing scripts. It is going to come in handy when you are automating administrative tasks, creating cronjobs, using git hooks, creating utility scripts etc.

Anyway, Python is **great**, JavaScript is **amazing**, Ruby is ...(I don't know, never used it), but Bash is **foundational**. I mean, if you ever want to write a script, in almost all scenarios, a **Bash** script might be exactly what you need. For instance, when you need to run a script inside a Linux container; you don't need to install a python, node or ruby runtime so as to run a script in those respective paradigms. Rather, you just utilize the inbuilt support for Bash in Linux systems to write your script and keep your containers as lightweight as possible.

Now, let's get to the fun part. We'll go through some bash syntax, just the basics, and hopefully, you'll have an easier time writing bash scripts.

> To run the following scripts, save them as "*sample.sh*" (*sample* is an arbitrary name, and you don't even need the *.sh* extension), run "*chmod +x sample.sh*" to give execution permissions to your script and finally run "*./sample.sh*"

##### Variables
declaring, manipulating and printing out a variable:
```sh
# initializing variables
i=12
s="something"
t=13

# modifying a variable
t=$((t+5))

# printing out variables
echo $i $s $t
```

##### Reading Input
reading user input and arguments provided to your script (if a variable or argument doesn't exist, the echo cmd will place an empty string in place of it)
```sh
echo "whats your name"
# read user input
read NAME
echo "Welcome $NAME, today's date is `date`"

# arguments are provided in this manner: ./sample.sh 100 200
echo "Your arguments are $1 and $2"
```

##### Looping
for, while and until are prevalent here; you can use your favorite loop control statements  too - **break** and **continue**; you'll normally wrap your statements inside a loop with **do** and **done** keywords
```sh
# for
for x in 1 2 3 4 5 a b c d
do
	echo "The value is $x"
done


# while
counter=1
while [ $counter -le 10 ]
do
	echo $counter
	((counter++))
done

# until => runs code till condition becomes true 
count=1
until [ $count -gt 5 ]
do
	echo $count
	((count++))
done
```

##### Using Test for Comparison
after conditions are set, the test cmd exits with a 0 for true and a 1 for false
```sh
# the cmd below outputs "false"
test 100 -lt 99 && echo "true" || echo "false"
```

##### Functions
function args are passed in a similar manner to passing args while invoking the script; use echo or a global variable to return values from functions; the return keyword is used to set an exit status from the function (usually set by the exit code from the last executed command)
```sh
# an add function that adds to params provided to it
function add
{
	echo $(( $1 + $2 ))
}

add 10 20 # outputs "30"

x=$(add 1 4)
echo $x # outputs "5"

# a function that initiates a global variable
function myname
{
	name="somebody"
}

myname
echo $name # outputs "somebody"
```

##### Comparison Operators
Here are ways to run comparisons and test equality;
| **comparison** | **command** |
| :---: | :---: |
| is equal to | ``[ "$a" -eq "$b" ]`` |
| is not equal to | ``[ "$a" -ne "$b" ]`` |
| is greater than| ``[ "$a" -gt "$b" ]`` or ``(("$a" > "$b"))`` |
| is greater than or equal to| ``[ "$a" -ge "$b" ]`` or ``(("$a" >= "$b"))`` |
| is less than| ``[ "$a" -lt "$b" ]`` or ``(("$a" < "$b"))`` |
| is less than or equal to| ``[ "$a" -le "$b" ]`` or ``(("$a" <= "$b"))`` |
e.g.
```sh

(("$a" <= "$b")) && echo "true" || echo "false"

[ "$a" -ge "$b" ] && echo "true"

if [ "$a" -lt "$b" ]; then echo "true"; fi
```

#### If-Else statements
Nothing new here if you've ever written code before;
```sh
# one liner
if (("$a" >= "$b")); then echo "true"; fi

# if cmd succeeds with an exit code of 0, run subsequent cmd
if cp -f file1 tmp/ 
then
	rm -f file1
elif cp -f file2 tmp/
then
	rm -f file2
	echo "The second file just moved"
else
	echo "No such file."
	echo "Can't do anything about it."
fi
```

##### String Comparisons
String comparisons are always in the picture; at some point, you will want to compare strings, so, here is another comparison table;
| **comparison** | **command** |
| :---: | :---: |
| is equal to | ``[ "$a" = "$b" ]`` or ``[ "$a" == "$b" ]`` |
| is not equal to | ``[ "$a" != "$b" ]`` |
| is greater than| ``[ "$a" \> "$b" ]`` or ``[[ "$a" > "$b" ]]`` |
| is less than| ``[ "$a" \< "$b" ]`` or ``[[ "$a" < "$b" ]]`` |
| is null (has 0 length)| ``[ -z "$a" ]`` |
| is not null (length > 0)| ``[ -n "$a" ]`` |
e.g.

```sh
a="something"
b=""

# outputs "length > 0"
if [ -z "$a" ]; then echo "is null"; else echo "length > 0"; fi

# outputs "it is null"
if [ -n "$b" ]; then echo "not null"; else echo "it is null"; fi
```

##### Template
Here's a boilerplate template to write a meaningful and readable script;
```sh
#!/bin/bash

# ================================================================
# Author: Sterling Archer
# Date: 1/4/2023
#
# Description: The purpose of this script is to showcase scripting
#
# ================================================================
# Version: x.x.y
#
# Special Instructions:
#
# Declared Variables:
#
# ================================================================
# Script Body
```

##### Sample Scripts
Some few bash scripts to look at:
1. a simple backup script
```sh
#!/bin/bash

echo "Enter your username for the backup server:"
read username

echo "Enter the name of your remote backup server:"
read remote_host

backup_filename=`date +"%m-%d-%y"-`backup.tar

home_directory=$HOME  

backup_location=$remote_host:/$home_directory/backups


tar -cvf $backup_filename `pwd`
scp $backup_filename $username@$backup_location
```
2. a script to create files
```sh
#!/bin/bash

# Initialize variables
name='test'
number=1
limit=2

# Check for the last created file and start algo from there
while [ -f "$name$number" ]
do
	((number++))
	((limit++))
done

# Make 25 files from last created file
# for i in {$number..$limit..1};

while [ $number -lt $((limit+1)) ]
do
	touch "$name$number"
	((number++))
done
```

##### Please Note!
- respect spaces while writing bash scripts, they can be a pain
- bash scripts are an old but reliable way of doing things, using modern scripting languages is recommended