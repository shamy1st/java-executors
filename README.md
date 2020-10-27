# Java Executors

**Thread Pool** is a set of threads ready to use.

[ExecutorService](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html) is a java interface representing the thread pool.
  * **Impl**: AbstractExecutorService, ForkJoinPool, ScheduledThreadPoolExecutor, ThreadPoolExecutor

**Demo**

    public class Main {
        public static void main(String[] args) {
            //Executors.newFixedThreadPool() is more easier than "new ThreadPoolExecutor"
            ExecutorService executor = Executors.newFixedThreadPool(2);
            //print: java.util.concurrent.ThreadPoolExecutor
            System.out.println(executor.getClass().getName());

            try {
                for(int i=0; i<10; i++) {
                    executor.submit(() -> {
                        System.out.println(Thread.currentThread().getName());
                    });
                }
            } finally {
                //terminate the program after finish all tasks
                executor.shutdown();
                //terminate the program even if it have running tasks
                //executor.shutdownNow();
            }
        }
    }
