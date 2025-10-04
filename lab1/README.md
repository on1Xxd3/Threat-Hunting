# lab1\_

------------------------------------------------------------------------

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Развить навыки работы в Rstudio IDE: установка пакетов работа с
    проектами в Rstudio настройка и работа с Git
3.  Закрепить знания базовых типов данных языка R и простейших операций
    с ними

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.1

## План

Заустить подкурсы и выполнить: 1. базовые структурные блоки (Basic
Building Blocks) 2. рабочие пространства и файлы (Workspace and Files)
3. последовательности чисел (Sequences of Numbers) 4. векторы (Vectors)
5. пропущенные значения (Missing Values)

## Шаги:

1.  Basic Building Blocks

    | In its simplest form, R can be used as an interactive calculator.
    Type 5 + 7 and press Enter.

> 5+7 \[1\] 12

Perseverance, that’s the answer.

To assign the result of 5 + 7 to a new variable called x, you type x |
\<- 5 + 7. This can be read as ‘x gets 5 plus 7’. Give it a try now.

> x \<- 5+7

|====================| 21% | You’ll notice that R did not print the
result of 12 this time. When you use the assignment operator, R |
assumes that you don’t want to see the result immediately, but rather
that you intend to use the result | for something else later on.

> x \[1\] 12

Nice work!

Now, store the result of x - 3 in a new variable called y.

> y \<- x-3

|============================| 29% | What is the value of y? Type y to
find out.

> y \[1\] 9

Now, let’s create a small collection of numbers called a vector. Any
object that contains data is called | a data structure and numeric
vectors are the simplest type of data structure in R. In fact, even a |
single number is considered a vector of length one.

…

|============================| 34% | The easiest way to create a vector
is with the c() function, which stands for ‘concatenate’ or | ‘combine’.
To create a vector containing the numbers 1.1, 9, and 3.14, type c(1.1,
9, 3.14). Try it now | and store the result in a variable called z.

> z \<- c(1.1, 9, 3.14)

|====================================| 37%

Anytime you have questions about a particular function, you can access
R’s built-in help files via the  
`?` command. For example, if you want more information on the c()
function, type ?c without the  
parentheses that normally follow a function name. Give it a try.

> ?c

You can combine vectors to make a new vector. Create a new vector that
contains z, 555, then z again in  
that order. Don’t assign this vector to a new variable, so that we can
just see the result immediately.

> c(z, 555, z) \[1\] 1.10 9.00 3.14 555.00 1.10 9.00 3.14

|===========================================| 45%

Numeric vectors can be used in arithmetic expressions. Type the
following to see what happens: z \* 2 + | 100.

> z\*2+100 \[1\] 102.20 118.00 106.28

|==============================================| 47%

Take the square root of z - 1 and assign it to a new variable called
my_sqrt. \> my_sqrt \<- sqrt(z-1)

Think about how R handled the other ‘vectorized’ operations:
element-by-element.

1: a vector of length 0 (i.e. an empty vector) 2: a single number (i.e a
vector of length 1) 3: a vector of length 3

Выбор: 3

That’s the answer I was looking for.

Print the contents of my_sqrt.

my_sqrt \[1\] 0.3162278 2.8284271 1.4628739

You are amazing!

|===========================================================| 61% | As
you may have guessed, R first subtracted 1 from each element of z, then
took the square root of each | element. This leaves you with a vector of
the same length as the original vector z.

Now, create a new variable called my_div that gets the value of z
divided by my_sqrt.

> my_div \<- z/my_sqrt

Perseverance, that’s the answer. Which statement do you think is true?
1: The first element of my_div is equal to the first element of z
divided by the first element of my_sqrt, and so on… 2: my_div is a
single number (i.e a vector of length 1) 3: my_div is undefined

Выбор: 1

You are really on a roll!

|==================================================================| 68%

Go ahead and print the contents of my_div.

> print(my_div) \[1\] 3.478505 3.181981 2.146460

Nice try, but that’s not exactly what I was hoping for. Try again. Or,
type info() for more options.

Type my_div and press Enter to see its contents.

> my_div \[1\] 3.478505 3.181981 2.146460

Keep up the great work!

|=====================================================================|
71% | When given two vectors of the same length, R simply performs the
specified arithmetic operation (`+`, | `-`, `*`, etc.)
element-by-element. If the vectors are of different lengths, R
‘recycles’ the shorter | vector until it is the same length as the
longer vector.

…

|======================================================================
| 74% | When we did z \* 2 + 100 in our earlier example, z was a vector
of length 3, but technically 2 and 100 | are each vectors of length 1.

…

|==========================================================================|
76% | Behind the scenes, R is ‘recycling’ the 2 to make a vector of 2s
and the 100 to make a vector of 100s. | In other words, when you ask R
to compute z \* 2 + 100, what it really computes is this: z \* c(2, 2,
2) + | c(100, 100, 100).

…

|=========================================================================|
79% | To see another example of how this vector ‘recycling’ works, try
adding c(1, 2, 3, 4) and c(0, 10). | Don’t worry about saving the result
in a new variable.

