
# Semaphore 类的基本使用


Semaphore 的基本使用
作用：多个共享资源互斥的使用！并发限流，控制最大的线程数。
Semaphore 的使用

```java
public class SemaphoreTest {
    public static void main(String[] args) {
        Semaphore semaphore= new Semaphore(3);

        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName() + "离开车位");

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },String.valueOf(i)).start();

        }
    }
}
```

原理：
semaphore.acquire() 获得，假设如果已经满了，等待，等待被释放为止！
semaphore.release() 释放，会将当前的信号量释放+1，然后唤醒等待的线程~