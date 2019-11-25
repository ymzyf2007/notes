```package com.hydbest.app.lbd.interview;

import java.util.concurrent.ArrayBlockingQueue;

/**
 * 现有的程序代码模拟产生了16个日志对象，并且需要运行16秒才能打印完这些日志，请在程序中增加4个线程去调用parseLog()方法来分头打印这16个日志对象，程序只需要运行4秒即可打印完这些日志对象
 *
 */
public class ProducerConsumerWithBlockingQueue {
	
//	public static void main(String[] args) {
//		System.out.println("begin:" + (System.currentTimeMillis() / 1000));
//		/* 模拟处理，当前代码需要运行16秒才能打印完这些日志 */
//		for(int i = 0; i < 16; i++) {	// 这行代码不能改动
//			final String log = "" + (i + 1);	// 这行代码不能改动
//			parseLog(log);
//		}
//	}
//	
//	/**
//	 * parseLog方法内部的代码不能改动
//	 * @param log
//	 */
//	public static void parseLog(String log) {
//		System.out.println(log + ":" + (System.currentTimeMillis() / 1000));
//		try {
//			Thread.sleep(1000);	// 模拟业务处理
//		} catch (InterruptedException e) {
//			e.printStackTrace();
//		}
//	}
	
	/****************************以上代码修改前*******************************************/
	// 按照要求，主要是parseLog内部耗费主要时间。需要在4s打印完结果的话，则需要4个线程来消费这些对象。解决思路，我们可以利用阻塞队列，生产对象的是在主线程，用4个线程去消费，则可以满足要求
	public static void main(String[] args) {
		// 创建一个阻塞队列，用于存储日志对象
		final ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<String>(16);
		
		/* 模拟处理，当前代码需要运行16秒才能打印完这些日志 */
		for(int i = 0; i < 16; i++) {	// 这行代码不能改动
			final String log = "" + (i + 1);	// 这行代码不能改动
			try {
				queue.put(log);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		// 创建4个消费线程用于消费日志对象
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					parseLog(queue.take());
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}).start();
	}
	
	/**
	 * parseLog方法内部的代码不能改动
	 * @param log
	 */
	public static void parseLog(String log) {
		System.out.println(log + ":" + (System.currentTimeMillis() / 1000));
		try {
			Thread.sleep(1000);	// 模拟业务处理
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```