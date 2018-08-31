# Building An App

We are going to pull our Jenkinsfile from our netcore-react example, from earlier.

You will need to go into Jenkins Credential Manager and include at least your dockerhub credentials.

**Credential**
`docker_registry` is the necessary `credential_id` - (Username/Password) to access DockerHub. 

**NOTE** Push will fail, if you are not authorized to write to the docker hub repo.

1. In your jenkins dashboard select `New Item`.
2. Call it `favorite-beer`, select `Pipeline` and click `Ok`.
3. Scroll to the `Pipeline` section, select the `Definition` dropdown and choose `Pipeline script from SCM`.
4. In the new `SCM` dropdown, select `Git`.
5. For repository URL, we will use: `https://github.com/redapt/favorite-beer`
6. If your repo is private, you will need to supply credentials, in this case it's public.
7. Under Script Path input `spa-react-netcore-redis/voting/voting/Jenkinsfile`