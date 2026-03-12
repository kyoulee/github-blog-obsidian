---
title: ping_main_funtion
description: kyoulee blog
preview: /post/templates/images/default-og-image.jpg
created: 2025-12-24T06:12:36+09:00
updated: 2025-12-27T06:12:36+09:00
tags:
  - ping_main_funtion
status: public
id: 0
writer: kyoulee
---

## Main 함수에 대한 관찰

첫 시작을 하는 main함수로 차근차근 정의된 코드를 보려한다.

```c
static struct argp argp =
    {argp_options, parse_opt, args_doc, doc, NULL, NULL, NULL};
  
int main (int argc, char **argv)
{
    int index;
    int one = 1;
    int status = 0;
    set_program_name (argv[0]);
    
# ifdef HAVE_SETLOCALE
    setlocale(LC_ALL, "");
# endif
    
    if (getuid () == 0)
        is_root = true;  
    
    /* Parse command line */
    iu_argp_init ("ping", program_authors);
    argp_parse (&argp, argc, argv, 0, &index, NULL);
    
    ping = ping_init (ICMP_ECHO, getpid ());
    if (ping == NULL)
        /* ping_init() prints our error message.  */
        exit (EXIT_FAILURE);
    
    ping_set_sockopt(ping, SO_BROADCAST, (char *) &one, sizeof (one));
    
    /* Reset root privileges */
    if (setuid (getuid ()) != 0)
        error (EXIT_FAILURE, errno, "setuid");
    
    /* Force line buffering regardless of output device.  */
    setvbuf (stdout, NULL, _IOLBF, 0);
    
    argv += index;
    argc -= index;
    
    if (count != 0)
        ping_set_count (ping, count);
    if (socket_type != 0)
        ping_set_sockopt (ping, socket_type, &one, sizeof (one));
    if (options & OPT_INTERVAL)
        ping_set_interval (ping, interval);
    if (ttl > 0)
        if (setsockopt (ping->ping_fd, IPPROTO_IP, IP_TTL, &ttl, sizeof (ttl)) < 0)
            error (0, errno, "setsockopt(IP_TTL)");
    if (tos >= 0)
        if (setsockopt (ping->ping_fd, IPPROTO_IP, IP_TOS, &tos, sizeof (tos)) < 0)
        error (0, errno, "setsockopt(IP_TOS)");
        
    init_data_buffer (patptr, pattern_len);

    while (argc--)
    {
        status |= (*(ping_type)) (*argv++);
        ping_reset (ping);
    }
    free (ping);
    free (data_buffer);
    return status;
}
```


다음 메인함수를 분해하여 설명하려 한다. 

### 기본 설정 부분

```c
int main (int argc, char **argv)
{
    int index;
    int one = 1;
    int status = 0;
    set_program_name (argv[0]);
    
# ifdef HAVE_SETLOCALE
    setlocale(LC_ALL, "");
# endif
    
    if (getuid () == 0)
        is_root = true;  
        
  // 생략
}
```

기본 설정 부분을 확인해보면 다음과 같이 시스템적으로 필요한 부분들 설정 혹은 확인 하는 것을 볼 수 있다.
각각의 부분을 함수 단위로 자세히 살펴보도록 하자 

#### set_program_name()

```c
#include <config.h>

/* Specification. */
#undef ENABLE_RELOCATABLE /* avoid defining set_program_name as a macro */
#include "progname.h"
#include <errno.h> /* get program_invocation_name declaration */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

const char *program_name = NULL;

void set_program_name (const char *argv0) 
{
    const char *slash;
    const char *base;

    if (argv0 == NULL)
    {
        fputs ("A NULL argv[0] was passed through an exec system call.\n", stderr);
        abort ();
    }

    slash = strrchr (argv0, '/');
    base = (slash != NULL ? slash + 1 : argv0);
    
    if (base - argv0 >= 7 && strncmp (base - 7, "/.libs/", 7) == 0)
    {
        argv0 = base;
        if (strncmp (base, "lt-", 3) == 0)
            {
            argv0 = base + 3;
#if HAVE_DECL_PROGRAM_INVOCATION_SHORT_NAME
            program_invocation_short_name = (char *) argv0;
#endif
        }
    }
    program_name = argv0;
#if HAVE_DECL_PROGRAM_INVOCATION_NAME
    program_invocation_name = (char *) argv0;
#endif
}
```

