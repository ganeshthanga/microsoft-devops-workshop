# Tagging and Versioning

- Tagging / Versioning
    - Does the chosen tag make sense now and for future releases?
    - Do I want to push this to the `latest` image tag?

Generally, when pulling an image from a repositoy, if you do not specify an image tag, the docker client will download the image tagged `latest`. You can take advantage of this by pushing your latest build to this tag, but you should avoid pushing **only** to `latest`, as some systems will default to using whats stored locally, rather than checking for diffs and re-downloading the new latest.

You can use the `docker tag` command to create additional tagged images, essentially as a clone, prior to pushing those to remote repositories.

When choosing a tag for your image, it should be something meaningful, that you can use to trace back to the source control commit. For instance, Jenkins build number could be used, because you can check that build in the Jenkins log, and determine which commit in SCM was used to build it. It's within reason to tag the images with the sha256 of the commit, in addition to other identifying information such as build number, etc. It really depends on what reverse lookup strategy you think would be best.