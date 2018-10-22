---
title: Collections Framework -- overview
category:
- JDK Source Code
- Collections Framework
date: 2017/04/17
---

# 1. Outline
![collection](/images/collections/collection.png)
![map](/images/collections/map.png)

## 1.1 Collection interfaces
- Collection
- Set
- List
- Queue
- Deque
- Map
- SortedSet
- SortedMap
- NavigableSet
- NavigableMap
- BlockingQueue
- TransferQueue
- BlockingDeque
- ConcurrentMap
- ConcurrentNavigableMap

## 1.2 Infrastructure
### 1.2.1 Iterators
- Iterable
- Iterator
- ListIterator
<!--more-->

### 1.2.2 Ordering
- Comparable
- Comparator

### 1.2.3 Runtime exceptions
- UnsupportedOperationException
- ConcurrentModificationException

### 1.2.4 Performance
- RandomAccess

## 1.3 Abstract implementations
- AbstractCollection
- AbstractSet
- AbstractLit
- AbstractSequentialList
- AbstractQueue
- AbstractMap

## 1.4 General-purpose implementations
All general-purpose implementations are serializable and all  support a public clone method.
- HashSet
- TreeSet
- LinkedHashSet
- ArrayList
- ArrayDeque
- LinkedList
- PriorityQueue
- HashMap
- TreeMap
- LinkedHashMap

## 1.5 Special-purpose implementations
- WeakHashMap
- IdentityHashMap
- CopyOnWriteArrayList
- CopyOnWriteArraySet
- EnumSet
- EnumMap

## 1.6 Concurrent implementations
- ConcurrentLinkedQueue
- LinkedBlockingQueue
- ArrayBlockingQueue
- PriorityBlockingQueue
- DelayQueue
- synchronousQueue
- LinkedBlockingDeque
- LinkedTransferQueue
- ConcurrentHashMap
- ConcurrentSkipListSet
- ConcurrentSkipListMap

## 1.7 Wrapper implementations
### 1.7.1 unmodifiable
- Collections.unmodifiableCollection
- Collections.unmodifiableList
- Collections.unmodifiableSet
- Collections.unmodifiableMap
- Collections.unmodifiableSortedSet
- Collections.unmodifiableSortedMap
- Collections.unmodifiableNavigableSet
- Collections.unmodifiableNavigableMap

### 1.7.2 synchronized
- Collections.synchronizedCollection
- Collections.synchronizedList
- Collections.synchronizedSet
- Collections.synchronizedMap
- Collections.synchronizedSortedSet
- Collections.synchronizedSortedMap
- Collections.synchronizedNavigableSet
- Collections.synchronizedNavigableMap

### 1.7.3 checked
- Collections.checkedCollection
- Collections.checkedLIst
- Collections.checkedSet
- Collections.checkedQueue
- Collections.checkedMap
- Collections.checkedSortedSet
- Collections.checkedSortedMap
- Collections.checkedNavigableSet
- Collections.checkedNavigableMap

## 1.8 Convenience implementations
- Arrays.asList
- Collections.emptyEnumeration
- Collections.emptyIterator
- Collections.emptyListIterator
- Collections.emptyList
- Collections.emptyMap
- Collections.emptySet
- Collections.emptySortedSet
- Collections.emptySortedMap
- Collections.emptyNavigableSet
- Collections.emptyNavigableMap
- Collections.singleton
- Collections.singletonList
- Collections.singletonMap
- Collections.nCopys

## 1.9 Adapter implementations
- Collections.newSetFromMap
- Collections.asLifoQueue

## 1.10 Legacy implementations
- Vector
- HashTable

## 1.11 Algorithms
- Collections.sort(List)
- Collections.binarySearch(List,Object)
- Collections.reverse(List)
- Collections.shuffle(List)
- Collections.fill(List,Object)
- Collections.copy(List dest,List Src)
- Collections.min(Collection)
- Collections.max(Collection)
- Collections.rotate(List list,List distance)
- Collections.replaceAll(List list,Object oldVal,Object newVal)
- Collections.indexOfSubList(List source,List target)
- Collections.lastIndexOfSubList(List source,List target)
- Collections.swap(List,int,int)
- Collections.frequency(Collection,Object)
- Collections.disjoint(Collection,Collection)
- Collections.addAll(Collection,T...)

## 1.12 Array Utilities
- Arrays

# 2. Category
## 2.1 Collection
- Basic Operations
  - size()
  - isEmpty()
  - add(E e)
  - remove(E e)
  - removeIf(Predicate predicate)  ---- default method since 1.8
  - stream()  ---- default method for Aggregate Operations since 1.8
  - parallelStream()  ---- default method for Aggregate Operations since 1.8
  - iterator()  ---- inherits from Iterable
  - spliterator()  ---- default method inherits from Iterable since 1.8
  - forEach(Consumer consumer)  ---- default method inherits from Iterable since 1.8
- Bulk Operations
  - c1.containsAll(c2)
  - c1.addAll(c2)
  - c1.retainAll(c2)
  - c1.removeAll(c2)
  - clear()
- Array Operations
  - toArray()
  - toArray(T[] a)

