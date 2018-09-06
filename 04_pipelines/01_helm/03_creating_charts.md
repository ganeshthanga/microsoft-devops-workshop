# Creating Helm Charts

The best way to learn how to make a good helm chart is to study the stable examples, 

There are some command that intializes or packages the chart, on your behalf, but you still have to populate the directory with the information and resources for your chart, prior to running package or install/update.

`helm create mychart` - Creates a directory called `mychart` which contains a basic helm chart that deploys an image, and potentially exposes it via a service. You can see an example in the examples folder: [examples/helm_create](examples/helm_create)

`helm package <path to chart>` will create a zip file that you can then use as the source of your chart, and or store in scm as a release.

As a helm chart developer the best place to go for information related to building out helm charts is their documentation.
https://docs.helm.sh/chart_template_guide/#the-chart-template-developer-s-guide

There you will find information about template functions, values files, built in objects, debugging, etc.


## Lets Revisit our Netcore ReactJs App

We have defined a chart in the app code repo, which can be updated with the code, and chart changes can be rolled out simultaneously. 

Discuss resources included, and the templating that is occurring in this iteration.

https://github.com/redapt/favorite-beer/tree/master/spa-react-netcore-redis/voting/voting/k8s