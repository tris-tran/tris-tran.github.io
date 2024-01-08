---
layout: post
category: java
title: Java - How to run a millon threads?
---


{% highlight java linenos %}
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

        //We work
        int[] work = {1, 2};
        long milis = System.currentTimeMillis();
        Methods.busyworkForMilis(milis, 1000, work);

        //We finish working
        allThreadsWait.countDown();
    }).start();
}

System.out.printf("All threads created %d\n", threadCount);
//We allow all threasd to work all at once.
countDownLatch.countDown();
//We wait for all threads to finish
allThreadsWait.await();
{% endhighlight %}

