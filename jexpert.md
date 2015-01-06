Here are some notes on various concepts of Java programming language.
The document is _not_ well organized and I'm not sure about the intended
audience. Nevertheless, I'm just putting down a bunch of stuff related
to Java that I've compiled over a span of a year or so. It's a good idea
to Google for more information on specific points. Some points below
concentrate on some lesser known things that Java _does_ and some focus
on counter-intuitive things that Java _doesn't_ do. A couple of
explanations have been (obviously) taken from StackOverflow.

---

Java is _always_ pass-by-value. More here, in this amazing discussion:
http://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value

---

Java _does_ allow "static import". It is used when repeatedly accessing
one or two static methods of a package. Not recommended to use it often.
One example is while accessing methods of `java.lang.Math` class:

```java
import static java.lang.Math.*;
```

and then directly use

```java
double r = cos(PI * theta);
```

instead of

```java
double r = Math.cos(Math.PI * theta);
```

---

POJO just means "Plain Old Java Objects" that should not extend or
implement any other class or get annotated.

---

In simple terms, "JavaBeans" is a class that encapsulates many objects
into a single object and has a "nullary constructor". The single object
is named "bean". They're reusable software components that allow access
to properties using getters and setters.

---

Java `assert`s are way to do validation. They should be used when
absolutely necessary and almost never be used in production code. It is
a good practice to not include any actions in assertion statements.
E.g.,

```java
assert someObject != null;
```

This asserts that `someObject` is not `null`. If `someObject` is `null`,
an `AssertionError` is thrown.

---

There is a keyword in Java called `strictfp`. It ensures that the JVM
evaluates the floating point in the same way on all the platforms. Without
this, JVM is free to do its own precision changes.

It's used like this:

```java
public strictfp class MyFPclass {
    // content
}
```

---

This is a lame one but still, it's perfectly legal but not recommended
to use static keyword before public which means

```java
static public void main(String args[]) {
}
```

does work.

---

In simplest terms, "autoboxing" and "unboxing" just mean casting and
un-casting from primitives to classes, i.e `int` to `Integer`, `char` to
`Char`, etc.

---

While compiling, the compiler performs "Type Erasure" in generics. For
example, the `<String>` part of `ArrayList<String>` will be forgotten.
A corollary would be to say that primitives cannot be used as types in
generics. However, the compiler does auto boxing.

---

"Reifiable" objects are the one that have full information during their
runtime. "Non-reifiable" objects don't. Non-reifiable types are types
where information has been removed at compile-time by type erasure &ndash;
invocations of generic types that are not defined as unbounded wildcards.
A non-reifiable type does not have all of its information available at
runtime. Examples of non-reifiable types are `List<String>` and
`List<Number>`; the JVM cannot tell the difference between these types
at runtime.

---

Java does _not_ allow first-class functions. First class functions
could be passed as parameters to other functions. For example,
JavaScript does allow that. Also, all functional programming languages
like Haskell, Erlang, etc. allow first-class functions. Even with the
inclusion of lambda expressions in Java 8, strictly speaking, there're
no first class functions in Java.

---

Creating string literals is much faster than creating `String` objects.
Full story and stats here:
http://www.precisejava.com/javaperf/j2se/StringAndStringBuffer.htm

---

There is a thing called "String Interning" in Java. String interning is
a method of storing only one copy of each distinct string value, which
must be immutable. Basically doing `String.intern()` on a series of
`String` objects will ensure that all `String`s having the same content
share same memory. So if there is a list of names where "john" appears
1000 times, by interning we ensure only one "john" is actually allocated
memory. One way to verify string interning is by examining the hashcodes
of String objects that have been interned.

This is an example of "Flyweight" design pattern.

---

There are no static classes in Java. There could however be
"static nested/inner" classes. But, if all the methods and variables in
a class are static, it's essentially a static Java class. Example is the
`java.lang.Math` class.

---

There _is_ a limit on the size of `java.util.HashMap`. A `HashMap`'s
maximum capacity is 2<sup>30</sup>. Infact, `HashMap`'s size will always
be a power of 2.

On a sidenote, the initial capacity of a `HashMap` is 16 and load factor
is 75% (12).

---

It's recommended to use

```java
mid = low + ((high - low) / 2);
```
rather than

```java
mid = (low + high) / 2;
```

to avoid overflowing. `low` + `high` might overflow before it's halved.

---

