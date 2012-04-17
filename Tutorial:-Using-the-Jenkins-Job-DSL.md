Before proceeding, make sure to have the latest plugin installed. Install from update center or .hpi file provided from this site.

Instead of creating each job from scratch, we recommend creating a "template" job. These are normal Jenkins jobs, configured via the Jenkins UI. An example could include one template for running tests on every commit and a daily sonar job. These templates wouldn't include anything specific to a project, like SCM. One strength of the DSL is that you only have to specify what is has changed from the template, and in this example only SCM information would have to be provided.

This tutorial will actually focus on creating three jobs, one for release, one for testing on every commit, and one for running Sonar once a day. We're assuming you have an existing Maven project.

## 1. Creating Template Jobs

* From the Jenkins main page, select "Create Job". Name it "TMPL-Release", base it on the Maven 2/3 template, and configure it to run the Maven release:release goal
* From the Jenkins main page, select "Create Job". Name it "TMPL-Test", base it on the Maven 2/3 template, and configure it to run the Maven test goal

## 2. Creating a Jenkins Job DSL Script

Add the following to source control and check it in:

```
job {
    name 'DSL-Tutorial-Release'
    using 'TMPL-Release'
    configure {
        //TBD
        configureSCM()
    }
}
job {
    name 'DSL-Tutorial-UnitTest
    using 'TMPL-Test'
    configure {
         // TBD
     }
}
```

## 3. Configuring the Jenkins Job DSL Manager

We need a job which will look at the DSL file and generate the jobs, this is called a "Seed" job. Go the Jenkins main page and click "Create Job". Add the build step called "Job DSL Manager". In the single text field, enter the path to the dsl file you checked into source control.

## 4. Creating Jenkins Jobs from your DSL Script

Run "Build Now" on your (Seed) job. Look at the build result to see a link to the job which has been created.