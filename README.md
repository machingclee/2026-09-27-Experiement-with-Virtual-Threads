# 2026-09-27-Experiement-with-Virtual-Threads
## Experiment with the physical limit
Change the line  
```java
var factory = Thread.ofVirtual().name("io-testing", 0).factory();
```
into 
```java
var factory = Thread.ofPlatform().name("io-testing", 0).factory();
```
and try to adjust the number `NUM_OF_REQUESTS` that exceeds your limit (`ulimit -n`) in your local machine. 

At the point you can catch


```
Exception in thread "main" java.lang.OutOfMemoryError: unable to create native thread: possibly out of memory or process/resource limits reached
  at java.base/java.lang.Thread.start0(Native Method)
  at java.base/java.lang.Thread.start(Thread.java:1444)
  at java.base/java.lang.System$1.start(System.java:2230)
  at java.base/java.util.concurrent.ThreadPerTaskExecutor.start(ThreadPerTaskExecutor.java:226)
  at java.base/java.util.concurrent.ThreadPerTaskExecutor.submit(ThreadPerTaskExecutor.java:264)
  at java.base/java.util.concurrent.ThreadPerTaskExecutor.submit(ThreadPerTaskExecutor.java:270)
  at com.machingclee.Main.main(Main.java:21)
```

then try to switch back the to the virtual thread `ThreadFactory`, and tries to see actually how many threads is in use from the `ForkJoinPool`. The result is quite surprising.

## Experiment with the real network call

Next you can even test it with a real network call, you can uncomment the `blockingCall` implementation
```java
    private String blockingCall(String callName, Integer delay) throws URISyntaxException, InterruptedException {
        System.out.println(callName + ": " + Thread.currentThread());
        Thread.sleep(20000);
        return "";
        // URI uri = new URI("http://httpbin.org/delay/" + delay);
        // try (var inputStream = uri.toURL().openStream()) {
        //     return new String(inputStream.readAllBytes());
        // } catch (Exception e) {
        //     throw new RuntimeException(e);
        // }
    }
```
into 
```java
    private String blockingCall(String callName, Integer delay) throws URISyntaxException, InterruptedException {
        URI uri = new URI("http://httpbin.org/delay/" + delay);
        try (var inputStream = uri.toURL().openStream()) {
            return new String(inputStream.readAllBytes());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```
