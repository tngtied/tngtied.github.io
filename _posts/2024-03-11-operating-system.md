---
layout: post
categories: [OS]
title: 운영 체제의 개요
author: tngtied
date: 2024-03-11
---

운영체제란 어플리케이션과 하드웨어를 중개하며, 어플리케이션으로 하여금 안전하고 효율적으로 하드웨어를 운용할 수 있도록 하는 프로그램이다.
유저의 시야에서 본 운영체제란 주로 시스템 프로그램의 모음이지, 실제 시스템 콜이 아니다. 시스템 프로그램은 파일 조작, 상태 정보, 파일 변경, 프로그래밍 언어 지원, 프로그램 로딩과 실행, 어플리케이션 프로그램으로 나뉠 수 있다.
유저에게는 간편하고 쉽고 안전성있고 빠른 운영체제를 제공하고, 시스템으로서는 유연하고, 확장가능하고, 안정하고, 효율적인 운영체제를 제공하는 것이 운영체제 디자인 목표이다. 운영 체제의 좋은 설계란 메커니즘 (어떻게 할 것인가)은 고정적이고 정책(policy, 무엇을 할 것인가)이 가변적인 것이다.

커널이란 운영체제 중 필수적으로 항상 메모리에 상주하는 부분을 의미한다. (\* 보충 필요)

운영체제는 기본적으로 커널 모드와 유저 모드라는 듀얼 모드로 운영된다. 듀얼 모드가 사용되는 이유는 어플리케이션의 실패로부터 시스템을 지키기 위함이다. 유저 모드에서 운영 체제에 하드웨어나 시스템에 접근할 수 있는 방법을 간접되도록 제한함으로써, 시스템에 크리티컬한 코드를 감추고 유저 어플리케이션에게 시스템 콜이라는 인터페이스를 제공한다.

# 커널 모드로의 진입

유저 모드에서 커널 모드로 진입하는 방법은 세 가지가 있다. I/O 인터럽트와 같은 하드웨어 인터럽트, exception과 같은 소프트웨어 인터럽트 그리고 시스템 콜이다. 시스템 콜도 엄밀히 말하자면 소프트웨어 인터럽트이지만, 다른 것과 다르게 유저 어플리케이션에서 커널 모드의 기능을 이용하기 위해 자발적으로 전환하는 것을 의미한다.
OS는 유저 서비스에게 프로그램의 실행, I/O 오퍼레이션, 파일 시스템 조작, 통신, 에러 디텍션을 제공하며, 또한 자원의 할당과 자원의 보호의 기능을 한다.
대부분의 프로그래밍 언어는 시스템 콜을 포장함으로써 간접적으로 사용할 수 있도록 한다.

# 시스템 콜

시스템 콜은 크게 프로세스 매니지먼트, 파일 매니지먼트, 파일 시스템 매니지먼트로 나뉜다. 우리가 사용하는 시스템 콜 중 하나는 파일을 read하는 것으로, 파일 매니지먼트에 속한다. 만약 자바 응용 프로그램에서 보통은 wrapper function인 파일에 대한 읽기를 호출할 때, 그것은 JVM에 의해 운영 체제에 따라 다른, 네이티브 API함수로 변환된다. UNIX 계열 운영체제인 경우 libc가 될 것이며, 윈도우 운영체제일 경우 WinAPI가 될 것이다. 이러한 네이티브 API 함수들은 유저 공간에 존재한다. 해당 네이티브 API 소스코드를 실행하는 중 커널 모드로 진입하는 명령어를 통해 운영체제가 진입하게 되며, 커널 모드 내부에 존재하는 시스템 콜 테이블에서 해당하는 것을 찾아 실행함으로써 시스템 콜을 핸들링하게 된다. 커널은 OS 내부 File System Management에게 해당 작업을 실행시키고, 스케줄링을 사용해 다른 가능한 유저 모드의 작업으로 모드 스위칭한다. 이후 파일 시스템으로부터 해당 파일의 read 실행이 끝났음이 하드웨어 인터럽트를 통하여 신호된다면, 다시 커널 모드로 진입해 해당 리턴을 받아오고 유저 모드로 돌아가 결과물을 반환한다.

# 모놀리식 커널 vs 마이크로커널

모놀리식 커널은 모든 OS 서비스를 하나의 레이어, 커널에 구현함으로써 어떤 기능도 다른 기능에 바로 접근할 수 있게 한 것이다. 그럼에 따라 성능이 좋은 대신 유지 보수 비용이 상대적으로 높다.
마이크로커널은 필수적인 OS 기능만을 커널에 자리시키고 여타 기능들은 마이크로커널에 위치시키고 유저 모드에서 작동시키는 것을 의미한다. 확장성, 모듈성, 유지 보수의 용이성의 면에서 장점을 가지지만 성능이 저하된다.
마이크로커널은 성능 문제 때문에 실제로 사용되지 않아다. 하지만 모놀리식 커널을 loadable하게 모듈화하자는 발전방향으로 이어졌다.

# Virtual Machine

VM은 OS의 하드웨어에 대한 의존성을 없애고 OS 소프트웨어의 cross-platform 호환성을 위해 제시되었다. 하드웨어에 의존적인 OS는 OS 자체의 보안적인 취약점이 발견되거나, 공유 자원에 악의적인 프로그램이 접근할 수 있다는 보안적인 이슈 또한 야기했기 때문이다.
VM 솔루션은 호스트 시스템에 게스트 시스템을 매핑하는 것이다. VM 솔루션의 경우 개개의 프로세스를 가상화하는 프로세스 VM과 완전한 시스템을 가상화한 System VM이 있다.
Process VM은 서로 다른 ISA(Instruction Set Architecture)와 OS에서 실행되며 ABI(Application Binary Interface)를 에뮬레이트한다. 그 예시로는 JVM이 있다. JVM은 Bytecode 바이너리를 운영체제에 맞춰 실행한다.
시스템 VM은 OS와 executable 모두를 실행하며 유저 레벨, 시스템 레벨 모두의 ISA를 에뮬레이트한다. 이는 VMM, Hypervisor라고 불리며 OS와 executable 모두를 실행한다.
하이퍼바이저는 아래 두 가지로 종류가 나뉜다.

- Hosted : 이미 존재하는 OS의 프로세스로서 돌아간다.
- Stand-alone: bare hardware 위에서 돌아간다.
