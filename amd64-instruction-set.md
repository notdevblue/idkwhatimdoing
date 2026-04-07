# AMD64 Instruction Set

진행하면서 명령 세트들을 알게 될 거임. 명령 세트들은 프로그래밍 메뉴얼이 아니라 레퍼런스 메뉴얼로 작성되어 있음.
이게 뭔 말이냐면, 각각의 명령은 이건 이런 짓을 함 이라고 작성되어 있지만, "어떻게 이걸 쓰는지" 에 대한 문서는 없음.

[AMD64 프로그래머 메뉴얼](https://docs.amd.com/v/u/en-US/24594_3.37)이나 [Intel 프로그래머 메뉴얼](https://cdrdv2.intel.com/v1/dl/getContent/671200) 에서 명령 셋들을 찾을 수 있음. (물론 다른곳도 인터넷 찾다보면 나옴)

명령을 한줄로 짧게 설명한 사이트도 있음 -> https://www.felixcloutier.com/x86/

AAA 부터 XTEST 까지... 우리가 사용할 수 있는 1500개가 넘는 명령이 있지만, 자주 쓰는건 이것보다 확실히 적음.

어셈블리 소스 코드의 한 줄 형식은 다음과 같음
```
[optional label] 명령
[optional label] 명령 피연산자
[optional label] 명령 피연산자1, 피연산자2
```
이게 조합되게 되면 명령은 opcode 가 되고, 피연산자는 바이트 시퀀스가 됨. 그러면 CPU가 이 명령을 실행할 수 있게 되고.

## Assembly Source

어셈블리 소스에선, NASM 어셈블러는 피연산자들이 `destination, source`(Intel syntax) 로 되어 있길 기대함. GAS 어셈블러는 `source, destination`(AT&T syntax) 로 되어 있길 기대하고.
다양한 CPU용(MC68000, AMD64, ARM 등등...) 어셈블러 문법은 왼쪽 피연산자가 source 인지 destination 인지 지정함.
GAS 어셈블러는 다양한 프로세서(아키택처)에 대해 소스를 어셈블 할 수 있기 때문에, 대부분 source, destinaion 형식임. Intel(NASM) 문법 쓰라고 할 수 있긴 함.

AT&T 는 다음과 같은 느낌:
* `movl %ebx, %eax` -> ebx 값을 eax 로 이동. (데이터 흐름, 예전 Unix 느낌)
NASM 은 다음과 같은 느낌:
* `mov eax, ebx` eax = ebx (변수 대입)

Intel 문법에선 세미콜론(;) 은 주석의 시작을 나타냄. 더 설명 안해도 잘 알 거 같으니까...

명령어 몇개 보기 전에, 주소 지정 방식을 좀 알아야 함.

## Addressing Modes

주소 지정 모드는 명령에 대해 어떤 피연산자와 실행 방법을 결정하는 수단임.
예를 들어 레지스터 피연산자는 특정 레지스터를 나타내고, 메모리 피연산자는 다양한 조합의 오프셋과/또는 레지스터 안에 들어있는 항목에 따라 주소를 지정할 수 있음.

주소 모드를 알아보기 위해, MOV 명령을 사용할꺼임. register 값을 memory 로 복사하거나, memory 값을 register 로 로드하거나 하는 명령어임.

한 설명은 각각 한 주소 지정 모드에 대한 설명임.
https://github.com/mschwartz/assembly-tutorial/blob/main/instruction-set/addressing.asm 파일은 다양한 주소 지정 모드를 알려줌

### Register Operands

메모리가 source 또는 destination 이 되는 것 대신, 피연산자는 레지스터임.
```asm
mov rax, rbx ; rbx 레지스터에 담긴 내용물을 rax 로 옮김
```

### Direct Memory Operands (Immediate operands 로 더 잘 알려져 있음)

이건 상수를 레지스터로 옮김. 상수는 opcode 다음으로 명령에 인코딩 됨.

```asm
mov rax, 10 ; source operand 는 상수임
```

#### Indirect Operands

이 모드는 레지스터를 연산을(로드 또는 저장 등..) 진행할 메모리의 주소로 사용함.

```asm
mov [rax], rbx ; rbx 레지스터의 내용물을 rax 레지스터에 담겨 있는 메모리 주소로 저장함
```

### Indirect with Displacement

이 모드는 레지스터를 메모리 위치의 기본 주소로 사용하고, 고정된 만큼의 오프셋을 더하여 최종적으로 연산을 진행할 메모리 주소를 얻어냄.

```asm
mov rax, [rbx+24] ; rbx 가 가리키는 메모리 주소 + 24 만큼의 메모리 주소에 있는 것을 rax 에 넣음
```

요거의 목적은 구조체의 멤버들을 접근하기 위해 존재함.

```c++
struct {
    char *name;
    char *address;
    char *phone;
} person;
person.name = nullptr;
person.address = nullptr;
person.phone = nullptr;
```
이런 식으로 있다고 하면, 어센블리로는 이렇게 할 수 있음.

```asm
NAME equ 0
ADDRESS equ 8
PHONE equ 12

mov rsi, person ; RSI(Source Index) 에 person 주소를 넣음
mov rax, 0      ; nullptr
mov NAME[rsi], rax      ; Intel syntax 기준 offset[register] 는 [register + offset] 과 동일
mov ADDRESS[rsi], rax   ; 구조체 필드 접근 느낌을 살리려고 일부로 이렇게 쓴다고 함
mov PHONE[rsi], rax     ; GAS 에선 rsi[rsi] 은 NAME(%rsi)
```

요 주소 지정 모드의 다른 사용 방법은 "C" 언어 같은 언어에서의 스택 프레임을 위해 사용할 수 있음. 특히 서브루튼 부르는 경우.
서브루틴은
* 호출될 때 스택에 뭔가 인자들을 값 형식이나(int 같은) 레퍼런스(구조체 또는 문자열의 주소나 무언가..) 를 넘겼을 수 있음.
* 로컬 변수들을 가지고 있을 수 있음.
서브루튼이 재귀적으로 불렸을 때, 각각의 재귀 호출은 인수와 로컬 변수들을 위해 스택을 준비해 둬야 함.

RBP(Base Pointer) 레지스터는 "C" 언어의 스택 규칙 중 함수를 호출할 때 사용되는 스택 프레임에 대해 사용됨.
* Call Stack -> 함수 호출 흐름 전채를 담는 메모리 영역
* Stack frame -> 특정 함수 하나의 실행 컨텍스트 (지역변수, 반환 주소 등...)

함수 호출 시, 인자들을(오른쪽부터 왼쪽 순서로) Stack 에 Push 함.
`foo(a, b, c)` 의 경우, 컴파일러는 c -> b -> a -> 순서로 stack 에 push 함. (cdecl 기준)
* `printf(fmt, ...)` 와 같은 가변 인자 때문에, 오른쪽부터 왼쪽으로 push 하게 됨. fmt 가 항상 top 에 오기 때문에, 스택 top 근처에 옴. 고로 첫 인자를 고정 위치에서 찾을 수 있음
```bash
printf("%d %d %d %d", 1, 2, 3, 4, 5);

stack 은 Last In First Out 구조고, 최상단 메모리부터 아래로 내려가는 방식이니:
10 "%d %d %d %d" # <- top
09 1
08 2
07 3
06 4
05 5
04 ...
03 ...
02 ...
01 ...
```

함수를 호출할 시, **RBP**값은 다음과 같은 의미를 가질 수 있음 (주소가 8바이트라고 기준.. RBP니까)
```
|-----------------|
| arguments       | <- rbp + 양수 (16 부터...)  # x86-64 기준, 레지스터 6개에 인자 전달되기 때문에 stack에는 7번째 인자부터 존재함.
|-----------------|
| return address  | <- rbp + 8                  # 실행 종료 후 돌아갈 주소
|-----------------| 
| old RBP         | <- rbp 0                    # caller 함수의 RBP 값
|-----------------|
| local variables | <- rbp - 양수
|-----------------|
```

```assembly
// source
void func() { }

; 는:
func:
    pushl %ebp      ; prologue: caller 의 EBP stack에 저장 
    movl %esp, %ebp ; prologue: 새 frame base 설정
    leave           ; epilogue: EBP 복원, ESP 복원
    ret             ; return

// source
void cup(int water_amount) {
    int dust, total;
    dust = 5;
    total = water_amount + dust;
}

cup:
    pushl   %ebp
    movl    %esp,       %ebp        ; frame base 설정
    subl    $8,         %esp        ; 로컬 변수 두개 (dust, total) 공간 확보

    movl    $5,         -4(%ebp)    ; dust = 5
    movl    -4(%ebp),   %eax        ; eax = dust
    addl    8(%ebp),    %eax        ; eax += water_amount
    movl    %eax,       -8(%ebp)    ; total = eax

    leave
    ret
```

함수가 함수 부르는 경우는, 다음과 같은 stack 구조가 됨.
```C
void coffee(int x) {
    int local = 154;
    vodka(x);
}

void vodka(int x) {
    int another_local = 40;
}

HIGH (0xffff)
|--------------------|
| # coffee frame     |
| int x              | rbo + 16
| return address     | rbp + 8
| old RBP            | rbp 0
| int local          | rbp - 4
|--------------------| 
| # vodka frame      |
| int x              | rbo + 16
| return address     | rbp + 8
| old RBP            | rbp 0
| int another_local  | rbp - 4
|--------------------|
LOW (0x0000)
```

### Indirect with displacement and scaled index

요 주소 지정 방식은 배열 요쇼를 접근하는데 사용됨.
* byte 배열의 요소는 1 byte
* word 배열의 요소는 2 bytes
* dword 배열의 요소는 4 bytes
* qword 배열으 요소는 8 bytes

```assembly
    member(rsi, rbx, 4) ; member 가 0 일 수 있음.
```
* member -> 구조체 멤버 오프셋 (Indirect with Displacement)
* rsi -> base (구조체/배열 시작 주소)
* rbx -> 인덱스
* 4 -> 크기

이건 이런 느낌
```c++
int rbx = 0;
rsi->member[rbx * 4];
```

C 의 `arr[i]` 가 `*(arr + i * sizeof(T))` 가 되는 의미. `*(member + rbx * 4)`
