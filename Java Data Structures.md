[toc]

# Java Data Structures

<img src="/Users/mdeliw/Library/Application%20Support/typora-user-images/image-20191121082444269.png" alt="image-20191121082444269" style="zoom:50%;" />

## ArrayList

- Slower, but offers flexibility in manipulations.
- It is initialized by size, and can grow dynamically
- Supports random access
- Doesn't support primitives

```java
//instance default
ArrayList<E> al = new ArrayList<E>();
ArrayList<E> al = new ArrayList<E>(size);

//instance from stream using list
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
List<E> list = stream.collect(Collectors.toList()); 
ArrayList<E> al = new ArrayList<E>(list);

//instance from stream using toCollection method
ArrayList<T> al = stream.collect(Collectors.toCollection(ArrayList::new));

//methods
al.clear();
al.isEmpty();

al.add(E element);
al.set(index, E element);

al.get(index);
al.contains(E element);
al.indexOf(E element);

al.lastIndexOf(E element);
al.ensureCapacity(newSize); 
al.trimToSize();

al.remove(index);
al.remove(E element);

//convert
Object[] = al.toArray();

//iterations
al.forEach(n -> Sout...)
al.removeIf(n -> (n % 3 == 0));

ListIterator<E> it = al.listIterator(); 
ListIterator<E> it = al.listIterator(fromIndex); 
while (it.hasNext()) { 
  System.out.println("Value is : "+ it.next()); 
} 
```

## LinkedList

```java
LinkedList<E> ll = new LinkedList<E>();

// add vs offer
// On failure, add throws exception, offer returns false.

ll.add(index, element); // tail
ll.addFirst(element);
ll.addLast(element); // last element

ll.offer(element);
ll.offerFirst(element);
ll.offerLast(element);

ll.removeFirst(element);
ll.removeLast(element);
getFirst();
getLast();

push(element);
peek(); // head, don't remove
poll(); // head, remove

peekFirst();
peekLast();

pollFirst();
pollLast();

```

## Stack

- LIFO

```java
Stack<E> stack = new Stack<E>();
push(element);
peek();
pop();
search(element);
removeElementAt(index);
```

## Queue

- FIFO - Insert at tail, remove at head
- Implemented by PriorityQueue and LinkedList

##  Deque

- FIFO and LIFO

- Offers insert, remove at both ends

- Faster than stacks and linked list

## Set

- Unordered list, dups are not allowed
- HashSet, LinkedHashSet, and TreeSet (sorted)
- HashSet
  - Doesn't preserve insertion order
  - Search is O(1)
- TreeSet
  - Doesn't preserve the insertion order, instead elements are sorted
  - Search is O(Log(n))

## Map

- Key/Value
- No Dups
- TreeMap and LinkedListMap maintain order, HashMap doesn't

### HashTable

- Same as Map, but synchronized

## String, StringBuilder, StringBuffer

- String is immutable
- StringBuilder is mutable
- StringBuffer is thread safe implementation of StringBuilder

```java
StringBuffer sbr = new StringBuffer(str); 
sbr.append(str);

StringBuilder sbl = new StringBuilder(str); 
sbl.append(str);
sbl.reverse(str);

sbr.toString();
sbl.toString();

Integer.toString(int);
String.valueOf(int);
Integer(n).toString();

int e = 12345; 
DecimalFormat df = new DecimalFormat("#,###"); 
String Str5 = df.format(e); 
System.out.println(Str5); // 12,345

int h = 255; 
Integer.toBinaryString(h); // 11111111
Integer.toHexString(j); // FF
Integer.toOctalString(i); // 377

Integer.toString(int, 7); // to base 7

```

## Arrays

```java
import java.utils.Array;

int arr[] = { 19, 20};
Arrays.toString(arr);
Arrays.asList(arr);
Arrays.toStream(arr);
Arrays.sort(arr);
Arrays.binarySearch(arr, key);
Arrays.binarySearch(arr, fromIndex, toIndex, key);
Arrays.compare(arr1, arr2); 

// Set of Integers to Set of Strings
Set<Integer> setOfInteger = 
  new HashSet<>(Arrays.asList(1, 2, 3, 4, 5)); 
Set<String> setOfString = setOfInteger
  .stream() 
  .map(String::valueOf) 
  .collect(Collectors.toSet()); 


```

## Stream

Stream is an abstraction and doesn’t hold data. Stream is NOT a data structure. The idea is to enable functional programming on stream of elements. You can’t point to an element in the stream, you can only specify a function that can operate on a element.

> Stream is an abstraction of a non-mutable collection of functions applied in some order to the data.

Common operations in Java Stream are:

- **Filter** - returns a new stream that contains some elements from the original stream.
- **Map** - transforms stream elements into something else.
- **Reduce** - reduces a stream to a single element.
- **Collect** - way to get out of stream and obtain a concrete collection of values.

Streams are created from various sources including collections, arrays, strings, IO resources, or generators. 

```java
//preferred assuming use of primitive or objects
Arrays.stream(array); 

// from collections
Arrays.asList().stream();
ArrayList().stream();

// iterating
Stream<T> stream = new ArrayList<>().stream();
Iterator<T> it = stream.iterator(); 
while (it.hasNext()) { 
  System.out.print(it.next() + " "); 
} 

// from Collections
List results = Arrays.asList(1,2,3,4).stream()
    .filter(e -> (e % 2) == 0)
    .map(e -> e * 2)
    .collect(toList())

// from values
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .forEach(p -> System.out.print(p + " "));

// from array
Stream<T> streamOfArray = Arrays.stream(arr); 
Stream<String> streamOfArray = Stream.empty(); 

// fluid
Stream.Builder<String> builder = Stream.builder(); 
Stream<String> stream = builder.add("a") 
  .add("b") 
  .add("c") 
  .build(); 

// infinite iteration
Stream.iterate(seedValue = 2, (Integer n) -> n * n) 
  .limit(limitTerms = 5) 
  .forEach(System.out::println); 
// 2, 4, 16, 256, 65536

// generate
Stream.generate(Math::random) 
  .limit(limitTerms = 5) 
  .forEach(System.out::println); 

// filter
list.stream() 
  .filter(p.asPredicate()) // p is a pattern
  .forEach(System.out::println); 

// convert primitive to object
int[] numbers = {1, 2, 3};
List<Integer> listOfIntegers = Arrays.stream(numbers)
    .boxed().collect(Collectors.toList());
```

## 

## Big Decimal

- Java Double and Float are floating point numbers stored as binary representation of fraction and exponent. Results have small errors 0.9999999 etc. 

- Big Decimal provide accurate representation .``

## Timer

```java
long start = System.currentTimeMillis();
// ...
long finish = System.currentTimeMillis();
long timeElapsed = finish - start;

long start = System.nanoTime();
// ...
long finish = Sy	stem.nanoTime();
long timeElapsed = finish - start;

Instant start = Instant.now();
// ... 
Instant finish = Instant.now();
long timeElapsed = 
  Duration.between(start, finish).toMillis();
```

## Tree

- Heavy mutations - use Red-Black tree
- Low mutations - use AVL tree

### Red-Black tree

- Every node is either black or red
- All roots aree black
- A red node can never have red parent
- All routes from root to leaf will have the same number of black nodes - also called the black height.
- Tree height = 2Log(n+1). Black height is  (tree height) / 2 

