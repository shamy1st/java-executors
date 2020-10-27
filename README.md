# Java Executors
**Thread Pool** is a set of threads ready to use.
[ExecutorService](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html) is a java interface representing the thread pool.
  * **Impl**: AbstractExecutorService, ForkJoinPool, ScheduledThreadPoolExecutor, ThreadPoolExecutor

### Create Thread Pool

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
    
### Callable, Future
* **Callable**: like a Runnable interface but return a value.
* **Future**: object will be ready in the future.

        public class Main {
            public static void main(String[] args) {
                ExecutorService executor = Executors.newFixedThreadPool(2);

                try {
                    Future future = executor.submit(() -> {
                        LongTask.simulate();
                        return 1;
                    });

                    System.out.println("some code ...");

                    try {
                        var result = future.get();
                        System.out.println(result);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (ExecutionException e) {
                        e.printStackTrace();
                    }
                } finally {
                    executor.shutdown();
                }
            }
        }

        public class LongTask {
            public static void simulate() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
        
### Asynchronous (Non-blocking) - CompletableFuture
[link](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)

    public class Main {
        public static void main(String[] args) {
            Supplier<Integer> task = () -> 1;
            var future = CompletableFuture.supplyAsync(task);

            try {
                var result = future.get();
                System.out.println(result);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
    
### Asynchronous API

    public class Main {
        public static void main(String[] args) {
            MailService service = new MailService();
            //service.send();       //block main thread
            service.sendAsync();    //don't block

            System.out.println("do some code ...");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("main thread finished!");
        }
    }

    public class MailService {
        public void send() {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Mail sent.");
        }

        public CompletableFuture<Void> sendAsync() {
            return CompletableFuture.runAsync(() -> send());
        }
    }

### Run Code on Completion


