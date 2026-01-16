---
title: Ping
description : kyoulee blog
preview: https://kyoulee.com/post/templates/images/default-og-image.jpg
created: 2025-12-26T06:12:05+09:00
updated: 2025-12-27T06:12:05+09:00
tags:
  - Ping
status: public
id : 0
writer : kyoulee
---

모든 이들이 간단하게 ping을 사용하는 것으로 실제 network상 해당 ip에 유요한 컴퓨터가 통신이 되는지 알아보기 위해 사용된다. 이는 [[NetWork/ICMP|ICMP]]  바탕으로 만들어진 프로토컬로 전세계 공용적으로 널리 사용되고 있다. 


## ping struct

본 글은 [https://notes.shichao.io](https://notes.shichao.io/)과 실제 코드를 바탕으로 참조하여 만들어 졌다.

> [!Caution]
> 내부 작성 코드는 운영체제마다 내부 코드가 일부 다를 수 있다. 작성자는 mac OS를 사용함으로 참고 바란다.

```c
typedef struct ping_data PING;

struct ping_data
{
    int ping_fd; /* Raw socket descriptor */
    int ping_type; /* Type of packets to send */
    size_t ping_count; /* Number of packets to send */
    struct timeval ping_start_time; /* Start time */
    size_t ping_interval; /* Number of seconds to wait between sending pkts */
    union ping_address ping_dest;/* whom to ping */
    char *ping_hostname; /* Printable hostname */
    size_t ping_datalen; /* Length of data */
    int ping_ident; /* Our identifier */
    union event ping_event; /* User-defined handler */
    void *ping_closure; /* User-defined data */
    
    /* Runtime info */
    int ping_cktab_size;
    char *ping_cktab;
    
    unsigned char *ping_buffer; /* I/O buffer */
    union ping_address ping_from
    size_t ping_num_xmit; /* Number of packets transmitted */
    size_t ping_num_recv; /* Number of packets received */
    size_t ping_num_rept; /* Number of duplicates received */
};

union ping_address {
    struct sockaddr_in ping_sockaddr;
    struct sockaddr_in6 ping_sockaddr6;
};

union event {
    ping_efp6 handler6;
    ping_efp handler;
};

struct in_addr {
    in_addr_t s_addr;
};

struct sockaddr_in {
    sa_family_t    sin_family;
    in_port_t      sin_port;
    struct in_addr sin_addr;
    unsigned char  sin_zero[8];
};

struct in6_addr {
    uint8_t s6_addr[16];
};

struct sockaddr_in6 {
    sa_family_t     sin6_family;
    in_port_t       sin6_port;
    uint32_t        sin6_flowinfo;
    struct in6_addr sin6_addr;
    uint32_t        sin6_scope_id;
};

struct sockaddr {
    sa_family_t sa_family;
    char        sa_data[14];
};
```

![[42/ft_ping/images/Pasted image 20251227174046.png]]

> [!summery]- protoent struct 정보
> ```netdb.h``` 에 숨어있는 구조체이다 ```protoent```의 경우 socket 프로토컬 번호에서 사용하고 ```servent```의 경우는 포트번호 예 ```http```를 불러 적용 하는대 사용한다
> ```c
> struct protoent
> {
>   char *p_name;		/* Official name of protocol.  */
>   char **p_aliases;	/* Alias list.  */
>   int p_proto;		/* Protocol number.  */
> };
> 
> struct servent
> {
>   char *s_name;		/* Official name of service.  */
>   char **s_aliases;	/* Alias list.  */
>   int s_port;		/* Port number, in network byte order.  */
>   char *s_proto;	/* Protocol to use.  */
> };
> ```


다음은 [[Unix/Unix Network Programming|Unix Network Programming]]을 참고하여 좀더 자세한 사항을 볼수 있다


## Reference

- [https://notes.shichao.io/unp/ch1/](https://notes.shichao.io/unp/ch1/)