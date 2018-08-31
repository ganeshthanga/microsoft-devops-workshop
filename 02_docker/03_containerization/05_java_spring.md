# Java - Spring

We dont have a formal example for a Java application, but there are many available on the web. If you have a java application, you can use those examples on git to formulate your `Dockerfile`.

It's worth mentioning special considerations when containerizing a Java application. Similar caveats may apply to other other applications that run in a virtual machine, inside of a container.

## The Problem
Even when JVM is running in a docker container, it starts threads and consumes memory based on the host system resources. If you are running this container on a 12 core node, even if you limit the container to 1 cpu, it will try to start 12 threads, with only one core dedicated to the container those threads may starve.

**Java 9** has better support for containers, allowing you to specify `-XX:+UseCGroupMemoryLimitForHeap` and `-XX:+UnlockExperimentalVMOptions` to have JVM better detect its containerized environment, but when orchestrators come into play, they dont always behave the way these features are expecting.

## The Solution
The solution that generally resolves this problem is to specify `-XMX` `-XX:ParallelGCThreads` and `-XX:ConcGCThreads` as part of the JVM commands. In docker, you can ovveride the command and add these flags to it at run time. The same principle applies when deploying workloads to orchestrators like Kubernetes.

## Dockerfile w/ External Build Process

https://spring.io/guides/gs/spring-boot-docker/
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```


`docker build . -t your-tag:version --build-arg JAR_FILE=/local/path/to/your/built.jar`

`docker run your-tag:version java -jar /app.jar -XMX256m`


## Options

If you want to also build you Jar inside of the container, you can follow a similar pattern as the netcore_reacy Dockerfile, using this FROM image, and installing your build dependencies inside of the `build` image, and ultimate copy it into the lightweight section similar to the Dockerfile defined above.