---
layout: post
title:  "Prime Shenanigans"
date:   2024-02-19 07:45:00 +0100
categories: projects
---

# BOO!
Wait I already did that a month ago..  
Oh well  
  
I was on the nerdy side of YouTube watching stuff, as you do, and I came across [b001's video on Dijkstra's Hidden Prime Finding Algorithm](https://youtu.be/fwxjMKBMR7s). Dijkstra's method is cool and all, but this Eratosthenes guy's method seemed really clever to me, especially for its time.  
  
[Eratosthenes of Cyrene](https://en.wikipedia.org/wiki/Eratosthenes) was a Greek polymath. That is, [someone with a lotta knowledge on a lotta things who tries to solve specific problems](https://en.wikipedia.org/wiki/Polymath). Amongst other things, he was a massive math nerd.  
  
Sometime between 276-194 BC, he introduced what's known nowadays as the [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).  
The Sieve of Eratosthenes is a prime finding algorithm that uses how composite numbers work to its advantage. Any integer composite number can be reduced to prime factors.  
The Sieve of Eratosthenes uses a huge boolean array of size n, starting from the number 2 (index 1) up to floor(sqrt(n)), which we'll write as m, and setting each index of its factors from m+m to n to false to indicate them as composite. These numbers can later be skipped when computing later composites, seeing as it'd be unnecessary to use them (as all composites have prime factors, it'd be inefficient to).  
  
A naive implementation of this algorithm, which only utilises the optimisation of checking up to floor(sqrt(n)), could be written as:
```
A -> boolean array of size n, filled with true values

for i = 2 up to floor(sqrt(n)) {
    if A[i] == true {
        for j = i+i, j += i up to n {
            set A[j] to false
        }
    }
}

return list of all true indices in A
```
  
I personally decided to turn this into an Iterator, so only the necessary values are computed.  
That's a lie, I just thought it'd be fun to. Yes, it was fun to.  
  
Here's my implementation in Java:  
```java
import java.util.Arrays;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class SieveOfEratosthenesIterator implements PrimeIterator<Integer> {
    private final boolean[] primeTape;
    private final int size;
    private final int maxCompareSize;

    private int index = 1;

    public static final int MAX_SIZE = Integer.MAX_VALUE - 8;

    public SieveOfEratosthenesIterator(int size) {
        if(size < 0)
            throw new UnsupportedOperationException("can't have a negative size, silly :3");

        if(size < 2)
            throw new UnsupportedOperationException("bit too small.. :( size should be >= 2");

        if(size > MAX_SIZE)
            throw new UnsupportedOperationException("that's too big! >:( size should be smaller than or equal to 2^31 - 9 (2147483639)");

        this.primeTape = new boolean[size];
        this.size = size;
        this.maxCompareSize = (int)Math.sqrt(size);

        Arrays.fill(this.primeTape, true);
        primeTape[0] = false;
    }

    public int getMaxCompareSize() {
        return maxCompareSize;
    }

    public Stream<Boolean> tapeStream() {
        return IntStream.range(0, primeTape.length).mapToObj(i -> primeTape[i]);
    }

    @Override
    public boolean hasNext() {
        boolean reachedEnd;

        while((reachedEnd = index + 1 != size) && !primeTape[index])
            index++;

        return reachedEnd;
    }

    @Override
    public Integer next() {
        int n = index + 1;

        if(index <= maxCompareSize)
            for(int i = index + n; i > 0 && i < size; i += n)
                primeTape[i] = false;

        if(n != size)
            index++;

        return n;
    }
}
```

...where PrimeIterator is:
```java
import java.util.Iterator;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;

public interface PrimeIterator<T extends Number> extends Iterator<T> {
    default Stream<T> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(this, Spliterator.ORDERED | Spliterator.IMMUTABLE);
    }
}
```

To just iterate over every prime up to n, you can do something like:
```java
PrimeIterator<Integer> priterator = new SieveOfEratosthenesIterator(n);

while(priterator.hasNext()) {
    int prime = priterator.next();
    System.out.println(prime);
}
```
...or
```java
new SieveOfEratosthenesIterator(n).forEachRemaining(prime -> {
    System.out.println(prime);
    // do something else with the prime, idk
});
```
Or if you want to have it as a list:
```java
PrimeIterator<Integer> priterator = new SieveOfEratosthenesIterator(n);
Stream<Integer> primeStream = priterator.stream(); // you can use this on its own, or alternatively:
List<Integer> primes = primeStream.collect(Collectors.toList()); // computes all primes up to n and stores them in a list
```

The reason why the max size is set to `(Integer.MAX_VALUE = 2^31-1) - 8` is because, unlike what most sources state, the maximum length the JVM lets you have on an array is 2^31 - 9. Try to make a bigger array, I dare you.  
  
Anyway, here's a [visualisation of the algorithm up to n=1849](https://github.com/Erdi-GitHub/Prime-Number-Experimental-Shenanigans/blob/master/src/main/java/me/erdi/primeshenanigans/demo/GraphicalDemo.java):  
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/914314090?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" style="position:absolute;top:0;left:0;width:100%;height:100%;" title="simplescreenrecorder-2024-02-19_07.28.45"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>  

Pretty neat, right?  
The source code is available on my GitHub: [Erdi-GitHub/Prime-Number-Experimental-Shenanigans](https://github.com/Erdi-GitHub/Prime-Number-Experimental-Shenanigans). You can build it with the usual `mvn install`, or get the binary from the releases section if you're lazy.
