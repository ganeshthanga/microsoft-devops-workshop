# Integration Testing MicroServices

With Kubernetes on IaaS such as AKS, it becomes reliably feasible to bring up entire stacks of micro-services in a temporary namespace, executing an integration or load test suite against the dynamically provisioned load balancers, and have it report status back to the team.

As mentioned in the previous sections, if you are performing stress tests, it makes sense to perform them in a seperate cluster that is an analog to production. 