다음 함수는 프로그램이 에러 혹은 어떠한 상황에 프로그램에 대한 명칭을 제공하기 위한 변수 저장이라 볼 수 있다. 이로서 어떠한 상황에서도 프로그램 이름을 출력하여 사용이 가능하도록 하는 부분이다.

#### setlocale()

> [!NOTE]
> setlocale은 환경에 따른 언어 그리고 시스템 구조를 참고하여 알맞은 설정을 할 수 있게 해주는 함수이다.
> 예 : 언어코드, 숫자, 단위 등

```c
#if 0
# if 0
# if !(defined __cplusplus && defined GNULIB_NAMESPACE)
# undef setlocale
# define setlocale rpl_setlocale
# define GNULIB_defined_setlocale 1
# endif
_GL_FUNCDECL_RPL (setlocale, char *, (int category, const char *locale));
_GL_CXXALIAS_RPL (setlocale, char *, (int category, const char *locale));
# else
_GL_CXXALIAS_SYS (setlocale, char *, (int category, const char *locale));
# endif
# if __GLIBC__ >= 2
_GL_CXXALIASWARN (setlocale);
# endif
#elif defined GNULIB_POSIXCHECK
# undef setlocale
# if HAVE_RAW_DECL_SETLOCALE
_GL_WARN_ON_USE (setlocale, "setlocale works differently on native Windows - "
"use gnulib module setlocale for portability");
# endif
#endif
```

다음을 보면 어떠한 환경에서도 동일하게 실행할 수 있게 만드는 Wrapper function이라 부르는거 같다.
대체 함수같은 뜻으로 명칭을 대치해서 사용하는것을 말하는 것 같다.

> [!CITE]
> 필자는 delegate function 인줄 알았다.

#### getuid()

> [!NOTE]
> getuid 함수는 user id를 추출하여 사용자의 번호를 알아보는 함수

다음 함수의 경우 root 관리자의 경우 0을 반환하기 때문에 root관리자임을 확인하는 것을 볼 수 있다.

### 파싱

parsing은 입력으로 온 부분을 정리하여 ping을 보내기 위한 정리로 볼 수 있다.

```c
int main (int argc, char **argv)
{
    // 생략 …
    
    /* Parse command line */
    iu_argp_init ("ping", program_authors);
    argp_parse (&argp, argc, argv, 0, &index, NULL);
    
    // 생략 …
```

#### iu_argp_init()

다음 함수는 내가만들었음을 알려주기 위한 함수로 상위에 정의된 작성자를 보여줌으로 자랑을 할 수 있다.😎

```c
const char *program_authors[] = {
    "Sergey Poznyakoff",
    NULL
};
```

상위에 보이는 것과 같이 배열에 이름을 작성하여 전달하는 것으로 기여를 표연할 수 있으며 다음 메크로가 작동되게 된다.

```c
/// config.h 
#define PACKAGE_BUGREPORT "bug-inetutils@gnu.org"

///libnetutils.h
#define iu_argp_init(name, authors) \
    argp_program_bug_address = "<" PACKAGE_BUGREPORT ">"; \
    argp_version_setup (name, authors);
```

> [!IMPORTANT]
> 다음 같은 구조로 만들어 져있으며 하위에 함수가 정의되있음으로 완벽하게 자랑 할 수 있다. 🚀

```c
void argp_version_setup (const char *name, const char * const *authors)
{
    argp_program_version_hook = version_etc_hook;
    program_canonical_name = name;
    program_authors = authors;
}
```

#### argp_parse()

