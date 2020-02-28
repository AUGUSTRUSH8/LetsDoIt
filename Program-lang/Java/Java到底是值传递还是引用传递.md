### Java到底是值传递还是引用传递

我们学习过程当中，越是简单的东西越是难以真正理解，比如标题的问题，Java到底是值传递还是引用传递？这个问题在StackOverflow上面得到了很好的解释。下面就以SK上面的问答来看看这个问题。

#### 问

I always thought Java was **pass-by-reference**.

However, I've seen a couple of blog posts (for example, [this blog](http://javadude.com/articles/passbyvalue.htm)) that claim that it isn't.

I don't think I understand the distinction they're making.

What is the explanation?

#### 答

I just noticed you referenced [my article](http://javadude.com/articles/passbyvalue.htm).

The Java Spec says that everything in Java is pass-by-value. There is no such thing as "pass-by-reference" in Java.

The key to understanding this is that something like

```java
Dog myDog;
```

is *not* a Dog; it's actually a *pointer* to a Dog.

What that means, is when you have

```java
Dog myDog = new Dog("Rover");
foo(myDog);
```

you're essentially passing the *address* of the created `Dog` object to the `foo` method.

(I say essentially because Java pointers aren't direct addresses, but it's easiest to think of them that way)

Suppose the `Dog` object resides at memory address 42. This means we pass 42 to the method.

if the Method were defined as

```java
public void foo(Dog someDog) {
    someDog.setName("Max");     // AAA
    someDog = new Dog("Fifi");  // BBB
    someDog.setName("Rowlf");   // CCC
}
```

let's look at what's happening.

- the parameter `someDog` is set to the value 42
- at line "AAA"
  - `someDog` is followed to the `Dog` it points to (the `Dog` object at address 42)
  - that `Dog` (the one at address 42) is asked to change his name to Max
- at line "BBB"
  - a new `Dog` is created. Let's say he's at address 74
  - we assign the parameter `someDog` to 74
- at line "CCC"
  - someDog is followed to the `Dog` it points to (the `Dog` object at address 74)
  - that `Dog` (the one at address 74) is asked to change his name to Rowlf
- then, we return

Now let's think about what happens outside the method:

*Did myDog change?*

There's the key.

Keeping in mind that `myDog` is a *pointer*, and not an actual `Dog`, the answer is NO. `myDog` still has the value 42; it's still pointing to the original `Dog` (but note that because of line "AAA", its name is now "Max" - still the same Dog; `myDog`'s value has not changed.)

It's perfectly valid to *follow* an address and change what's at the end of it; that does not change the variable, however.

Java works exactly like C. You can assign a pointer, pass the pointer to a method, follow the pointer in the method and change the data that was pointed to. However, you cannot change where that pointer points.

In C++, Ada, Pascal and other languages that support pass-by-reference, you can actually change the variable that was passed.

If Java had pass-by-reference semantics, the `foo` method we defined above would have changed where `myDog` was pointing when it assigned `someDog` on line BBB.

Think of reference parameters as being aliases for the variable passed in. When that alias is assigned, so is the variable that was passed in.