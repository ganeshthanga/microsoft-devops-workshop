# Java - Spring

We dont have a formal example for a Java application, but there are many available on the web. If you have a java application, you can use those examples on git to formulate your `Dockerfile`.

It's worth mentioning special considerations when containerizing a Java application. Similar caveats may apply to other other applications that run in a virtual machine, inside of a container.

## The Problem
Even when JVM is running in a docker container, it starts threads and consumes memory based on the host system resources. If you are running this container on a 12 core node, even if you limit the container to 1 cpu, it will try to start 12 threads, with only one core dedicated to the container those threads may starve.

**Java 9** has better support for containers, allowing you to specify `-XX:+UseCGroupMemoryLimitForHeap` and `-XX:+UnlockExperimentalVMOptions` to have JVM better detect its containerized environment, but when orchestrators come into play, they dont always behave the way these features are expecting.

## The Solution
The solution that generally resolves this problem is to specify `-XMX` `-XX:ParallelGCThreads` and `-XX:ConcGCThreads` as part of the JVM commands. In docker, you can ovveride the command and add these flags to it at run time. The same principle applies when deploying workloads to orchestrators like Kubernetes.