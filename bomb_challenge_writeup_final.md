# Phase-1
## Running the file
Run the file - gdb bomb (Use sestatus (install with sudo apt install policycoreutils) )(it is an executable file already)
## Method to solve
1. disassemble main using the command disas main
2. Set break points at the lines of phase 1 and explode bomb
3. disassemble phase_1
4. Use the nexti command until you get to the strings_not_equal line
We disas the strings_not_equal function and find that the rsi value is moved to rbp, then rbp is moved to rdi
5. since in the before line we moved something(the value to compare eax)(it is a memory address) into esi, read the rsi register using the command x/s $rsi
6. It gives a string called "I turned the moon into something I call a Death Star."
7. Now give run
8. Then enter the string and put c for breakpoints

### Final answer: I turned the moon into something I call a Death Star.

# Phase-2
## Finding conditions for the input
1. If cmp sets the sign flag, then the bomb explodes, hence the numbers are positive (Sign flag is 0 if it is positive, and 1 if it is negative)

cmp DWORD PTR [rsp],0x0

0x0000000000400dd2 <+17>:  js     0x400ddb <phase_2+26>
 

2. This part checks if we have 5 inputs

cmp    eax,0x5

0x0000000000401325 <+51>:jle    0x40132c <read_six_numbers+58>

3. The read six numbers actually just appends values into the stack and registers nothing more.

4. This part only does some computation, hence we set a breakpoint and try analyzing this part with various inputs

## Finding the actual computation of the program

 0x0000000000400dee <+45>:	add    eax,DWORD PTR [rsp+rbx*4-0x4]
 
 0x0000000000400df2 <+49>:	cmp    DWORD PTR [rsp+rbx*4],eax
 
since it is compared to eax we will check eax value using 
info registers eax

eax            0x1                 1

Now we will check what is there after the computation 

x/1dw $rsp+$rbx*4 

0x7fffffffdd44:	1

Hence first number is 1. Similarly, we have to start from the first, but this time, in input, give first as 1, so it will now check the 2nd value

info register eax

eax            0x2                 2

(gdb) x/1dw $rsp+$rbx*4

0x7fffffffdd44:	2

When we give continue 

info register eax

eax            0x4                 4

(gdb) x/1dw $rsp+$rbx*4

0x7fffffffdd48:	3

Similarly, we give continue and find all other numbers

### final answer 1 2 4 7 11 16

# Phase 3
## Condition for the input 
1. No negative numbers and 1
   
 rax =2

 cmp    eax,0x1
 
 0x0000000000400e24 <+32>:	jle    0x400e3c <phase_3+56>

3. first number less than 7

   
cmp    DWORD PTR [rsp+0xc],0x7

0x0000000000400e2b <+39>:	ja     0x400eb3 <phase_3+175>

5. first number greater than 5
   
0x0000000000400e72 <+110>:	cmp    DWORD PTR [rsp+0xc],0x5

0x0000000000400e77 <+115>:	jg     0x400e7f <phase_3+123>

## Actual computation of the program

 0x0000000000400e31 <+45>:	mov    eax,DWORD PTR [rsp+0xc]
 
 0x0000000000400e35 <+49>:	jmp    QWORD PTR [rax*8+0x402220]
   
To solve this phase, I first observed that the program uses the first input number to calculate a jump target, so I tested with the input 6 and found that it jumped to address 0x0000000000400e9e. At this location, the program sets eax to 0 and then continues into a sequence of addition and subtraction instructions that modify eax. Since the final comparison checks whether eax equals the second input number, the goal is to choose a first input that causes the program to jump to a block where the arithmetic operations cancel out, keeping eax unchanged. By examining the jump table, I found that the jump we need is to 0x0000000000400e97, because this section contains balanced addition and subtraction instructions that leave eax at 0. Therefore, the correct first number to reach this jump target is 4, and since eax stays at 0, the second input must also be 0. Hence, the solution for this phase is the pair: 4 0.
   
### final answer: 4 0
   
phase 4
 0x0000000000400f1b <+29>:	cmp    eax,0x2
   0x0000000000400f1e <+32>:	jne    0x400f27 <phase_4+41>
number of input is 2 only
