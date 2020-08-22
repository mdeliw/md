[toc]

# Creating Objects

## 1. Prefer Static factory methods over constructors <u>when the list of parameters is not optional</u>.

<u>Good</u>

- Static factory methods have names. 
- Static factory are not required to create a new object each time they're invoked.
- Static factory can return object of any subtype of their return type.
- Static factory can return object of a different class type based on input paramenter.
- Static factory doesn't need the class of the return type to exist - Service Provider Interface.

<u>Bad</u>

- Static methods are hard to find. Javadocs don't call out static factory methods clearly as compared to constructors.
- Only providing static factory method means that the class without a public or protected constructors can't be subclassed. 

<u>Naming convention</u>

-  `class.static-factory-method`
  - `.from` for  single parameter and return an instance. Alternative is `.valueOf`, `.getInstance`, `.instance`.
  - `.of`for multiple parameters and returns an instance. Alternative is `.valueOf`, `.getInstance`, `.instance`.
  - `.create`, `.newInstance` returns a new instance for each call.
- `helper-class.static-factory-method`:
  - `.getType`, `.newType`, `.type` when the static factory method is in a different class, i.e. the method is not in the class whose instance is returned.

## 2. Prefer Builder method over constructor <u>when the list of parameters is optional</u>.

The Builder pattern is verbose and needs an additional Builder class. The builder should be used only if we are dealing with with a large number of parameters.  The approach uses ==three distinct patterns== and validatity checks for mandatory and optional params <u>are necessary</u> in the Builder. Use `IllegalArgumentException` when necessary.  

1. Call the static factory method to return <u>a builder object</u> instead of the instance of the requested class. 
2. Use the setter methods of the builder object to further set any optional parameter.
3. Finally, call the `build` method of the builder object to generate the instance of the requested class.

```java
public class SomeClass {
  // required fields ... 
  // optional fields...
  
  //private constructor with builder as param
  private SomeClass(Builder builder) {
  	// transfer state from builder to SomeClass  
  }
  
  public static class Builder {
    // required fields ...
    // optional fields ...
    
    // constructor for required fields
    public Builder(mandatory params) {
      //... moves state to required fields
    }
    
    //setter for optional fields
    public Builder fieldName(optional params e.g. int x) {
      this.x = x;
      return Builder;
    }
    
    // return SomeClass
    public SomeClass build() {
      return new SomeClass(this);
    }    
  }
}

//usage
SomeClass sc = new SomeClass.Builder(a,b)
  .c(100)
  .d(200)
  .build();
```

>  This pattern can be included in the `abstract` class definition to force the builder implementation especially when multiple subtypes of a class are involved.

```java
// absttact class
public abstract class Pizza {
  public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE};
  final Set<Topping> toppings;
  
  // generic type with recursive type parameter.
  abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.Class);
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
    	return self();
    }

  	abstract SomeClass build();
  	protected abstract T self();
  }
  
  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone();
  }
  
}
```

## 3. Prefer the Singleton property.

Three common approaches use a private constructor and a static member to provide access to the sole instance. In two approaches, the static member is `final`.

<u>First approach</u>

A priviledged client can call the private constructor using reflection. To defend against this, have the constructor throw an exception if it is asked to create a second instance.

```java
public class SomeClass {
  public static final SomeClass INSTANCE = new SomeClass();
  private SomeClass() { ... }
}
```

<u>Second approach</u>

Allows the option to remove the singleton behavior at a later time and uses the `SomeClass::instance`pattern.

```java
public class SomeClass {
  private static final SomeClass INSTANCE = new SomeClass();
  private SomeClass() { ... }
  public static SomeClass getInstance() {
    return INSTANCE;
  }
}
```

For the above two approaches to be Serializable, its not enough to just implment Serializable. Make all fields transient, and implement `readResolve`, otherwise evertime the singleton is deserialized, a new object will be created. 

```java
private Object readResolve() {
    // Return the one true Object
    // let the garbage collector take care of other impersonators. 
    return INSTANCE;
}
```

<u>Third approach</u>

The third approach below, is similar to the first approach, it is concise, offers ironclad guarantee and <u>provides the serialization</u>. However it can't be used if the singleton must extend a superclass other than `Enum`.

```java
public enum SomeClass {
  INSTANCE;
}
```

## 4. Prefer non-instantiation with private constructor.

Some classes that are just a group of static fields and static methods aren't designed to be instantiated. To enforce non-instantiation  means to prevent subclassing. Java creates a default constructor only if the class contains no explicit constructor -  and usually the utility classes don’t need a constructor. 

```java
// Noninstantiable class
public class SomeClass {
  private SomeClass() {
    // just in case the class is instantiated from within the class.
    throw new AssertionError();
  }
}
```

## 5. Prefer dependency injection.

