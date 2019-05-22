开篇
  AtomicBoolean位于java.util.concurrent.atomic包下，是java提供给的可以保证数据的原子性操作的一个类。
  Atomicxxxx系列主要核心在于Unsafe这个类的运用保证线程安全，而Unsafe这个类应该是通过JNI调用的底层实现。

AtomicBoolean类构造器
  AtomicBoolean的构造器分为有参和无参两种：
    参数为boolean initialValue的构造器函数内部会将
    无参数的构造函数默认初始化value为0
  
  AtomicBoolean的关键逻辑在于static代码块中通过unsafe接口初始化value的内存地址，后续直接通过内存地址进行操作。
  
  public class AtomicBoolean implements java.io.Serializable {
    private static final long serialVersionUID = 4654671469794556979L;
    // 设置为使用Unsafe进行原子更新
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    // 保存修改变量的实际内存地址，通过unsafe.objectFiledOffset读取
    private static final long valueOffset;
    
    // 初始化的时候计算出保存的value的内存地址便于直接进行内存操作
    static {
      try {
        valueOffset = unsafe.objectFieldOffset(AtomicBoolean.class.getDeclaredField("value"));
      } catch (Exception ex) {
        throw new Error(ex):
      }
    }
    
    private volatile int value;
    
    // 根据bool类型的入参initialValue初始化int类型的value
    public AtomicBoolean(boolean initialValue) {
      value = initialValue ? 1 : 0;
    }
    
    // int类型的value默认初始化为0
    public AtomicBoolean() {
    }
  }
  
  AtomicBoolean的get操作
    AtomicBoolean的get操作当中直接转化为int类型的value为boolean对象进行返回。
    AtomicBoolean的getAndSet操作可以好好研究研究，首先get之前的值prev，然后通过compareAndSet进行设置。
设置成功就返回之前的值，否则直到设置成功后才返回原来的值，有种乐观锁的概念。
    compareAndSet函数内部是通过unsafe.compareAndSwapInt进行原子操作且必须期待着为expect的情况下才能设置update。如果
如果expect和实际存储的值不一致那么就一直循环更新。
    getAndSet无法区分ABA的场景，也就是说原来是A的时候被其他线程更新为B后再由其他线程更新为A，这种它是区分不出来的。
  
  public final boolean get() {
    return value !=0;
  }
  
  public final boolean getAndSet(boolean newValue) {
    boolean prev;
    do {
      prev = get();
    } while (!compareAndSet(prev, newValue));
    return prev;
  }
  
  public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e ,u);
  }
  
  AtomicBoolean的set操作
    AtomicBoolean的set相关操作中分为两大类：
    
    1、set()直接覆盖value值
    2、compareAndSet()、weakCompareAndSet()、lazySet()内部都是通过unsafe接口进行操作。
    3、compareAndSet()、weakCompareAndSet()内部都是通过compareAndSwapInt进行交换保证满足expect后才能更新
为udpate值
    4、lazySet()方法通过unsafe的putOrderedInt进行保存，网上说法是这个新修改的值会有一定的延迟可见性的效果，
没法继续深究。

  public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
  }
  
  public boolean weakCompareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
  }
  
  public final void set(boolean newValue) {
    value = newValue ? 1 : 0;
  }
  
  public final void lazySet(boolean newValue) {
    int v = newValue ? 1 : 0;
    unsafe.putOrderedInt(this, valueOffset, v);
  }
  
  
  
