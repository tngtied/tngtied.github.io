---
layout: post
categories: [Java]
title: Collections
author: tngtied
date: 2024-03-08
---

Collections의 필요성은 기존 자료형이 immutable하다는 데서부터 대두되어, mutable한 자료형을 제공하는 데에서 나온다.
Collections가 포함하는 자료형 인터페이스는 Set, List, Queue, Map이 있다.

# List

List 인터페이스를 구현하는 클래스로는 ArrayList, LinkedList, Vector가 있다.
LinkedList는 삽입과 삭제가 빠르지만 조회가 ArrayList보다 느리다. 이 말은 반대로 하면 ArrayList는 삽입과 삭제가 보다 느리지만 indexing을 통한 조회가 빠르다는 것이 된다. 이 둘과 비교해봤을 때 Vector thread safe하다는 점에서 차이점을 가진다.

## List.of()에 대하여

List가 인터페이스라면 List의 스태틱 메소드인 of()는 무엇을 반환하는 것일까? 인터페이스는 인스턴스를 가질 수 없는데 우리는 무엇을 다루고 있는 것일까?
보통 List.of()로 생성된 것은 List<T>로 타입 지정이 된다. 그러나 실제로는 내부적으로 `java.util.ImmutableCollections`의 정적 하위 클래스 중 하나의 인스턴스를 다루게 된다. 우리는 구체적으로 클래스 이름을 알 필요가 없으며, 해당 클래스가 List 인터페이스의 메소드를 지원한다는 것만 알면 된다.

# Set

Set은 기본적으로 순서가 없고, 중복이 허용되지 않는 자료구조이다. 이를 구현한 클래스로는 HashSet, LinkedHashSet, TreeSet이 있다.
HashSet은 정렬 순서, 삽입 순서를 신경쓰지 않는다. LinkedHashSet은 삽입 순서를 따른다. TreeSet은 정렬 순서를 따른다.

# Map

Map 인터페이스를 구현하는 클래스로는 HashTable, HashMap, LinkedHashMap, TreeMap이 있다.
HashMap과 달리 HashTable은 synchronized하므로 thread-safe하다. 또한 해시맵은 key : null를 저장할 수 있지만 hashtable은 불가능하다.
TreeMap은 다른 맵과 달리 NavigableMap을 구현하므로 다른 Map 자료에서는 불가능한 연산을 다양하게 할 수 있다.

# Queue

Queue 인터페이스를 구현하는 클래스로는 PriorityQueue가 있다. PriorityQueue를 구현할 때 있어서 Comparator 인터페이스를 구현하는 커스텀 Comparator Class와 내부의 compare 함수를 작성함으로써 큐의 우선순위 비교자를 지정할 수 있다.
