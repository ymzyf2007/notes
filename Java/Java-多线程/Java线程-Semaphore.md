~~~java
package test.java.util.concurrent.semaphore;

import java.util.concurrent.Semaphore;

public class 线程交替处理 {

    private Semaphore semaphore1 = new Semaphore(1);    // 初始的可用许可数目
    private Semaphore semaphore2 = new Semaphore(0);
    private Semaphore semaphore3 = new Semaphore(0);

    public int count = 10;

    public void run() {

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    int i = 0;
                    while(i++ < count) {
                        semaphore1.acquire();    // 从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
                        System.out.print("A");
                        semaphore2.release();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    int i = 0;
                    while(i++ < count) {
                        semaphore2.acquire();
                        System.out.print("B");
                        semaphore3.release();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    int i = 0;
                    while(i++ < count) {
                        semaphore3.acquire();
                        System.out.print("C");
                        semaphore1.release();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    public static void main(String args[]) throws Exception {
        线程交替处理 t = new 线程交替处理();
        t.run();
    }
}
~~~

