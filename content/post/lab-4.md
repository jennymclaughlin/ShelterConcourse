+++
Categories = ["lab"]
Tags = ["spring","registry-server","cloudfoundry"]
date = "2017-11-29T00:19:42-04:00"
title = "Lab 8: Spring Cloud Registry Server"
weight = 8

+++


### Goal

To create a Spring boot application using Spring Cloud Registry Server and deploy it on the Cloud Foundry Platform.

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

5. Pivotal Web Services Account.  Create a free trial account here [Pivotal Web Services](http://run.pivotal.io/)

6. Gradle for build (https://projects.eclipse.org/projects/tools.buildship)


Steps
--
In this workshop we are going to follow these steps to build our first Cloud Native Spring Boot app on Cloud foundry using the Spring Cloud Registry Server.


Learn how to


    - Config a Spring Cloud Service Registry
    - Use Service Registry (Eureka) in a Spring Boot application
    - Register services (fortune-service) with Service Registry (Eureka)
    - Discover and Consume services (greeting-feign) with Service Registry (Eureka)


Desired the architecture of this Cloud Native Spring boot app is:

<img src="/images/spring-3.png" alt="Registry Server with Cloud Native Spring App" style="width: 100%;"/>


***
## PART 1: Register a service.

### Step 1
##### Get the fortune-service app

Clone the git repo which has a simple boilerplate Spring boot app built using Spring Initializer.

The Spring Labs repo contains multiple apps, we are going to focus on fortune-service app in this exercise.

````
git clone https://github.com/bbyers-pivotal/pcf-workshop-spring-labs.git
````


### Step 2
##### Login into Pivotal Cloud Foundry

The students have userId's (student-xx) and the passwords will be distributed in the workshop.
Each student is assigned their own Space (student-xx)

````
cf login -a https://api.sys.gcp.pcf.cloud --skip-ssl-validation
  Email: <student-XX>
  Password: ••••••••
````

Login to the App Console at https://apps.sys.gcp.pcf.cloud

<img src="/images/pcf-console.png" alt="PCF App Console" style="width: 100%;"/>



### Step 3
##### Configure the Spring Cloud Registry Service Instance from the marketplace

1. In the PCF App Console, create a instance of the Registry Service from the marketplace.

      <img src="/images/pcf-console-2.png" alt="Marketplace Services" style="width: 100%;"/>

2. Select the default plan.
3. Name the service instance as 'studentXX-service-registry'

      <img src="/images/pcf-registry-service-1.png" alt="Registry Service" style="width: 100%;"/>

4. This will create the studentXX-service-registry service instance. To view the configuration of this service by clicking manage.

      <img src="/images/pcf-registry-service-2.png" alt="Registry Service" style="width: 100%;"/>


### Step 4

##### Code walk through (fortune-service)

Let's walk through the code in the fortune-service app in the source repo (Step #1) using your favorite editor (Atom/Sublime/Eclipse/IntelliJ/STS)

1. Review the *fortune-service/src/main/resources/bootstrap.yml* file. The name of this app is fortune-service.

      ````
      server:
       port: 8787
      spring:
       application:
         name: fortune-service
      ````
      spring.application.name is the name the application will use when registering with Eureka.

2. Review the *fortune-service/build.gradle* file. By adding *spring-cloud-services-starter-service-registry* to the classpath this application is eligible to register and discover services with the service-registry.

      ````
      dependencies {
           compile('io.pivotal.spring.cloud:spring-cloud-services-starter-service-registry')
            ...
      }
      ````

3. Review the *fortune-service/src/main/java/io/pivotal/FortuneServiceApplication.java*

      Notice the ````@EnableDiscoveryClient.```` This enables a discovery client that registers the fortune-service with the service-registry application.

        @SpringBootApplication
        @EnableDiscoveryClient
        public class FortuneServiceApplication {

            public static void main(String[] args) {
                SpringApplication.run(FortuneServiceApplication.class, args);
            }
        }



### Step 5
##### Push the app to cloud Foundry

1. Change the manifest.yml file in the fortune-service/ to reflect the name of the app and the service-registry

        applications:
        - name: fortune-service
          memory: 1G
          instances: 1
          host: <student-XX>-fortune-service
          path: ./build/libs/fortune-service-0.0.1-SNAPSHOT.jar
          services:
          - <student-XX>-service-registry
          env:
            TRUST_CERTS: api.sys.gcp.pcf.cloud


2. Build the app using gradle

      ````
      ./gradlew clean build
      ````

3. Push the app using cf cli

      ````
      cf push
      ````

6. Open in the browser the App
   Get the route to your app

      ````
      http://student1-fortune-service.cfapps.gcp.pcf.cloud/
      ````

### Step 6
##### Verify the App is registered in the Service Registry

After the a few moments, check the service-registry dashboard. Confirm the fortune-service is registered.

<img src="/images/pcf-registry-service-3.png" alt="Service Registry" style="width: 100%;"/>




***
## PART 2: Consume a service.

### Step 7
##### Build Restful interface for consuming services using Netflix Feign

You have the greeting-service app in the cloned repo (Step 1) which has the client to consume service.

Lets walk through the code

1. In the greeting-feign/pom.xml file , with the *spring-cloud-starter-feign*  dependency,  this application is eligible to discover services with the service-registry.
      ````
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
      </dependency>
      ````

2. In the *GreetingFeignApplication.java* includes the @EnableDiscoveryClient annotation on a configuration class. To have Feign client interfaces automatically configured, it must also use the @EnableFeignClients annotation.

      ````
      @SpringBootApplication
      @EnableDiscoveryClient
      @EnableFeignClients
      public class GreetingFeignApplication {

          public static void main(String[] args) {
              SpringApplication.run(GreetingFeignApplication.class, args);
          }

      }
      ````


3. To call a registered service, a consuming application can use a URI with a hostname matching the name with which the service is registered in the Service Registry. This way, the consuming application does not need to know the service application’s actual URL; the Registry will take care of finding and routing to the service.

      Pivotal Cloud Foundry installation is configured to only allow HTTPS traffic, you must specify the https:// scheme in the base URI used by your client application.

      ````
      @FeignClient("https://fortune-service")
      public interface FortuneServiceClient {

      	 @RequestMapping(method = RequestMethod.GET, value = "/")
      	 String getFortune();
      }
      ````


### Step 8
##### Build and Push the Consuming app

1. Next, update the manifest.yml

        ---
        applications:
        - name: greeting-feign
          memory: 1G
          instances: 1
          host: student-XX-greeting-feign
          path: ./build/libs/greeting-feign-0.0.1-SNAPSHOT.jar
          services:
          - student-XX-service-registry
          env:
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

      Get the route to your app

      ````
    http://studentXXX-greeting-feign.cfapps.gcp.pcf.cloud/
      ````

      <img src="/images/pcf-registry-example.png" alt="Service Registry Example" style="width: 100%;"/>

### Step 9
##### Verify the App is registered in the Service Registry

This app is also registered as a service in the Service registry. Check the service-registry dashboard. Confirm the fortune-service and greeting-feign is registered.

<img src="/images/pcf-registry-service-4.png" alt="Service Registry" style="width: 100%;"/>
