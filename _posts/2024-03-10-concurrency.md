---
layout: post
categories: [Java]
title: Concurrency and Thread
author: tngtied
date: 2024-03-10
---

Concurrency란 다중의 스레드를 동시에 실행하는 기법이다. 스레드란 동시성을 가진 프로세스로 효율적인 자원 활용을 가능하게 한다.

# 방법

자바에서 멀티스레딩을 하는 방법 중 하나는 `extends Thread`이고 다른 하나는 `implements Runnable`이다.
`extends Thread의` 경우 클래스에 public void run() 메서드를 작성하고 start()를 실행하는 것만으로도 가능하지만, `implements runnable`을 구현한 경우 Thread 오브젝트를 생성하고 그 안에 해당 멀티스레딩 클래스를 주입해준 후에 스레드를 start()해야 한다.
스레드 관리 및 다중 스레드 제어를 위해서는 ExecutorService를 사용해야 한다.

# 스레드의 상태

스레드의 가능한 상태는 아래와 같다.

- NEW
- RUNNABLE
- RUNNING
- BLOCKED/WAITING
- TERMINATED/DEAD
  다른 스레드가 실행됨에 따라 실행을 기다리고 있는 경우 RUNNABLE 상태가 된다. BLOCKED/WAITING의 경우 I/O와 같은 작업 혹은 다른 스레드의 출력을 받아야 하는 상황에 의해 기다리는 경우를 뜻한다.

# 스레드의 제어

스레드에는 setPriority를 통해 우선순위를 부여할 수 있다. 이는 강제가 아니고 리퀘스트의 정도에 불과하다.
join() 메소드를 통해 특정 라인 이후의 코드가 해당 스레드가 종료된 이후에만 실행되도록 강제할 수 있다.
스레드 클래스 내부에서 Thread.yield()를 통해 사용 중인 자원을 해제할 수 있다.
synchronized 메소드의 경우 shared 리소스에 대해서 스레드 중 하나만이
executor thread 중 newFixedThreadPool을 사용하면 현재 활성화된 스레드의 수를 조절할 수 있다.

# 값을 리턴하는 태스크

특정 값을 리턴하는 태스크의 경우 Callable interface를 구현해주어야 한다. Callable interface를 구현할 때 정한 제네릭을 리턴 값으로 하는 call()메소드를 오버라이드하고, 실제 실행 시 ExceutorService.submit(task)로 실행한다. 해당 submit()은 Future<T>를 리턴할 것이다. 이를 .get()을 통해 가져오고자 한다면, 해당 스레드가 종료되고 값을 리턴할때까지를 기다리게 된다.
