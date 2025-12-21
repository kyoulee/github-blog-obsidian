---
title: autotool
description: autotool에 대한 관찰
preview: /images/sketch-compile-preview.png
created: 2025-12-21T09:12:47+09:00
updated: 2025-12-21T09:12:47+09:00
tags:
  - 빌드
  - autotools
  - configure
  - icmp
  - network
  - ping
categories:
  - 개발
  - 빌드/배포
status: public
id: 0
writer: kyoulee
---

## 빌드 프로세스의 심층 분석: `./configure` 실행 후 `Makefile` 생성 메커니즘

오픈 소스 소프트웨어 ```inetutils-2.0``` 를 보는중 `./configure` 스크립트 실행하여 최종 실행 가능한 결과물 탄생시키는 것을 보게 되었다.
결과적으로 보았을때는 ```Makefile```을 생성하기 위한 작업으로 보여지는데 이를 어떻게 생성하는 규칙을 같는지 알아보도록 해보자

> [!NOTE]
> **이번 글에서는 `./configure` 실행 후 `Makefile`이 생성되기까지의 단계를 기술적인 관점에서 심층적으로 분석하고, 실제 코드 예시와 더불어 `configure.ac`에서 사용되는 주요 Autoconf 매크로에 대한 표를 제공하여 빌드 프로세스의 핵심을 명확하고 정확하게 파악할 수 있도록 집중적으로 논의합니다.**

빌드 환경 정의의 핵심: `configure.ac`와 `Makefile.am`의 역할 (코드 예시 및 주요 매크로 설명) 하도록 하겠다.

### configure.ac

빌드 프로세스는 개발자가 정의하는 두 가지 중요한 설정 파일을 보자 

* **`configure.ac`**: 프로젝트의 빌드 환경을 기술하는 선언적 명세서라 생각하면 좋다. 다음은 주요 Autoconf 매크로를 포함한 간단한 예시로

```autoconf.ac
AC_INIT([my-awesome-project], [1.0], [bug-report@example.com])
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([-Wall -Werror foreign])

AC_PROG_CC
AC_CHECK_LIB([z], [compress], [], [AC_MSG_ERROR([zlib required.])])
AC_DEFINE([HAVE_ZLIB], [1], [Define if zlib is available.]
AC_OUTPUT([Makefile])

```

설정, 정의 그리고 시작 같은 간단한 것을 볼 수 있다.

다음은 실제 파일에 있는 부분이다

