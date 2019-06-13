# 字符串连接

分析字符串连接的常用方式 

- `String.concat();` 
- `StringBuilder.append();` 
- `StringBuffer.append();`  
- `String+String;` 

## 1.String.concat();

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    //创建len+otherLen长度的字符数组buf，将value复制到buf
    char buf[] = Arrays.copyOf(value, len + otherLen);
    //将str复制到buf中，从buf的len位置开始复制
    str.getChars(buf, len);
    //构造函数String(char[] value, boolean share)
    //直接使用buf创建字符串，如果此时buf被改变，字符串中的value也被改变，违反字符串不变性。
    return new String(buf, true);
    //而new String(buf)，是将buf copy进入String的value
}
```
1. String c = a.concat(b); 原理是，新建一个字符串ab，将a和b分别复制进去，然后赋给c。期间并不会产生额外的垃圾。
	但是，String d = a.concat(b).concat(c);原理是，新建一个字符串ab，将a和b分别复制进去，然后新建一个字符串abc，将ab和c分别复制进去，然后赋给d。期间产生的垃圾，字符串ab。
	所以，如果同时concat的字符串越多，产生的垃圾就越多。
	
2. 构造函数String(char[] value, boolean share) 是String中唯一不是public的构造方法，下面是该方法的分析。

```java
/*
 * Package private constructor which shares value array for speed.
 * this constructor is always expected to be called with share==true.
 * a separate constructor is needed because we already have a public
 * String(char[]) constructor that makes a copy of the given char[].
 *
 * 这是一个package级别的构造函数，为了提高速度，而分享 value数组（其他都有复制操作保证字符串不变性）
 * 这个构造方法总是期望使用 share==true 来调用它（因为不支持unshared）。
 * 需要这个单独的构造函数，是因为我们已经有一个public String(char[])构造函数，该函数制作一个给定的char[]的copy。
 */
```

## 2.StringBuilder和StringBuffer.append()

```java
//2.StringBuilder, StringBuffer
StringBuilder stringBuilder = new StringBuilder();
String strBu = stringBuilder.append(one).append(two).toString();
StringBuffer stringBuffer = new StringBuffer();
strBu = stringBuffer.append(one).append(two).toString();
```

涉及的方法，new StringBuilder/StringBuffer()、append()、toString();

**StringBuffer**在append和toString方法加了**synchronized**关键字，其他和StringBuilder一样。

分析一下这三个方法
```java
	//构造方法
    public StringBuilder() {
        super(16);
    }

    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
    
    //append方法
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        //count是sb的字符串长度，len是新增的字符串长度
        ensureCapacityInternal(count + len);
        //复制len到value中
        str.getChars(0, len, value, count);
        //更新count
        count += len;
        return this;
    }
    
    //传入连接后的字符串总长度
    private void ensureCapacityInternal(int minimumCapacity) {
        if (minimumCapacity - value.length > 0)
        	//如果字符串总长度大于现在的数组长度，扩展数组
            expandCapacity(minimumCapacity);
    }
    
    //扩展数组的逻辑
    void expandCapacity(int minimumCapacity) {
    	//新的数组长度，比如原始数组为16，新的则是34
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
        	//如果新的数组长度还是 小于 字符串总长度，则新的数组长度为 字符串总长度
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        //创建新的数组(更长)，复制旧数组的内容进去
        value = Arrays.copyOf(value, newCapacity);
    }
    
    //toString方法
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }
    //新建一个字符串(等于copy了一遍最终结果)
```

1. append的原理是，将字符串复制到字符数组，如果数组长度不够，则需要扩展产生新的数组，此时旧数组则变为垃圾。

2. toString的原理是，新建一个字符串，复制一遍结果(等于用多了一倍的空间)。

3. 如果事先知道你字符串的长度，你可以在new的时候指定长度。
	这样可以节省扩展数组时使用的空间和时间。

## 3.String+String

下面分析"+"的使用

```java
    public static void main(String[] args) {
        String a = "a";
        String result = a + "bbb" + "cccccc";
        for(int i=0;i<5;i++){
            a+="ccc";
        }
        StringBuilder sbuilder = new StringBuilder(a);
        for(int i=0;i<5;i++){
            sbuilder.append("ccc");
        }
    }
