### Goal
In this workshop, you will learn how to Build Pipelines to for unit testing, staging and production deployment to Cloud Foundry using Concourse.ci

<!--more-->

### Introduction

Concourse's end goal is to provide an expressive CI system with as few distinct moving parts as possible.

Concourse CI decouples your project from your CI's details, and keeping all configuration in declarative files that can be checked into version control.

Concourse limits itself to three core concepts: tasks, resources, and the jobs that compose them. Interesting features like timed triggers and synchronizing usage of external environments are modeled in terms of these, rather than as layers on top.

With these primitives you can model any pipeline, from simple (unit → integration → deploy → ship) to complex (testing on multiple infrastructures, fanning out and in, etc.).

Prerequisites
--

1. Brian's CF Env (https://apps.run.pcf.cloud) or Pivotal Web Services Account.  Create a free trial account here [Pivotal Web Services](http://run.pivotal.io/)

2. Fly cli. The fly tool is a command line interface to Concourse, it available when you bring up Concourse


Steps
--
In this workshop we are going to follow these steps to use the Concourse.CI to build pipelines and trigger pipelines to continuously deploy code on Cloud Foundry.


Learn how to

    - Start and Configure Concourse.CI server
    - Create a Pipeline
    - Trigger a Pipeline using Fly
    - Run a pipeline to test, stage and deploy on Cloud Foundry


***

## Part 1: Configure Concourse.CI Server in Google

### Step 1
##### Download Fly and Target Concourse CI Server in Google

The Concourse Server in Google is already configured from an existing Concourse installation for this workshop.

   Open a browser window and launch ***https://ci.google.pcf.cloud/***

   The userid/password for this server is
   ```
      userid: student-XX
      password: <distributed in the workshop>
   ```

   Download the fly cli from the main web page based on your OS 

Open a cmd/terminal and target the concourse server.

We will call our Concourse CI Server as *```gcp```*

    $ fly -t gcp login -c https://ci.google.pcf.cloud -k -n student-XX

    Use the same userid / password combination.

### Step 2
##### Create your first pipeline

We have an existing project `flight-school` in a git repo, which we can clone and use for our first pipeline.

````
$ git clone https://github.com/bbyers-pivotal/flight-school.git
$ cd flight-school

````

In the ci folder there is a properties file, flight-school-properties-sample.yml

This contains the cf and git specific configuration which is read by the pipeline. You don't want your configuration user name/passwords etc to be stored in the repository. Hence you will make a local copy of the file and add your specific configuration. This makes the CI pipeline portable across teams.

Make a local copy of the flight-school-properties-sample.yml


````
// These instructions are for Linux/OS. Please use the correct commands for Windows
mkdir ~/.concourse
cp ci/flight-school-properties-sample.yml ~/.concourse/flight-school-properties.yml
// This command is not required for Windows
chmod 600 ~/.concourse/flight-school-properties.yml
````

Now edit the ~/.concourse/flight-school-properties.yml
Be sure to point to the correct github-uri. The simplest uri to use is the same from the clone command in step 2.

````
github-uri: https://github.com/bbyers-pivotal/flight-school.git
github-branch: master
cf-api: https://api.run.pcf.cloud
cf-username: <student-XX>
cf-password: <password>
cf-org: ObjectPartners
cf-space: student-XX
cf-manifest-host: <student-XX>-flight-school-ci

````

Review your pipeline file in the ci folder\pipeline.yml

````
resources:
- name: flight-school
  type: git
  source:
      uri: {{github-uri}}
      branch: {{github-branch}}
- name: staging-app
  type: cf
  source:
      api: {{cf-api}}
      username: {{cf-username}}
      password: {{cf-password}}
      organization: {{cf-org}}
      space: {{cf-space}}
      skip_cert_check: true

jobs:
- name: test-app
  plan:
  - get: flight-school
    trigger: true
  - task: tests
    file: flight-school/ci/tasks/build.yml
  - put: staging-app
    params:
      manifest: flight-school/manifest.yml

````

This pipeline has two resources and a single job. The resource `flight-school` is the `git` repo and the resource `staging-app` is the `cloud-foundry` space to stage the app.

The single job in the pipeline `test-app` gets the source code from git on any commits to the repo, and triggers the task defined in the `build.yml`. Next, on completion of this task, it puts the output artifact in to the staging-app resource using the manifest file defined for `cf push`

Edit the manifest file to reflect your app name

````bash
cd ..
nano manifest.yml
````

````
name: flight-school
memory: 128M
random-route: true
path: .

````


### Step 3
##### Set the Pipeline using Fly


Now you have your pipeline defined, it is ready to be uploaded to the CI Server.

Name your pipeline based on your student-XX id

````
$ fly -t gcp set-pipeline -p student-XX-flight-school -c ci/pipeline.yml -l ~/.concourse/flight-school-properties.yml
````


### Step 4
##### Trigger the Pipeline and stage the app

Test the tasks manually before you run the whole Pipelines
````
$ fly -t gcp execute -c ci/tasks/build.yml
````


````
$ fly -t gcp pipelines // This will list all the pipelines

$ fly -t gcp unpause-pipeline -p student-XX-flight-school  // this will unpause the pipeline. You can also press play in the web ui

$ fly -t gcp trigger-job --job student-XX-flight-school/test-app
````


<img src="/images/concourse-2.png" alt="Concourse CI" style="width: 100%;"/>


Open up a new browser window and launch you just pushed app. You can get the URL of the app using

````
cf apps
````

<img src="/images/concourse-5.png" alt="Concourse FLY" style="width: 100%;"/>



### Step 5
##### Drill into the Concourse UI

Select the pipeline from the Web UI and double click on the test-app stage. This will bring up the up the details of the step.


<img src="/images/concourse-6.png" alt="Concourse FLY" style="width: 100%;"/>

To delete the pipeline, run the following command:

````
$ fly -t gcp destroy-pipeline -p student-XX-flight-school // This will DELETE the pipeline from Concourse
````
