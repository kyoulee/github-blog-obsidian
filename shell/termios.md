
터미널 제어를 하다모면 termios를 접하게 된다

다음은 ```termios.h```파일을 가져온 것으로 특정 속성을 넣는 것으로 터미널의 옵션을 설정 할 수 있게된다.

```c
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define ECHOKE 0x00000001 /* visual erase for line kill */
#endif /*(_POSIX_C_SOURCE && !_DARWIN_C_SOURCE) */
#define ECHOE 0x00000002 /* visually erase chars */
#define ECHOK 0x00000004 /* echo NL after line kill */
#define ECHO 0x00000008 /* enable echoing */
#define ECHONL 0x00000010 /* echo NL even if ECHO is off */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE
#define ECHOPRT 0x00000020 /* visual erase mode for hardcopy */
#define ECHOCTL 0x00000040 /* echo control chars as ^(Char) */
#endif /*(_POSIX_C_SOURCE && !_DARWIN_C_SOURCE) */
#define ISIG 0x00000080 /* enable signals INTR, QUIT, [D]SUSP */
#define ICANON 0x00000100 /* canonicalize input lines */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define ALTWERASE 0x00000200 /* use alternate WERASE algorithm */
#endif /*(_POSIX_C_SOURCE && !_DARWIN_C_SOURCE) */
#define IEXTEN 0x00000400 /* enable DISCARD and LNEXT */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define EXTPROC 0x00000800 /* external processing */
#endif /*(_POSIX_C_SOURCE && !_DARWIN_C_SOURCE) *
#define TOSTOP 0x00400000 /* stop background jobs from output */
#if !defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE)
#define FLUSHO 0x00800000 /* output being flushed (state) */
#define NOKERNINFO 0x02000000 /* no kernel output from VSTATUS */
#define PENDIN 0x20000000 /* XXX retype pending input (state) */
#endif /*(_POSIX_C_SOURCE && !_DARWIN_C_SOURCE) */#define NOFLSH 0x80000000 /* don't flush after interrupt */

typedef unsigned long tcflag_t;
typedef unsigned char cc_t;
typedef unsigned long speed_t

struct termios {
tcflag_t c_iflag; /* input flags */
tcflag_t c_oflag; /* output flags */
tcflag_t c_cflag; /* control flags */
tcflag_t c_lflag; /* local flags */
cc_t c_cc[NCCS]; /* control chars */
speed_t c_ispeed; /* input speed */
speed_t c_ospeed; /* output speed */
};
```

## struct

구조체는 다음과 같은 방식으로 구분된다.

|**멤버 명칭**|**타입**|**설명**|**주요 역할**|
|---|---|---|---|
|**`c_iflag`**|`tcflag_t`|Input Flags|입력 시 문자 변환 (예: `CR`을 `NL`로 변환)|
|**`c_oflag`**|`tcflag_t`|Output Flags|출력 시 문자 변환 (예: 출력 지연, 탭 처리)|
|**`c_cflag`**|`tcflag_t`|Control Flags|하드웨어 제어 (데이터 비트, 패리티, 정지 비트)|
|**`c_lflag`**|`tcflag_t`|**Local Flags**|**사용자 인터페이스(에코, 정규 모드 등) 제어**|
|**`c_cc[]`**|`cc_t`|Control Chars|특수 문자 설정 (VEOF, VINTR, VERASE 등)|

상위와 같이 생각보다 다양한 옵션을 새밀히 조정할 수 있는 flag들이 있으며 이에 대한 설정 flag는 다음과 같다

## flag

다음은 flag에 설정할 수 있는 옵션을 나열하였다.


### control chars