> [!NOTE]
> GNU 할아버지의 파싱의 철학을 담아볼 수 있는 구간이다.
> ```argp``` 의 명칭도 argument parser로 추측하여 본다

함수는 기본적으로 정의된 ```argp_options```에 대하여 검색하고 필요한 부분을 설정하도록 설계되어 있다. 
```argp_options```의 구조는 다음과 같다.

```c
static struct argp_option argp_options[] = {
  /* [1]긴이름, [2]키(ID), [3]인자명, [4]플래그, [5]설명, [6]그룹번호 */
  {"echo", ARG_ECHO, NULL, 0, "send ICMP_ECHO packets (default)", GRP+1},
  // example
  // {"count", 'c', "NUMBER", 0, "stop after sending NUMBER packets", GRP+1},
  
  {NULL, 0, NULL, 0, NULL, 0} // 배열의 끝을 알리는 필수 종료자
};
```
다음과 같은 설정으로 프로그램의 명령어와 설정 flag들을 정의 하는 방식을 취하고 있다.

ping의 정의코드는 다음과 같다.

> [!IMPORTANT]-  argp_options 정의 보기
> ```c
> static struct argp_option argp_options[] = {
> #define GRP 0
>     {NULL, 0, NULL, 0, "Options controlling ICMP request types:", GRP},
>     {"address", ARG_ADDRESS, NULL, 0, "send ICMP_ADDRESS packets (root only)",
>      GRP+1},
>     {"echo", ARG_ECHO, NULL, 0, "send ICMP_ECHO packets (default)", GRP+1},
>     {"mask", ARG_ADDRESS, NULL, 0, "same as --address", GRP+1},
>     {"timestamp", ARG_TIMESTAMP, NULL, 0, "send ICMP_TIMESTAMP packets", GRP+1},
>     {"type", 't', "TYPE", 0, "send TYPE packets", GRP+1},
>     /* This option is not yet fully implemented, so mark it as hidden. */
>     {"router", ARG_ROUTERDISCOVERY, NULL, OPTION_HIDDEN, "send "
>     "ICMP_ROUTERDISCOVERY packets (root only)", GRP+1},
> #undef GRP
> #define GRP 10
>     {NULL, 0, NULL, 0, "Options valid for all request types:", GRP},
>     {"count", 'c', "NUMBER", 0, "stop after sending NUMBER packets", GRP+1},
>     {"debug", 'd', NULL, 0, "set the SO_DEBUG option", GRP+1},
>     {"interval", 'i', "NUMBER", 0, "wait NUMBER seconds between sending each "
>     "packet", GRP+1}
>     {"numeric", 'n', NULL, 0, "do not resolve host addresses", GRP+1},
>     {"ignore-routing", 'r', NULL, 0, "send directly to a host on an attached "
>     "network", GRP+1},
>     {"tos", 'T', "NUM", 0, "set type of service (TOS) to NUM", GRP+1},
>     {"ttl", ARG_TTL, "N", 0, "specify N as time-to-live", GRP+1},
>     {"verbose", 'v', NULL, 0, "verbose output", GRP+1},
>     {"timeout", 'w', "N", 0, "stop after N seconds", GRP+1},
>     {"linger", 'W', "N", 0, "number of seconds to wait for response", GRP+1},
> #undef GRP
> #define GRP 20
>     {NULL, 0, NULL, 0, "Options valid for --echo requests:", GRP},
>     {"flood", 'f', NULL, 0, "flood ping (root only)", GRP+1},
>     {"preload", 'l', "NUMBER", 0, "send NUMBER packets as fast as possible "
>     "before falling into normal mode of behavior (root only)", GRP+1},
>     {"pattern", 'p', "PATTERN", 0, "fill ICMP packet with given pattern (hex)",
>     GRP+1},
>     {"quiet", 'q', NULL, 0, "quiet output", GRP+1},
>     {"route", 'R', NULL, 0, "record route", GRP+1},
>     {"ip-timestamp", ARG_IPTIMESTAMP, "FLAG", 0, "IP timestamp of type FLAG, "
>     "which is one of \"tsonly\" and \"tsaddr\"", GRP+1},
>     {"size", 's', "NUMBER", 0, "send NUMBER data octets", GRP+1},
> #undef GRP
>     {NULL, 0, NULL, 0, NULL, 0}
> };
> ```
> 
> 다음과 같이 ```ping```함수의 doc과 명령어 flag들이 작성 되있다.

