---
layout: post
title:  "BF2JC Bug Fixes and QOL"
date:   2024-01-24 18:21:00 +0100
categories: projects
---

Before reading this, I recommend reading "[BF to Java Compiler!]({{ "/2024/01/23/bf2jc" | relative_url }})".  
  
Welp. What's new?  
First thing's first, the tape implementation in v1.0.0 was scuffed. Now that's fixed, in more ways than one, it uses a fixed-length array this time and actually wraps the pointer around like most BF interpreters tend to do.  
  
You can specify the tape's length using the --length option, the default value is 30000, seeing as, [according to Wikipedia](https://en.wikipedia.org/wiki/Brainfuck#Language_design):
> The brainfuck language uses a simple machine model consisting of the program and instruction pointer, as well as a one-dimensional array of at least **30,000** byte cells initialized to zero

You can set it to less than 30k though, I'm not strict, I want BF2JC to be a playground. Negative length of course doesn't work, feel free to try, the Java code won't compile.  
The max length would be 0X7FFFFFFF, or, for sane people: 2,147,483,647 (must be unstylised when passed, i.e. --length 2147483647). Though I recommend not setting it too high if you want predictable behavior, due to how BF2JC optimises code, if you end up having multiple pointer-increments (>) in a sequence of length n where ptr + n > 2147483647, you will have a negative pointer. GLHF.  
  
Let's see an example where wrapping actually matters:  
  
