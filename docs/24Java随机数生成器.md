# Java随机数生成器

- ## Random
   ### 先分析构造函数
```java
public class Random implements java.io.Serializable {

    //种子唯一标识与系统当前计数器的纳秒数，按位异或
    public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }

    //获取种子唯一标识(AtomicLong)，当前值->新值，CAS更新，返回新值
    private static long seedUniquifier() {
        for (;;) {
            long current = seedUniquifier.get();
            long next = current * 181783497276652981L;
            if (seedUniquifier.compareAndSet(current, next))
                return next;
        }
    }

    //种子唯一标识，初始化
    private static final AtomicLong seedUniquifier = new AtomicLong(8682522807148012L);

    //原子种子
    private final AtomicLong seed;

    public Random(long seed) {
        if (getClass() == Random.class)
            //通过加工种子生成原子种子
            this.seed = new AtomicLong(initialScramble(seed));
        else {
            // subclass might have overriden setSeed
            this.seed = new AtomicLong();
            setSeed(seed);
        }
    }

    //加工种子
    private static long initialScramble(long seed) {
        return (seed ^ multiplier) & mask;
    }
    private static final long multiplier = 0x5DEECE66DL;
    private static final long mask = (1L << 48) - 1;

    synchronized public void setSeed(long seed) {
        this.seed.set(initialScramble(seed));
        haveNextNextGaussian = false;
    }
    private boolean haveNextNextGaussian = false;
}
```

可以看出主要的变量有3个，
AtomicLong seedUniquifier（CAS更新），Long seed，AtomicLong seed

   ### 分析主要用到的方法，nextInt()和nextInt(int bound)。
   nextInt()会返回int范围内的随机数。nextInt(int bound)返回[0,bound)间的int随机数，左闭右开。
   ```java
    public int nextInt() {
        return next(32);
    }

    public int nextInt(int bound) {
        if (bound <= 0)
            throw new IllegalArgumentException("bound must be positive");
        //获取31位随机数的int，符号位默认为0，正数
        int r = next(31);
        int m = bound - 1;
        if ((bound & m) == 0)  // i.e., bound is a power of 2
            r = (int)((bound * (long)r) >> 31);//左移bound位，再右移31位
        else {
            for (int u = r;
                 u - (r = u % bound) + m < 0;
                 u = next(31))
                ;
            //取余
        }
        return r;
    }

    protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask;
            //此处进行CAS操作，原子种子替换为新种子
        } while (!seed.compareAndSet(oldseed, nextseed));
        //根据需要的bit位数来进行返回
        return (int)(nextseed >>> (48 - bits));
    }
    private static final long addend = 0xBL;
   ```
两个方法都主要调用了next(int bits)方法，包含CAS操作。所以当并发线程越多时，Random性能就越低。

- ## ThreadLocalRandom
JDK1.7之后提供了新类**ThreadLocalRandom**来代替Random。
  
   ### 初始化部分
```java
    static final void localInit() {
        int p = probeGenerator.addAndGet(PROBE_INCREMENT);
        int probe = (p == 0) ? 1 : p; // skip 0
        long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
        Thread t = Thread.currentThread();
        UNSAFE.putLong(t, SEED, seed);
        UNSAFE.putInt(t, PROBE, probe);
    }

    //long SEED,long PROBE是哪里来的
   //静态模块初始化3个变量
   static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> tk = Thread.class;
            SEED = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSeed"));
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
            SECONDARY = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSecondarySeed"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }

//3个变量是来自线程内里面的long
class Thread {
    /** The current seed for a ThreadLocalRandom */
    @sun.misc.Contended("tlr")
    long threadLocalRandomSeed;

    /** Probe hash value; nonzero if threadLocalRandomSeed initialized */
    @sun.misc.Contended("tlr")
    int threadLocalRandomProbe;

    /** Secondary seed isolated from public ThreadLocalRandom sequence */
    @sun.misc.Contended("tlr")
    int threadLocalRandomSecondarySeed;
    }
```
三个变量来自Thread类，加上@sun.misc.Contended("tlr")注解，处理伪共享问题。
     @sun.misc.Contended注解会在对象或字段的前后各增加128字节大小的padding（[出自](https://www.jianshu.com/p/c3c108c3dcfd)）。

SEED 是控制随机数的种子。Probe控制初始化。SECONDARY 是二级种子。

   ### 生成随机数部分
   主要用到了nextSeed方法
```java
    final long nextSeed() {
        Thread t; long r; // read and update per-thread seed
        UNSAFE.putLong(t = Thread.currentThread(), SEED,
                       r = UNSAFE.getLong(t, SEED) + GAMMA);
        return r;
    }  
```
获取currentThread()，进行getLong和putLong操作，利用线程间的隔离，避免进行CAS。