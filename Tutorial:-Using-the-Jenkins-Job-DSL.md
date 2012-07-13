----------------------------
Before proceeding, make sure to have the latest plugin installed. Install from update center or .hpi file provided from this site.
----------------------------

This tutorial will walk you through create a single job from the DSL, then to add a few more.

## 1. Creating Seed Job
We use a Free-style project as a place to run the DSL, which means you'll get all the benefits of all your plugins: history, logs, emails, etc. We further enhance the job to show which jobs got created from the DSL, in each build and on the Seed Job page. The first step is to create this job. 

# From the Jenkins main page, select "Create Job". A new job creation page will show up.
# Fill in the name field, e.g. "tutorial-job-dsl-1"
# Select the "Build a free-style software project" radio button.
# Click OK button

![Create Creation Screen](createjob.png)

## 2. Adding a DSL Script

Now that we have a job, we're going to add a build step to execute the Job DSL. Then we can paste in an example job.

# From the confiure screen, scroll down the "Add build step"
# From the pull down menu, select "Process Job DSLs". You should be presented with two radio buttons. The default should be "Use the provided DSL script" and be showing a text input box.
# Copy the following block into the input box. (Note: This job will be called DSL-Tutorial-1-Test. It'll check a GitHub repo every 15 minutes, then run 'clean test' if there's any change.)
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
# Click "Save" button.

## 3. Run Seed Job

The Seed Job is all setup now and needs to be run. As it stands right now, we didn't setup any triggers to run the job, but we could have using the standard Jenkins UI in Step 2. Let's run it.

# Click "Build Now" from tutorial-job-dsl-1. It should only take a second to run.
# Look at the build result to see a link to the job which has been created. You should see a section called "Generated Jobs".
# Follow this link to the new job. You can run the job manually or wait 15 minutes for the scm trigger to kick in. (Note: if you have a new Jenkins server, you might be missing the Git plugin or a Maven install. That could cause this job to fail when running.)

## 4. Adding additional Jobs

To show some of the power of the DSL, let's create a bunch more jobs.

* Go back to 'tutorial-job-dsl-1'
* Click configure and navigate back down the the Process Job DSLs build step.
* Add the following into the text box. (Note: The practicality of this block is questionable, but it could be used to shard your tests into different jobs.)
```
def giturl = 'git://github.com/jgritman/aws-sdk-test.git'
for(i in 0..10) {   
    job {
        name "DSL-Tutorial-1-Test-${i}"
        scm {
            git(giturl)
        }
        steps {
            maven("test -Dtest.suite=${i}")
        }
    }
}
```

## 5. Enjoy the results

Now you know how to make Seed Jobs, which can create a multitude of child jobs. Read up on the DSL or follow the other tutorials for more fun.
