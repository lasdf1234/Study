AtomicBoolean的妙用

设置开关
// 这是一个全局变量
boolean isStart = false;

然后在某一个需要设置开关的位置：
if (!isStart) {
  isStart = true;
  doSomething();
}

显然这是线程不安全的，通过使用AtomicBoolean可以这样做：
public class InitXxxService {
  private AtomicBoolean initState = new AtomicBoolean(false);
  
  @Override
  public void init() {
    if (! initState.compareAndSet(false, true)) {   // init once
      return ;
    }
    
    // TODO 写初始化代码
  }


  原理简介
  public final boolean compareAndSet(boolean expect, boolean update);
  
  这个方法的意思是，如果当前AtomicBoolean对象的值与expect相等，那么我们就去更新值为update，并且返回true，
否则返回false。

  这里其实作了两件事：
    1、当前值与expect相比较。如果相等继续第二步（即// TODO的初始化代码），如果不相等直接返回false
    2、把当前值更新为update，并返回true
