# Test Pipeline

Once you have Jenkins up, it is safe to go ahead and deploy our test pipeline, which will deploy a pod template with our necessary build containers, and print some text.

1. Log in to Jenkins
2. `cd examples` in this page's directory.
3. `cat Jenkinsfile.test`
4. Copy the contents to your clipboard.
5. Click `New Item`, call it `test`, select `Pipeline` and click `Ok`.
6. Scroll to the bottom, you'll find the `Pipeline` section, where you can paste `Jenkinsfile.test` and click `Save`.
7. Click our job `test` and then on the left, click `Build Now`.
8. Click on the build that is at the top of `Build History` on the left.
9. Click Console Output 

Your output should looks similar to the following:

```
Started by user admin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Still waiting to schedule task
Waiting for next available executor
Agent jenkins-slave-2zf0j-mmmhx is provisioned from template Kubernetes Pod Template
Agent specification [Kubernetes Pod Template] (io-publish): 
* [docker] docker:latest
* [helm] lachlanevenson/k8s-helm:v2.8.2
* [kubectl] lachlanevenson/k8s-kubectl:v1.11.1

Running on jenkins-slave-2zf0j-mmmhx in /home/jenkins/workspace/test
[Pipeline] {
[Pipeline] container
[Pipeline] {
[Pipeline] stage
[Pipeline] { (➡ Get Current Service Name)
[Pipeline] sh
[test] Running shell script
+ echo production-instance
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // container
[Pipeline] container
[Pipeline] {
[Pipeline] stage
[Pipeline] { (➡ Fetch Current Service Selector "app")
[Pipeline] sh
[test] Running shell script
+ echo testing
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // container
[Pipeline] container
[Pipeline] {
[Pipeline] stage
[Pipeline] { (➡ Deployment)
[Pipeline] echo
Running dry-run deployment with: 
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```