`>>` is an arithmetic right shift (preserves sign) and `>>>` is a logical
right shift (doesn't care about the signedness). Yes, that implies that
there are `<<` and `<<<` too, but they do the same job.

---

Surprising (or not) but Java does _not_ have unsigned integer :-)

---

In Java, `null` is _not_ an object, i.e.

```java
if (null instanceof Object)
    ...
```

is false.

---

There is _no_ `sizeof` in Java. To find the size of an object in Java
(since Java doesn't have a `sizeof` operator), we can do GC, then:

```java
long initialFree = runtime.freeMemory();
// initialize the object
long objectSize = initialFree - runtime.freememory();
```

Generally, the size is 16 bytes.

---

I know premature optimization is the root of all evil but counting down
(i.e., `for (int i=n; i>0; i--)`) is much faster than counting up.
On my machine, with this program - https://gist.github.com/anoopdixith/74c47a25ca52b9baefcf:

    Time taken by count-up in nanoseconds is 2700251000
    Time taken by count-down in nanoseconds is 6892000
    Ratio of diffDown to diffUp is 391.0

---

On similar lines as the above one: (A few obtained brom Peter Norvig's legendary blog)

1. Calling `Math.max(a,b)` is far slower than `(a > b) ? a : b`.
2. Arrays are 15 to 30 times faster than `Vector`s. Hash tables are 2/3
   the speed of `Vector`s.

---

Java compilers are very poor at lifting constant expressions out of
loops. So,

```java
for (int i = 0; i < str.length(); i++) {}
```

is far _slower_ than

```java
int len = str.length();
for (int i = 0; i < len; i++) {}
```

---

The instance variable initialization, if you were wondering, is actually
put in the constructor(s) by the compiler.

---

Synchronized Blocks in Static Methods:

An example where methods are synchronized on the class object of the
class the methods belong to:

```java
public class MyClass {
    public static synchronized void log1(String msg1, String msg2){
        log.writeln(msg1);
        log.writeln(msg2);
    }

    public static void log2(String msg1, String msg2){
        synchronized(MyClass.class){
            log.writeln(msg1);
            log.writeln(msg2);
        }
    }
}
```

Only one thread can execute inside any of these two methods at the same
time.

Had the second synchronized block been synchronized on a different
object than `MyClass.class`, one thread could execute inside each method
at the same time.

---

(Taken from StackOverflow)

Arbitrary Number of Arguments:

You can use a construct called "varargs" to pass an arbitrary number of
values to a method. You use varargs when you don't know how many of a
particular type of argument will be passed to the method. It's a
shortcut to creating an array manually.

To use varargs, you follow the type of the last parameter by an ellipsis
(three dots, like this: `...`), then a space, and the parameter name.
The method can then be called with any number of that parameter,
including none.

```java
public Polygon polygonFrom(Point... corners) {
    int numberOfSides = corners.length;
    double squareOfSide1, lengthOfSide1;
    squareOfSide1 = (corners[1].x - corners[0].x)
                     * (corners[1].x - corners[0].x)
                     + (corners[1].y - corners[0].y)
                     * (corners[1].y - corners[0].y);
    lengthOfSide1 = Math.sqrt(squareOfSide1);

    // more method body code follows that creates and returns a
    // polygon connecting the Points
}
```

Note that, inside the method, `corners` is treated like an array. The
method can be called either with an array or with a sequence of
arguments. The code in the method body will treat the parameter as an
array in either case.

---

In Java, a "shallow copy" (e.g. when assigning an array to an object's
array) is just pointing references. It is _unsafe_, a change in the
array will change the object's array also. "Deep copying" creates new
array in memory altogether. "Lazy copying" is a mixture of these two
where first, a shallow copy is made but with a counter to check if the
data is being shared or not. And when the program wants to modify the
data, if it finds that the copy has been shared, then a deep copy is
done. Cloning is a shallow copy of the original object. Serializable is
deep copy, however.

---

Anonymous class is basically an expression, used when the class is
fairly simple with just a method and when we know it's not going to be
reused. It's concise.

---

An interface is called as a functional interface if it has _just one_
abstract method.

---

In Java 8, Lambda expressions have been introduced. Lambda is extension
of anonymous classes. Lambda expressions enable us to treat
"functionality as method argument", or "code as data".

```java
public int operateBinary(int a, int b, IntegerOp op) {
    return op.operation(a, b);
}

...

IntegerOp addition = (a, b) -> a + b;
operateBinary(40, 2, addition);

```

Inside Lambda, it's not necessary to mention types. Compiler figures it
out using "target types". So, lambda could be used everywhere where
target types are known.

---

There is nothing called strong/weak typed (Or to say the least, it's
debated). But dynamic/static typing exists.

* Static typing - less prone to errors and helps the compiler in
  optimization.
* Dynamic - faster since any code change need not check for type safety
  and thus need not revisit.

---

In Java, we _cannot_ have a synchronized constructor. We can only
synchronize the block inside the constructor.

---

Regarding the serialization UID:

The serialization runtime associates with each serializable class a
version number, called a `serialVersionUID`, which is used during
deserialization to verify that the sender and receiver of a serialized
object have loaded classes for that object that are compatible with
respect to serialization.

```java
static final long serialVersionUID = 42L;
```

---

Regarding Class Loaders in Java:

The initial

```java
public static void main(String args[])
```

is _not_ loaded by the class loader, rest every class is.

```java
Class r = loadClass(String className, boolean resolveIt);
```

Notice that since `String` is used in class loader, it's important to
have it immutable.

Primordial classloaders come by default with JVM. They implement default
implementation of `loadClass`.

Classes are loaded when

1. byte code sees 'new' is used
2. byte code makes a static reference to a class (e.g. `System.out`).

User's custom class loader gets _higher_ priority than the primordial
class loader.

---

Virtual method is a method whose behavior can be overridden within an
inheriting class by a function with the same signature. All functions in
Java are virtual by default.

They could be made non-virtual by adding the `final` keyword. Or, by
adding `private` keyword. In Java applications, the executed method is
determined by the object type in _runtime_.

---

In Java, you _cannot_ use ternary operators without returning something.
A ternary expression is an expression, and cannot be used as a
statement.

---

Finalized...

* classes: cannot be subclassed.
* methods: cannot be overridden.
* variables: can only be assigned once.

---

Regarding Serialization:

Not all fields could be serialized. Non-serializable fields (like `int`)
should be marked `transient` ("transient" is a keyword in Java). They'll
get their default values when deserialized. On a side note, the
serialized `.ser` file _cannot_ be viewed as a text-file or such.

---

```java
System.identityHashCode(someObject);
```

gives the _original_ hash code _even_ if you _override_ the `hashCode()`
method. How about that!

---

In Java, there is a keyword called "native".
The `native` keyword is applied to a method to indicate that the method
is implemented in native code using JNI. E.g. in hash code
implementation. It marks a method that it will be implemented in other
languages, not in Java. It works together with JNI (Java Native
Interface).

---

This is "duh" but nevertheless including. While using iterators, users
shouldn't call `remove()` before they call `next()` or `hasNext()`.

---

We always knew `java.util.HashMap` is non-concurrent (and hence faster
than `java.util.Hashtable`) while `Hashtable` is (and is thus safer).
But in Java, there _is_ a collection called `ConcurrentHashMap`. It's
exactly what its name says.

---

Regarding PermGen:

The PermGen is where the JVM stores the metadata about classes. It _no
longer exists_ in Java-8, having been replaced with "Metaspace".
Generally, PermGen doesn't require any tuning above ensuring it has
enough space, although it is possible to have leaks if classes are not
being unloaded properly.

---

Expressions like `2*+1` are perfectly valid. It breaks down to
_2 × (+1)_, where `+` is used as a unary operator.

---

It's _highly recommended_ to close resources (like file reader or
buffered reader) inside `finally` blocks.

---

In Java, priority queues are implemented as a _heap_. And on a sidenote,
one amazing real life example of priority queue can be found here:
http://stackoverflow.com/a/14687156/1148287

---

`Iterator` and `Iterable` interfaces are different.
`Iterator` has `hasNext()`, `next()` and `remove()` while `Iterable` has
`iterator()`.

---

Regarding Double-Brace Initialization:

One of the starngest syntaxes in Java is the double-brace
initialization. In Java, we can create and initialize a new collection
as an expression by using the "double-brace" syntax:

```java
Set<String> sampleSet = new HashSet<String>() {{
    add("abc");
    add("def");
    add("abc");
}};
```

---

More to come - duck typing, reflection and how JUnit uses it for
testing, Java robots, multiple inheritance, TimSort (sorting algorithm
currently used to sort arrays in Java SE 7 and on Android), writing own
class Loaders, interesting stuff about garbage collection and more.