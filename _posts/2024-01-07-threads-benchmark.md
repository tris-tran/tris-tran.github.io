---
layout: post
category: benchmark
title: What Does It Takes To Run One Millon Concurrent Tasks?
---

Some time ago I came across a disapoiting article about [Running One Millon
Concurrent Tasks](https://pkolaczk.github.io/memory-consumption-of-async/) I
decided to compare java (with and without virtual threads), go, erlang, c (with
linux pthreads), rust and nodejs. I wanted to include more languages, Haskell
comes to mind but dealving a little deep in how to configure and run this
languages feels hard enogh.

## The Game Plan

My idea is to spawn any number of threads, block all threads once started, then
we take messurments and let all threads execute and let the process finish
timing the execution.

```
for 0..N {
    SpawnThread(| -> {
        AwaitAllOtherThreads
        DoBusyWork
    })
}
-- We await user imput so we can take mesurments before letting the threads
execute
AwaitUserInput
StartAllThreads
AwaitAllThreadsToFinish
```

## Java

{% highlight java linenos %}

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

public class OldThreads {
	public static void main(String[] args) throws InterruptedException {
		int threadCount = Methods.getNumberOfThreads(args);
		AtomicInteger atomicErrors = new AtomicInteger(1);

		CountDownLatch countDownLatch = new CountDownLatch(1);
		CountDownLatch allThreadsWait = new CountDownLatch(threadCount);

		ThreadFactory threadFactory = r -> {
			Thread thread = new Thread(r);
			thread.setDaemon(true);
			return thread;
		};

		for (int i = 0; i < threadCount; i++) {
			threadFactory.newThread(() -> {
				//We wait for all threads to start
				try {
					countDownLatch.await();
				} catch (InterruptedException e) {
					atomicErrors.addAndGet(1);
				}

				Methods.busyWork();

				//We finish working
				allThreadsWait.countDown();
			}).start();
		}

		var ignore = System.console().readLine("All threads created");
		//We allow all threasd to work all at once.
		countDownLatch.countDown();
		//We wait for all threads to finish
		allThreadsWait.await();
	}
}

{% endhighlight %}