> c(1,2,3,4)+c(0,10) \[1\] 1 12 3 14

Perseverance, that’s the answer.

|=============================================================================|
82% | If the length of the shorter vector does not divide evenly into
the length of the longer vector, R will | still apply the ‘recycling’
method, but will throw a warning to let you know something fishy might
be | going on.

…

|==============================================================================|
84% | Try c(1, 2, 3, 4) + c(0, 10, 100) for an example.

> c(1,2,3,4)+c(0,10,100) \[1\] 1 12 103 4 Предупреждение: В c(1, 2, 3,
> 4) + c(0, 10, 100) : длина большего объекта не является произведением
> длины меньшего объекта

Excellent job!

|=================================================================================|
87% | Before concluding this lesson, I’d like to show you a couple of
time-saving tricks.

…

|=================================================================================|
89% | Earlier in the lesson, you computed z \* 2 + 100. Let’s pretend
that you made a mistake and that you | meant to add 1000 instead of 100.
You could either re-type the expression, or…

…

|==================================================================================|92%
| In many programming environments, the up arrow will cycle through
previous commands. Try hitting the up | arrow on your keyboard until you
get to this command (z \* 2 + 100), then change 100 to 1000 and hit |
Enter. If the up arrow doesn’t work for you, just type the corrected
command.

> z\*2+1000 \[1\] 1002.20 1018.00 1006.28

Nice work!

|====================================================================================95%
| Finally, let’s pretend you’d like to view the contents of a variable
that you created earlier, but you | can’t seem to remember if you named
it my_div or myDiv. You could try both and see what works, or…

…

|==================================================================================|97%
You can type the first two letters of the variable name, then hit the
Tab key (possibly more than once). | Most programming environments will
provide a list of variables that you’ve created that begin with ‘my’. |
This is called auto-completion and can be quite handy when you have many
variables in your workspace. | Give it a try. (If auto-completion
doesn’t work for you, just type my_div and press Enter.)

> my_div \[1\] 3.478505 3.181981 2.146460

You got it right!

|==================================================================================|100%

1.  рабочие пространства и файлы (Workspace and Files)

In this lesson, you’ll learn how to examine your local workspace in R
and begin to explore the | relationship between your workspace and the
file system of your machine.

…

|== | 3% | Because different operating systems have different
conventions with regards to things like file paths, | the outputs of
these commands may vary across machines.

…

|===== | 5% | However it’s important to note that R provides a common
API (a common set of commands) for interacting | with files, that way
your code will work across different kinds of computers.

…

|======= | 8% | Let’s jump right in so you can get a feel for how these
special functions work!

…

|========== | 10% | Determine which directory your R session is using as
its current working directory using getwd().

> getwd() \[1\] “C:/Users/PC/Documents/Threat-Hunting”

Nice work!

|============ | 13% | List all the objects in your local workspace using
ls().

> ls() \[1\] “my_div” “my_sqrt” “q” “x” “y” “z”

That’s a job well done!

|=============== | 15% | Some R commands are the same as their
equivalents commands on Linux or on a Mac. Both Linux and Mac |
operating systems are based on an operating system called Unix. It’s
always a good idea to learn more | about Unix!

…

|================= | 18% | Assign 9 to x using x \<- 9.

> x \<- 9

That’s correct!

|==================== | 21% | Now take a look at objects that are in
your workspace using ls().

ls() \[1\] “my_div” “my_sqrt” “q” “x” “y” “z”

You got it right!

|======================| 23% | List all the files in your working
directory using list.files() or dir(). dir() \[1\] “lab1” “README.md”
“Threat-Hunting.Rproj”

Keep up the great work!

|========================= | 26% | As we go through this lesson, you
should be examining the help page for each new function. Check out the |
help page for list.files with the command ?list.files.

> ?list.files

That’s the answer I was looking for.

|=========================== | 28% | One of the most helpful parts of
any R help file is the See Also section. Read that section for |
list.files. Some of these functions may be used in later portions of
this lesson.

…

|============================== | 31% | Using the args() function on a
function name is also a handy way to see what arguments a function can |
take.

…

|================================ | 33% | Use the args() function to
determine the arguments to list.files(). args(list.files) function (path
= “.”, pattern = NULL, all.files = FALSE, full.names = FALSE, recursive
= FALSE, ignore.case = FALSE, include.dirs = FALSE, no.. = FALSE) NULL

That’s correct!

|===================================| 36% | Assign the value of the
current working directory to a variable called “old.dir”. old.dir \<-
getwd()

You’re the best!

|===================================== | 38% | We will use old.dir at
the end of this lesson to move back to the place that we started. A lot
of query | functions like getwd() have the useful property that they
return the answer to the question as a result | of the function.

…

|======================================== | 41% | Use dir.create() to
create a directory in the current working directory called “testdir”.
dir.create(‘testdir’)

You got it right!

|========================================== | 44% | We will do all our
work in this new directory and then delete it after we are done. This is
the R analog | to “Take only pictures, leave only footprints.”

