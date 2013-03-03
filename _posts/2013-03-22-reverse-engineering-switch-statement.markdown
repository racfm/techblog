---
layout: post
date: 2013-03-22
title: "Reverse Engineering: "Switch" statement"
tags: [$TAGLIST]
author: Oprea Matei
---

In this article, i will present a simple program that uses "switch" statement
and then i'll try to reverse engineer the compiled version of the program to
figure out how we can determine the usage of "switch" statement in the assembly
code. First i will show you the program i wrote for this article. It's a very
simple switch statement program: 

{% highlight cpp %}
#include <stdio.h>   

int main () {
	int a=8;
	int b=9;
	int c=10;
	int input;
	scanf ("%d",&input);
	switch (input) {
		case 8:
		printf ("a");
		break;
		case 9:
		printf ("b");
		break;
		case 10:
		printf ("c");
		break;
		default:
		printf ("This is techblog");
		break;
	}
	return 0;
}
{% endhighlight %}
[1]: ../img/switch_compiled_ex1.png
I compiled the program and then i ran it, using "8" for input. It gave me the
following output, how it was expected to be: 
   
![Switch 1][1]

Now we will interpret the assembly code for our program :

In assembly language programming, the function prologue is a few lines of code
at the beginning of a function, which prepare the stack and registers for use
within the function ( Similarly, the function epilogue appears at the end of
the function, and restores the stack and registers to the state they were before the
function was called )

In our program, the function prologue will be: 
`
int main () {
	   0x0804845c <+0>:	push   %ebp
	      0x0804845d <+1>:	mov    %esp,%ebp
	         0x0804845f <+3>:	and    $0xfffffff0,%esp
		    0x08048462 <+6>:	sub    $0x20,%esp
`
Now we will analyze the input :

`    int a=8;
	       0x08048465 <+9>:	movl   $0x8,0x14(%esp) // 0x8, our value for a
	       						// 8
     int b=9;
	         0x0804846d <+17>:	movl   $0x9,0x18(%esp) // 0x9 our value for b
	 						// 9
     int c=10;
		   0x08048475 <+25>:	movl   $0xa,0x1c(%esp) // 0xa our value for c 
	   						// 10
`

As we initialized the first three variables we want to load into registers
the input, so we can compare what we get with the initialzed ones.

`
scanf("%d",&input);
   0x0804847d <+33>:	lea    0x10(%esp),%eax // we load the address of the
   //location reference by the source operand to the destination operand
      0x08048481 <+37>:	mov    %eax,0x4(%esp) 
         0x08048485 <+41>:	movl   $0x8048588,(%esp)
	    0x0804848c <+48>:	call   0x8048360 <__isoc99_scanf@plt> // we
	    // call the scanf function in C
`

After we called the "scanf" function, we have to go through switch statement. How
does the switch function works ? Well, it's allowing the value of a variable or
expression to control the flow of a program executiion via a "goto". The main
reason for using switch statement is for improving clarity and reducing
repetitive code.
Now, i'll present you the switch statement in assembly language : 

`
case 8: // if the input is 8 
	printf("a\n");
	    		0x080484a4 <+72>:	movl   $0x804858b,(%esp) //this is what we have to
   						//print
      0x080484ab <+79>:	call   0x8048330 <puts@plt // 0x8048330 is the address of the
      //puts function. To be more precise, it points to the entry for puts() in
      //the program linker table (PLT)
      //You can see an description for PLT right here [2]. 
`
Actually, the compiler was smart enough to see that he can calls the puts()
functions which is executed faster than printf().
`
	break;
        		0x080484b0 <+84>:	jmp    0x80484db <main+127> // if it is true
				// we jump to <main+127> which will go into 
				// break 
	return 0;
			x080484db <+127>:	mov 	$0x0,%eax
				// end of the program.
`


[2] - http://stackoverflow.com/questions/5469274/what-does-plt-mean-here/5469334#5469334