Look at v1.0.0
```bash
$ cat in.bf
<<<<-[------->+<]>-.-[->+++++<]>++.+++++++..+++.[--->+<]>-----.---[->+++<]>.-[--->+<]>---.+++.------.--------.-[--->+<]>.

$ java -jar BF2JC-1.0.0-SNAPSHOT.jar
Transpiled in 2.636071ms!

$ cat out.java
import java.util.List;
import java.util.ArrayList;
import java.io.IOException;

public class out {
	public static void main(String[] args) throws IOException {
		List<Byte> data = new ArrayList<>();
		data.add((byte) 0);
		int pos = 0;

		pos -= 4;
		data.set(pos, (byte) (data.get(pos) - 1));
		while(data.get(pos) != 0) {
			data.set(pos, (byte) (data.get(pos) - 7));
			pos += 1; while(pos >= data.size()) data.add((byte) 0);
			data.set(pos, (byte) (data.get(pos) + 1));
			pos -= 1;
		}
		pos += 1; while(pos >= data.size()) data.add((byte) 0);
		data.set(pos, (byte) (data.get(pos) - 1));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) - 1));
		while(data.get(pos) != 0) {
			data.set(pos, (byte) (data.get(pos) - 1));
			pos += 1; while(pos >= data.size()) data.add((byte) 0);
			data.set(pos, (byte) (data.get(pos) + 5));
			pos -= 1;
		}
		pos += 1; while(pos >= data.size()) data.add((byte) 0);
		data.set(pos, (byte) (data.get(pos) + 2));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) + 7));
		System.out.print((char)(byte) data.get(pos));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) + 3));
		System.out.print((char)(byte) data.get(pos));
		while(data.get(pos) != 0) {
			data.set(pos, (byte) (data.get(pos) - 3));
			pos += 1; while(pos >= data.size()) data.add((byte) 0);
			data.set(pos, (byte) (data.get(pos) + 1));
			pos -= 1;
		}
		pos += 1; while(pos >= data.size()) data.add((byte) 0);
		data.set(pos, (byte) (data.get(pos) - 5));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) - 3));
		while(data.get(pos) != 0) {
			data.set(pos, (byte) (data.get(pos) - 1));
			pos += 1; while(pos >= data.size()) data.add((byte) 0);
			data.set(pos, (byte) (data.get(pos) + 3));
			pos -= 1;
		}
		pos += 1; while(pos >= data.size()) data.add((byte) 0);
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) - 1));
		while(data.get(pos) != 0) {
			data.set(pos, (byte) (data.get(pos) - 3));
			pos += 1; while(pos >= data.size()) data.add((byte) 0);
			data.set(pos, (byte) (data.get(pos) + 1));
			pos -= 1;
		}
		pos += 1; while(pos >= data.size()) data.add((byte) 0);
		data.set(pos, (byte) (data.get(pos) - 3));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) + 3));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) - 6));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) - 8));
		System.out.print((char)(byte) data.get(pos));
		data.set(pos, (byte) (data.get(pos) - 1));
		while(data.get(pos) != 0) {
			data.set(pos, (byte) (data.get(pos) - 3));
			pos += 1; while(pos >= data.size()) data.add((byte) 0);
			data.set(pos, (byte) (data.get(pos) + 1));
			pos -= 1;
		}
		pos += 1; while(pos >= data.size()) data.add((byte) 0);
		System.out.print((char)(byte) data.get(pos));
		System.out.print((char)(byte) data.get(pos));
	}
}

$ javac out.java

$ java out
Exception in thread "main" java.lang.IndexOutOfBoundsException: Index -4 out of bounds for length 1
	at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64)
	at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70)
	at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266)
	at java.base/java.util.Objects.checkIndex(Objects.java:361)
	at java.base/java.util.ArrayList.get(ArrayList.java:427)
	at out.main(out.java:12)
```
An OOBE. Nasty. Pathetic. Grim. Disgusting.  
Now, look at how v1.2.1 flawlessly handles it, whilst taking less disc space:
```bash
$ cat in.bf
<<<<-[------->+<]>-.-[->+++++<]>++.+++++++..+++.[--->+<]>-----.---[->+++<]>.-[--->+<]>---.+++.------.--------.-[--->+<]>.

$ java -jar BF2JC-1.2.1-SNAPSHOT.jar --replace
Transpiled in 2.517951ms!

$ cat out.java
import java.io.IOException;

public class out {
	public static void main(String[] args) throws IOException {
		int len = 30000;
		byte[] tape = new byte[len];
		int ptr = 0;

		ptr = Math.floorMod(ptr - 4, len);
		tape[ptr] -= 1;
		while(tape[ptr] != 0) {
			tape[ptr] -= 7;
			ptr = (ptr + 1) % len;
			tape[ptr] += 1;
			ptr = Math.floorMod(ptr - 1, len);
		}
		ptr = (ptr + 1) % len;
		tape[ptr] -= 1;
		System.out.print((char)tape[ptr]);
		tape[ptr] -= 1;
		while(tape[ptr] != 0) {
			tape[ptr] -= 1;
			ptr = (ptr + 1) % len;
			tape[ptr] += 5;
			ptr = Math.floorMod(ptr - 1, len);
		}
		ptr = (ptr + 1) % len;
		tape[ptr] += 2;
		System.out.print((char)tape[ptr]);
		tape[ptr] += 7;
		System.out.print((char)tape[ptr]);
		System.out.print((char)tape[ptr]);
		tape[ptr] += 3;
		System.out.print((char)tape[ptr]);
		while(tape[ptr] != 0) {
			tape[ptr] -= 3;
			ptr = (ptr + 1) % len;
			tape[ptr] += 1;
			ptr = Math.floorMod(ptr - 1, len);
		}
		ptr = (ptr + 1) % len;
		tape[ptr] -= 5;
		System.out.print((char)tape[ptr]);
		tape[ptr] -= 3;
		while(tape[ptr] != 0) {
			tape[ptr] -= 1;
			ptr = (ptr + 1) % len;
			tape[ptr] += 3;
			ptr = Math.floorMod(ptr - 1, len);
		}
		ptr = (ptr + 1) % len;
		System.out.print((char)tape[ptr]);
		tape[ptr] -= 1;
		while(tape[ptr] != 0) {
			tape[ptr] -= 3;
			ptr = (ptr + 1) % len;
			tape[ptr] += 1;
			ptr = Math.floorMod(ptr - 1, len);
		}
		ptr = (ptr + 1) % len;
		tape[ptr] -= 3;
		System.out.print((char)tape[ptr]);
		tape[ptr] += 3;
		System.out.print((char)tape[ptr]);
		tape[ptr] -= 6;
		System.out.print((char)tape[ptr]);
		tape[ptr] -= 8;
		System.out.print((char)tape[ptr]);
		tape[ptr] -= 1;
		while(tape[ptr] != 0) {
			tape[ptr] -= 3;
			ptr = (ptr + 1) % len;
			tape[ptr] += 1;
			ptr = Math.floorMod(ptr - 1, len);
		}
		ptr = (ptr + 1) % len;
		System.out.print((char)tape[ptr]);
	}
}

$ javac out.java

$ java out
Hello World!
```
Beautiful. Marvelous. Scrumptious. Meticulous. Clean.  
No OOBE! [It just works!](https://youtu.be/YPN0qhSyWy8) (Teehee.)  
  
What else?  
Now there's a --minify (or -m) option! Let's look at how absolutely tiny code can become:
```brainfuck
[
    in.bf
]
<<<<-[------->+<]>-.-[->+++++<]>++.+++++++..+++.[--->+<]>-----.---[->+++<]>.-[--->+<]>---.+++.------.--------.-[--->+<]>.
```
```bash
$ java -jar BF2JC-1.2.1-SNAPSHOT.jar --minify
```
```java
// out.java
public class out{public static void main(String[]$)throws java.io.IOException{int l=30000;byte[]t=new byte[l];int p=0;p=Math.floorMod(p-4,l);t[p]-=1;while(t[p]!=0){t[p]-=7;p=(p+1)%l;t[p]+=1;p=Math.floorMod(p-1,l);}p=(p+1)%l;t[p]-=1;System.out.print((char)t[p]);t[p]-=1;while(t[p]!=0){t[p]-=1;p=(p+1)%l;t[p]+=5;p=Math.floorMod(p-1,l);}p=(p+1)%l;t[p]+=2;System.out.print((char)t[p]);t[p]+=7;System.out.print((char)t[p]);System.out.print((char)t[p]);t[p]+=3;System.out.print((char)t[p]);while(t[p]!=0){t[p]-=3;p=(p+1)%l;t[p]+=1;p=Math.floorMod(p-1,l);}p=(p+1)%l;t[p]-=5;System.out.print((char)t[p]);t[p]-=3;while(t[p]!=0){t[p]-=1;p=(p+1)%l;t[p]+=3;p=Math.floorMod(p-1,l);}p=(p+1)%l;System.out.print((char)t[p]);t[p]-=1;while(t[p]!=0){t[p]-=3;p=(p+1)%l;t[p]+=1;p=Math.floorMod(p-1,l);}p=(p+1)%l;t[p]-=3;System.out.print((char)t[p]);t[p]+=3;System.out.print((char)t[p]);t[p]-=6;System.out.print((char)t[p]);t[p]-=8;System.out.print((char)t[p]);t[p]-=1;while(t[p]!=0){t[p]-=3;p=(p+1)%l;t[p]+=1;p=Math.floorMod(p-1,l);}p=(p+1)%l;System.out.print((char)t[p]);}}
```
Ah, I sure do wish there was another more compact way to print "Hello World!" in Java. Oh well.  
  
Welp. Those are the notable changes, though there are some other minimal changes and a bugfix that need not mention because I'm lazy and they're uninteresting.  
  
[Click me to get to Erdi-GitHub/BF2JC! I promise I'm not a ](https://github.com/Erdi-GitHub/BF2JC)[rickroll.](https://youtu.be/6XK4S8OQPuU)