또한 argp struct의 정의는 이러한 구조를 가지고 있으며 다양한 상황에서 help또한 만들어 줄 수 있도록 설정해 놓았다.

```c
struct argp_option
{
    const char *name
    int key;
    const char *arg;
    int flags;
    const char *doc;
    int group;
};

struct argp
{
    const struct argp_option *options;
    argp_parser_t parser;
    const char *args_doc
    const char *doc;
    const struct argp_child *children;
    char *(*help_filter) (int __key, const char *__text, void *__input);
    const char *argp_domain;
};
```

상세한 ```argp_parse```의 함수를 살펴보면 다음과 같다. 
이에 대하여는 추후 parse함수를 만들때 확인하는 것으로 하겟다.

```c
error_t __argp_parse (const struct argp *argp, int argc, char **argv, unsigned flags, int *end_index, void *input)
{
    error_t err;
    struct parser parser
    /* If true, then err == EBADKEY is a result of a non-option argument failing to be parsed (which in some cases isn't actually an error). */
    int arg_ebadkey = 0;
    #ifndef _LIBC
    if (!(flags & ARGP_PARSE_ARGV0))
    {
        #if HAVE_DECL_PROGRAM_INVOCATION_NAME
        if (!program_invocation_name)
            program_invocation_name = argv[0];
        #endif
        #if HAVE_DECL_PROGRAM_INVOCATION_SHORT_NAME
        if (!program_invocation_short_name)
            program_invocation_short_name = __argp_base_name (argv[0]);
        #endif
    }
    #endif
    if (! (flags & ARGP_NO_HELP))
    /* Add our own options. */
    {
        struct argp_child *child = alloca (4 * sizeof (struct argp_child));
        struct argp *top_argp = alloca (sizeof (struct argp));
        /* TOP_ARGP has no options, it just serves to group the user & default argps. */
        memset (top_argp, 0, sizeof (*top_argp));
        top_argp->children = child;
        memset (child, 0, 4 * sizeof (struct argp_child));
        if (argp)
            (child++)->argp = argp;
        (child++)->argp = &argp_default_argp;
        if (argp_program_version || argp_program_version_hook)
            (child++)->argp = &argp_version_argp;
        child->argp = 0;
        argp = top_argp;
    }
    /* Construct a parser for these arguments. */
    err = parser_init (&parser, argp, argc, argv, flags, input);
    if (! err)
    /* Parse! */
    {
        while (! err)
            err = parser_parse_next (&parser, &arg_ebadkey);
        err = parser_finalize (&parser, err, arg_ebadkey, end_index);
    }
    return err;
}

#ifdef weak_alias
weak_alias (__argp_parse, argp_parse)
#endif
```

> [!TIP]
> 상위 보이듯이 ```__argp_parse``` 함수를  ```argp_parse``` 로 선언하여 사용하는 것도 볼 수 있다.

### socket 생성

```c
int main (int argc, char **argv)
{
    // 생략 …   
    
    ping = ping_init (ICMP_ECHO, getpid ());
    if (ping == NULL)
        /* ping_init() prints our error message.  */
        exit (EXIT_FAILURE);
    
    ping_set_sockopt(ping, SO_BROADCAST, (char *) &one, sizeof (one));
    
    // 생략 …   
}    
```

#### ping_init()

```libping.c``` 파일안에 있는 ``` ping_init``` 함수를 보면 다음 사항들을 주로 이룬다.