Adhoc use of injection increased clutter. Use Guice or Spring instead. 

The static utility classes and singletons are not appropriate when a state used inside a class is to be parameterized. What is required is the ability to support multiple instances of a class, each instance uses a resource desired by the client.  

In the below good example, imagine otherwise, that Lexicon is implemented as a singleton or static field. You won’t be able to change it once implement. 

```java
Class SpellChecker {
  private final Lexicon dictionary;
  public SpellChecker(Lexicon dictionary) {
    this.dictionary = dictionary;
  }
  // ... 
}
```

A Java 8 variation is to pass a `Supplier<T>` in the constructor. 

```java
public SomeClass(Supplier<? extends Base> baseFactory) { ... }
```

## 6. Avoid creating unnecessary objects.

```java
String s = new String("hello"); // bad
String s = "hello"; // good
```

With the second approach, it is guaranteed that the object will be reused by  any other code running in the same VM that happens to contains the same string literal. 

- Use `Pattern.compile()` where heavy use of `String.matches` is done.
- Prefer primitives over boxed primitives.
- Stacks manage their own memory, and popped items aren't automatically garbage collected. Explicitly assign null on the popped index.
- Listeners and Callbacks are another source of memory leaks if not unregistered.
- Make a defensive copy when reuse of object is not a good thing. Conversely, avoid duplicating objects when the object can be reused.
- Read about  `WeakHashMap`

## 7. Prefer try-with-resources over finalizers and cleaners

Avoid finalizers. The specification provides no guarantees that the finalizer will run promptly. Additionally it provides no guarantee that they'll be run at all. Uncaught exception thrown in the finalizer is ignored. Instead implement `AutoCloseable` and have client involve the `close` method on each instance using the `try`-with-resources approach. 

```java
//clean the rented room before returning to landlord.
public class Room implements AutoCloseable {
  private static final Cleaner cleaner = Cleaner.create();
  
  private static class State implemented Runnable {
    int numJunkPiles;
    State(int junk) {
      this.numJunkPiles = junk;
    }
    @Override public void run() {
      System.out.println("Cleaining room");
      numJunkPiles = 0;
    }
  }
  
  private final State state;
  private final Cleaner.Cleanable cleanable;
  
  public Room(int numJunkPiles) {
    state = new State(numJunkPiles);
    cleanable = cleaner.register(this, state);
  }
  
  @Override public void close() {
    cleanable.clean();
  }
}

// right usage
try (Room room = new Room(7)) {
 	System.out.println("Goodbye") 
}

// wrong usage
new Room(7);
System.out.println("Goodbye");
```

# Methods common to all objects.

Although `Object` is a concrete class, it is designed for extension, and its non-final methods are meant for overriding. 

## 1. Overriding `equals`.

- Best option is to not override this method - in which case the instance is only equal to itself.  
- Don't override if you're sure that the `equals` doesn't apply or if the super class already has a method that applies to your class.
- Override if a logical equality use case applies - such as a a value class, or when two different references have the same value. 
- Always override `hashCode` when you override `equals`.
- You mush adhere to the following rules to implement the `equals`:

```java
// reflexive
x.equals(x); // true if x is not null

// symmetric
x.equals(y);
y.equals(x); // true if x and y are not null

// transitive
x.equals(y);
y.equals(z);
z.equals(x);

// consistently return the same value.
x.equals(y);
x.equals(null); //must return false if x is not null;
```

#### 2. Always override `toString` and `clone`.

#### 3. Consider implementing `Comparable`.

### Classes and Interfaces.

#### 1. Minimize accessibility.

- Encourage information hiding. 
- Make a class as inaccessible as possible.
-  `private - restricted to class`
- `package-private`- restricted to package
- `protected`- restricted to subclass or class in the same package
- `public` - open to all.
- For testing, the compromise is to make `private` into `package-private` which means the test classes need to be in the same package as the class being tested.
- Forcing immutability

```java
public static final Things[] VALUES = { ... } //bad

//good
private static final Things[] PRIVATE_VALUES = { ... }
public statis final List<Things> VALUES = 
 Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

//or good
private static final Things[] PRIVATE_VALUES = { ... }
public static final Things[] values() {
  return PRIVATE_VALUES.clone();
}
```

#### 2. In public classes, use accessor methods instead of public fields.

- Definately apply this for public classes.
- Not mandatory for private and package-private classes.

#### 3. Minimize mutability.

> Immutable classes are simple, thread-safe, encourage reuse. You can avoid making defensive copies. No concept of temporary consistency, offers atomicity for free.

