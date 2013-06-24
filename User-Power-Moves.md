When you get a little bit expert in your usage of the JOB DSL and Plugin, you might want to try the following Power Moves:

# Run a DSL Script locally
Before you push a new DSL script to jenkins, it's helpful to run it locally and eyeball the resulting XML. To do this follow these steps:

1. git clone https://github.com/jenkinsci/job-dsl-plugin.git
1. cd job-dsl-plugin
1. ./gradlew :job-dsl-core:oneJar
1. DSL_JAR=$(find job-dsl-core -name '*standalone.jar'|tail -1)
1. java -jar $DSL_JAR sample.dsl.groovy

If you already have the source code checked out then you can ignore step 1. 

What's going on here is that there's a static main method that can run the DSL, you just have to give it a filename. It'll output all the jobs' XML to the current directory. Likewise, if you use "using" (the templates-like feature) it'll look in the current directory for a file with the name of the job appended with ".xml" at the end of it.

# Generate a Job config.xml without having to fire up Jenkins
1. Add some job dsl content to a file, say job.dsl
1. Run the gradle command:  ./gradlew run -Pargs=job.dsl
Note: the run task loads the file relative to the job-dsl-core directory, so I always just put my test files in there.
Note2: if your dsl code contains a task named "myJob", the run task will generate myJob.xml.

[The original discussion about this on the Newsgroup](https://groups.google.com/forum/#!msg/job-dsl-plugin/lOYH7bL7AcM/70N1AEW219cJ)

# Access the Jenkins Environment Variables
To access the Jenkins Environment variables (such as BUILD_NUMBER) from within DSL scripts just wrap them in '${}'. E.g.: 

`println " BUILD_NUMBER = ${BUILD_NUMBER}"`

Some of the available variables are as follows:

* BUILD_CAUSE
* BUILD_CAUSE_USERIDCAUSE
* BUILD_ID
* BUILD_NUMBER
* BUILD_TAG
* EXECUTOR_NUMBER
* HOME
* HUDSON_HOME
* HUDSON_SERVER_COOKIE
* JENKINS_HOME
* JENKINS_SERVER_COOKIE
* JOB_NAME
* LANG
* LOGNAME
* NODE_LABELS
* NODE_NAME
* OLDPWD
* PWD
* SHELL
* TERM
* TMPDIR
* USER

[Original  discussion on the newsgroup](https://groups.google.com/d/msg/job-dsl-plugin/ArgUBsLgumo/v77k5G6fllkJ)