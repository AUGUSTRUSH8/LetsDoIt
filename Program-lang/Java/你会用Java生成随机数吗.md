### 你会用Java生成某个特定区间的随机数吗

很多人对于这个问题并不是很感兴趣，但事实是这个问题在现实开发当中也有不少的用处哦，比如让你开发一个简单的抢红包功能，功能需求就是在用户发出的金额范围内按照人数生成随机红包（其实这个需求描述里面也有很多bug，可以想想bug在哪里）。

#### 错误尝试

```java
//Attempt 1:
randomNum = minimum + (int)(Math.random() * maximum);
// Bug: `randomNum` can be bigger than `maximum`.
```

```java
//Attempt 2:
Random rn = new Random();
int n = maximum - minimum + 1;
int i = rn.nextInt() % n;
randomNum =  minimum + i;
// Bug: `randomNum` can be smaller than `minimum`.
```

#### 正确解法

**Java 1.7 or later**

```java
import java.util.concurrent.ThreadLocalRandom;

// nextInt is normally exclusive of the top value,
// so add 1 to make it inclusive
int randomNum = ThreadLocalRandom.current().nextInt(min, max + 1);
```

**Before Java 1.7**

```java
import java.util.Random;

/**
 * Returns a pseudo-random number between min and max, inclusive.
 * The difference between min and max can be at most
 * <code>Integer.MAX_VALUE - 1</code>.
 *
 * @param min Minimum value
 * @param max Maximum value.  Must be greater than min.
 * @return Integer between min and max, inclusive.
 * @see java.util.Random#nextInt(int)
 */
public static int randInt(int min, int max) {

    // NOTE: This will (intentionally) not run as written so that folks
    // copy-pasting have to think about how to initialize their
    // Random instance.  Initialization of the Random instance is outside
    // the main scope of the question, but some decent options are to have
    // a field that is initialized once and then re-used as needed or to
    // use ThreadLocalRandom (if using at least Java 1.7).
    // 
    // In particular, do NOT do 'Random rand = new Random()' here or you
    // will get not very good / not very random results.
    Random rand;

    // nextInt is normally exclusive of the top value,
    // so add 1 to make it inclusive
    int randomNum = rand.nextInt((max - min) + 1) + min;

    return randomNum;
}
```

