# Monitoring

```
$ docker top <container_name>
$ docker stats <container_name>
```

* Examples:
```
$ docker top container1
UID   PID   PPID  C  STIME  TTY    TIME      CMD
root  1019  999   0  13:27  pts/0  00:00:00  sh

$ docker stats container1
CONTAINER ID   NAME         CPU %   MEM USAGE / LIMIT     MEM %   NET I/O         BLOCK I/O   PIDS
c5c8b37f47e4   container1   0.00%   1.504MiB / 31.24GiB   0.00%   25.5kB / 672B   0B / 0B     1
```
