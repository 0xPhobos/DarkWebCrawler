# How to run Trandoshan ?

As said before Trandoshan is designed to run on distributed systems and is available as docker image which make it a great candidate for the cloud. In fact there is a repository which hold all configurations files needed to deploy a production instance of Trandoshan on a Kubernetes cluster. The files are available here: https://github.com/trandoshan-io/k8s and the containers images are available on docker hub.

If you have a kubectl configured correctly, you can deploy Trandoshan in a simple command:

``./boostrap.sh``

Otherwise you can run Trandoshan locally using docker and docker-compose. In the trandoshan-parent repository there is a compose file and a shell script that allow the application to run using the following command :

``./deploy.sh``