```autoconf.ac
AC_PREREQ(2.64)

AC_INIT([GNU inetutils],
 m4_esyscmd([build-aux/git-version-gen .tarball-version]),
 [bug-inetutils@gnu.org])

AC_CONFIG_SRCDIR([src/inetd.c])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_HEADERS([config.h:config.hin])
AC_CANONICAL_HOST

AM_INIT_AUTOMAKE([1.11.1 dist-xz -Wall -Werror])

…

### Checks for programs.
AC_PROG_CC
gl_EARLY
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])
AC_CHECK_TOOL(AR, ar)
AC_PATH_PROG(DD, dd, dd)
AC_PATH_PROG(MKTEMP, mktemp, mktemp)
AC_PATH_PROG(NETSTAT, netstat, netstat)
AC_PATH_PROG(RM, rm, rm)
AC_PROG_CPP
AC_PROG_EGREP
AC_PROG_FGREP
AC_PROG_INSTALL
AC_PROG_MAKE_SET
AC_PROG_RANLIB
AC_PROG_YACC
AC_PROG_LN_S
AC_PROG_SED
AM_MISSING_PROG(HELP2MAN, help2man, $missing_dir)
AC_ARG_VAR(GREP, [Location of preferred 'grep' utility.])
AC_ARG_VAR(EGREP, [Location of preferred 'egrep' utility.])
AC_ARG_VAR(FGREP, [Location of preferred 'fgrep' utility.])
AC_ARG_VAR(SED, [Location of preferred 'sed' utility.])
AC_ARG_VAR(DD, [Location of 'dd'.])
AC_ARG_VAR(MKTEMP, [Location of 'mktemp'.])
AC_ARG_VAR(NETSTAT, [Location of 'netstat'.])
AC_ARG_VAR(TARGET, [IP address used while testing. @<:@127.0.0.1@:>@])
AC_ARG_VAR(TARGET6, [IPv6 address used while testing. @<:@::1@:>@])

…

## Checks for CPP macros.

# Look for the posix SEEK_ macros (for lseek), and if not found, try
# the similar berkeley L_ macros; if neither can be found, use the
# classic unix values.
IU_CHECK_MACRO(SEEK_ macros,
  [#include <unistd.h>], SEEK_SET SEEK_CUR SEEK_END,
  :,
  IU_CHECK_MACRO(L_ seek macros,
    [#include <unistd.h>], L_SET L_INCR L_XTND,
    AC_DEFINE([SEEK_SET], L_SET, [Define to L_SET as replacement])
    AC_DEFINE([SEEK_CUR], L_INCR, [Define to L_INCR as replacement])
    AC_DEFINE([SEEK_END], L_XTND, [Define to L_XTND as replacement]),
    AC_DEFINE([SEEK_SET], 0, [Define to 0 if missing])
    AC_DEFINE([SEEK_CUR], 1, [Define to 1 if missing])
    AC_DEFINE([SEEK_END], 2, [Define to 2 if missing])))

# Look for the posix _FILENO macros; if not found, use the classic
# unix values.
IU_CHECK_MACRO(_FILENO macros,
  [#include <unistd.h>], STDIN_FILENO STDOUT_FILENO STDERR_FILENO,
  :,
  AC_DEFINE([STDIN_FILENO], 0, [Define to 0 if missing])
  AC_DEFINE([STDOUT_FILENO], 1, [Define to 1 if missing])
  AC_DEFINE([STDERR_FILENO], 2, [Define to 2 if missing]))

…
AC_OUTPUT

```

기본적인 틀을 볼 수 있으며 컴퓨터마다 실행에 필요한 설정들과 라이브러리들을 가져오는 것으로 보인다.

#### **주요 Autoconf 매크로 설명:**

