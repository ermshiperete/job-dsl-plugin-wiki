# Job Factory
The DSL execution engine exposes only one method, called 'job'. This 'job' method implies the creation of a Jenkins job and the closure to this method then further exposed some methods where things get interesting. See the later sections to learn the specific available methods available, for when a DSL helper method isn't available, look to the [[configure block]]. Below is the simplest job possible:

```groovy
job {
    name 'Simplest Job Possible'
}
```

Because the engine is just Groovy, you can call other Groovy classes available in the workspace. When in those methods the 'job' method is no longer available, so it is recommended to pass in the current context to make this method available to other context. For example, when making utility methods, you would call them like this:

```groovy
BuildFramework.ant(this, arg1, arg2)
```

Then the BuildFramework class has enough to make 'job' calls, it would look like this:

```groovy
public class BuildFramework {
    public void static ant(jobFactory, arg1, arg2) {
        jobFactory.job {
            name arg1
            steps {
                ant(arg2)
            }
        }
    }
}
```

TBD, Current this method is hard-coded to free-form and maven projects. It wouldn't be too hard to make it implement multi-job and ivy projects. File a bug if you'd find this useful.

# DSL Methods

This is the formal documentation of the available DSL methods. In the Closure provided to job there are a few top level methods, like label and chucknorris. Others are nested deeper in blocks which represent their role in Jenkins, e.g. the publishers block contains all the publisher steps. The one caveat is that scm and multiscm are mutually exclusive. Likewise, when using the scm block, only one scm can be specified.

Further sections will define in detail how they work, in a Java-like syntax. If an argument is followed with an equals, this means it's a default value. DSL Methods can be cumulative or overriding, meaning that some methods will add nodes (e.g. publishers and steps) and some will replace nodes (e.g. disabled() will replace any existing disabled tags).

**NOTE: when using these commands, remember that you need to use them in context.  I.e. to use the "downstream" command, it needs to be enclosed in a "publisher" command block.**

Here's a high level overview of what's available:

```groovy
job(attributes) {
    name(nameStr)
    using(templateNameStr)
    description(descStr)
    label(labelStr)
    timeout(timeoutInMinutes, shouldFailBuild)
    disabled(shouldDisable)
    block(projectNames)
    logRotator(daysToKeepInt, numToKeepInt, artifactDaysToKeepInt, artifactNumToKeepInt)
    jdk(jdkStr)
    rootPOM(rootPOMStr)
    goals(goalsStr)
    mavenOpts(mavenOptsStr)
    perModuleEmail(shouldSendEmailPerModule)
    archivingDisabled(shouldDisableArchiving)
    runHeadless(shouldRunHeadless)
    environmentVariables(vars)
    environmentVariables(closure) // See below for details of EnvironmentVariablesContext
    priority(value)
    authorization {
        permission(permissionStr) // e.g. hudson.model.Item.Workspace:authenticated
        permission(String permEnumName, String user)
        permission(Permission perm, String user)
        permissionAll(securityGroup)
    }
    scm {
        hg(url, branch) {}
        git(url, branch) {}
        svn(svnUrl, localDir) {}
        p4(viewspec, user, password) {}
    }
    triggers {
        cron(cronString)
        scm(cronString)
        gerrit(gerritClosure) // See below for gerritClosure syntax
        snapshotDependencies(checkSnapshotDependencies)
    }
    multiscm {
        hg(url, branch) {}
        git(url, branch) {}
        github(ownerAndProject, branch, protocol, host) {}
        svn(svnUrl) {}
        p4(viewspec, user, password) {}
    }
    steps {
        shell(String commandStr)
        batchFile(String commandStr)
        gradle(tasksArg, switchesArg, useWrapperArg) {}
        maven(targetsArg, pomArg) {}
        ant(targetsArg, buildFileArg, antInstallation, antClosure) // See below for antClosure syntax
        copyArtifacts(jobName, includeGlob, targetPath, flattenFiles, optionalAllowed, copyArtifactClosure) // See below for copyArtifactClosure syntax
        groovyCommand(commandStr) {} // See below for groovyClosure syntax
        groovyScriptFile(fileName) {} // See below for groovyClosure syntax
        systemGroovyCommand(commandStr) {} // See below for systemGroovyClosure syntax
        systemGroovyScriptFile(fileName) {} // See below for systemGroovyClosure syntax
    }
    publishers {
        extendedEmail(recipients, subjectTemplate, contentTemplate ) {}
        archiveArtifacts(glob, excludeGlob, latestOnlyBoolean)
        archiveJunit(glob, retainLongStdout, allowClaimingOfFailedTests, publishTestAttachments)
        publishHtml {
            report(reportDir, reportName, reportFiles, keepAll)
        }
        publishJabber(target, strategyName, channelNotificationName, jabberClosure) // See below for jabberClosure syntax
        publishScp(site, scpClosure) // See below for scpClosure syntax
        publishCloneWorkspace(workspaceGlob, workspaceExcludeGlob, criteria, archiveMethod, overrideDefaultExcludes, cloneWorkspaceClosure)
        downstream(projectName, thresholdName)
        downstreamParameterized(downstreamClosure) // See below for downstreamClosure syntax
        violations(perFileDisplayLimit, violationsClosure) // See below for violationsClosure syntax
        chucknorris() // Really important
        irc(ircClosure) // See below for ircClosure syntax
    }
    parameters {
        booleanParam(parameterName, defaultValue, description)
	listTagsParam(parameterName, scmUrl, tagFilterRegex, sortNewestFirst, sortZtoA, maxTagsToDisplay, defaultValue, description)
	choiceParam(parameterName, options, description)
	fileParam(fileLocation, description)
	runParam(parameterName, jobToRun, description)
	stringParam(parameterName, defaultValue, description)
	textParam(parameterName, defaultValue, description)
    }
}
```