- Don't provide methods that modify the object's state.
- Ensure that the class can't be extended. You can make the class final or better to make constructor private and use static factories.
- Make all fields final.
- Make all fields private.
- Resist writing a setter.
- Ensure exclusive access to any mutable components.
- Offer a mutable companion class for your immutable class only if necessary.
- Avoid offering a public initializer and instead use constructors to completely initialize all variations.
- Avoid reinitializer code.
- ==Limitation== is that every distincy value needs a separate object.

> If your immutable class depends upon BigInteger and BigDecimal, you have to take extra care as these instances weren't designed for immutability.

```java
public static BigInteger safeInstance(BigInteger val) {
  return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
}
```

#### 4. Favor composition over inheritance.

- Inheritance (scope is limited to class extending super class) makes sense when you control the ecosystem (class, and its subclasses). Inheritance causes problems when it crosses package boundaries. 
- Instead of extending, use a private field referencing the instance of another class.

### Enums

#### 1. Use enums instead of int constants

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
```

Enums are classes that export one instance for each enumerated constant via a public static field.  Enums types are effectively final, and have no accessible constructors. Clients can neither create instance nor extend it.  

You can add arbitrary methods, fields, and interfaces. You can provide implementation of all the `Object` methods, also implement `Comparable` and `Serializable`. 

```java
public enum Planet {
  private final double mass;
  private final double radius;
  private final double surfaceGravity;
  private static final double universalG = 6.67300E-011;
  
  //constructor
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = universalG * mass / (radius * radius);
  }
  
  public double mass() { return mass;}
  public double radius() { return radius;}
  public double surfaceGravity() { return surfaceGravity;}
  public double surfaceWeight(double mass) { return mass * surfaceGravity;}
  
  MERCURY(3.302e+23, 2.439e6),
  VENUS(m2, r2),
  EARTH(m3, r3),
  MARS(m4, r4),
  JUPITER(m5, r6),
  SATURN(m6, r6),
  URANUS(m7, r7),
  NEPTUNE(m8, r8);
}

// usage
double mass = earthWeight/Planet.EARTH.surfaceGravity;
for (Planet p: Planet:values())
  System.out.printf("Weight on %s is %f%n", p, p.surfaceWeight(mass));
```

### Methods

- Choose method names carefully. Your primary goal should be to choose names that are understandable and consistent with other names in the same package. Your secondary goal should be to choose names with broader consensus.
- Every method should "pull its weight". 
- Aim for 4 or fewer arguments.
- Return empty collections or arrays, not nulls.

### General Programming

- Minimize scope of local variables. Declare them when used.
- Prefer `for-each` over `'for'.`

### Exceptions ✅

- Exceptions are designed for exceptional circumstances.
- Placing code inside `try-catch` block inhibits certain jvm optimizations.
- Follow the iterator's state dependent method `hasNext` to execute the `next` method later.
- Use checked exceptions for reasonably recoverable conditions and runtime exceptions for programming errors. The checked exceptions are supposed to be caught and the runtime  exceptions aren't. This checked exceptions put burden on the caller to determine how to recover or to throw it upward. 
- Don't use  `Error` class, the general convention is that `Error` is reserved for JVM's use.
- Don't use `Exception`, `RuntimeException`, `Throwable`directly. Treat them as abstract. You can't reliably test for the exceptions are many scenarios lead to throwing these exceptions. 
- Use exception translation at higher layers and prevent pollution of lower layer implementation related exceptions.
- Design methods throwing exceptions to have failure-atomic property. Generally speaking,  failed method invocation should leave the object in the state that it was in prior to the invocation. 
- Use standard exceptions as much as possible.

### Concurrency

- Remember that reading and writing a variable is atomic unless the variable is of type `long` or `double`. 
- Read about `AtomicLong`, `volatile`. 
- Prefer executors, tasks, and streams to threads.
- The higher-level utilities in `java.util.concurrent` fall into three categories: `ExecutorFramework`, `Concurrent Collections`, and `Synchronizers`. 
- `Synchronizers` enable threads to wait for one another, allowing them to coordinate their activities. The most commonly used synchronizers are `CountDownLatch` and `Semaphore`. Less commonly used are `CyclicBarrier` and `Exchanger`. The most powerful synchronizer is `Phaser`.
- Don't depend upon thread scheduler, best is to ensure that the average number of runnable threads is not significantly greater than the number of processors.
- When in doubt, don't synchronize, and document that the code is not thread-safe.
- Lazy initializers have their uses, but avoid them as much as possible. 

### Serialization

- Prefer alternatives to Java serialization.
- The attack surface on serialized data is too big to protect.

There are other simpler mechanisms (==json==, ==protobuf==) for translating between objects and byte sequences that avoid many of the dangers of Java serialization. These don't support automatic serialization and deserialization, instead, they support simple, structured data-objects consisting of a collection of name-value pairs, with very limited support for primitives and array data types. This simple abstraction has proven sufficient to build extremely powerful distributed systems. 