-  [[NetWork/ICMP|ICMP]] 에 대한 정의를 가져와 사용한다.
- fd를 할당 받아 새로운 socket을 만드는 것을 확인 할 수 있다.
- ping을 만들어 fd 를 담아 반환한다.

```c
PING * ping_init (int type, int ident)
{
    int fd;
    struct protoent *proto;
    PING *p;
    
    /* Initialize raw ICMP socket */
    proto = getprotobyname ("icmp");
    if (!proto)
    {
        fprintf (stderr, "ping: unknown protocol icmp.\n");
        return NULL;
    }
    
    fd = socket (AF_INET, SOCK_RAW, proto->p_proto);
    if (fd < 0) {
        if (errno == EPERM || errno == EACCES) {
            errno = 0;
            /* At least Linux can allow subprivileged users to send ICMP
            * packets formally encapsulated and built as a datagram socket,
            * but then the identity number is set by the kernel itself.
            */
            fd = socket (AF_INET, SOCK_DGRAM, proto->p_proto);
            if (fd < 0) {
                if (errno == EPERM || errno == EACCES || errno == EPROTONOSUPPORT)
                    fprintf (stderr, "ping: Lacking privilege for icmp socket.\n");
                else
                    fprintf (stderr, "ping: %s\n", strerror (errno));
                return NULL;
            }
            useless_ident++; /* SOCK_DGRAM overrides our set identity. */
        }
        else
            return NULL;
    }
    
    /* Allocate PING structure and initialize it to default values */
    p = malloc (sizeof (*p));
    if (!p) {
        close (fd);
        return p;
    }
    
    memset (p, 0, sizeof (*p));
    
    p->ping_fd = fd;
    p->ping_type = type;
    p->ping_count = 0;
    p->ping_interval = PING_DEFAULT_INTERVAL;
    p->ping_datalen = sizeof (icmphdr_t);
    /* Make sure we use only 16 bits in this field, id for icmp is a unsigned short. */
    p->ping_ident = ident & 0xFFFF
    p->ping_cktab_size = PING_CKTABSIZE;gettimeofday (&p->ping_start_time, NULL);
    return p;
}
```



### ping_set_sockopt()

> [!NOTE]
> 생성된 socket을 설정하는 함수로 이 또한 커널 단의 작업으로 OS별로 다른 함수를 가지고 있다 
> setsockopt을 사용함으로 공통적인 부분을 맞추어 만들도록 한다.

```c
void ping_set_sockopt (PING * ping, int opt, void *val, int valsize)
{
    setsockopt (ping->ping_fd, SOL_SOCKET, opt, (char *) &val, valsize);
}
```


```c
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
```

|**매개변수**|**의미**|**상세 설명**|
|---|---|---|
|**`sockfd`**|소켓 디스크립터|설정할 대상 소켓의 '번호표' (파일 디스크립터)|
|**`level`**|프로토콜 계층|옵션이 적용될 층 (SOL_SOCKET, IPPROTO_TCP, IPPROTO_IP 등)|
|**`optname`**|옵션 이름|바꾸고 싶은 구체적인 항목 (예: 타임아웃, 주소 재사용 등)|
|**`optval`**|설정할 값의 주소|변경할 값이 담긴 **메모리의 주소** (포인터)|
|**`optlen`**|값의 크기|`optval`이 가리키는 데이터의 크기 (sizeof 이용)|

```setsocketopt``` 함수의 경우 다음과 같은 특징을 가지고 있다. 
상위를 참조하여 어떠한 방식으로 세팅되는지 감을 잡을 수 있는데 등록된 fd 를 

```shell
% lsof -p 58818
COMMAND   PID    USER   FD   TYPE             DEVICE SIZE/OFF                NODE NAME
ping    58818 kyoulee  cwd    DIR               1,13      288             1733322 /Users/kyoulee/Documents/Project/ft_ping
ping    58818 kyoulee  txt    REG               1,13   170640 1152921500312561618 /sbin/ping
ping    58818 kyoulee  txt    REG               1,13  2299152 1152921500312563341 /usr/lib/dyld
ping    58818 kyoulee    0u   CHR               16,4   0t6787                 777 /dev/ttys004
ping    58818 kyoulee    1u   CHR               16,4   0t6787                 777 /dev/ttys004
ping    58818 kyoulee    2u   CHR               16,4   0t6787                 777 /dev/ttys004
ping    58818 kyoulee    3u  IPv4 0xb6753cbf816189c5      0t0                ICMP *:*
```