|**분류**|**매크로**|**설명 및 용도**|
|---|---|---|
|**기본 초기화**|`AC_PREREQ(VER)`|필요한 최소 Autoconf 버전을 확인합니다.|
||`AC_INIT(NAME, VER, BUG)`|프로젝트 이름, 버전, 버그 보고처를 설정합니다.|
||`AC_CONFIG_SRCDIR([FILE])`|소스 디렉토리의 유효성을 검사합니다.|
||`AC_CONFIG_HEADERS([FILE])`|시스템 검사 결과를 저장할 C 헤더(`config.h`)를 설정합니다.|
||`AM_INIT_AUTOMAKE([OPTS])`|Automake(Makefile.am 처리용)를 초기화합니다.|
|**사용자 옵션**|`AC_ARG_WITH(NAME, HELP)`|`--with-xxx` 옵션을 정의합니다. (외부 라이브러리 연동)|
||`AC_ARG_ENABLE(NAME, HELP)`|`--enable-xxx` 옵션을 정의합니다. (기능 켜기/끄기)|
||`IU_ENABLE_CLIENT/SERVER`|`inetutils` 전용: 특정 도구(ping 등) 빌드 여부를 결정합니다.|
|**프로그램 체크**|`AC_PROG_CC`|시스템의 C 컴파일러(gcc 등)를 찾습니다.|
||`AC_PROG_INSTALL`|파일 설치 도구(`install`)를 찾습니다.|
||`AC_PATH_PROG(VAR, PROG)`|특정 프로그램(dd, rm 등)의 절대 경로를 찾아 변수에 저장합니다.|
|**라이브러리/함수**|`AC_SEARCH_LIBS(FUNC, LIB)`|특정 함수를 사용하기 위해 필요한 라이브러리를 찾습니다.|
||`AC_CHECK_HEADERS([LIST])`|시스템에 특정 헤더 파일이 있는지 확인합니다.|
||`AC_CHECK_FUNCS([LIST])`|시스템에 특정 함수가 있는지 확인합니다.|
||`AC_CHECK_TYPES([LIST])`|특정 데이터 타입(`socklen_t` 등)이 있는지 확인합니다.|
||`AC_CHECK_MEMBERS([S.M])`|구조체 안에 특정 멤버 변수가 있는지 확인합니다.|
|**결과 정의/치환**|`AC_DEFINE(VAR, VAL, DESC)`|`config.h`에 `#define` 매크로를 추가합니다. (C 코드용)|
||`AC_SUBST(VAR)`|쉘 변수를 Makefile 변수로 바꿉니다. (Makefile용 `@VAR@`)|
||`AM_CONDITIONAL(NAME, COND)`|Makefile.am 내에서 쓸 수 있는 `if` 조건을 만듭니다.|
||`AH_TEMPLATE(KEY, DESC)`|`config.h.in` 템플릿에 매크로 설명을 미리 등록합니다.|
|**상태 메시지**|`AC_MSG_CHECKING(MSG)`|"checking MSG..." 메시지를 화면에 출력합니다.|
||`AC_MSG_RESULT(RES)`|검사 결과(yes/no 등)를 메시지 옆에 출력합니다.|
||`AC_MSG_ERROR/WARN(MSG)`|에러(중단) 또는 경고 메시지를 출력합니다.|
|**최종 실행**|`AC_CONFIG_FILES([LIST])`|생성할 파일(보통 `Makefile`) 목록을 지정합니다.|
||`AC_OUTPUT`|**최종 단계:** 모든 설정을 바탕으로 실제 파일을 생성합니다.|

> [!TIP]
> **`AC_`로 시작하는 매크로**: Autoconf 표준 매크로입니다.
> **`AM_`로 시작하는 매크로**: Automake 전용 매크로입니다.
> **`IU_`로 시작하는 매크로**: `inetutils` 개발자가 내부적으로 만든 커스텀 매크로입니다. (상위 디렉토리의 `am/` 폴더 등에 정의되어 있음)

### Makefile.am

* **`Makefile.am`**: 각 소프트웨어 컴포넌트의 빌드 규칙을 명확하게 정의하는 빌드 레시피로 설명이 나온다.

```makefile.am
ACLOCAL_AMFLAGS = -I am -I m4

EXTRA_DIST = paths ChangeLog.0 summary.sh.in CHECKLIST

SUBDIRS = lib \
	libinetutils libtelnet libicmp libls \
	src telnet telnetd ftp ftpd talk talkd whois ping ifconfig \
	doc man \
	tests

DISTCLEANFILES = pathdefs.make paths.defs \
	$(PACKAGE)-$(VERSION).tar.gz $(PACKAGE)-$(VERSION).tar.xz

# git-version-gen
EXTRA_DIST += $(top_srcdir)/.version
BUILT_SOURCES = $(top_srcdir)/.version
$(top_srcdir)/.version:
	echo $(VERSION) > $@-t && mv $@-t $@
dist-hook:
	echo $(VERSION) > $(distdir)/.tarball-version

snapshot:
	$(MAKE) dist distdir=$(PACKAGE)-`date +"%Y%m%d"`
```

makefile 부분은 기본적인 우리가 아는 부분이랑 같은것으로 보인다

간단한 차이점을 보면 다음 표와 같다.

