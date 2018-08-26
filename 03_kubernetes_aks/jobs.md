# Jobs

A *[Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/#what-is-a-job)* creates one or more pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the Job itself is complete. Deleting a Job will cleanup the pods it created.

A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).

A Job can also be used to run multiple Pods in parallel.

## Example

Below is an example Job config. It computes π to 2000 places and prints it out. It takes around 10 seconds to complete.
```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

```
$ kubctl create -f job-pi.yml
job "pi" created

$ kubectl describe jobs/pi
Name:           pi
Namespace:      default
Selector:       controller-uid=19aa42d0-f7df-11e7-8370-08002721601f
Labels:         controller-uid=19aa42d0-f7df-11e7-8370-08002721601f
                job-name=pi
Annotations:    <none>
Parallelism:    1
Completions:    1
Start Time:     Fri, 24 Aug 2018 13:25:23 -0800
Pods Statuses:  1 Running / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=19aa42d0-f7df-11e7-8370-08002721601f
           job-name=pi
  Containers:
   pi:
    Image:  perl
    Port:   <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  8s    job-controller  Created pod: pi-rfvvw
```

* Get the result of the Job run (i.e., the value of π):
```
$ pods=$(kubectl get pods --show-all --selector=job-name=pi --output=jsonpath={.items..metadata.name})
$ echo $pods
pi-rfvvw

$ kubectl logs ${pods}
3.1415926535897932384626433832795028841971693...
```

## Cron Jobs

Support for creating Jobs at specified times/dates (i.e. cron) is available in Kubernetes 1.4. See [here](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) for details.

Below is an example Cron Job. Every minute, it runs a simple job to print current time and then say hello:
```
$ cat << EOF >cronjob.yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
EOF
```

``` 
$ kubectl create -f cronjob.yml
cronjob "hello" created
 
$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
hello     */1 * * * *   False     0         <none>          11s
 
$ kubectl get jobs --watch
NAME               DESIRED   SUCCESSFUL   AGE
hello-1515793140   1         1            7s
 
$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
hello     */1 * * * *   False     0         22s             48s
```

* Get the results:
```
$ pods=$(kubectl get pods -a --selector=job-name=hello-1515793140 --output=jsonpath={.items..metadata.name})
$ echo $pods
hello-1515793140-plp8g
 
$ kubectl logs $pods
Fri Jan 12 21:39:07 UTC 2018
Hello from the Kubernetes cluster
```

* Cleanup
``` 
$ kubectl delete cronjob hello
```
