+++

Categories = ["lab"]
Tags = ["scaling","microservices","cloudfoundry"]
date = "2017-11-29T07:49:11-04:00"
title = "Lab 5: Scale an Application"
weight = 5

+++

### Goal

To take a previously deployed microservice and demonstrate how to scale the applicaiton in the Application Manager and through the command line.

<!--more-->

Prerequisites
--

### Step 1
##### Scale the Application in Apps Manager

Applications can be scaled via the command line or the console. When we talk about scale, there are two different types of scale: Vertical and Horizontal. Read [Scaling Apps](http://docs.cloudfoundry.org/devguide/deploy-apps/cf-scale.html) doc on more details on scaling applications.

When you vertically scale your application, you are increasing the amount of memory made available to your application. You would vertically scale your application while profiling your app, do performance tuning and to find the best memory settings before you deploy it in production.
Scaling your application horizontally means that you are adding application instances to increase your application throughput and performance under load.

Lets vertically scale the application to 1 GB of RAM.
  ````bash
  $ cf scale <YOUR INITIALS>-cities-service -m 1G
  ````


### Step 2
##### Scale the Application in the Console

Now scale your application down to 512 MB.

Next, lets scale up your application to 2 instances
  ````bash
  $ cf scale <YOUR INITIALS>-cities-service -i 2
  ````


To check the status of your applications you can check from the command line to see how many instances your app is running and their current state
  ````bash
  $ cf app <YOUR INITIALS>-cities-service
  ````


Once the second instance as started, scale the app back down to one instance.

You can also use the Autoscaler service from the marketplace and bind it to your app, so automatically scale it up or down.
<img src="/images/pcf-autoscaler.png" alt="Autoscaler" style="width: 70%;"/>


<br>
### Step 3
##### Verify the app from the Console

To verify that the application is running, use the following curl commands (or use your browser) to retrieve data from the service or use a browser to access the URL:

  ````bash
  $ curl -i -k https://<YOUR INITIALS>-cities-service.cfapps.gcp.pcf.cloud/cities
  ````

  ````bash
  $ curl -i -k https://<YOUR INITIALS>-cities-service.cfapps.gcp.pcf.cloud/cities/49
  ````

  ````bash
  $ curl -i -k https://<YOUR INITIALS>-cities-service.cfapps.gcp.pcf.cloud/cities?size=5
  ````

  For Windows, use your browser and visit the corresponding URLs.

<br>

##### Discussion

In this part of the lab we app and scaled the app. How do you horizontally scale your applications today?
