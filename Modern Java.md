[toc]

# Modern Java Recipes

## The Basics

### Lamda expression

Body with a single line with no braces is a `Lambda expression`. It is an implementation of the interface method, and can be assigned to a reference of that interface type as long as it is compatible with the method signature. A lambda can be an argument to a function, a return type from a method, or assigned to a reference, and in each case, the type of the assignment must be a functional interface.

```java
//old 
public class RunnableDemo() {
    psvm() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello");
            }
        }).start();
    }
}

//lamba as a method argument
new Thread(()-> System.out.println("Hello")).start();

// lambda assigned to a reference of an interface type
Runnable r = () -> System.out.println("Hello")).start();
new Thread(r).start();
```

### Block Lamda

Braces allow for multiple statements, and the `return` keyword is required. 

### Method Reference

Treat an existing method (static or instance) as a lamda expression. The `forEach` method takes a `Consumer` as an argument. The `Consumer` can be implemented as an expression or a method reference which is shorter and includes the name of the class containing the method.

```java
//lambda
Stream.of(3, 1, 4, 1, 5, 9)
    .forEach(x -> System.out.println(x));

//method reference
Stream.of(3, 1, 4, 1, 5, 9)
    .forEach(System.out::println);

//assigning method reference to functional interface
Consumer<Integer> printer = System.out::println;
Stream.of(3, 1, 4, 1, 5, 9)
    .forEach(printer);

Stream.generate(Math::random)		// static method as a Supplier
    .limit(10)
    .forEach(System.out::println);	// instance method
```

There are three forms of the method reference.

`object::instanceMethod` - instance method on the supplied object.

```java
x -> System.out.println(x); // context provides x as method argument
// becomes 
x -> System.out::println;
```

`Class::staticMethod` - static method on the supplied class.

```java
(x,y) -> Math.max(x,y);	// context provides x,y as method argument.
// becomes
(x,y) -> Math::max
```

`Class::instanceMethod` - instance method on the object supplied by the context as a target  to the method.

```java
x -> x.length(); // context provides x as supplied object, rather than as an argument.
// becomes
x -> String::length;
```

Another variation is if the method takes multiple arguments via class name, the first element supplied by context is the supplied object, and the remaining elements are the arguments to the methods.

```java
List<String> strings = 
    Arrays.asList("this", "is", "a", "list", "of", "strings");
List<String> sorted = strings.stream()
    .sorted((s1, s2) -> s1.compareTo(s2))  // multiple arguments, s1 is the object
    .collect(Collectors.toList());

List<String> sorted = strings.stream()
    .sorted(String::compareTo)	// s1 is the object, s2 is the argument
    .collect(Collectors.toList());

// another example
Stream.of("this", "is", "a", "list", "of", "strings")
    .map(String::length)			// instance method via class
    .forEach(System.out::println);	// instance method via object
```

### Constructor References

Instantiate an object using method reference.

```java
List<String> names = people.stream()
    .map(person->person.getName()) // Lambda expression
    .collect(Collectors.toList());
// becomes
List<String> names = people.stream()
    .map(Person::getName) // Method reference
    .collect(Collectors.toList());
// ---
List < String > names = 
    Arrays.asList ( "Grace Hopper" ,"Barbara Liskov" ,"Ada Lovelace" ,"Karen Spärck Jones" ) ; 

List<Person> person = names.stream()
    .map(name -> new Person(name)) // Lambda
	.collect(Collectors.toList());
// becomes
List<Person> person = names.stream()
    .map(Person::new) // Constructor reference
    .collect(Collectors.toList());
```

## java.util.function package

### Consumer

Takes an argument and returns nothing.

```java
List < String > strings = Arrays . asList ( "this" , "is" , "a" , "list" , "of" , "strings" ) ; 

strings.forEach(new Consumer<String>() {
    @Override
    public void accept(String s) {
        System.out.println(s);
    }
});

// becomes
strings.forEach(s -> System.out.println(s)); // lambda
strings.forEach(System.out::println); // method reference
```

The `BiConsumer` takes two arguments and returns nothing. The arguments are assumed to be of different types, and the second argument is primitive. 

```java
// other examples
Optional.ifPresent(Consumer);
Stream.forEach(Consumer);
Stream.peek(Consumer); // for debugging
```

### Supplier

Takes no argument and returns a value. Good use case is deferred execution. 

```java
DoubleSupplier randomSupplier = new DoubleSupplier() {
    @Override
    public double getAsDouble() {
        return Math.randon();
    }
};
// becomes
randomSupplier = Math.random; // lambda - so confusing
randomSupplier = Math::random; // method reference
```

