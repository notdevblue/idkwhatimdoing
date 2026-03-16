# x64/AMD64 Registers

> 레지스터 구조와 관련해선: https://en.wikipedia.org/wiki/X86#x86_registers, https://en.wikipedia.org/wiki/X86#Structure

Intel 과 AMD 프로세서는 변태적인 기능들(하드웨어 비디오 디코딩) 제외하곤 동일한 레지스터를 가지고 있음.
이 문서에선 x64와 AMD64를 치환 가능한 것으로 두고 설명한다고 함.

## General Purpose Registers (GPR: 범용 레지스터)

4개의 범용 레지스터가 있음, A B C D. (네이밍 겁나 단순하네 A B C D...)
레지스터 이름을 명시적으로 사용하지는 않지만, 레지스터/컨텐츠의 크기가 중요함.
* Byte (8bit) 값의 경우, AL 또는 AH, BL/BH, CL/CH, DL/DH 를 사용함. L 은 'low order byte', H 는 'high order byte'
* Word (16bit) 값의 경우, AX, BX, CX, DX 를 사용함. (X 자체에 특별한 의미는 없음, 이름 규칙 맞추려고...)
* 32bit word 값의 경우, EAX, EBX, ECX, EDX 를 사용함 (E -> Extended AX)
* 64bit word 값은, RAX, RBX, RCX, RDX (R -> 64-bit Register AX)

64 bits 보다 작은 사이즈 레지스터를 사용하는 경우, 레지스터의 남은 비트들은 영향받지 않음.
AX 가 0x0102 있고, AL 에 0x03 로드하면, AX 는 0x0103 이 됨.
byte 레지스터에 값 넣고, word 레지스터로 더하는 경우 이슈임.
레지스터에 로드/저장을 좀 더 효율적으로 하기 위해 몇가지 꼼수도 잇긴 함. (근데 그건 안 알려주네? 왜??)

AMD64 랑 x64는 8개의 추가 범용 레지스터가 있음, R8, R9, R10, R11, R12, R13, R14 그리고 R15. (64-bit mode only)
요것들은 8, 16, 32, 64 비트 레디스터로 접근됨. R8-R15 (64bits), R8D-R15D (32bits), R8W-R15W (16bits), 그리고 R8B-R15B (8bits).
아마도 네이밍이 요모양인 이유는
* R8 -> 64bit word 는 기존에 R... 네이밍
* R8D -> dword
* R8W -> word
* R8B -> byte

## Special Purpose Registers (SPR: 특별 기능 레지스터)

RCX/ECX/CX (CX) 레지스터는 특정 명령어에 대한 카운터 역할도 함.
AMD64 명령체계는 채우기, 복사, 메모리 비교, 그리고 루프를 돌며 요 레지스터를 bytes/words/dwords/qwords 의 숫자로 사용하면서 채우기/복사/비교를 수행하는 명령들을 포함하고 있음.
요 특별한 루프 명령은 요 레지스터를 루프 카운터로도 사용함.
(Counter 라 C)

RSI/ESI/SI 랑 RDI/EDI/DI 레지스터는 채우기, 복사, 비교 명령을 수행하는데 사용되는 범용 "source"와 "destination" 레지스터임.
(Source Index 라 SI, Destination Index 라 DI)

RBP 레지스터는 기본 주소 레지스터나 high level 언어 컴파일러가 함수의 스택 프레임(인자, 반환할 주소, 스택에 할당된 로컬 변수들)을 유지하는데 사용되는 범용 레지스터임.
(Base Point 라서 BP)

## CPU Control Registers

### Stack

RSP 레지스터는 프로세서 스택 최상단 주소를 가지고 있음. (Stack Pointer 라 SP)
레지스터 값 보존을 위해 스택에 푸쉬할 수 있고, 팝 하여 값을 다시 얻을 수 있고, 인덱스를 사용하여 스택에 이미 존재하는 값을 접근할 수 있음.

### Instruction Pointer

RIP 레지스터는 다음으로 실행된 명령의 주소를 가지고 있음. (Instruction Pointer 라 IP)
CPU는 명령을 실행하면서 자동으로 올바를 숫자를 더해 다음 실행될 명령어를 가리키게 만듬.
서브루틴을
* 호출하는 경우, RIP는 RSP 스택에 푸쉬되고 RIP 는 서브루틴의 주소로 로드됨.
* 반환하는 경우, 호출 전 푸쉬된 RIP가 스택에서 팝 되어 RIP에 들어감, 실행은 호출 이후 계속 진행됨.

### Flags

FLAGS 레지스터는 x86 CPU의 현재 상태를 나타내는 값들을 가지고 있음.
모든 비트가 사용되는건 아님. (https://en.wikipedia.org/wiki/FLAGS_register 요기에 자세히 명시되어있다고 함)

CPU가 설정하는 FLAGS 의 예시 비트 중 하나는 Carry Flag(올림수 플래그) 임, 산술 연산 이후 올림수가 발생한 경우 설정됨.
예를 들어 255가 담겨있는 AL 레지스터에 1을 더하는 경우, AL=0 Carry=1 을 얻게 됨. AL=254에 1을 더하면 Carry=0.

프로그램이 설정하는 FLAGS 의 예시 비트 중 하나는 Direction Flag 임.
* 0인 경우, 채우기/복사하기/기타등등 명령은 시작 주소부터 앞으로 진행함. (SI, DI 를 증가시킴)
* 1인 경우, 채우기/복사하기/기타등등 명령은 시작 주소부터 뒤로 진행함. (SI, DI 를 감소시킴)

FLAGS 레지스터는 쓰라고 있는거긴 하지만, 대부분 Carry/Direction bit 만 직접적으로 사용할 거임.
Carry bit 을 통해 함수의 true/false 를 반환한다거나..
CLC/STC 명령을 통해 Carry bit 를 지우거나 설정하거나..
(CLear Carry bit, SeT Carry bit)

다양한 명령어들이 내부적으로 Carry/Zero bit 을 사용함. (zero flag -> 산술 연산 결과를 확인하기 위해 사용됨, 1인 경우 결과가 0임, 아닌 경우 리셋됨.)
프로그래밍적으로 요 비트들을 설정하거나 지울 수 있음.

(쓰면서 넣은거. carry, sign, overflow, zero... 는 뭔가 산술 연산 시 자주 쓸 듯 함)