실제 ping 프로세서 ```fd``` 를 가져와 봤다  다음과 같이 나오는것을 볼 수 있다.
그중 IPv4 Type에 나와있듯 IPv4의 특징을 가지고 ```0xb6753cb…```의 메모리 주소를 할당 하고 있다는 것을 볼 수 있다.

신기하게 ```0t0``` 으로 읽는 부분이 0부분을 가리키고 있는데 이부분에 대하여는 메모리 주소로부터 0개의 데이터 부분까지 읽었다 (가르키고 있다) 라고 볼 수 있는데 그뜻은 fd에 대하여 읽은거 (가르키는거) 가 없다는 것을 의미한다. 이는 하위에  ```*:*``` 부분이 아직 특정되지 않았다는 뜻으로 보이는데 이러한 이유로 사이즈도 볼 수 없는거 같다. 
하지만 상위에 설정에 따르면 ```icmphdr_t``` 의 크기를 socket에 설정하였기 때문에 ```8byte``` 크기로 지정하였다는 것은 추측할 수 있다.

그리고 ICMP 프로토컬을 사용하고 있다는 것을 알려준다



### ping 설정

```c
int main (int argc, char **argv)
{
    // 생략 …   

    /* Reset root privileges */
    if (setuid (getuid ()) != 0)
        error (EXIT_FAILURE, errno, "setuid");
    
    /* Force line buffering regardless of output device.  */
    setvbuf (stdout, NULL, _IOLBF, 0);
    
    argv += index;
    argc -= index;
    
    if (count != 0)
        ping_set_count (ping, count);
    if (socket_type != 0)
        ping_set_sockopt (ping, socket_type, &one, sizeof (one));
    if (options & OPT_INTERVAL)
        ping_set_interval (ping, interval);
    if (ttl > 0)
        if (setsockopt (ping->ping_fd, IPPROTO_IP, IP_TTL, &ttl, sizeof (ttl)) < 0)
            error (0, errno, "setsockopt(IP_TTL)");
    if (tos >= 0)
        if (setsockopt (ping->ping_fd, IPPROTO_IP, IP_TOS, &tos, sizeof (tos)) < 0)
        error (0, errno, "setsockopt(IP_TOS)");
        
    init_data_buffer (patptr, pattern_len);

    // 생략 …   

```

이제 socket에 Ping에 대한 설정을 하려 한다.

#### root 권한설정

상위에서 봣던 root권한 확인하는 부분과 같은 함수 이다. 
이는 root권한으로 설정하여 실행하겠다는 함수로 권한 부여가 안될경우 error 를 반환하도록 되어 있다.

```c
if (setuid (getuid ()) != 0)
    error (EXIT_FAILURE, errno, "setuid");
```

#### buffer 설정

```c
setvbuf (stdout, NULL, _IOLBF, 0);
```

이는  stdout ```fd``` 에 ```\n``` 을 만나면 터미널에 출력되도록 설정하는 것으로 stdout 을 주시했다가 \n 을 감지하여 출력하는 것을 볼 수 있다. 이런식으로 설계하면 내가 실행한 터미널에서 볼 수 있도록 text를 출력하기 때문이다.

다음은 ```_IO***``` 옵션들을 볼 수 있다.

|**상수명**|**모드**|**동작 방식**|**주로 사용되는 곳**|
|---|---|---|---|
|**`_IONBF`**|No Buffering|버퍼링 없음. 데이터가 생기자마자 즉시 출력.|에러 메시지(`stderr`)|
|**`_IOLBF`**|**Line Buffering**|**`\n`을 만날 때마다 출력.**|**터미널 출력(`stdout`)**|
|**`_IOFBF`**|Full Buffering|버퍼가 꽉 차야만 출력.|파일 쓰기/읽기|

