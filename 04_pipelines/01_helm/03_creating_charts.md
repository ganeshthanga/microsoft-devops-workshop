# Creating Helm Charts

The best way to learn how to make a good helm chart is to study the stable examples, 

There is a command that packages the chart, on your behalf, but you still have to populate the directory with the information and resources for your chart, prior to running it.

`helm package <path to chart>` will create a zip file that you can then use as the source of your chart, and or store in scm as a release.

