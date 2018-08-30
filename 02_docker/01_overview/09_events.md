# Events

Use `docker events` to get real-time events from the server. These events differ per Docker object type.

```
$ docker events
$ docker events --since '1h'
$ docker events --since '2018-03-08T16:00'
$ docker events --filter event=attach
$ docker events --filter event=destroy
$ docker events --filter event=attach --filter event=die --filter event=stop
```

See the [Docker Docs](https://docs.docker.com/engine/reference/commandline/events/) for a full set of features and options.
