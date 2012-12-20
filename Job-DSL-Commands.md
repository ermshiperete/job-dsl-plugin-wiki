# Job Factory
The DSL execution engine exposes only one method, called 'job'. This 'job' method implies the creation of a Jenkins job and the closure to this method then further exposed some methods where things get interesting. See the later sections to learn the specific available methods available, for when a DSL helper method isn't available, look to the [[configure block]]. Below is the simplest job possible:

```
job {
    name 'Simplest Job Possible'
}
```

Because the engine is just Groovy, you can call other Groovy classes available in the workspace. When in those methods the 'job' method is no longer available, so it is recommended to pass in the current context to make this method available to other context. For example, when making utility methods, you would call them like this:

```
BuildFramework.ant(this, arg1, arg2)
```

Then the BuildFramework class has enough to make 'job' calls, it would look like this:

```
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

TBD, Current this method is hard-coded to free-form projects. It wouldn't be too hard to make it generate maven and ivy projects. File a bug if you'd find this useful.

# DSL Methods

This is the formal documentation of the available dsl methods. In the Closure provided to job there are a few top level methods, like label and chucknorris. Others are nested deeper in blocks which represent their role in Jenkins, e.g. the publishers block contains all the publisher steps. The one caveat is that scm and multiscm are mutually exclusive. Likewise, when using the scm block, only one scm can be specified.

Further sections will define in detail how they work, in a Java-like syntax. If an argument is followed with an equals, this means it's a default value. DSL Methods can be cumulative or overriding, meaning that some methods will add nodes (e.g. publishers and steps) and some will replace nodes (e.g. disabled() will replace any existing disabled tags).

Here's a high level overview of what's available:

```groovy
job {
    name(nameStr)
    using(templateNameStr)
    description(descStr)
    label(labelStr)
    timeout(timeoutInMinutes, shouldFailBuild)
    chucknorris() // Really important
    disabled(shouldDisable)
    authorization {
        permission(permissionStr) // e.g. hudson.model.Item.Workspace:authenticated
        permission(String permEnumName, String user)
        permission(Permission perm, String user)
        permissionAll(securityGroup)
    }
    scm {
        hg(url, branch) {}
        git(url, branch) {}
        svn(svnUrl) {}
        p4(viewspec, user, password) {}
    }
    multiscm {
        hg(url, branch) {}
        git(url, branch) {}
        svn(svnUrl) {}
        p4(viewspec, user, password) {}
    }
    steps {
        shell(String commandStr)
        gradle(tasksArg, switchesArg, useWrapperArg) {}
        maven(targetsArg, pomArg) {}
    }
    publishers {
        extendedEmail(recipients, subjectTemplate, contentTemplate ) {}
    }
}
```

The plugin tries to provide dsl methods to cover "common use case" scenarios as simple method calls. When these methods fail you, you can always generate the XML yourself via the [[configure block]]. Sometimes, a DSL method will provide a configure block of its own, which will set the a good context to help modify a few fields.  This gives native access to the Job config XML, which is typically very straight forward to understand.

(Note: The full XML can be found for any job by taking the Jenkins URL and appending "/config.xml" to it. We find that creating a job the way you like it, then viewing the XML is the best way to learn what fields you need.)

## Name
```groovy
name(String jobName)
```

The Name of the job, **required**. This could be a static name but given the power of Groovy you should get very fancy with the these.

## Using
```groovy
using(String templateName)
```

Refers to a template Job to be used as the basis for this job. These are loaded before any configure blocks or DSL commands.  Template Jobs are just standard Jenkins Jobs which are used for their underlying config.xml. When they are changed, the seed job will attempt to re-run, which has the side-effect of cascading changes of the template the jobs generated from it.

## Description
```groovy
description(String desc)
```

Sets description of the job. This is a not a good way of creating a dynamic description of a job.

## Label
```groovy
label(String labelStr)
```

Label which specifies which nodes this job can run on, e.g. 'X86&&Ubuntu'

## Timeout
```groovy
timeout(int timeoutInMinutes, Boolean shoudFailBuild = true)
```

Using the build timeout plugin, it can fail a build after a certain amount of time.

## Chuck Norris

```groovy
chucknorris()
```

Enables the Cordell Walker plugin.

## Disable

```groovy
disabled(Boolean shouldDisable)
```

Provides ability to disable a job.

## Security
```groovy
permission(String)
permission(String permEnumName, String user)
permission(Permissions perm, String user)
permissionAll(String user)
```

Creates permission records. The first form adds a specific permission, e.g. 'hudson.model.Item.Workspace:authenticated', as seen in the config.xml. The second form simply breaks apart the permission from the user name, to make scripting easier. The third uses a helper Enum called Permissions to hide some of the names of permissions. It is available by importing javaposse.jobdsl.dsl.helpers.Permissions. By using the enum you get some basic type checking. A flaw with this system is that Jenkins plugins can create their own permissions, and the job-dsl plugin doesn't necessarily know abou them. The last form will take everything in the Permissions enum and gives them to the user, this method also suffers from the problem that not all permissions from every plugin are included.

Permissions looks like this:

```java
Enum Permissions {
    ItemConfigure, ItemWorkspace, ItemDelete, ItemBuild, ItemRead, ItemRelease, ItemExtendedRead
    RunDelete, RunUpdate, etc.
```

# Source Control

## Mercurial

```groovy
hg(String url, String branch = null, Closure configure = null)
```

Add Mercurial SCM source. Will not clean by default, to change this use the configure block, e.g.

```groovy
hg('http://scm') { node ->
    node / clean('true')
}
```

## Git
```groovy
git(String url, String branch = null, Closure configure = null)
```

Add Git SCM source. The Git plugin has a lot of configurable options, which are all available in the configure block,

```groovy
git('git@git') { node -> // Is hudson.plugins.git.GitSCM
    node / gitConfigName('DSL User')
    node / gitConfigEmail('me@me.com')
}
```

## Subversion
```groovy
svn(String svnUrl, Closure configure = null)
```

Add Subversion source. Configure block is handed a hudson.scm.SubversionSCM.

## Perforce
```groovy
p4(String viewspec, String user = 'rolem', String password = '', Closure configure = null)
```

Add Perforce SCM source. The user probably has to be specified. The password will be properly encrypted. Sets p4Client to builds-${JOB_NAME}. The configure block is handed a hudson.plugins.perforce.PerforceSCM. Perforce requires a few fields to be setup, so it's very likely that the configure block will be needed, especially for things like p4Port.

```groovy
p4('//depot/Tools/build') { node ->
    node / p4Port('perforce:1666')
    node / p4Exec('/usr/bin/p4')
    node / exposeP4Passwd('false')
    node / pollOnlyOnMaster('true')
}
```

# Triggers


Triggers block contains the available triggers.

## Cron
```groovy
cron(String cronString)
```

Triggers job based on regular intervals.

## Source Control Trigger
```groovy
scm(String cronString)
```

Polls source control for changes at regular intervals.

## Gerrit
```groovy
gerrit {
    events(Closure eventClosure) // Free form listing of event names
    project(String projectName, List<String> branches) // Can be called multiple times
    project(String projectName, String branches) // Can be called multiple times
    configure(Closure configureClosure) // Handed com.sonyericsson.hudson.plugins.gerrit.trigger.hudsontrigger.GerritTrigger
}
```

Polls gerrit for changes. This DSL method works slightly differently by exposing most of its functionality in its own block. This is accomadating how the plugin can be pointed to multiple projects and trigger on many events. The most complex part is the events block, which takes the "short name" of an event. When looking at the raw config.xml for a Job which has Gerrit trigger, you'll see multiple class names in the triggerOnEvents element. The DSL method will take the names in the events block and prepend it with "com.sonyericsson.hudson.plugins.gerrit.trigger.hudsontrigger.events.Plugin" and append "Event", meaning that shorter names like ChangeMerged and DraftPublished can be used. Straight from the unit test:

```groovy
gerrit {
    events {
        ChangeMerged
        DraftPublished
    }
    project('reg_exp:myProject', ['ant:feature-branch', 'plain:origin/refs/mybranch'])
    project('test-project', '**')
    configure { node ->
        node / gerritBuildSuccessfulVerifiedValue << '10'
    }
}
```

# Build Steps

Adds step block to contain an ordered list of build steps.

## Shell command
```groovy
shell(String commandStr)
```

Runs a shell command.

## Gradle
```groovy
gradle(String tasksArg = null, String switchesArg = null, Boolean useWrapperArg = true, Closure configure = null)
```

Runs Gradle, defaulting to the Gradle Wrapper. Configure block is handed a hudson.plugins.gradle.Gradle node.


## Maven
```groovy
maven(String targetsArg = null, String pomArg = null, Closure configure = null)
```

Runs Apache Maven. Configure block is handed hudson.tasks.Maven.

# Publishers

Block to contain list of publishers.

## Extended Email Plugin
```groovy
extendedEmail(String recipients = null, String subjectTemplate = null, String contentTemplate = null, Closure emailClosure = null)
```

Supports the Extended Email plugin. DSL methods works like the gerrit plugin, providing its own block to help set it up. The emailClosure is primarily used to specify the triggers, which is optional. Its definition:

```groovy
extendedEmail {
    trigger(String triggerName, String subject = null, String body = null, String recipientList = null,
            Boolean sendToDevelopers = null, Boolean sendToRequester = null, includeCulprits = null, Boolean sendToRecipientList = null)
    trigger(Map args)
    configure(Closure configureClosure) // Handed hudson.plugins.emailext.ExtendedEmailPublisher
}
```

The first trigger method allow complete control of the email going out, and maps directly to what is seen in the config.xml of a job. The triggerName needs to be one of these values: 'PreBuild', 'StillUnstable', 'Fixed', 'Success', 'StillFailing', 'Improvement', 'Failure', 'Regression', 'Aborted', 'NotBuilt', 'FirstFailure', 'Unstable'. Those names come from classes prefix with 'hudson.plugins.emailext.plugins.trigger.' and appended with Trigger. The second form of trigger, uses the names from the first, but can be called with a Map syntax, so that values can be left out more easily. To help explain it, here an example from the unite tests:

```groovy
extendedEmail('me@halfempty.org', 'Oops', 'Something broken') {
    trigger('PreBuild')
    trigger(triggerName: 'StillUnstable', subject: 'Subject', body:'Body', recipientList:'RecipientList',
            sendToDevelopers: true, sendToRequester: true, includeCulprits: true, sendToRecipientList: false)
    configure { node ->
        node / contentType << 'html'
    }
}
```

## Archive Artifacts
```groovy
archiveArtifacts(String glob, String excludeGlob = null, Boolean latestOnlyBoolean = false)
```

Supports archiving artifacts with each build. Simple example:

```groovy
publishers {
    archiveArtifacts 'build/test-output/**/*.html'
}
```

## HTML Publisher
```groovy
publishHtml {
    report(String reportDir, String reportName = null, String reportFiles = 'index.html', Boolean keepAll = false)
    report(Map args) // same names as the method above
}

```

Provides context to add html reports to be archive. The report method can be called multiple times in the closure. Simple
example with variations on how to call the report method:

```groovy
publishers {
    publishHtml {
        report('build/test-output/*', 'Test Output')
        report 'build/coverage/*', 'Coverage Report', 'coverage.html' // Without parens
        report reportName: 'Gradle Tests', reportDir: 'test/*', keepAll: true // Map synxtax
    }
}
```

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

* Publish - Downstream job
* Steps - groovy
* Steps - System Groovy
* Publish - xUnit
* Publish - TestNG

@quidryan:
* Steps - Ant
* Publish - Checkstyle, FindBugs, PMD, Cobertura, Emma, Analysis
* Publish - Junit
* Publish - Javadoc
* Jabber
* Publish to scp

## To Be Designed
* Build - CopyArtifact
* Parameterized Builds
* Maven projects
* Publish - DeployPublisher
* Build - Python
* Report - MavenMailer
* Publish - BuildTrigger
* Publish - SVNTagPublisher
* Publish - RedeployPublisher
* Extend SCM (SVN and others?) to handle browser URL's nicely and multiple Module Locations
* Configure - LogRotator
* Publish - JoinTrigger