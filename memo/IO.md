linux中每启动一个bash代表一个进程，在当前bash中再启动一个/bin/bash那么相当于再fork出来一个进程，并且跟原来的进程是父子关系(可以通过pstree观察)

由于进程间的隔离机制，所以在父进程中定义的变量，在子进程中是看不到的，比如:

在父进程中定义变量a

a=1

echo $a

在当前bash下启动一个新的bash

/bin/bash

此时在子进程中a是没有定义的

echo $a

再退回到父进程中，export a之后，在进入子进程的话，a就可见了

exit

export a

/bin/bash

echo $a



在linux中{}代表程序块。注意{的后面一定先先跟空格，然后命令必须以;结束

当管道的一端是程序块的时候，linux会启动一个新的进程来执行程序块里的逻辑

echo $$和echo $BASHPID都是查看当前bash的进程ID，但是echo $$命令的优先级较高，比管道命令:|的优先级要高，

{ echo $$ ; ll ./ } | { cat ; }打印出来的是父进程的进程ID，因为linux在解释的时候先执行echo $$，所以$$会被先解析成当前进程的进程ID，然后才根据管道命令分别启动两个进程执行代码块

{ echo $BASHPID ; ll ./ ; } | { cat ; }打印出来的是子进程的进程ID，因为linux先会根据管道命令分别启动两个进程执行代码块，然后在子进程中分别执行两个代码块中的命令

echo $$

echo $BASHPID

{ echo $$ ; ll ./ } | { cat ; }

{ echo $BASHPID ; ll ./ ; } | { cat ; }