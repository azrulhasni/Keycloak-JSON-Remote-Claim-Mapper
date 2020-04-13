Keycloak + JSON Remote Claim Mapper
===================================

 

By Azrul MADISA

April 2020

 

Introduction
------------

Lets face it, adding extra claims in Keycloak’s access token is ‘messy’. You
have to create a jar file (containing a Java or Javascript application - yes,
Javascript - in a jar), upload it somewhere, modify some config file etc. So, if
we have to do it, we might as well do it ONCE and allow for dynamic claim to
come through it.

 

Keycloak has an extension that can help us here
<https://www.keycloak.org/extensions.html >called ‘JSON Remote Claim Mapper’.
This plugin will allow Keycloak, upon authentication, to call a third party
restful service, get back a JSON payload and integrate it to an access token.

![](README.images/5xVv4d.jpg)

This tutorial will go through the steps of integrating the JSON Remote Claim
Mapper extension to Keycloak, creating a Docker image and subsequently deploying
it to Kubernetes.

 

For more information on the extension, please visit
<https://github.com/groupe-sii/keycloak-json-remote-claim>

 

Installation
------------

### Pre-requisites

-   A working Docker platform (e.g. Docker Desktop)

-   A running Kubernetes cluster

-   A working Helm installation. Follow the instructions here
    [<https://helm.sh/docs/intro/install/>] to install Helm

 

### Install Echo-Query

-   Follow the tutorial here to install Echo-query
    [<https://github.com/azrulhasni/Echo-query]>

-   One difference is that, instead of exposing a LoadBalancer service, we will
    expose a NodePort service

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
> kubectl expose deployment echoquery-app --type=NodePort --name=echoquery-http
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

### Folder

We will create a project folder, say under /Users/\<user
name\>/Documents/Keycloak-JSON-Remote-Claim-Mapper. We will address this folder
as \$WORKING_FOLDER from now on

 

### Steps

-   Download config-azrulhasni.yaml into working folder. This yams file is Helm
    configuration file

-   JSON Remote Claim Mapper is already integrated into the azrulhasni/keycloak
    image. The  config-azrulhasni.yaml will use that image for installation. To
    install, run Helm:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
> helm install keycloak -f config-azrulhasni.yaml codecentric/keycloak
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

Building on your own
--------------------

### Pre-requisite

-   Docker Hub subscription (get it here for free https://hub.docker.com) and a
    docker id

-   A working Docker platform

-   A running Kubernetes cluster

 

### Steps

-   Follow the instructions in
    <https://github.com/groupe-sii/keycloak-json-remote-claim >to obtain 3
    files:

    -   standalone.xml

    -   module.xml

    -   json-remote-claim.jar

-   We would also need the Keycloak Dockerfile. Go to and copy the content of
    the tab Dockerfile to a an empty text file called Dockerfile. Place
    Dockerfile in \$WORKING_FOLDER

![](README.images/KV6U8J.jpg)

-   First, we need to make a few adjustments to Dockerfile

    -   Delete openssl

    ![](README.images/E7hxNc.jpg)

    -   To copy our files to their respective folders as instructed, we leverage
        on COPY  instruction. Add these instructions just above ‘USER 1000'

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    COPY json-remote-claim.jar /opt/jboss/keycloak/modules/system/layers/base/fr/sii/keycloak/mapper/json-remote-claim/main/
    COPY module.xml /opt/jboss/keycloak/modules/system/layers/base/fr/sii/keycloak/mapper/json-remote-claim/main/
    COPY standalone.xml /opt/jboss/keycloak/standalone/configuration/
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   Next, build the docker image. Don’t forget the trailing dot.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
> docker build -t <your docker id>/keycloak:9.0.2 .
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   Then push the image to Docker Hub

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
> docker push <your docker id>/keycloak:9.0.2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   Run the installation instructions above to install
