# Revisiting Section 2.3.4 .NET Core - ReactJS

Recall: [.NET Core - ReactJS](../../02_docker/03_containerization/04_netcore_react.md)

With some understanding of the primitives, we can take the application we built in the containerization section and deploy it to our cluster.

Our application stack includes two microservices. 
1. Our .NET Core app (The image has been publicly staged on docker hub at `redaptcloud/favorite-beer:v1`)
2. The redis data store.

When deploying to Kubernetes it is important to deploy everyhing at the same time, or dependencies first, of course. We'll setup our redis pods, and then deploy our web app pods.

## Table of Contents

1. [Setting Up Redis](01_setting_up_redis.md)
2. [Setting Up Our Web App](02_setting_up_our_web_app.md)
