# Assembly language computing 101
## Set register
section .text
global _start

_start:
    mov rdi,0x1337

## Set multiple registers
section .text
global _start

_start:
    mov rax,0x1337
    mov r12,[0xCAFED00D1337BEEF]
    mov rsp,[0x31337]
    
## Add to register
section .text
global _start

_start:
    add rdi,0x331337

## linear equation register
section .text
global _start

_start:
    imul rdi,rsi
    add rdi,rdx
    mov rax,rdi
## intger division
section .text
global _start

_start:
    mov rax,rdi
    div rsi

## Modulo operation
section .text
global _start

_start:
    mov rax,rdi
    div rsi
    mov rax, rdx

## Setting upper byte
section .text
global _start
_start:
    mov ah, 0x42

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

## Increment:
section .text
global _start

_start:
  mov rax,[0x404000]
  mov rbx, 0x1337             
  add [0x404000], rbx

  ## Little endian write
  section .text
  global _start

  _start:
     mov rax,0xdeadbeef00001337
     mov [rdi],rax
     mov rbx,0xc0ffee0000
     mov [rsi],rbx
## Memory sum
section .text
global _start

_start:
  mov rax,[rdi+0]
  mov rbx,[rdi+8]
  add rax,rbx
  mov [rsi],rax
## Subtract stack
section .text
global _start
_start:
  pop rbx
  sub rbx,rdi
  push rbx
## Swap stack
section .text
global _start

_start:
  push rdi
  push rsi
  pop rdi
  pop rsi
## Stack average
section .text
global _start

_start:
  mov rax,[rsp]
  add rax,[rsp+8]
  add rax,[rsp+16]
  add rax,[rsp+24]
  mov rbx, 4
  div rbx
  push rax
## absolute jump
section .text
global _start

_start:
  mov rax,0x403000
  jmp rax
## relative jump





