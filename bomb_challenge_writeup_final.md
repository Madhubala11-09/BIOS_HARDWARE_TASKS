# Phase-1
## Running the file
Run the file - gdb bomb (Use sestatus (install with sudo apt install policycoreutils) )(it is an executable file already)
## Method to solve
1. disassemble main using the command disas main
2. Set break points at the lines of phase 1 and explode bomb
3. disassemble phase_1
4. Use the nexti command until you get to the strings_not_equal line
We disas the strings_not_equal function and find that the rsi value is moved to rbp, then rbp is moved to rdi, then the function string_length is called, where all registers values change.
6. since in the before line we moved something(the value to compare eax)(it is a memory address) into esi, read the rsi register using the command x/s $rsi
7. It gives a string called "I turned the moon into something I call a Death Star."
8. Now give run
9. Then enter the string and put c for breakpoints
    


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


## Finding the actual computation of the program

This part only does some computation, hence we set a breakpoint and try analyzing this part with various inputs

 0x0000000000400dee <+45>:	add    eax,DWORD PTR [rsp+rbx*4-0x4]
 
 0x0000000000400df2 <+49>:	cmp    DWORD PTR [rsp+rbx*4],eax
 
since it is compared to eax, we will check the  eax value using 
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
   
To solve this phase, I first observed that the program uses the first input number to calculate a jump target, so I tested with the input 6 and found that it jumped to address 0x0000000000400e9e. At this location, the program sets eax to 0 and then continues into a sequence of addition and subtraction instructions that modify eax. Since the final comparison checks whether eax equals the second input number, the goal is to choose a first input that causes the program to jump to a block where the addition and subtraction operations cancel out, keeping eax unchanged. By examining the lines that it can jump to, I found that the jump we need is to 0x0000000000400e97, because this section contains balanced addition and subtraction instructions that leave eax at 0. Therefore, the correct first number to reach this jump target is 4, and since eax stays at 0, the second input must also be 0. Hence, the solution for this phase is 4,0.
   
### final answer: 4 0
   
phase 4

 0x0000000000400f1b <+29>:	cmp    eax,0x2
   0x0000000000400f1e <+32>:	jne    0x400f27 <phase_4+41>

number of input is 2 only

 0x400f1b <phase_4+001d>   cmp    eax, 0x2
     0x400f1e <phase_4+0020>   jne    0x400f27 <phase_4+41>
     0x400f20 <phase_4+0022>   cmp    DWORD PTR [rsp+0xc], 0xe
     0x400f25 <phase_4+0027>   jbe    0x400f2c <phase_4+46>	TAKEN [Reason: C || Z]

The input number should be less then 14.

   0x0000000000400f36 <+56>:	mov    edi,DWORD PTR [rsp+0xc]
rsp+0xc is the first input number we give.

   0x0000000000400f2c <+46>:	mov    edx,0xe //edx=14
   0x0000000000400f31 <+51>:	mov    esi,0x0  /esi=0
   0x0000000000400f36 <+56>:	mov    edi,DWORD PTR [rsp+0xc] /edi=first input
   0x0000000000400f3a <+60>:	call   0x400ebf <func4>


   0x0000000000400ec3 <+4>:	mov    eax,edx / eax=14
   0x0000000000400ec5 <+6>:	sub    eax,esi /eax=eax-esi / 14-0=14 / eax=14
   0x0000000000400ec7 <+8>:	mov    ecx,eax /ecx=eax
   0x0000000000400ec9 <+10>:	shr    ecx,0x1f / ecx=14>>31; ecx=0
   0x0000000000400ecc <+13>:	add    ecx,eax / ecx=ecx+eax; ecx=14
   0x0000000000400ece <+15>:	sar    ecx,1 / ecx=ecx/2; ecx=7
   0x0000000000400ed0 <+17>:	add    ecx,esi /ecx=ecx+esi; ecx=14
   0x0000000000400ed2 <+19>:	cmp    ecx,edi /14, first input
   0x0000000000400ed4 <+21>:	jg     0x400ee4 <func4+37> /14>first input
   0x0000000000400ed6 <+23>:	mov    eax,0x0 
   0x0000000000400edb <+28>:	cmp    ecx,edi 
   0x0000000000400edd <+30>:	jl     0x400ef0 <func4+49> /14<first input
   0x0000000000400edf <+32>:	add    rsp,0x8
   0x0000000000400ee3 <+36>:	ret
   0x0000000000400ee4 <+37>:	lea    edx,[rcx-0x1] 
   0x0000000000400ee7 <+40>:	call   0x400ebf <func4>
   0x0000000000400eec <+45>:	add    eax,eax
   0x0000000000400eee <+47>:	jmp    0x400edf <func4+32>
   0x0000000000400ef0 <+49>:	lea    esi,[rcx+0x1]
   0x0000000000400ef3 <+52>:	call   0x400ebf <func4>
   0x0000000000400ef8 <+57>:	lea    eax,[rax+rax*1+0x1]
   0x0000000000400efc <+61>:	jmp    0x400edf <func4+32>