### 2.1.1 Set
equals and hashCode methods are overrided through AbstractSet
#### 2.1.1.1 Operations
all inherit from Collection

#### 2.1.1.2 Implementations
- General-purpose
  - HashSet
  - TreeSet
  - LinkedHashSet
- Special-purpose
  - EnumSet
  - CopyOnWriteArraySet

### 2.1.2 List
equals and hashCode methods are overrided through AbstractList
#### 2.1.2.1 Operations
- Collection Operations
all inherit from Collection
  - sort(Comparator c)  ---- default method since 1.8
  - replaceAll(UnaryOperator operator)  ---- default method since 1.8
- Positional Access and Search Operations
  - get(int index)
  - set(int index,E e)
  - add(int index,E e)
  - remove(int index)
  - addAll(int index,Collection c)
  - indexOf(Object o)
  - lastIndexOf(Object o)
- Iterators
  - listIterator()
  - listIterator(int index)
- Range-View Operation
  - subList(int fromIndex, int toIndex)
- List Algorithms
  - Collections.sort(List)
  - Collections.binarySearch(List,Object)
  - Collections.reverse(List)
  - Collections.shuffle(List)
  - Collections.fill(List,Object)
  - Collections.copy(List dest,List Src)
  - Collections.rotate(List list,List distance)
  - Collections.replaceAll(List list,Object oldVal,Object newVal)
  - Collections.indexOfSubList(List source,List target)
  - Collections.lastIndexOfSubList(List source,List target)
  - Collections.swap(List,int,int)

#### 2.1.2.2 Implementations
- General-purpose
  - ArrayList
  - LinkedList
- Special-purpose
  - CopyOnWriteArrayList

### 2.1.3 Queue
#### 2.1.3.1 Operations
  |Type of Operation | Throws exception | Returns special value|
  |------------------|------------------|----------------------|
  |Insert            |add(e)            |offer(e)
  |Remove	           |remove()	        |poll()
  |Examine	         |element()         |peek()
#### 2.1.3.2 Implementations
- General-purpose
  - PriorityQueue
- Concurrent
  - ConcurrentLinkedQueue  ---- implements Queue
  - LinkedBlockingQueue  ----- implements BlockingQueue
  - ArrayBlockingQueue  ---- implements BlockingQueue
  - PriorityBlockingQueue  ---- implements BlockingQueue
  - DelayQueue  ---- implements BlockingQueue
  - SynchronousQueue  ---- implements BlockingQueue
  - LinkedTransferQueue  ---- implements TransferQueue

### 2.1.4 Deque
#### 2.1.4.1 Operations
- Main Methods
  - addFirst(e);offerFirst(e)
  - addLast(e);offerLast(e)
  - removeFirst();pollFirst()
  - removeLast();pollLast()
  - getFirst();peekFirst()
  - getLast();peekLast()
  - removeFirstOccurrence(o);removeLastOccurrence(o)
- Queue Methods
  - add(e)  ---- equivalent to addLast(e)
  - offer(e)  ---- equivalent to offerLast(e)
  - remove()  ---- equivalent to removeFirst()
  - poll()  ---- equivalent to pollFirst()
  - element()  ---- equivalent to getFirst()
  - peek()  ---- equivalent to peekFirst()
- stack Methods
  - push(e)  ---- equivalent to addFirst(e)
  - pop()  ---- equivalent to removeFirst()
- Collection Methods
  - remove(o)  ---- equivalent to removeFirstOccurrence(o)
  - contains(o)
  - size()
  - iterator()
  - descendingIterator()

#### 2.1.4.2 Implementations
- General-purpose
  - LinkedList
  - ArrayDeque
- Concurrent
  - LinkedBlockingDeque  ---- implements BlockingDeque
  - ConcurrentLinkedDeque

## 2.2 Map
### 2.2.1 Operations
- Basic Operations
  - size()
  - isEmpty()
  - containsKey(key)
  - containsValue(value)
  - get(key)
  - put(key,value)
  - remove(key)
  - getOrDefault(key, defaultValue)  ---- default method since 1.8
  - forEach(action)  ---- default method since 1.8
  - replaceAll(function)  ---- default method since 1.8
  - putIfAbsent(key,value)  ---- default method since 1.8
  - remove(key, value)  ---- default method since 1.8
  - replace(key, oldValue, newValue)  ---- default method since 1.8
  - replace(key, value)  ---- default method since 1.8
  - computeIfAbsent(key,mappingFunction)  ---- default method since 1.8
  - computeIfPresent(key,remappingFunction)  ---- default method since 1.8
  - compute(key,remappingFunction)  ---- default method since 1.8
  - merge(key, value,remappingFunction)  ---- default method since 1.8

- Bulk Operations
  - putAll(map)
  - clear()
- Collection Views
  - keySet()
  - values()
  - entrySet()

### 2.2.2 Implementations
- General-Purpose
  - HashMap
  - TreeMap  ---- implements NavigableMap
  - LinkedHashMap
- Special-Purpose
  - EnumMap
  - WeakHashMap
  - IdentityHashMap
- Concurrent
  - ConcurrentHashMap
  - ConcurrentSkipListMap  ---- implements ConcurrentNavigableMap
