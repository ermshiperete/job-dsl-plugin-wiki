When you get a little bit expert in your usage of the JOB DSL and Plugin, you might want to try the following Power Moves:

# Run a DSL Script locally
Before you push a new DSL script to jenkins, it's helpful to run it locally and eyeball the resulting XML. To do this follow these steps:

1. git clone https://github.com/jenkinsci/job-dsl-plugin.git
1. cd job-dsl-plugin
1. ./gradlew :job-dsl-core:oneJar
1. DSL_JAR=`find job-dsl-core -name '*standalone.jar'|tail -1
1. java -jar $DSL_JAR sample.dsl.groovy

If you already have the source code checked out then you can ignore step 1. 

What's going on here is that there's a static main method that can run the DSL, you just have to give it a filename. It'll output all the jobs' XML to the current directory. Likewise, if you use "using" (the templates-like feature) it'll look in the current directory for a file with the name of the job appended with ".xml" at the end of it.