### Predicates

Takes a single argument and returns a boolean.

```java
return Arrays.stream(names)
    .filter(s -> s.length() == 10) // lamba predicate
    .collect(Collectors.joining(", "));
// becomes
public String getNamesMatchingCondition( // pass predicate as argument
	Predicate<String> condition, String... names) {
    return Arrays.stream(names)
    	.filter(condition) // apply predicate
    	.collect(Collectors.joining(", "));
}
```

### Function

Takes a single argument and returns a value. Transform an input  into an output.

```java
List <Integer> nameLengths = names.stream() 
    .map(new Function<String, Integer>() { // anonymous function
        @Override public Integer apply (String s) { 
            return s.length (); 
        } 
    }).collect(Collectors.toList()); 
// becomes
.map(s -> s.length());
.map (String::length );
```

A BiFunction takes two arguments and returns a value. If inputs and outputs are of same type, you can use `BinaryOperator`. 

## Creating Streams

```java
// from static values
String names = Stream.of("a", "b", "c").collect(Collectors.joining(","));

List<String> bunch = Arrays.asList("a", "b", "c", "d");
String names = bunch.stream().collect(Collectors.joining(","));

String[] munsters = { "Herman" , "Lily" , "Eddie" , "Marilyn" , "Grandpa" }; 
String names = Arrays.stream(munsters).collect(Collectors.joining(",")); 

List(BigDecimal) nums = Stream.iterate(BigDecimal.ONE, n -> n.add(BigDecimal.ONE))
    	.limit(10).collect(Collectors.toList());
List(BigDecimal) nums =Stream.iterate(BigDecimal.ONE, n -> n.add(BigDecimal.ONE))
    	.limit(10).forEach(System.out::println);
Stream.generate(Math::random)
    .limit(10).forEach(System.out::println);
```

### Boxed Streams

Create a collection from a primitive stream.

```java
int[] intArray = IntStream.of(3,1,4,1,5,9).toArray();

List<Integer> ints = IntStream.of(3,1,4,1,5,9)
    .boxed().collect(Collectors.toList());

```

#### # Reduction Operations - wip

Produce a single value from a stream.

- Map transforms a stream of one type into another type (such as String to Int)
- Filter takes the Int stream and creates another stream of desired elements
- Finally, a terminal operator to create a single value (sum, avg) from the stream

```java
String[] strings = "this is an array of strings".split(" ");
long count = Arrays.stream(strings).map(String::length).count();
int totalLength = Arrays.stream(strings).mapToInt(String::length).sum();
// .average(), .max(), .min()

// summing numbers using reduce
int sum = IntStream.rangeClosed(1, 10)
    .reduce((x, y) -> {
        sout(x, y)
        return x + y;
    }).orElse(0);

int sum = Stream.of(1,2,3,4,5,6,7,8,9,10)
    .reduce(0, Integer::sum);

// find max
Integer max = Stream.of(3,1,4,1,5,9)
    .reduce(Integer.MIN_VALUE, Integer::max);

// concatenate string from stream
String s = Stream.of("this", "is", "a", "list")
    .reduce("", String::concat);
```

### Stream of Random numbers

```java
Random r = new Random();
r.ints(5).sorted().forEach(...);
r.doubles()...;
r.longs()...;
```



### Strings to Streams and back

Although Strings are collections of characters, `String` is not part of the Collections framework, and doesn’t implement `Iterable`, so there is no stream factory to convert String into a Stream. Also there is no `Arrays.stream` for `char[]`.

```java
string.toLowerCase().codePoints() // returns IntStream
    .collect(StringBuilder::new, // supplier
             StringBuilder::appendCodePoint, // BiConsumer accumulator
             StringBuilder::append) // BiConsumer combiner
    .toString();
```

### Counting

```java
long count = Stream.of(...).count();
long count = Stream.of(...).collect(Collectors.counting());
Map<Boolean, Long> numberLengthMap = strings.stream()
    .collect(Collectors.partitioningBy(
    	s -> s.length() % 2 == 0, // predicate 
    	Collectors.counting())); // collector
```

### findFirst, findAny

```java
Optional<Integer> firstEven = Stream.of(...)
    .filter(n -> n % 2 == 0)
    .findFirst();

Optional<Integer> firstEvenGT10 = Stream.of(...)
    .filter( n -> n > 10)
    .filter(n -> n % 2 == 0)
    .findFirst();

Optional<Integer> any = Stream.of(...)
    .unordered()
    .parallel()
    .findAny();


```