The plugin tries to provide DSL methods to cover "common use case" scenarios as simple method calls. When these methods fail you, you can always generate the XML yourself via the [[configure block]]. Sometimes, a DSL method will provide a configure block of its own, which will set the a good context to help modify a few fields.  This gives native access to the Job config XML, which is typically very straight forward to understand.

(Note: The full XML can be found for any job by taking the Jenkins URL and appending "/config.xml" to it. We find that creating a job the way you like it, then viewing the XML is the best way to learn what fields you need.)

# Job
```groovy
job(Map<String, Object> attributes=[:], Closure closure)
```

The above method will return a _Job_ object that can be re-used and passed around. E.g.

```groovy
def myJob = job {
    name 'SimpleJob'
}
myJob.with {
    description 'A Simple Job'
}
```

A job can have optional attributes. Currently only a 'type' attribute with value of 'Freeform', 'Maven', 'Multijob' is supported. When no type attribute is specified, a free-style job will be generated. 

```groovy
job(type: Maven) {
  name 'maven-job'
}
```

Please see the [[Job Reference]] page for details to the _job_ component.

# Queue

```groovy
queue(String jobName)
queue(Job )
```

This provide the ability to 
#  Configure
_This is primarily defined in the [[configure block]] page. This is a short overview._

A lot of property directory on a project are best accessed directly with the configure block and via a DSL command.
Here are some simple examples:
```
configure {
    description 'My Description'
    jdk = 'JDK 6'
    disabled false
    labels 'MASTER' // Need way to disable too
}
```

## To Be Implemented

These are the ones in pipeline, and will be implemented sooner than later. If you're looking on working on one, claim it.

* Publish - xUnit
* Publish - Checkstyle, FindBugs, PMD, Cobertura, Emma, Analysis
* Publish - Javadoc

@daspilker:
* Config File Provider Plugin

@garethbowles:
* Publish - TestNG

## To Be Designed
* Publish - DeployPublisher
* Build - Python
* Report - MavenMailer
* Publish - BuildTrigger
* Publish - SVNTagPublisher
* Publish - RedeployPublisher
* Extend SCM (SVN and others?) to handle multiple Module Locations
* Publish - JoinTrigger