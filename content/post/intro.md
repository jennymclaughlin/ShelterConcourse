+++
Categories = ["agenda","workshop"]
Tags = ["logistics","speakers"]
date = "2017-11-29T07:44:03+04:00"
title = "Welcome to the Pivotal Cloud Foundry Workshop @ Object Partners"
type = "Introduction"
weight = 1
+++

##### Date and Time
Date: **December 2 2017**

Time: **9:00 AM - 4:00 PM**

Click to get the Agenda, Prerequisites and Setup for the workshop.

<!--more-->


#### Agenda
* Introductions
* Marketplace Observations
* Cloud Native Architecture
* Introduction to Pivotal Cloud Foundry
* PCF at Audatex
* Lab: Deploying applications
* Services
* Lab: Bind to a Service
* Logging and Metrics
* Lab: Scale an Application
* Lab: Bind to an external service
* Domains and Routing
* Lab: Blue Green Deployments
* Buildpacks
* Spring Cloud Services
* Lab: Spring Cloud Services
* Concourse

---

#### Speakers
+ Brian Byers - Platform Architect at Pivotal


---

#### Prerequisites
1. Java SDK 1.8+ [Java from Oracle](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

          Set the JAVA_HOME variable to the installation dir, in case it is not automatically set

2. Git CLI for [Windows](https://github.com/git-for-windows/git/releases/download/v2.15.1.windows.1/Git-2.15.1-64-bit.exe)
   or Git from [github.com](https://desktop.github.com)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Curl for [Windows](http://winampplugins.co.uk/curl/)
   Or for [Mac] (http://pdb.finkproject.org/pdb/package.php/curl)

5. An account on your PCF Environment

##### Setup

1. Install the Prerequisites software (cf cli)

2. Check if you are able to use the cf cli to connect your PCF  Environment.

          cf login -a https://api.sys.gcp.pcf.cloud  --skip-ssl-validation

3. Check if you are able to connect to Git repo and download / clone the repo using CLI
4. Login to the App Manager Console at

        https://apps.sys.gcp.pcf.cloud



#### EBooks
Beyond the Twelve-Factor App - https://pivotal.io/beyond-the-twelve-factor-app
Cloud Foundry: The Cloud Native Platform - https://pivotal.io/cloud-foundry-the-cloud-native-platform
Migrating to Cloud Native Application Architectures - https://pivotal.io/platform/migrating-to-cloud-native-application-architectures-ebook