### anyMatch, allMatch, and noneMatch - wip

### Stream flatMap versus Map - wip

### Concatenating Streams

```java
Stream<String> first = Stream.of(a,b,c);
Stream<String> second = Stream.of(x,y,z);
List<String> strings = Stream.concat(first, second)
    .collect(Collectors.toList());
```

###

### Lazy Stream - wip

## Comparators and Collectors

### Sorting using Comparator

```java
string.stream()
	.sorted
    .sorted((s1,s2) -> s1.length() - s2.length())
    .sorted(Comparator.comparingInt(String::length))
	.collect(Collectors.toList());
```

### Converting Stream into a Collection

```java
List<String> lists = Stream.of(...)
    .collect(Collectors.toList());
Set<String> sets = Stream.of(...)
    .collect(Collectors.toSet());
List<String> linkedLists = Stream.of(...)
    .collect(Collectors.toCollection(LinkedList::new));
Stringp[] arrs = Stream.of(...)
    .toArray(String[]::new);

Set<Action> actors = ..getActors();
Map<String, String> maps = actors.stream()
    .collect(Collectors.toMap(Actor::getName, Actor::getRole));
```

## java.time Package

Uses ISO 8601 standard formatting.

```java
Instant.now();
LocalDate.now();
LocalTime.now();
LocalDateTime.now();
ZonedDateTime.now();
```

## Concurrency

### Future Interface

```java
ExecutorService service = Executors.newCachedThreadPool();
Future<String> future = service.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        Thread.sleep(100);
        return "Hello World!";
    }
});
System.out.println("Processing...");
try {
    if (!future.isCancelled()) {
        System.out.println(future.get()); // blocking
    } else {
        System.out.println("Cancelled");
    }
} catch (InterruptedException | ExecutuinException e) {
    e.printStackTrace();
}

// lambda
Future<String> future = service.submit(() -> {
    Thread.sleep(100);
    return "Hello World!";
});
System.out.println("Processing...");
while (!future.isDone()) { // blocking
    System.out.println("Waiting...");
}
try {
    if (!future.isCancelled()) {
        System.out.println(future.get()); // blocking call
    } else {
        System.out.println("Cancelled");
    }
} catch (InterruptedException | ExecutuinException e) {
    e.printStackTrace();
}

// cancelling
future.cancel(true);
```

### CompletableFuture

#### Completing a CompletableFuture

```java
// when you already have a CF and want to give a specific value
boolean complete(T value);  

// create a CF with an already computed value
static <U> CompletableFuture<U> completedFuture(U value);

// completes an existing CF with exception
completeExceptionally(Throwable ex);

// example with read from cache or sor
public CompletableFuture<Product> getProduct(int id) {
    try {
        Product product = getLocal(id);
        if (product != null) {
            return CompletableFuture.completedFuture(product);
        } else {
            return CompletableFuture.supplyAsync(() -> {
	            Product p = getRemote(id);
                return p;
            });
        }
    } catch (Exception e) {
        CompletableFuture<Product> future = new CompletableFuture<>();
        future.completeExceptionally(e);
        return future;
    }
}
```

#### Coordinating CompletableFuture

Completion of one `Future` triggers another action.

```java
CompletableFuture.supplyAsync(...) // Supplier
    .thenApply(...) // Function
    .thenApply(...)
    .thenAccept(...) // Consumer
    .join(); // blocking - retrieve result
```

Chain another `Future` to the original `Future` with the benefit that the result of the first `Future` is available to the second `Future`.

```java
int x = 2;
int y = 3;
// chain
CompletableFuture<Integer> completableFuture = 
    CompletableFuture.supplyAsync(() -> x)
    	.thenCompose(n -> CompletableFuture.supplyAsync(() -> n + y));
// combine
CompletableFuture<Integer> completableFuture = 
    CompletableFuture.supplyAsync(() -> x)
    	.thenCombine(
    		CompletableFuture.supplyAsync(() -> y), (n1, n2) -> n1 + n2));
System.out.println(completableFuture.get());


```

## Common Code

### Create Array with random values within a range

```java
Random r = new Random();
// 100 elements, range of -10 to 20
int[] arr = r.ints(100, -10, 20).toArray();
```

### Print an Array

```java
Arrays.toString(arr);
```

### Find a max in Array

```java
int max = Arrays.stream(arr).max().getAsInt();
```