```

上述内容的字节码如下

```
0: ldc           #2                  // String a
2: astore_1
3: new           #3                  // class java/lang/StringBuilder
6: dup
7: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
10: aload_1
11: invokevirtual #5                  // Method java/lang/StringBuilder.append:
										(Ljava/lang/String;)Ljava/lang/StringBuilder;
14: ldc           #6                  // String bbbcccccc
16: invokevirtual #5                  // Method java/lang/StringBuilder.append:
										(Ljava/lang/String;)Ljava/lang/StringBuilder;
19: invokevirtual #7                  // Method java/lang/StringBuilder.toString:
										()Ljava/lang/String;
22: astore_2
23: iconst_0
24: istore_3
25: iload_3
26: iconst_5
27: if_icmpge     56
30: new           #3                  // class java/lang/StringBuilder
33: dup
34: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
37: aload_1
38: invokevirtual #5                  // Method java/lang/StringBuilder.append:
										(Ljava/lang/String;)Ljava/lang/StringBuilder;
41: ldc           #8                  // String ccc
43: invokevirtual #5                  // Method java/lang/StringBuilder.append:
										(Ljava/lang/String;)Ljava/lang/StringBuilder;
46: invokevirtual #7                  // Method java/lang/StringBuilder.toString:
										()Ljava/lang/String;
49: astore_1
50: iinc          3, 1
53: goto          25
56: new           #3                  // class java/lang/StringBuilder
59: dup
60: aload_1
61: invokespecial #9                  // Method java/lang/StringBuilder."<init>":
										(Ljava/lang/String;)V
64: astore_3
65: iconst_0
66: istore        4
68: iload         4
70: iconst_5
71: if_icmpge     87
74: aload_3
75: ldc           #8                  // String ccc
77: invokevirtual #5                  // Method java/lang/StringBuilder.append:
										(Ljava/lang/String;)Ljava/lang/StringBuilder;
80: pop
81: iinc          4, 1
84: goto          68
87: return
```

分析字节码可以看出

1. String result = a + "bbb" + "cccccc"; 具体过程是：new StringBuilder对象，append字符串a，创建字符串常量"bbbcccccc"，append该字符串，最后调用toString方法生成字符串。

2. 循环 a+="ccc"; 语句，会循环创建StringBuilder对象，执行append(a).append("ccc").toString方法。

3. 循环sbuilder.append("ccc");，则可以避免上述问题，因为sbuilder本来就是StringBuilder对象。

## 4.StringBundler

liferay公司的一个StringBuilder优化版的类。[StringBundler提供的网址](https://issues.liferay.com/browse/LPS-6072)

```java
public StringBundler() {
	_array = new String[_DEFAULT_ARRAY_CAPACITY];
}
```

首先，StringBuilder内部存放的是String[]，而不是Char[]。使用字符串数组好处在于不需要经常进行扩展操作。

```java
private static String _toString(String[] array, int arrayIndex) {
		if (arrayIndex == 0) {
			return StringPool.BLANK;
		}

		if (arrayIndex == 1) {
			return array[0];
		}

		if (arrayIndex == 2) {
			return array[0].concat(array[1]);
		}

		if (arrayIndex == 3) {
			if (array[0].length() < array[2].length()) {
				return array[0].concat(array[1]).concat(array[2]);
			}

			return array[0].concat(array[1].concat(array[2]));
		}

		int length = 0;

		for (int i = 0; i < arrayIndex; i++) {
			length += array[i].length();
		}

		UnsafeStringBuilder usb = null;

		if (length > _THREAD_LOCAL_BUFFER_LIMIT) {
			Reference<UnsafeStringBuilder> reference =
				_unsafeStringBuilderThreadLocal.get();

			if (reference != null) {
				usb = reference.get();
			}

			if (usb == null) {
				usb = new UnsafeStringBuilder(length);

				_unsafeStringBuilderThreadLocal.set(new SoftReference<>(usb));
			}
			else {
				usb.resetAndEnsureCapacity(length);
			}
		}
		else {
			usb = new UnsafeStringBuilder(length);
		}

		for (int i = 0; i < arrayIndex; i++) {
			usb.append(array[i]);
		}

		return usb.toString();
	}
```

可以看出字符串连接的几个优化建议

1. 小于等于3个字符串用concat方法

2. 先concat大的字符串

3. 大于3个字符串用append方法