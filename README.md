# PA5: "Hacking" the PA4 Gradebook

In this assignment, you will type in a password in a text (.txt) file,
and run it through a provided executable:

1.	To display "your" PA4 grade
2.	To get access to change "your" PA4 grade

Of course, the binary we provide and the grades are fake (everyone has an
80% to start), and nothing really changes about your grade.

## Getting Started 

Click [this link](https://classroom.github.com/a/UXMPAU_Y) to get the github classroom starter code. 

## 1. Background

Imagine that you're a less ethical student than I'm sure you actually are. You
overhear from some other students in lab that there's a binary available on the
pi-cluster that can show you your grades on assignments before we formally
release them. You hear quieter whispers that someone found a way to use it to
_change_ their grade. Given our less-than-ethical assumption about your state
of mind, you might be tempted to exploit this for yourself.

## 2. Seeing Your Score

You hear that the binary is available at ~/../public/pa4-gradebook. You can try running it with

```
$ ~/../public/pa4-gradebook
```

You should at first see output that says you need to create a password.txt
file. Try creating one, and putting a few characters into it. If you run the
program again, it will tell you the password is incorrect (unless you're an
extremely lucky guesser).

Your task is to figure out the password that is assigned to you by
reverse-engineering the binary, and putting it into `password.txt` in order to
gain access to viewing your "grade". You primary tools will be `gdb`, your
knowledge of stack layout, and a few hints we give along the way.

You can run the program in `gdb` (using `gdb ~/../public/pa4-gradebook`), and trace through
the program, setting breakpoints and inspecting memory as necessary.

A reminder of some useful GDB commands:

- `layout reg` will show you the contents of registers
- `x/10x $sp` will e**x**amine **10** he**x** formatted words of memory starting at
`$sp` (you can parameterize the number of words and the register used here)
- `x/10x 0xFFFFFFFF` will e**x**amine **10** he**x** formatted words of memory
starting at the address you provide (you can copy paste an address into this
command to explore more memory, for instance)
- `b *label` will set a **b**reakpoint directly at the given label
- `si` will **s**tep one **i**nstruction
- `start` will begin the program and pause at the beginning of `main`
- `continue` or `c` will **c**ontinue the program running when it's stopped at a breakpoint

**Hint**: Consider passwords, and how they must work in a program for a moment.
Perhaps, somewhere in the program, the contents of `password.txt` is
compared against some other string or other value to check if the password is
correct.  Maybe some well-known library function, like `strlen` or `strcmp` or
`strncmp`, is used for that purpose? Or maybe the course staff wrote a helper
function having to do with `password` somewhere?

The UNIX command `strings` looks for string constants and labels within a
binary. You can run it with

```
$ strings ~/../public/pa4-gradebook
```

Try running this and look at the output -- does it suggest to you any labels
that might be interesting to stop at?

If you figure out what values your password is being compared against, you can
try putting strings in the password file to match. Don't be afraid to try a
number of things; you can try running as often as you'd like!

When you get the password correct, you will see a prompt like:

```
--------------------------------------------
Welcome cs30f to the CSE 30 PA4 gradebook!
--------------------------------------------

Press 'v' to view your grade. If you are an admin, then you can press 'c' to change your grade.
v <--- You type this as the user
The grade for cs30f is : 80%.
```

## 3 Getting Admin Permissions

If you try the `'c'` option, you'll get an error:

```
--------------------------------------------
Welcome cs30f to the CSE 30 PA4 gradebook!
--------------------------------------------

Press 'v' to view your grade. If you are an admin, then you can press 'c' to change your grade.
c
Access denied; user is not admin
```

You'll need to be a little more sneaky to get administrator permissions to
_change_ your grade.

### 3.1 A Reminder about Stacks

Consider this program:

<img src="https://github.com/ucsd-cse30-f17/pa5-public/blob/master/stack_example_1.png" width="500">

Recall how its data is laid in the stack when the program is run:

<img src="https://github.com/ucsd-cse30-f17/pa5-public/blob/master/stack_example_2.png" width="500">

Notice how the stack grows upward in memory, from high memory addresses
to low memory addresses. The locally declared variables are laid out at
the bottom of the frame, with the arrays and structs above them while
structs' and arrays' members are in the opposite manner (from low memory
address to high memory address).

### 3.2 The Plot Thickens

You see some code open on the professor's laptop during office hours.  You do
your best to commit it to memory and write it down (remember, you're acting
quite unethically in this story), because it strikes you that the code was
something regarding usernames and keys.

You vaguely recall some code that provided a struct definition:

<img src="https://github.com/ucsd-cse30-f17/pa5-public/blob/master/Struct.png" width="400">

and some code relating to the layout of some local variables, somewhere inside
a `main`-looking function

<img src="https://github.com/ucsd-cse30-f17/pa5-public/blob/master/local_vars.png" width="900">

You're pretty sure that you saw the `permissions` field used in a conditional
somewhere; something about checking permissions. If only you could get that to
be set to just the right value, you think, there'd be a chance of enabling the
change feature...

Your task is to craft a `password.txt` file that not only gets you access to
see your grade, but tricks the binary into thinking the current user has the
permission to change the grade. You will be able to do this solely by editing
`password.txt`.

You'll use GDB again, paying careful attention to what locations in memory you
might be able to control by manipulating your password.  All the commands
mentioned above will be handy again to see what memory location you should be
changing and how you can go about changing it.  Keep examining the stack while
stepping, and see what's really happening in the registers.

**HINT**
Since the course staff are programmers with great style, they may have written
helper functions related to checking permissions.

**HINT** 
The pa4-gradebook program uses the password you placed in password.txt
by reading it in from a file and storing it in a stack allocated string.
If you make the password _longer_ than it needs to be, is the password still
accepted?  Where might the extra characters be going?

**HINT**
When important values are adjacent on the stack, overflowing an array with
values that you control can let you assign into other stack-allocated values.



## 4. Ethics, Responsibility, and Legality

What a fun assignment (at least, we think it is)!

It's fun to joke about this kind of situation, and it's really useful to have courses so we can
explore it in a controlled environment. That said, these kinds of vulnerabilities can and do come up
in real software. Given that you have some potentially exploitable knowledge, this comes with
responsibility.

First, *the ability to break a system does not give you permission to exploit it*. This is important
legally and morally. If a person steals from a car whose doors are unlocked, it is still theft, even
though the car's owner was perhaps less careful than they should be. If you exploit a system by reverse
engineering or triggering a buffer overflow, it can still be a crime, despite the carelessness of the
system's creators. The Computer Fraud and Abuse Act covers many of these cases https://en.wikipedia.org/wiki/Computer_Fraud_and_Abuse_Act. In addition, it's a good question, if you find a vulnerability but don't exploit it, how you should report it. The community has standards
around these practices, for example Responsible Disclosure:
https://en.wikipedia.org/wiki/Responsible_disclosure

Use your knowledge responsibly.

Second, from the reverse perspective as a system builder, before creating a system that could handle
sensitive information, you need to understand the risks of storing it and manipulating it. The "mistake"
in the program that enables these attacks is a somewhat trivial one that we all could make. It could also
have serious consequences for people who put their trust in your system. As a software engineer, you have
responsibility for these issues, and you should educate yourself (by taking security courses, or working
under a mentor who deals with these issues) before taking on projects where a mistake could result in
losing or modifying information on behalf of other people.


## 5. README

1. Write into your README file the following sentence:

<img src="https://github.com/ucsd-cse30-f17/pa5-public/blob/master/readme_statement.png">

This language is borrowed from the computer security course here.

2. In the "story" of this assignment, you played the unethical student who
exploited the system. In reality, the right option would be to immediately
report the issue to the course staff. In addition, if you see sensitive code
open on someone's laptop, it's a good idea to let them know you saw it so they
know it was in plain view, and then you should not use this knowledge for your
own gain.

    Write a draft of an email that you would send to the course staff (either a
    TA, tutor, or the instructor), to let them know about the vulnerability you
    found, describing the issue and how to reproduce it. Keep it to 100 words
    or less.


## 6. Handin

Commit and push the following files to the GitHub repository that was created for you by 11:59 PM on
Tuesday, November 21, 2017.

1. password.txt
2. README.txt
