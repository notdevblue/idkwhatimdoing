# Commonly Used Instructions

## Arithmetic

```assembly
ADC - ADd with Carry
ADD - 두 레지스터 더함
DEC - 1 감소
DIV - 부호 없는 나눗샘
IDIV - 부호 있는 나눗셈
IMUL - 부호 있는 곱샘
INC - 1 더함
MUL - 부호 없는 곱샘
NEG - 2의 보수로 만듬
SBB - 빌림 사용한 빼기 (carry flag)
SUB - 뺴기
LEA - 주소 계산 결과 레지스터에 저장 (주소 그 자체를 저장함, 주소에 있는 값을 저장하는거 아님)
```

## Boolean Algebra

```assembly
AND - AND 연산이죠 뭐..
NOT - 1의 보수 (비트 반전시키는거)
OR - OR 이죠..
XOR - 이것도 뭐..
TEST - AND 수행하되, 플래그만 설정하고 결과를 저장하지 않음.
```

## Branching and Subroutines

```assembly
CALL - 서브루틴, 함수, 프로시저 호출
SYSCALL - OS 함수 호출
```

[분기와 서브루틴](https://github.com/mschwartz/assembly-tutorial/blob/main/README.md#branching-and-subroutines)