>**Before proceeding, make sure to have the latest Job DSL plugin installed. Install it from the update center or use the .hpi file provided from [this site](https://github.com/downloads/JavaPosseRoundup/job-dsl-plugin/job-dsl.hpi).**

This tutorial will walk you through how to create a single job using a DSL script; and then add a few more.

## 1. Creating the Seed Job
We use a Free-style Jenkins Job as a place to run the DSL scrpits. We call this a "Seed Job". Since it's a normal Job you'll get all the standard benefits of Jenkins: history, logs, emails, etc. We further enhance the Seed Job to show which Jobs got created from the DSL script, in each build and on the Seed Job page. 

The first step is to create this Job.

* From the Jenkins main page, select either the "New Job" or "Create new Jobs" link. A new job creation page will be displayed.

[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/newjob.png|center|frame]]

* Fill in the name field, e.g. "tutorial-job-dsl-1"
* Select the "Build a free-style software project" radio button.
* Click the OK button

[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/createjob.png|center|frame]]

## 2. Adding a DSL Script

Now that we have created our empty Seed Job we need to configure it. We're going to add a build step to execute the Job DSL script. Then we can paste in an example script as follows:

* On the configure screen, scroll down to the "Build: Add build step" pull down menu
[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/AddBuildStep.png|center|frame]]

* From the pull down menu, select "Process Job DSLs". You should be presented with two radio buttons. The default will be "Use the provided DSL script" and a text input box will be displayed below it.
[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/AddBuildStepSelected.png|center|frame]]

* Copy the following DSL Script block into the input box. (Note: The job resulting from this will be called DSL-Tutorial-1-Test. It'll check a GitHub repo every 15 minutes, then run 'clean test' if there's any changes found.)
```
job {
    name 'DSL-Tutorial-1-Test'
    scm {
        git('git://github.com/jgritman/aws-sdk-test.git')
    }
    triggers {
        scm('*/15 * * * *')
    }
    steps {
        maven('-e clean test')
    }
}
```
[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/DslBuildStep.png|center|frame]]

* Click the "Save" button.  You'll be shown the overview page for the new Seed job you just created.

[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/EmptySeed.png|center|frame]]

## 3. Run the Seed Job

The Seed Job is now all set up and can be run, generating the Job we just scripted. 

Note: As it stands right now, we didn't setup any build triggers to run the job automatically but we could have, using the standard Jenkins UI in Step 2. Let's just run it ourselves manually.

* Click the "Build Now" link/button on the tutorial-job-dsl-1 overview page. It should only take a second to run.
[[https://github.com/JavaPosseRoundup/job-dsl-plugin/raw/master/images/Build1.png|center|frame]]

* Look at the build result to see a link to the job which has been created. You should see a section called "Generated Jobs".

* Follow this link to the new job. You can run the job manually or wait 15 minutes for the scm trigger to kick in. (Note: if you have a new Jenkins server, you might be missing the Git plugin or a Maven installation. That could cause this job to fail when run.)

## 4. Adding additional Jobs

To show some of the power of the DSL, let's create a bunch more jobs.

* Go back to 'tutorial-job-dsl-1'
* Click configure and navigate back down the the Process Job DSLs build step.
* Add the following into the text box. (Note: The practicality of this block is questionable, but it could be used to shard your tests into different jobs.)
```
def project = 'quidryan/aws-sdk-test'
def branchApi = new URL("https://api.github.com/repos/${project}/branches")
def branches = new groovy.json.JsonSlurper().parse(branchApi.newReader())
branches.each { 
    def branchName = it.name
    job {
        name "${project}-${branchName}".replaceAll('/','-')
        scm {
            git("git://github.com/${project}.git", branchName)
        }
        steps {
            maven("test -Dproject.name=${project}/${branchName}")
        }
    }
}
```

## 5. Enjoy the results

Now you know how to make Seed Jobs, which can create a multitude of child jobs. Read up on the DSL or follow the other tutorials for more fun.
