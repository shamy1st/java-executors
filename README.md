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
            //runAsync() - no return value, run in separate thread
            return CompletableFuture.runAsync(() -> send());
        }
    }

### Run Code on Completion

    public class Main {
        public static void main(String[] args) {
            CompletableFuture future = CompletableFuture.supplyAsync(() -> 1);
            //thenRun(),thenRunAsync() are  provided by CompletionStage

            //run in the current thread "main"
            future.thenRun(() -> {
                //print: main
                System.out.println(Thread.currentThread().getName());
                System.out.println("Task");
            });

            //run in a separate thread
            future.thenRunAsync(() -> {
                //print: ForkJoinPool.commonPool-worker-3
                System.out.println(Thread.currentThread().getName());
                System.out.println("Task Asynchronous");
            });

            //run after the task is completed, in the current thread "main"
            future.thenAccept(result -> {
                //print: main
                System.out.println(Thread.currentThread().getName());
                System.out.println(result);
            });

            //run after the task is completed, in a separate thread
            future.thenAcceptAsync(result -> {
                //print: ForkJoinPool.commonPool-worker-3
                System.out.println(Thread.currentThread().getName());
                System.out.println(result);
            });

            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
### Task Handling Exceptions

    public class Main {
        public static void main(String[] args) {
            CompletableFuture future = CompletableFuture.supplyAsync(() -> {
                System.out.println("doing some task ...");
                throw new IllegalStateException();
            });

            try {
                //future.get(); // throw exception
                //to prevent crashing, use .exceptionally()
                var result = future.exceptionally(exception -> 9).get();
                System.out.println(result);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
    
### Transform CompletableFuture
transform result to another form.

    public class Main {
        public static int toFahrenheit(int celsius) {
            return (int)(celsius * 1.8) + 32;
        }

        public static void main(String[] args) {
            var future = CompletableFuture.supplyAsync(() -> 20);
            future.thenApply(Main::toFahrenheit)
                    .thenAccept(fahrenheit -> System.out.println(fahrenheit));
        }
    }

### Compose CompletableFuture
start a task on completion of another task.

    public class Main {
        public static CompletableFuture<String> getEmail() {
            return CompletableFuture.supplyAsync(() -> "ahmed@example.com");
        }

        public static CompletableFuture<String> getPlaylist(String email) {
            return CompletableFuture.supplyAsync(() -> "music01,music02,music03");
        }

        public static void main(String[] args) {
            //using id get email, using email get music-playlist
            getEmail()
                .thenCompose(Main::getPlaylist)
                .thenAccept(playlist -> System.out.println(playlist));
        }
    }

### Combine CompletableFuture
start two tasks asynchronously and then combine the results.

    public class Main {
        public static void main(String[] args) {
            var productPrice = CompletableFuture.supplyAsync(() -> 20);
            var usdToEur = CompletableFuture.supplyAsync(() -> 0.9);

            productPrice.thenCombine(usdToEur, (price, exchangeRate) -> price * exchangeRate)
                    .thenAccept(result -> System.out.println(result));
        }
    }

### Waiting Many Tasks to Complete

    public class Main {
        public static void main(String[] args) {
            var task1 = CompletableFuture.supplyAsync(() -> 1);
            var task2 = CompletableFuture.supplyAsync(() -> 2);
            var task3 = CompletableFuture.supplyAsync(() -> 3);

            var allTasks = CompletableFuture.allOf(task1, task2, task3);
            allTasks.thenRun(() -> {
                try {
                    var result1 = task1.get();
                    var result2 = task2.get();
                    var result3 = task3.get();

                    System.out.println("sum: " + (result1+result2+result3));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }

                System.out.println("All tasks done!");
            });
        }
    }

### Wait Fastest Task
if you have multiple task and you want the result from the fastest task

    public class Main {
        public static void main(String[] args) {
            var task1 = CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return 100;
            });

            var task2 = CompletableFuture.supplyAsync(() -> 200);

            CompletableFuture.anyOf(task1, task2)
                    .thenAccept(result -> System.out.println(result));
        }
    }
    
### Handle Timeout

    public class Main {
        public static void main(String[] args) {
            var task = CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return 1;
            });

            try {
                //throw exception java.util.concurrent.TimeoutException
                //var result = task.orTimeout(500, TimeUnit.MILLISECONDS).get();
                var result = task.completeOnTimeout(-1, 500, TimeUnit.MILLISECONDS).get();
                System.out.println(result);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
