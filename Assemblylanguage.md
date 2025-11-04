# Assembly language computing 101
## Set register
section .text
global _start

_start:
    mov rdi,0x1337;

## Set multiple registers
section .text
global _start

_start:
    mov 
## Add to register
section .text
global _start

_start:
    mov 

## linear equation register
section .text
global _start

_start:
    mov 
## intger division
section .text
global _start

_start:
    mov 

## Modulo operation
section .text
global _start

_start:
    mov rax,rdi;
    div rsi;
    mov rax, rdx;

## Setting upper byte
section .text
global _start
_start:
    mov ah, 0x42;

## Efficient modulo
section .text
global _start

_start:
   mov al,dil
   mov bx, si

## Byte extraction:
section .text
global _start

_start:
  shr rdi, 32
  mov al, dil

## Bitwise and
section .text
global _start

_start:
  and rdi,rsi
  lea rax,[rdi]

## Check even:
section .text
global _start

_start:
  and rdi,1
  xor rax,rax
  xor rdi,1
  or rax, rdi

## Read memory:
section .text
global _start

_start:
  mov rax,[0x404000]

## Write memory:
section .text
global _start

_start:
  mov [0x404000],rax

#Increment:
section .text
global _start

_start:
  mov rax,[0x404000]
  mov rbx, 0x1337             
  add [0x404000], rbx