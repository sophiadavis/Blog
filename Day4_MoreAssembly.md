# Day 4 at Hacker School
## I dream of circuits and write some Assembly.

-----

Finally, I got around to watching [a YouTube video on how shift registers work](https://www.youtube.com/watch?v=6fVbJbNPrEU). (I think?) a shift register is a chain of elements, which each have an input and output. A bit array of data enters the first element, which processes the first chunk of the data and passes the rest as its output, which becomes input to the next element in the circuit. That element processes the next chunk, and passes the rest as output, etc. It's a key piece of hardware in the LED strip I played with yesterday. I really want to build a circuit with a shift register, but I don't know enough about circuits/electricity to follow [the tutorial from the same YouTube stream](https://www.youtube.com/watch?v=oB_pz18AinI) by myself. 

-----

Bummed by my lack of electrical engineering background, I went to brush my teeth, and instead got caught up in a conversation about Assembly, different processor builds, and basic programs I could write in Assembly. Eric agreed to help me write some Assembly programs of my own. Yay!

We started by writing a super simple program in C (all it does is set the return value):

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

Notice that this is a more-or-less un-optimized. A more optimized version looks like this (compilers are scary):  

```
	.section	__TEXT,__text,regular,pure_instructions
	.globl	_main
	.align	4, 0x90
_main:
Leh_func_begin1:
	pushq	%rbp
Ltmp0:
	movq	%rsp, %rbp
Ltmp1:
	movl	$42, -8(%rbp)
	movl	-8(%rbp), %eax
	movl	%eax, -4(%rbp)
	movl	-4(%rbp), %eax
	popq	%rbp
	ret
Leh_func_end1:

	.section	__TEXT,__eh_frame,coalesced,no_toc+strip_static_syms+live_support
EH_frame0:
Lsection_eh_frame:
Leh_frame_common:
Lset0 = Leh_frame_common_end-Leh_frame_common_begin
	.long	Lset0
Leh_frame_common_begin:
	.long	0
	.byte	1
	.asciz	 "zR"
	.byte	1
	.byte	120
	.byte	16
	.byte	1
	.byte	16
	.byte	12
	.byte	7
	.byte	8
	.byte	144
	.byte	1
	.align	3
Leh_frame_common_end:
	.globl	_main.eh
_main.eh:
Lset1 = Leh_frame_end1-Leh_frame_begin1
	.long	Lset1
Leh_frame_begin1:
Lset2 = Leh_frame_begin1-Leh_frame_common
	.long	Lset2
Ltmp2:
	.quad	Leh_func_begin1-Ltmp2
Lset3 = Leh_func_end1-Leh_func_begin1
	.quad	Lset3
	.byte	0
	.byte	4
Lset4 = Ltmp0-Leh_func_begin1
	.long	Lset4
	.byte	14
	.byte	16
	.byte	134
	.byte	2
	.byte	4
Lset5 = Ltmp1-Ltmp0
	.long	Lset5
	.byte	13
	.byte	6
	.align	3
Leh_frame_end1:


.subsections_via_symbols
```

Crazy! 

Eric helped me reduce the first version to an even more simple Assembly program, which I could use as a template for other programs:

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
running it (```./basic```), and then checking the return value (```echo $?```). It was 42!

I played around with [conditionals](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/conditional.s) and [loops](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/loops.s) (using comparisons and jump statements to execute different sections of code), learned a bit of [how to use constants](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/helloworld.s), called C's [printf](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/print.s) (Eric did most of this one), and used a [function call with a local variable](https://github.com/sophiadavis/Learning-x86-Assembly/blob/master/localVars.s) (although not necessary for the task at hand). All any of my programs (except print and hello world) do is set the return value of the 'main' function, so you have to check that they work by running ```echo $?```.

-----

Some things to note are that the 'prolog' and 'epilog' are part of the Assembly's calling convention -- all functions should start with the prolog and finish with the epilog. Calling convention also specifies which registers contain data that must be saved elsewhere by caller and callee (so no data is overwritten), that parameters should be pushed to the stack last to first (so the callee can reference them easily with respect to the base pointer), etc. It's also important to note that addresses in the stack grow down towards lower numbers, while memory is still written upwards. So in order to allocate space for a local variable, you need to subtract from the address stored in RSP. That's pretty confusing, and my notebook got filled up with lots of drawings of stacks with arrows pointing to them. 

I think [this](http://www.cs.princeton.edu/courses/archive/spr11/cos217/lectures/15AssemblyFunctions.pdf) is a pretty helpful guide on calling conventions, but it's kind of confusing because it's written for a 32 bit processor (so whenever they deal with 4 bits, I had to deal with 8, etc. Thanks Eric!), and all the drawings of the stack are upside down (I think).

Here are some more cool things I learned about Assembly syntax:  

* .text or .data or .asczi all specify the type of the thing that follows. 

* ```push``` and ```pop``` are cool because they automatically update the stack pointer as necessary. 
* Whenever a function is called, an instruction pointer (a return address -- the address of the next instruction to be executed after the function returns -- stored in register RIP) is also pushed on to the stack. 
* The ```ret``` keyword automatically pops this from the stack, so the program can continue running.
* Whether register names start with 'R' or 'E' or something else has something to do with processor type and maintaining backwards compatibility...?

-----


I got stuck trying to deal with caller-save and callee-save registers. Something is wrong with my stack pointers! Hopefully someone can help me work that out. My next goal is to use Assembly to write a function to calculate the factorial of a number, then I'll probably stop. I have scratched my itch for Assembly, unless it involves blinking lights.






 