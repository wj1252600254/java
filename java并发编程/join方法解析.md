<h1>join方法解析</h1>
<h2>源码</h2>
```    
public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
//如果millis=0，则不断轮询，如果线程存活，则等待。
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```
<h2>解析</h2>
被Synchronized修饰的方法，如果要调用该方法，必须获得对象的锁。可以看到join的内部是用wait()实现的。如果在main函数中，调用线程的join方法，那么<b>main线程会获得该线程的锁</b>，main线程获取锁后，根据情况调用wait()方法等待，等待唤醒。
```
public class HelloJava {

    public static void main(String[] args) {
        Object oo = new Object();
        MyThread t1 = new MyThread("线程t1--", oo);
        MyThread t2 = new MyThread("线程t2--", oo);
        t2.start();
        t1.start();
        try {
            t1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

class MyThread extends Thread{
    private String name;
    private Object oo;
    public MyThread(String name,Object oo){
        this.name = name;
        this.oo = oo;
    }
    @Override
    public void run() {
        synchronized (oo) {
            for(int i = 0; i < 1000; i++){
                System.out.println(name + i);
            }
        }
    }
}
```
上面程序中，假设t2先执行，那么t2先获取了oo的锁，那么t1必须等t2执行完。join方法会造成当前线程wait，就如你看到的这里的wait(0)，是当前线程wait，并不是调用者wait，正如join方法的说明一样，Waits for this thread to die. 你的程序里，就是说主线程等到t1线程执行完以后再执行，主线程的wait状态，应该是由t1执行完成之后调用的notify解除，这个应该是native的。<br>
主线程main执行t1.start()和t2.start()两行代码之后，创建了t1线程和t2线程，它们竞争oo这把锁，谁拿锁谁执行。之后main线程执行t1.join，Thread类中的join方法是同步的，（被synchronized修饰的方法是锁住整个对象的也就是synchronized(t1)），所以main线程拿到了t1这把锁。main线程继续执行t1.join，判断t1这条线程是不是活着的，如果是就调用wait()方法，因为这里调用的是t1.wait，虽然t1是线程，但是在main线程执行的步骤中t1是一把锁，跟object.wait一样，让执行的线程等待，所以这里就让main线程等待了。同时t1、t2线程一直在竞争执行，t1线程的run方法执行完了，t1线程的生命周期也到头了，在API中join方法的描述中说：当线程终止时，调用this.notifyAll方法，t1线程调用了notifyAll，唤醒了main线程.main线程继续判断t1是否存活，因为t1已经执行完run方法了，这时join方法才执行完。
