+++
Categories = ["lab"]
Tags = ["spring","config-server","cloudfoundry"]
date = "2017-11-29T23:37:14-04:00"
title = "Lab 7: Spring Cloud Config Server"
weight = 7
+++


### Goal

To create a Spring boot application using Spring Cloud Config Server to store and fetch configuration information and deploy it on the Pivotal Cloud Foundry Platform.


<!--more-->

### Introduction

Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer’s own laptop, bare metal data centers, and managed platforms such as Cloud Foundry.

The big picture : Use Spring Cloud Services design patterns to build cloud Native applications

<img src="/images/spring-1.png" alt="Cloud Native Spring Application Architecture" style="width: 100%;"/>



Prerequisites
--

1. Java SDK 1.7+

2. Git from [github.com](https://mac.github.com/)

3. Cloud Foundry CLI for [Mac](https://github.com/cloudfoundry/cli/releases) or [Windows](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html#windows)

4. Curl from [curl](http://curl.haxx.se/)

5. Gradle for build (https://projects.eclipse.org/projects/tools.buildship)

6. Access to a Pivotal Cloud Foundry instance


Steps
--
In this workshop we are going to follow these steps to build our first Cloud Native Spring Boot app on Cloud foundry using the Spring Cloud Config Server.


Learn how to

    - Set up a Git repository to hold configuration data
    - Configure Spring Cloud Config server (config-server) on Pivotal Cloud Foundry with a Git backend
    - Set up a client (greeting-config) to pull configuration from the config-server
    - Use @ConfigurationProperties to capture configuration changes (greeting-config)
    - Use @RefreshScope to capture configuration changes (greeting-config)
    - Use Cloud Bus to notify applications (greeting-config) to refresh configuration at scale
    - Config a Spring Cloud Service Registry

Desired the architecture of this Cloud Native Spring boot app is:

<img src="/images/spring-2.png" alt="Config Server with Cloud Native Spring App" style="width: 100%;"/>


* * *

### Step 1
##### Get the greeting-config app

Clone the git repo which has a simple boilerplate Spring boot app built using Spring Initializer.

The Spring Labs repo contains multiple apps, we are going to focus on greeting-config app in this exercise.

````
git clone https://github.com/bbyers-pivotal/pcf-workshop-spring-labs.git
````


### Step 2
##### Login into Pivotal Cloud Foundry

The participants have userId's and the passwords will be distributed prior to the workshop.


    ````
    cf login -a https://api.sys.gcp.pcf.cloud --skip-ssl-validation
    Email: student-x
    Password: ••••••••
    ````

Login to the App Console at https://apps.sys.gcp.pcf.cloud

    <img src="/images/pcf-console.png" alt="PCF App Console" style="width: 100%;"/>


### Step 3
##### Set the config data

The greeting-config app uses the Spring Cloud Services Config Server to read config data.

Fork the repo (http://www.github.com/bbyers-pivotal/workshop-app-config)

<img src="/images/config-server-fork.png" alt="Fork" style="width: 100%;"/>

You can make changes to the application config files in the forked repo at http://github.com/your-github-account/workshop-app-config

In case you want to make local changes and commit to the repo, then clone the git repo which has the config properties which are read by the greeting-config

````bash
$git clone https://github.com/your-github-account/workshop-app-config.git
$cd workshop-app-config/
````

In this repo you have the following config files:

<img src="/images/git-1.png" alt="Git Config Server Files" style="width: 200px;"/>

The config server serves the configuration request using the following path formats, where the application name is set in the application.yml for the client application, profile and label are set as environment variables.

````
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.Properties
````
For more details on the Config Server config files refer to the documentation (http://docs.pivotal.io/spring-cloud-services/config-server/server.html)

You could also have multiple branches in your Git repo, and in the Config Service instance, you can configure which branch to read the config information.

````
master
------
https://github.com/myorg/configurations
|- myapp.yml
|- myapp-development.yml
|- myapp-production.yml

tag v1.0.0
----------
https://github.com/myorg/configurations
|- myapp.yml
|- myapp-development.yml
|- myapp-production.yml

````

### Step 4
##### Configure the Spring Cloud Config Service Instance from the marketplace

1. In the PCF App Console, create a instance of the Config Server service from the marketplace.

<img src="/images/pcf-console-1.png" alt="Marketplace Services" style="width: 100%;"/>

2. Select the default plan.
3. Name the service instance as '<YOUR INITIALS>-config-service'

<img src="/images/pcf-config-service-1.png" alt="Config Server" style="width: 100%;"/>

4. This will create the <YOUR INITIALS>-config-service service instance. .

<img src="/images/pcf-config-service-2.png" alt="Config Server" style="width: 100%;"/>

5. Next, to configure the Config server to point to the Git Data store do the following from the `cf cli`

For Windows

        `cf update-service student1-config-service -c "{\"git\": {\"uri\": \"https://github.com/your-github-account/workshop-app-config.git\"}}"`

For Mac/Linux

        `cf update-service student1-config-service -c '{"git": {"uri": "https://github.com/your-github-account/workshop-app-config.git"}}'`

Note: The Git repository URL is the URL of your cloned git repo in Step 3. If you are using the instructor git url, you can specify that.


For detailed documentation on the other configuration items, refer to the product documentation.



### Step 5

##### Code walk through (greeting-config)

Import the greeting-config project in Eclipse as a Gradle Project.

Let's walk through the code in the greeting-config app in the source repo (Step #1) using your favorite editor (Atom/Sublime/Eclipse/IntelliJ/STS)

1. greeting-service

      In GreetingProperties.java, @ConfigurationProperties annotation allows for reading of configuration values. Configuration keys are a combination of the prefix and the field names. In this case, there is one field (displayFortune). Therefore greeting.displayFortune is used to turn the display of fortunes on/off. Remaining code is typical getter/setters for the fields.

      ````
      @ConfigurationProperties(prefix="greeting")
      public class GreetingProperties {

      	private boolean displayFortune;

      	public boolean isDisplayFortune() {
      		return displayFortune;
      	}

      	public void setDisplayFortune(boolean displayFortune) {
      		this.displayFortune = displayFortune;
      	}
      }
      ````

      greetingProperties.isDisplayFortune() is used to turn the display of fortunes on/off. There are times when you want to turn features on/off on demand.

      ````
      @EnableConfigurationProperties(GreetingProperties.class)
      public class GreetingController {

      	Logger logger = LoggerFactory
      			.getLogger(GreetingController.class);


      	@Autowired
      	GreetingProperties greetingProperties;

      	@Autowired
      	FortuneService fortuneService;

      	@RequestMapping("/")
      	String getGreeting(Model model){

      		logger.debug("Adding greeting");
      		model.addAttribute("msg", "Greetings!!!");

      		if(greetingProperties.isDisplayFortune()){
      			logger.debug("Adding fortune");
      			model.addAttribute("fortune", fortuneService.getFortune());
      		}

      		//resolves to the greeting.vm velocity template
      		return "greeting";
      	}

      }
      ````

2. quote-service

      QuoteService uses the @RefreshScope annotation. Beans with the @RefreshScope annotation will be recreated when refreshing configuration. The @Value annotation allows for injecting the value of the quoteServiceURL configuration parameter.

      ````
      @Service
      @RefreshScope
      public class QuoteService {
      	Logger logger = LoggerFactory
      			.getLogger(QuoteController.class);

      	@Value("${quoteServiceURL}")
      	private String quoteServiceURL;

      	public String getQuoteServiceURI() {
      		return quoteServiceURL;
      	}

      	public Quote getQuote(){
      		logger.info("quoteServiceURL: {}", quoteServiceURL);
      		RestTemplate restTemplate = new RestTemplate();
      		Quote quote = restTemplate.getForObject(
      				quoteServiceURL, Quote.class);
      		return quote;
      	}
      }
      ````

3. greeting-config.yml

      In the app-config repo in the Github, review the greeting-config.yml file, which has the displayFortune turned on and the quoteService point to an existing URL.

      ````
      security:
        basic:
          enabled: false

      management:
        security:
          enabled: false

      logging:
        level:
          io:
            pivotal: DEBUG

      greeting:
        displayFortune: true # <----Change to true

      quoteServiceURL: http://quote-service-dev.cfapps.gcp.pcf.cloud/quote
      ````


### Step 6
##### Push the app

1. Change the manifest.yml file in the greeting-config/ to reflect the name of the app and the config-service

            ---
        applications:
        - name: greeting-config
          memory: 1G
          buildpack: https://github.com/cloudfoundry/java-buildpack
          instances: 1
          host: <student-XX>-greeting-config
          path: build/libs/greeting-config-0.0.1-SNAPSHOT.jar
          services:
            - <student-XX>-config-server
          env:
            SPRING_PROFILES_ACTIVE: dev
            TRUST_CERTS: api.sys.gcp.pcf.cloud


2. Build the app using gradle

      ````
      ./gradlew clean build
      ````

3. Push the app using cf cli

      ````
      cf push
      ````

4. Open in the browser the App

      ````
      http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/
      http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/random-quote
      ````

### Step 7
##### Change the property and curl to RefreshScope

1. In the app-config repo, edit the greeting-config.yml file.

      ````
      greeting:
        displayFortune: false # <----Change to true

      quoteServiceURL: http://quote-service-qa.cfapps.gcp.pcf.cloud/quote
      ````

2. Force refresh the beans

      ````
      curl -X POST http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/refresh
      ````

      This will output the properties which changed
      ````
      ["quoteServiceURL","greeting.displayFortune"]
      ````

3. Open in the browser the App

      You will see the Greetings doesn't have any fortune and the random-quote is from qa service

        http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/
        http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/random-quote


### Step 8
##### Change the profile and Push

1. Next, update the manifest.yml to point to the SPRING_PROFILES_ACTIVE to `qa`

        ---
        applications:
        - name: <studentXXX>-greeting-config
          memory: 512M
          buildpack: https://github.com/cloudfoundry/java-buildpack
          instances: 1
          host: <studentXXX>-greeting-config
          path: target/greeting-config-0.0.1-SNAPSHOT.jar
          services:
            - <studentXXX>-config-service
          env:
            SPRING_PROFILES_ACTIVE: qa

2. Push the app using cf cli

      ````
      cf push
      ````


3. Now the properties will be served by app-config/greeting-config-qa.yml

      You can verify by opening the two URLs

        http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/
        http://<student-XX>-greeting-config.cfapps.gcp.pcf.cloud/random-quote


### Step 9
##### Refreshing Application Configuration at Scale with Cloud Bus

When running several instances of your application, this poses several problems:

1. Refreshing each individual instance is time consuming and too much overhead
2. When running on Cloud Foundry you don’t have control over which instances you hit when sending the POST request due to load balancing provided by the router

Spring Cloud Bus addresses the issues listed above by providing a single endpoint to refresh all application instances via a pub/sub notification.

1. Create a RabbitMQ service instance, bind it to greeting-config

      ````
      $ cf create-service p-rabbitmq standard cloud-bus
      $ cf bind-service <studentXXX>-greeting-config cloud-bus
      ````

2. Add the dependency to the build.gradle

      ````

      dependencies {
        compile('org.springframework.cloud:spring-cloud-starter-bus-amqp')

      ````

3. Build the app and push 2 app instances

      ````
      $./gradlew clean build
      $cf push -i 2
      ````

4. Change the app-config/greeting-config.yml and refresh all the app instances using Cloud Bus

      ````
      curl -X POST http://<studentXXX>-greeting-config.cfapps.gcp.pcf.cloud/bus/refresh
      ````

5. Verify by opening the two URLs

        http://<studentXXX>-greeting-config.cfapps.gcp.pcf.cloud/
        http://<studentXXX>-greeting-config.cfapps.gcp.pcf.cloud/random-quote

### Step 10
##### Spring Actuator Endpoints

Check the Actuator Endpoints

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/beans``

Dumps all of the beans in the Spring context.

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/autoconfig``

Dumps all of the auto-configuration performed as part of application bootstrapping.

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/configprops``

Displays a collated list of all @ConfigurationProperties.

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/env``

Dumps the application’s shell environment as well as all Java system properties.

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/mappings``

Dumps all URI request mappings and the controller methods to which they are mapped.

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/dump``

Performs a thread dump.

``http://<YOUR INITIALS>-greeting-config.cfapps.gcp.pcf.cloud/trace``

# Advanced Topics

## Encryption and Decryption of data
The Config Server can serve encrypted property values from a configuration file. If the Config Server is configured with a symmetric or asymmetric encryption key and the encrypted values are prefixed with the string {cipher}, the Config Server will decrypt the values before serving them to client applications. The Config Server has an /encrypt endpoint, which can be used to encrypt property values.

      {"git": {"uri": "https://github.com/spring-cloud-services-samples/cook-config.git" }, "encrypt": { "key": "KEY" }}

And to get the encrypted values, first get the OAuth TOKEN_STRING

        $ cf env cook
        Getting env variables for app cook in org myorg / space development as admin...
        OK

        System-Provided:
        {
         "VCAP_SERVICES": {
          "p-config-server": [
           {
            "credentials": {
             "access_token_uri": "https://p-spring-cloud-services.uaa.sys.gcp.pcf.cloud/oauth/token",
             "client_id": "p-config-server-876cd13b-1564-4a9a-9d44-c7c8a6257b73",
             "client_secret": "rU7dMUw6bQjR",
             "uri": "https://config-86b38ce0-eed8-4c01-adb4-1a651a6178e2.cfapps.gcp.pcf.cloud"
            },
        [...]


  To get the token and assign to $TOKEN

        TOKEN=$(curl -k ACCESS_TOKEN_URI -u CLIENT_ID:CLIENT_SECRET -d
        grant_type=client_credentials | jq -r .access_token);

  Use this token in your curl request.

        curl -k -H "Authorization: bearer $TOKEN" -H "Accept: application/json" URI/ENDPOINT | jq
        curl -H "Authorization: bearer $TOKEN" http://SERVER/encrypt -d 'Value to be encrypted'`

The Config Server returns the encrypted value. You can use the encrypted value in a configuration file with the {cipher} prefixed
In a file within the configuration repository, properties whose values are prefixed with {cipher} will be decrypted before they are served to client applications.

Example encrypted values in the properties file

        secretMenu: '{cipher}AQA90Q3GIRAMu6ToMqwS++En2iFzMXIWX99G66yaZFRHrQNq64CntqOzWymd3xE7uJp
        ZKQc9XBIkfyRz/HUGhXRdf3KZQ9bqclwmR5vkiLmN9DHlAxS+6biT+7f8ptKo3fzQ0gGOBaR4kTnWLBxmVaIkjq1
        Qze4aIgsgUWuhbEek+3znkH9+Mc+5zNPvwN8hhgDMDVzgZLB+4YnvWJAq3Au4wEevakAHHxVY0mXcxj1Ro+H+Zel
        IzfF8K2AvC3vmvlmxy9Y49Zjx0RhMzUx17eh3mAB8UMMRJZyUG2a2uGCXmz+UunTA5n/dWWOvR3VcZyzXPFSFkhN
        ekw3db9XZ7goceJSPrRN+5s+GjLCPr+KSnhLmUt1XAScMeqTieNCHT5I='


SCS 1.4 adds support for Hashicorp Vault backends to Config Server service instances and adds support for declaratively defining composite (i.e. combined Git + Vault) configuration of multiple backends to Config Server service instances.


## Git SSH and HTTPS and HTTP/S Proxy access

You can configure a Config Server configuration source so that the Config Server accesses it using the Secure Shell (SSH) protocol

      '{"git": { "uri": "ssh://git@github.com/spring-cloud-services-samples/cook.git", "hostKey": "AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+...", "hostKeyAlgorithm": "ssh-rsa", "privateKey": "-----BEGIN RSA PRIVATE KEY-----\nMIIJKQIB..."} }'


You can configure a Config Server service instance to access configuration sources using an HTTP or HTTPS proxy.

      '{"git": { "proxy": { "http": { "host": "proxy.wise.com", "port": "80" } } } }'
      '{"git": { "proxy": { "http": { "host": "proxy.wise.com", "port": "80" } } } }'

You can configure a Config Server service instance to use multiple configuration sources, which will be used only for specific applications or for applications which are using specific profiles

      '{"git": { "repos": { "cookie": { "uri": "https://github.com/myorg/config-repo", "label": "develop" } } } }'
      '{"git": { "repos": { "cookie": { "pattern": "co*/dev*", "uri": "https://github.com/spring-cloud-services-samples/cook-config" } } } }'