#### input options 설정

> [!NOTE]
> 다음은 input argments 에 대한 설정을 담은 것임으로 하위 표를 참고하여 활성화 되는 것을 보도록 한다.

```c
if (count != 0)
    ping_set_count (ping, count);
if (socket_type != 0)
    ping_set_sockopt (ping, socket_type, &one, sizeof (one));
if (options & OPT_INTERVAL)
    ping_set_interval (ping, interval);
if (ttl > 0)
    if (setsockopt (ping->ping_fd, IPPROTO_IP, IP_TTL, &ttl, sizeof (ttl)) < 0)
        error (0, errno, "setsockopt(IP_TTL)");
if (tos >= 0)
    if (setsockopt (ping->ping_fd, IPPROTO_IP, IP_TOS, &tos, sizeof (tos)) < 0)
```

|**변수명**|**옵션 (Flag)**|**예시 사용법**|**설명**|
|---|---|---|---|
|**`count`**|**`-c`**|`ping -c 5 google.com`|핑을 몇 번 보낼지 결정합니다. (예: `-c 5`이면 5번만 보내고 종료)|
|**`socket_type`**|**`-d`**|`ping -d google.com`|소켓의 특정 동작 방식을 활성화합니다. (보통 `SO_DEBUG`나 `SO_REUSEADDR` 등)|
|**`interval`**|**`-i`**|`ping -i 0.2 google.com`|패킷 사이의 대기 시간을 설정합니다. (기본은 1초, `-i 0.2`이면 0.2초마다 송신)|
|**`ttl`**|**`-t`**|`ping -t 64 google.com`|IP 헤더의 `Time To Live`를 설정하여 패킷이 거칠 수 있는 라우터 수를 제한합니다. (패킷의 생존 시간(TTL)을 **64**로 지정합니다.)|
|**`tos`**|**`-Q`** 또는 **`-T`**|`ping -Q 16 google.com`| IP 헤더의 `Type of Service`를 설정하여 패킷의 우선순위를 결정합니다. (우선도 16를 설정합니다.) |


#### buffer 세팅

모든 설정이 끝이 나고 버퍼를 만드는 과정이다.

```c
init_data_buffer (patptr, pattern_len);
```

##### init_data_buffer;

```c  
void init_data_buffer (unsigned char * pat, size_t len)
{
    size_t i = 0;
    unsigned char *p;
    
    if (data_length == 0)
        return;
    data_buffer = xmalloc (data_length);
    if (pat)
    {
        for (p = data_buffer; p < data_buffer + data_length; p++)
        {
            *p = pat[i];
            if (++i >= len)
                i = 0;
         }
    } else {
        for (i = 0; i < data_length; i++)
            data_buffer[i] = i;
    }
}
```

다음은 상위 함수를 보면 간단하게 설정 되어 있다 

```xmalloc```의 경우는 ```malloc```의 에러를 catch 하기 위한 커스텀 함수로 exit를 자동화 한 것이다. 그럼으로 

크기는 MAXPATTERN이라는 크기로 넣어줌으로 문자와 같은 부분을 패턴에 맞게 16진수로 수정해주는것 같다.

크기는 다음 고정된 크기를 변수로 받는다.

```c
#define MAXPATTERN 16 /* Maximal length of pattern. */
```

> [!IMPORTANT]
> 좀 당황스러운 부분은 ```patptr ``` 을 사용하는 곳이 없다는 것이다.
> 실제 만들어 지나 테스트를 하는 용도로 만든건지 모르겠지만 사용하는 곳은 따로 없기 때문에 넘어 가도 된다.


### 실행

마즈막 실행과 종료 부분이다. 

