# Day 4 at Hacker School
## I dream of circuits and write some Assembly.

-----

Finally, I got around to watching [a YouTube video on how shift registers work](https://www.youtube.com/watch?v=6fVbJbNPrEU). A shift register is a chain of inputs and outputs. A bit array of data enters the first element, which processes the first chunk of the data and passes the rest as its output, which becomes input to the next element in the circuit, etc. It's a key piece of hardware in the LED strip I played with yesterday. I really want to build one, but I don't know enough about circuits/electricity to follow [the tutorial from the same YouTube stream](https://www.youtube.com/watch?v=oB_pz18AinI) by myself. 

-----

I went to brush my teeth, and instead got caught up in a conversation about Assembly, different processor builds, and basic programs I could write in Assembly with Eric and someone else. Eric agreed to help me write some Assembly programs of my own. We started by writing a super simple program in C (all it does is set the return value):

```
int main(void) {  
    return 42;  
}  
```
compiling it to x86 AT&T syntax assembly:

```
clang basic.c -S -mllvm --x86-asm-syntax=att -o basic.s
```
and looking at the file:

```
	.section	__TEXT,__text,regular,pure_instructions
	.globl	_main
	.align	4, 0x90
_main:                                  ## @main
	.cfi_startproc
## BB#0:
	pushq	%rbp
Ltmp2:
	.cfi_def_cfa_offset 16
Ltmp3:
	.cfi_offset %rbp, -16
	movq	%rsp, %rbp
Ltmp4:
	.cfi_def_cfa_register %rbp
	movl	$42, %eax
	movl	$0, -4(%rbp)
	popq	%rbp
	ret
	.cfi_endproc


.subsections_via_symbols
```
Then Eric helped reduce it to the most simple Assembly program, to use as a template for other programs:

```
.data # constants and variables go here
.text # labels next part as instructions

.globl	_main
_main:
    # prolog
	push	%rbp	# save 'old value' of RBP (base pointer) onto stack 
	mov	%rsp, %rbp	# set the RBP (for this stack frame) to point where current  stack pointer is pointing
	
	mov $42, %rax	# move the value of 0 into RAX (register used for return values)
	
	# epilog
	pop	%rbp 		# retrieve 'old value' of RBP
	ret				# return from the function
```
We checked that this was working by compiling it to binary:

```
gcc basic.s -o basic
```
and then running it (```./basic```) and checking the return value (```echo $?```). It was 42!

I played around with [conditionals](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/conditional.s) and [loops](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/loops.s) (using comparisons and jump statements to execute different sections of code), learned a bit of [how to use constants](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/helloworld.s), called C's [printf](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/print.s) (Eric did most of this one), and used a [local variable inside a function](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/localVars.s) (although not necessary for the task at hand).

Some things to note are that the 'prolog' and 'epilog' are part of the Assembly's calling convention -- all functions should start with the prolog and finish with the epilog. Calling convention also specifies which registers contain data that must be saved elsewhere by caller and callee (so no data is overwritten), and that parameters should be pushed to the stack last to first (so the callee can reference them easily with respect to the base pointer). It's also important to note that addresses in the stack grow down towards lower addresses, while memory is still written upwards. So in order to allocate space for a local variable, you need to subtract from the address stored in RSP. That's pretty confusing, and my notebook got filled up with lots of drawings of stacks with things pointing to them. 

I think [this](http://www.cs.princeton.edu/courses/archive/spr11/cos217/lectures/15AssemblyFunctions.pdf) is a pretty helpful guide on calling conventions, but it's kind of confusing because it's written for 32 bit architecture (so whenever they deal with 4 bits, I had to deal with 8, etc. Thanks Eric!), and all the drawings of the stack are upside down (I think).

Here are some more cool things I learned about Assembly syntax:  

* .text or .data or .asczi all specify the type of the thing that follows. 

* ```push``` and ```pop``` are cool because they automatically update the stack pointer as necessary. Whenever a function is called, a return address (the address of the next instruction to be executed after the function returns) is also pushed on to the stack. 
* The ```ret``` keyword automatically pops this from the stack, so the program can continue running.

-----


I got stuck trying to deal with caller-save and callee-save registers. Something is wrong with my stack pointers! Hopefully someone can help me work that out. My next goal is to use Assembly to write a function to calculate the factorial of a number, then I'll probably stop. I have scratched my itch for Assembly, unless it involves blinking lights.






 