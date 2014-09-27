# Day 3 at Hacker School
## I read a lot about Assembly, and played with an Arduino. 

-----

I started off the day reading and taking notes on [Hardware for AVR-Assembler-Programming](http://www.avr-asm-tutorial.net/avr_en/beginner/). "AVR" is a type of microcontroller (= mini computer) produced by [Atmel](http://www.atmel.com/products/microcontrollers/avr/), and it's also used to refer to the variety of Assembly language used to program AVR chips. Arduino Unos are based on the AVR ATmega328 chip. I learned more about concepts like:  

|Thing|Explanation|  
| ----- | :----- |  
|Registers|blocks for storing data; directly accessible to the CPU; size varies from system to system|    
|Ports|'gates' between the CPU and internal/external hardward and software; can hold up to 8 bits (on an AVR at least); each has a fixed address|  
|MCUCR|MCU (a port) General Control Register -- controls the general properties of a chip|  
|SRAM|Static RAM -- memory that is not directly accessible to the CPU -- so registers are usually used as intermediate storage if you want to access data here; can be accessed using fixed addresses or using pointers; this is where the Stack is!|  
|Assembly syntax|Lots of manipulating/moving around/setting bits & bytes, but more ways to control program flow than I expected (using macros and subroutines, conditional jumps, and labels for sections of code), and some commands that might make accessing the stack correctly somewhat more straightforward (like pushing and popping of stack frame pointers)... and lots of other stuff! Each instruction is executed in 1 unit of the processor's clock cycle, which makes it ideal for the LED stuff I poked at later in the day|

Then, I learned that this guide was written with Intel-style assembly syntax ... and to play with writing inline-assembly to run on an Arduino, I would need to use AT&T syntax. They differ, for example, in how registers are named, and the ordering of command arguments, more on that [here](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html). Both Intel and AT&T also differ from the syntax used in the [cool virtual processor](http://ivanzuzak.info/FRISCjs/webapp/) that I played with a bit, too. Who knew there were so many types of Assembly. Bummer.

--------------

I found the Clang option to compile C to AT&T syntax (and also some [documentation](http://llvm.org/docs/CommandGuide/llc.html)!):

```
clang programName.c -S -mllvm --x86-asm-syntax=att -o nameForCompiledFile
```
So I poked around with some more little C programs.

Meanwhile, Alex taught me to solder, and I hooked some wires up to an old telephone for use in Hacker School's future Robot Bartender.

--------------

I was starting to fade, writing inline Assembly proved too much for today. So I asked Dana to borrow her Arduino. After running some of the tutorial programs, I started looking at [some of the code she'd written](https://github.com/danasf/simplepixel/blob/master/simple.ino) to control an LED strip.  The lights run only at a certain frequency, so in order to control them, you need to know exactly how many processor instruction cycles will be needed to pass each byte of data (? -- I'm still not quite sure how Dana figured this all out.). Today I just used C for-loops to manipulate Dana's inline Assembly -- I managed to make white lights move down the strip and then green lights, and I also did some fancy stuff with lots of colors but I couldn't quite figure out why. Next step will be to manipulate the inline Assembly myself, which will probably be very hard. 

--------------   

At the very end of the day, Eric came over to help explain some of the code my Hello World program had compiled to, which was awesome.
  