…

|============================================= | 46% | Set your working
directory to “testdir” with the setwd() command.

> setwd(‘testdir/’)

You are doing so well!

|=============================================== | 49% | In general, you
will want your working directory to be someplace sensible, perhaps
created for the | specific project that you are working on. In fact,
organizing your work in R packages using RStudio is | an excellent
option. Check out RStudio at http://www.rstudio.com/

…

|================================================== | 51% | Create a
file in your working directory called “mytest.R” using the file.create()
function.

> file.create(‘myTest.R’) \[1\] TRUE

One more time. You can do it! Or, type info() for more options.

file.create(“mytest.R”) will get the job done!

> file.create(“mytest.R”) \[1\] TRUE

That’s correct!

|==================================================== | 54% | This
should be the only file in this newly created directory. Let’s check
this by listing all the files in the current directory. list.files()
\[1\] “myTest.R”

That’s correct!

|======================================================= | 56% | Check
to see if “mytest.R” exists in the working directory using the
file.exists() function.

> file.exists(“mytest.R”) \[1\] TRUE

You got it!

|========================================================= | 59% | These
sorts of functions are excessive for interactive use. But, if you are
running a program that loops | through a series of files and does some
processing on each one, you will want to check to see that each | exists
before you try to process it.

…

|============================================================ | 62% |
Access information about the file “mytest.R” by using file.info().

> file.info(‘mytest.R’) size isdir mode mtime ctime atime exe uname
> mytest.R 0 FALSE 666 2025-10-04 23:53:54 2025-10-04 23:53:39
> 2025-10-04 23:53:54 no PC udomain mytest.R DESKTOP-VG3OM9I

You are doing so well!

|============================================================== | 64% |
You can use the $ operator — e.g., file.info(“mytest.R”)$mode — to grab
specific items.

…

|================================================================= | 67%
| Change the name of the file “mytest.R” to “mytest2.R” by using
file.rename().

> file.rename(‘mytest.R’, ‘mytest2.R’) \[1\] TRUE

Your dedication is inspiring!

|=================================================================== |
69% | Your operating system will provide simpler tools for these sorts
of tasks, but having the ability to | manipulate files programatically
is useful. You might now try to delete mytest.R using |
file.remove(‘mytest.R’), but that won’t work since mytest.R no longer
exists. You have already renamed | it.

…

|======================================================================
| 72% | Make a copy of “mytest2.R” called “mytest3.R” using file.copy().

> file.copy(‘mytest2.R’, “mytest3.R”) \[1\] TRUE

Excellent job!

|========================================================================
| 74% | You now have two files in the current directory. That may not
seem very interesting. But what if you | were working with dozens, or
millions, of individual files? In that case, being able to
programatically | act on many files would be absolutely necessary. Don’t
forget that you can, temporarily, leave the | lesson by typing play()
and then return by typing nxt().

…

|===========================================================================
| 77% | Provide the relative path to the file “mytest3.R” by using
file.path().

> file.path(“mytest3.R”) \[1\] “mytest3.R”

You’re the best!

|=============================================================================
| 79% | You can use file.path to construct file and directory paths that
are independent of the operating system | your R code is running on.
Pass ‘folder1’ and ‘folder2’ as arguments to file.path to make a |
platform-independent pathname.

> file.path(‘folder1’, ‘folder2’) \[1\] “folder1/folder2”

You are doing so well!

|================================================================================
| 82% | Take a look at the documentation for dir.create by entering
?dir.create . Notice the ‘recursive’ | argument. In order to create
nested directories, ‘recursive’ must be set to TRUE.

> ?dir.create

Your dedication is inspiring!

|==================================================================================
| 85% | Create a directory in the current working directory called
“testdir2” and a subdirectory for it called | “testdir3”, all in one
command by using dir.create() and file.path().
dir.create(file.path(‘testdir2’, ‘testdir3’), recursive = TRUE)

You got it!

|=====================================================================================
| 87% | Go back to your original working directory using setwd().
(Recall that we created the variable old.dir | with the full path for
the orginal working directory at the start of these questions.)
setwd(old.dir)

Excellent work!

|=======================================================================================
| 90% | It is often helpful to save the settings that you had before you
began an analysis and then go back to | them at the end. This trick is
often used within functions; you save, say, the par() settings that you
| started with, mess around a bunch, and then set them back to the
original values at the end. This isn’t | the same as what we have done
here, but it seems similar enough to mention.

…

|==========================================================================================
| 92% | After you finish this lesson delete the ‘testdir’ directory that
you just left (and everything in it)

…

|============================================================================================
| 95% | Take nothing but results. Leave nothing but assumptions. That
sounds like ‘Take nothing but pictures. | Leave nothing but footprints.’
But it makes no sense! Surely our readers can come up with a better
motto | . . .

…

|===============================================================================================
| 97% | In this lesson, you learned how to examine your R workspace and
work with the file system of your | machine from within R. Thanks for
playing!

…

|=================================================================================================|
100%

## Оценка результата

## Вывод