```c
int main (int argc, char **argv)
{
    // 생략 …   

    while (argc--)
    {
        status |= (*(ping_type)) (*argv++);
        ping_reset (ping);
    }
    free (ping);
    free (data_buffer);
    return status;
    
    // 생략 …   
}
```

다음은 인자로 [[#argp_parse()]] 되지 않은 부분에 대하여 모든 인자를 주소로 취급하여 실행하는 것을 볼 수 있다. (운영 체제 마다 다름으로 없을 수 있음)

이는 


```c
  
int

ping_echo (char *hostname)

{

#ifdef IP_OPTIONS

char rspace[MAX_IPOPTLEN]; /* Maximal IP option space. */

#endif

struct ping_stat ping_stat;

int status;

  

if (options & OPT_FLOOD && options & OPT_INTERVAL)

error (EXIT_FAILURE, 0, "-f and -i incompatible options");

  

memset (&ping_stat, 0, sizeof (ping_stat));

ping_stat.tmin = 999999999.0;

  

ping_set_type (ping, ICMP_ECHO);

ping_set_packetsize (ping, data_length);

ping_set_event_handler (ping, handler, &ping_stat);

  

if (ping_set_dest (ping, hostname))

error (EXIT_FAILURE, 0, "unknown host");

  

if (options & OPT_RROUTE)

{

#ifdef IP_OPTIONS

memset (rspace, 0, sizeof (rspace));

rspace[IPOPT_OPTVAL] = IPOPT_RR;

rspace[IPOPT_OLEN] = sizeof (rspace) - 1;

rspace[IPOPT_OFFSET] = IPOPT_MINOFF;

if (setsockopt (ping->ping_fd, IPPROTO_IP,

IP_OPTIONS, rspace, sizeof (rspace)) < 0)

error (EXIT_FAILURE, errno, "setsockopt");

#else

error (EXIT_FAILURE, 0, "record route not available in this "

"implementation.");

#endif /* IP_OPTIONS */

}

else if (options & OPT_IPTIMESTAMP)

{

int type;

  

if (suboptions & SOPT_TSPRESPEC)

#ifdef IPOPT_TS_PRESPEC_RFC791

type = IPOPT_TS_PRESPEC_RFC791;

#else

type = IPOPT_TS_PRESPEC;

#endif

else if (suboptions & SOPT_TSADDR)

type = IPOPT_TS_TSANDADDR;

else

type = IPOPT_TS_TSONLY;

  

#ifdef IP_OPTIONS

memset (rspace, 0, sizeof (rspace));

rspace[IPOPT_OPTVAL] = IPOPT_TS;

rspace[IPOPT_OLEN] = sizeof (rspace);
if (type != IPOPT_TS_TSONLY)
rspace[IPOPT_OLEN] -= sizeof (n_time); /* Exsessive part. */
rspace[IPOPT_OFFSET] = IPOPT_MINOFF + 1;
# ifdef IPOPT_POS_OV_FLG
rspace[IPOPT_POS_OV_FLG] = type;
# else
rspace[3] = type;
# endif /* !IPOPT_POS_OV_FLG */  
if (setsockopt (ping->ping_fd, IPPROTO_IP,
IP_OPTIONS, rspace, rspace[IPOPT_OLEN]) < 0)
error (EXIT_FAILURE, errno, "setsockopt");
#else /* !IP_OPTIONS */
error (EXIT_FAILURE, 0, "IP timestamp not available in this "
"implementation.");
#endif /* IP_OPTIONS */
}
printf ("PING %s (%s): %zu data bytes",
ping->ping_hostname,
inet_ntoa (ping->ping_dest.ping_sockaddr.sin_addr), data_length);
if (options & OPT_VERBOSE)
printf (", id 0x%04x = %u", ping->ping_ident, ping->ping_ident);
printf ("\n");

status = ping_run (ping, echo_finish);
free (ping->ping_hostname);
return status;

}
```

```c
void ping_reset (PING * p)
{
    p->ping_num_xmit = 0;
    p->ping_num_recv = 0;
    p->ping_num_rept = 0;
}
```