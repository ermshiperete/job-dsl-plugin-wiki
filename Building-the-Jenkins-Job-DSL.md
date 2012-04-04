This page covers the steps required to build and package the Jenkins-JOB-DSL from source.  These are command-line instructions. If you have a IDE, go for your life. The steps should be very similar.

## Pre-Requisites
* Java 1.6 JDK (or above)
* Gradle (version???)

## Steps
1. Get the source code by cloning the project's git repo
1. Open a command line window and cd to the jenkins-job-dsl directory that you cloned to
1. Run "gradle:server" which will compile and package the plugin .jpi and also start a local Jenkins server with the plugin installed
1. Open a browser and navigate to http://localhost:8080 to see the Jenkins UI