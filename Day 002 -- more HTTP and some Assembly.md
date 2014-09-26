# Day 2 at Hacker School
## I wrote my first "program" in Assembly.

-----

I started the day off by looking at what Kyle and Dana have been working on. They have a string of LED lights connected to a Raspberry Pi, which they're using to build a sound spectrum analyser (?). Kyle talked to me about some of the issues they've been having -- like each of the two different strands of lights uses a different type of chip, and each chip has a different encoding (?) and voltage. They're using a Python library to control the lights. Cool stuff that I wish I understood better.

--------------

I spent longer than I wanted getting my simple http server to a point where I was happy with it. My goal was to:

1. Receive the client's http request
2. Figure out the requested path (assuming a GET request)
3. Serve the files corresponding to the path

I did step 1 yesterday and step 2 was easy, but there were major hangups with step 3... I had added some simple html and css files and a jpeg image to the directory with the script that runs my server. The idea was to serve either a text document or the image, depending on the requested path. So I spent a long time trying to figure out how to add something to the http response that would tell the client "and now go look at this file". After talking to another Hacker Schooler who spent two weeks building a much more involved, complicated, functional server, I realized that I was trying to use the "Rails way." You can't just tell the client to look at one of YOUR local files -- you have to actually give them the content. Once I tacked the actual *bytes* that made up each file into the http response (and set the correct Content-Type header), it finally worked. It was also cool to see that when I linked to the css file from an html file, the client started sending a separate request for the css path, too.

[Here's](https://github.com/sophiadavis/http-server) the complete code, with lots of (hopefully) helpful comments. I should mention that the code is based heavily on [this tutorial](http://www.binarytides.com/python-socket-programming-tutorial/).

--------------

Next, I talked to Dana to see if she had any advice for diving in to Assembly. She's written a fair bit of Assembly to work with LEDs connected to an Arduino, and she showed me what some of her code does. To get my feet wet, I spent the rest of the day compiling silly C programs to Intel x86 (well, after I re-learned some basic C syntax...): 

```
clang programName.c -S -mllvm --x86-asm-syntax=intel -o nameForCompiledFile
```

Hopefully, [StackOverflow](http://stackoverflow.com/questions/10990018/how-to-generate-assembly-code-with-clang-in-intel-syntax) had directed me to the correct flags for Clang. I was having trouble finding documentation on these.

This little program:

```
#import <stdio.h>
#import <stdlib.h>

int main(int argc, char *argv[]) {
    int a = 3;
    int *b = malloc(sizeof(int));
    int g = 9;
    int h = g * a;
    *b = 5;
    printf("Did it work: %p\n", b);
    int c = a + *b;
    printf("Did it work: %i\n", c);
    free(b);
}
```
compiled to this:

```
	.section	__TEXT,__text,regular,pure_instructions
	.globl	_main
	.align	4, 0x90
_main:                                  ## @main
	.cfi_startproc
## BB#0:
	push	RBP
Ltmp2:
	.cfi_def_cfa_offset 16
Ltmp3:
	.cfi_offset rbp, -16
	mov	RBP, RSP
Ltmp4:
	.cfi_def_cfa_register rbp
	sub	RSP, 64
	movabs	RAX, 4
	mov	DWORD PTR [RBP - 4], EDI
	mov	QWORD PTR [RBP - 16], RSI
	mov	DWORD PTR [RBP - 20], 3
	mov	RDI, RAX
	call	_malloc
	lea	RDI, QWORD PTR [RIP + L_.str]
	mov	QWORD PTR [RBP - 32], RAX
	mov	DWORD PTR [RBP - 36], 9
	mov	ECX, DWORD PTR [RBP - 36]
	imul	ECX, DWORD PTR [RBP - 20]
	mov	DWORD PTR [RBP - 40], ECX
	mov	RAX, QWORD PTR [RBP - 32]
	mov	DWORD PTR [RAX], 5
	mov	RSI, QWORD PTR [RBP - 32]
	mov	AL, 0
	call	_printf
	lea	RDI, QWORD PTR [RIP + L_.str1]
	mov	ECX, DWORD PTR [RBP - 20]
	mov	RSI, QWORD PTR [RBP - 32]
	add	ECX, DWORD PTR [RSI]
	mov	DWORD PTR [RBP - 44], ECX
	mov	ESI, DWORD PTR [RBP - 44]
	mov	DWORD PTR [RBP - 48], EAX ## 4-byte Spill
	mov	AL, 0
	call	_printf
	mov	RDI, QWORD PTR [RBP - 32]
	mov	DWORD PTR [RBP - 52], EAX ## 4-byte Spill
	call	_free
	mov	EAX, 0
	add	RSP, 64
	pop	RBP
	ret
	.cfi_endproc

	.section	__TEXT,__cstring,cstring_literals
L_.str:                                 ## @.str
	.asciz	 "Did it work: %p\n"

L_.str1:                                ## @.str1
	.asciz	 "Did it work: %i\n"


.subsections_via_symbols
```

Even much simpler programs were rather complicated...

I also read about CPUs, the flash, registers... stuff like that. Registers are chips that store data. Unlike main memory (which also stores data), registers are located within the Central Processing Unit (CPU). The CPU manipulates the information stored in its various registers (and main memory), in accordance with instructions (written in hex) from the flash storage. Assembly is a human-readable language that is translated into the hex instructions executed by the CPU. (Except this is all a lot more complicated. [This guy](http://www.avr-asm-tutorial.net/avr_en/beginner/) knows better.)


Dana showed me a [cool processor simulator](http://ivanzuzak.info/FRISCjs/webapp/), so I ended up played around with that and wrote my first 'program':

```
ADD R1, 5, R3
MOVE R3, R2
MOVE 0, R3
`END
```
I think it adds 5 to the contents of register R1, and puts the result at R3. Then it copies the contents of R3 to R2, and places a 0 at register R3.

I don’t know what the `END does -- because the simulator's “PC” (program counter -- which keeps track of the next instruction the CPU should execute) keeps increasing. Are more instructions being executed somehow? 

------

### To Do:  
* More research about registers  
* More research on Assembly (different types, syntax)  
* Find out how Clang is actually compiling my programs  
* Write some Assembly embedded inside a C program
* Do cool things with blinking lights     