|**매크로**|**색인(Index)**|**활성화 조건**|**설명**|
|---|---|---|---|
|**VEOF**|0|ICANON|파일 끝(End of File) 신호|
|**VEOL / VEOL2**|1 / 2|ICANON|추가적인 줄바꿈 문자|
|**VERASE**|3|ICANON|한 글자 삭제 (Backspace)|
|**VWERASE**|4|ICANON + IEXTEN|한 단어 삭제|
|**VKILL**|5|ICANON|입력 중인 한 줄 전체 삭제|
|**VREPRINT**|6|ICANON + IEXTEN|현재 입력 버퍼의 내용을 다시 출력|
|**VINTR**|8|ISIG|인터럽트 시그널 (`Ctrl+C`)|
|**VQUIT**|9|ISIG|종료 시그널 (`Ctrl+\`)|
|**VSUSP / VDSUSP**|10 / 11|ISIG|작업 일시 중지 (Suspend)|
|**VSTART / VSTOP**|12 / 13|IXON / IXOFF|흐름 제어 시작/중지 키|
|**VLNEXT**|14|IEXTEN|다음 문자를 해석하지 않고 그대로 입력|
|**VDISCARD**|15|IEXTEN|출력을 버림|
|**VMIN / VTIME**|16 / 17|**!ICANON**|비정규 모드에서 읽기 대기 시간/최소 바이트|
|**VSTATUS**|18|ICANON + IEXTEN|상태 요청 시그널|

### input flag

|**플래그**|**비트값**|**설명**|
|---|---|---|
|**IGNBRK**|`0x0001`|브레이크(Break) 신호 무시|
|**BRKINT**|`0x0002`|브레이크 발생 시 `SIGINT` 발생|
|**IGNPAR**|`0x0004`|패리티 오류가 있는 문자 무시|
|**PARMRK**|`0x0008`|패리티 오류 마킹|
|**INPCK**|`0x0010`|입력 패리티 검사 활성화|
|**ISTRIP**|`0x0020`|8번째 비트 제거 (7비트 통신)|
|**INLCR**|`0x0040`|입력받은 `NL`을 `CR`로 변경|
|**IGNCR**|`0x0080`|입력받은 `CR` 무시|
|**ICRNL**|`0x0100`|입력받은 `CR`을 `NL`로 변경|
|**IXON / IXOFF**|`0x0200/0400`|출력/입력 소프트웨어 흐름 제어|
|**IXANY**|`0x0800`|아무 키나 누르면 정지된 출력 재개|
|**IMAXBEL**|`0x2000`|입력 큐가 꽉 차면 벨소리 울림|
|**IUTF8**|`0x4000`|UTF-8 모드 유지 (정확한 글자 삭제 위함)|

### output flag

|**플래그**|**비트값**|**설명**|
|---|---|---|
|**OPOST**|`0x0001`|출력 가공 전체 활성화|
|**ONLCR**|`0x0002`|`NL`을 `CR-NL`로 변환하여 출력|
|**OXTABS**|`0x0004`|탭(`\t`)을 공백으로 확장하여 출력|
|**ONOEOT**|`0x0008`|출력 시 `EOT`(`^D`) 문자 무시|
|**OCRNL**|`0x0010`|출력 시 `CR`을 `NL`로 변환|
|**ONOCR**|`0x0020`|0번 컬럼에서 `CR` 출력 안 함|
|**ONLRET**|`0x0040`|`NL`이 `CR` 기능 수행|
|**NLDLY ~ VTDLY**|-|줄바꿈, 탭, CR, 폼피드, 백스페이스, 수직탭 지연 설정|

### control flag

|**플래그**|**비트값**|**설명**|
|---|---|---|
|**CSIZE (CS5~CS8)**|`0x0300`|전송 데이터 비트 수 (5~8비트)|
|**CSTOPB**|`0x0400`|정지 비트 2개 사용 (꺼져있으면 1개)|
|**CREAD**|`0x0800`|문자 수신 가능 상태로 설정|
|**PARENB / PARODD**|`0x1000/2000`|패리티 사용 여부 및 홀수 패리티 설정|
|**HUPCL**|`0x4000`|마지막 프로세스 종료 시 모뎀 끊기|
|**CLOCAL**|`0x8000`|모뎀 제어선 무시|
|**CRTSCTS**|-|하드웨어 흐름 제어 (RTS/CTS)|

### local flag

|**플래그**|**비트값**|**설명**|
|---|---|---|
|**ECHOKE**|`0x0001`|줄 삭제 시 시각적으로 깔끔하게 지움|
|**ECHOE**|`0x0002`|한 글자 삭제 시 시각적으로 지움|
|**ECHOK**|`0x0004`|줄 삭제 후 줄바꿈 문자 출력|
|**ECHO**|`0x0008`|입력한 문자 화면 출력 (에코)|
|**ECHONL**|`0x0010`|`ECHO`가 꺼져도 엔터키는 출력|
|**ECHOPRT**|`0x0020`|하드카피(종이 출력)용 지우기 모드|
|**ECHOCTL**|`0x0040`|제어 문자를 `^C` 형태로 출력|
|**ISIG**|`0x0080`|특수 제어 키(`Ctrl+C` 등)의 시그널화|
|**ICANON**|`0x0100`|정규 모드 (줄 단위 입력)|
|**IEXTEN**|`0x0400`|확장 기능 활성화 (LNEXT, REPRINT 등)|
|**TOSTOP**|`0x0040_0000`|백그라운드 작업의 출력 제어|
|**NOFLSH**|`0x8000_0000`|시그널 발생 시 큐 비우기 방지|