|**구분**|**일반 Makefile**|**Makefile.am (Automake)**|
|---|---|---|
|**작성 주체**|개발자가 직접 작성|개발자가 작성 (원본 설계도)|
|**복잡성**|컴파일 옵션, 의존성 등을 모두 수동 입력|"소스 파일 목록" 위주로 아주 간결하게 작성|
|**호환성**|작성한 OS/환경에서만 잘 돌아감|`./configure`를 거치며 어떤 OS든 자동 최적화됨|
|**추상화**|`gcc -o ping ...` 처럼 구체적 명령어 사용|`bin_PROGRAMS = ping` 처럼 선언적 방식 사용|
|**자동 생성**|없음|`Makefile.in`을 거쳐 최종 `Makefile`로 변신함|

이러한 automake의 특징을 가지고 있다. 즉 상위 configure를 참조하여 만들게 되는 차이점을 가지고 있다 생각하면 된다.


### `configure`와 `makefile`의 코드변환

```makefile.am``` 에서 확인된 코드를 가지고 ```makefile.in```에는 사용자 시스템에 맞는 환경 변수들을 기입해준다.

이결과를 이제 makefile에 환경변수 등을 넣어주면서 정의된 시스템들에서 정상작동하도록 한다. (내부 집접설계한 개발자가 아님으로 완벽히 맞다고 할 수 는 없다.)

```makefile.in
* **`automake`**: `Makefile.am` 파일을 처리하여 `Makefile.in` 템플릿 파일을 생성합니다.

bindir = @bindir@
CC = @CC@
CFLAGS = @CFLAGS@
LDFLAGS = @LDFLAGS@
LIBS = @LIBS@
HAVE_ZLIB = @HAVE_ZLIB@

bin_PROGRAMS = awesome-tool

awesome_tool_SOURCES = src/main.c
awesome_tool_LDADD = @awesome_tool_LDADD@

```

상위 같은 환경 설정들을 시스템에 맞는 환경설정들로 makefile에 넣어 준다 생각하면 좋다.

`Makefile.in` 파일은 `configure` 스크립트에 의해 시스템 환경에 따라 `@VARIABLE@` 형태의 변수들이 치환될 준비가 된것이다.

다음은 실제 파일 이다
```makefile.in

DIST_ARCHIVES = $(distdir).tar.gz $(distdir).tar.xz
GZIP_ENV = --best
DIST_TARGETS = dist-xz dist-gzip
distuninstallcheck_listfiles = find . -type f -print
am__distuninstallcheck_listfiles = $(distuninstallcheck_listfiles) \
  | sed 's|^\./|$(prefix)/|' | grep -v '$(infodir)/dir$$'
distcleancheck_listfiles = find . -type f -print
ACLOCAL = @ACLOCAL@
ALLOCA = @ALLOCA@
ALLOCA_H = @ALLOCA_H@
AMTAR = @AMTAR@
AM_DEFAULT_VERBOSITY = @AM_DEFAULT_VERBOSITY@
APPLE_UNIVERSAL_BUILD = @APPLE_UNIVERSAL_BUILD@
AR = @AR@
```

대부분이 자동 환경 변수 설정을위한 ```@TEXT@``` 식으로 되어있는걸 볼 수 있다.

이로서 makefile을 생성하게 되는것이다.

### 결론: 코드 및 매크로 설명을 통한 빌드 프로세스 심층 이해

`configure.ac`의 주요 Autoconf 매크로와 함께 실제 코드 변환 과정을 살펴보았다. 이를 통해 개발자가 정의한 빌드 설정이 자동화 도구를 거쳐 최종 빌드 스크립트인 `Makefile`로 어떻게 구체화되는지 더욱 명확하게 이해할 수 있었으며. 이 심층적인 이해는 오픈 소스 소프트웨어 빌드 과정의 복잡성을 해소하고, 효율적인 개발 및 문제 해결 능력을 향상시키는 데 중요한 토대가 될 것이다.

> [!TIP]
> 추가적으로 ```config.h``` 와 같은 c파일용 header파일도 만들어준다.