+++

Categories = ["lab"]
Tags = ["building","microservices","cloudfoundry"]
date = "2017-11-29T07:49:11-04:00"
title = "Lab 1: Build and Deploy Apps on PCF"
weight = 1

+++

### Goal


To deploy and configure a microservice and UI, leverage the platform for monitoring & management of the microservice, and do a blue green deployment with zero downtime.

<!--more-->

Prerequisites
--

1. Java SDK 1.8+ [Java from Oracle](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

    Set the JAVA_HOME variable to the installation dir, in case it is not automatically set

2. Git CLI for [Windows](https://github.com/git-for-windows/git/releases/download/v2.15.1.windows.1/Git-2.15.1-64-bit.exe)
   or Git from [github.com](https://desktop.github.com)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Gradle for build (https://projects.eclipse.org/projects/tools.buildship)

5. Clone or Download the Source Code

	[PCF Workspace:  https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/](https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/)

	##### Get the source code repository app

	<img src="/images/git-clone.png" alt="Git" style="width: 70%;"/>

	Download the source code. Download as Zip file and save it in local folder

	```bash
	   unzip pcf-workspace-devops-master.zip
	```
	##### ---OR---

	Fork and Clone

	[PCF Workspace:  https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/](https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/)

	For Linux/Mac:
	```bash
	$ git clone https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops.git
	```

	For Windows
	```
	C:\<Some Directory to save code>\> git clone https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops.git
	```

5. Environment Proxy Setup (Optional depending on network configuration)

	If you are connected to a network that requires using a proxy, you'll need to set the appropriate environment variables. In the repository there is a file called *setenv.bat*.

	Open up a terminal/command prompt, change directory to the top level folder and run the setenv.bat file.

	For Mac
	```
	export HTTP_PROXY=<your http proxy>
	export HTTPS_PROXY=<your https proxy>
	```

	For Windows
	```
	set HTTP_PROXY=<your http proxy>
	set HTTPS_PROXY=<your https proxy>
	```

6. Gradle Environment Setup (Optional depending on network configuration)

	Create or edit the proxy information in gradle.properties.

	For example:
	```
    systemProp.http.proxyHost=yourproxy.example.com
    systemProp.http.proxyPort=8080
    systemProp.https.proxyHost=yourproxy.example.com
    systemProp.https.proxyPort=8080
	```

Steps
--

In this workshop we are going to follow these steps to deploy apps on Cloud foundry.

    - Get a Spring boot app from an existing Git Repository.
    - Deploy it to Pivotal Cloud foundry.
    - Manage the lifecycle of the application.

__NOTE__

> The instructions in this document are for Mac/Linux based CLI/Shell. If you are using Windows, you may need to adjust your slashes.

***
### Step 1
##### Build the app
By this point, you should have cloned (or forked, or downloaded) the [workspace repo](https://github.com/Pivotal-Field-Engineering/pcf-workspace-devops/).  Now you will build the project and deploy it to Cloud Foundry.

For Linux/Mac:


    cd pcf-workspace-devops
    ./gradlew clean build


Windows:


    cd pcf-workspace-devops
    gradlew.bat clean build


### Step 2
##### Login into Pivotal Cloud Foundry

Each participant will have their own user ids and passwords.  

````
cf login -a https://api.sys.gcp.pcf.cloud --skip-ssl-validation
  Email: student-x
  Password: ••••••••
````

Login to the App Console at https://apps.sys.gcp.pcf.cloud

<img src="/images/pcf-console.png" alt="PCF App Console" style="width: 70%;"/>


### Step 4
##### Push the app


1. Push the cities-hello, put your initials in the app name so we don't get conflicts

    ```bash
    $ cd cities-hello
    $ cf push <YOUR INITIALS>-cities-hello
    // This will give an output which is similar to this
    requested state: started
    instances: 1/1
    usage: 512M x 1 instances
    urls: cities-hello-lactiferous-unanswerableness.cfapps.gcp.pcf.cloud
    last uploaded: Mon Jun 15 14:53:10 UTC 2017
    stack: cflinuxfs2
    ```
2. Open the app url

    When you push the apps, it will give the url route to the app.
    <img src="/images/welcome.png" alt="Welcome to PCF Workshop" style="width: 70%;"/>

3. If you haven't already it is a good time to walk through the AppsManager:

        https://apps.sys.gcp.pcf.cloud

4. When you are finished, let's delete the application we just created. In the next labs we will be pushing a diferent application

    ```bash
    cf delete -r <YOUR INITIALS>-cities-hello
    ```

##### Recap:

> Cloud Foundry Haiku </br>
  Here is my source code </br>
  Run it on the cloud for me </br>
  I do not care how</br>

##### Discussion

* How do you deploy an application today?
* How does your platform understand which runtime to use to run the app?

***
