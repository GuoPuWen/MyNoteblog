前面JVM章节都是在说明JVM理论性知识，而学习JVM最重要的是通过理解Java程序运行的每一个状态，当项目上线之后如何通过JVM的这些知识进行性能调优，或者当出现服务器CPU到100%，死锁，内存泄漏的时候该如何进行排查问题，本文便从这些角度真实模拟环境进行排查

# CPU占满

创建SpringBoot工程，有如下请求

```java
    @GetMapping("/cpu")
    public String cpuover(){
        System.out.println("请求cpu死循环");
        Thread.currentThread().setName("loop-thread-cpu");
        int num = 0;
        while (true) {
            num++;
            if (num == Integer.MAX_VALUE) {
                System.out.println("reset");
            }
            num = 0;
        }
    }
```

浏览器请求：http://192.168.18.107:8080/cpu，使用top查看linux虚拟机运行状态，发现CPU立马到100%，是有java程序造成的

![image-20210330153447387](http://cdn.noteblogs.cn/image-20210330153447387.png)