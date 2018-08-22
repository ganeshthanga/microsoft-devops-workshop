# Events

```
$ docker events
$ docker events --since '1h'
$ docker events --since '2018-03-08T16:00'
$ docker events --filter event=attach
$ docker events --filter event=destroy
$ docker events --filter event=attach --filter event=die --filter event=stop
```
