# Docker Image Repositories

The most popular image repository is DockerHub, it functions similar to GitHub, in that anything you make public is free, however private storage requires a subscription.

Different repositories have different mechanisms for retrieving login information, which you can pass to the `docker login` command. You can also deploy your own container registry, which will use the same protocols as dockerhub.

```
$ docker login [OPTIONS] [SERVER]
```

- `--password` - Your password for authenticating against the container repo.
- `--username` - Your username for authenticating against the container repo.
- `--password-stdin` - Retrieves the password from a file, or other stdin.

In order to push your image to a repository, you must have appropriate permissions, on the user or organization, that you are targeting, and on the container repo itself, if it already exists.

You can use the docker tag commands to stage your images for repo storage.

For instance:

```
$ docker tag my_local_image redaptcloud/my_local_image:latest

$ docker push redaptcloud/my_local_image:latest

$ docker tag redaptcloud/my_local_image:latest redaptcloud/my_local_image:v1

$ docker push redaptcloud/my_local_image:v1
```

In this case, the default `DockerHub` is our repository, and `redaptcloud` is the organization we are targeting. We could also push images to a username, being the repo prefix, similar to GitHub. `my_local_image` is the tag we would have originally provided in the build phase, `docker build -t my_local_image .`. `v1` and `latest` are different versions of `my_local_image`, wherein `latest` has a special case, that an un-versioned pull fetches latest.

For example:

```
$ docker pull redaptcloud/my_local_image
```

is functionally equivalent to:

```
$ docker pull redaptcloud/my_local_image:latest
```

## Credentials

When you run the docker login command, a local file `~/.docker/config` is updated to include your targeted server.

Different operating systems may generate and maintain credentials, in a different way, for example on `macosx` the passwords are stored in Keychain, and referenced here via `credsStore`. In many cases the username, password, and email would be contained here as a portable config that you can move to other systems, and authenticate against container registries with. 

```
{
    "auths": {
        "https://index.docker.io/v1/": {}
    },
    "HttpHeaders": {
        "User-Agent": "Docker-Client/17.09.1-ce (darwin)"
    },
    "credsStore": "osxkeychain"
}
```

Kubernetes will ultimately make use of this portability, when it needs to pull private images.
