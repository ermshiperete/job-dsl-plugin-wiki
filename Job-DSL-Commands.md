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
        violations(perFileDisplayLimit, violationsClosure) // Seed below for violationsClosure
        chucknorris() // Really important
    }
}
```

The plugin tries to provide dsl methods to cover "common use case" scenarios as simple method calls. When these methods fail you, you can always generate the XML yourself via the [[configure block]]. Sometimes, a DSL method will provide a configure block of its own, which will set the a good context to help modify a few fields.  This gives native access to the Job config XML, which is typically very straight forward to understand.

(Note: The full XML can be found for any job by taking the Jenkins URL and appending "/config.xml" to it. We find that creating a job the way you like it, then viewing the XML is the best way to learn what fields you need.)

## Job
```groovy
job(Map<String, Object> attributes=[:], Closure closure)
```

A job can have optional attributes. Currently only a 'type' attribute with value 'maven' is supported, which will change the job to a maven2/3 project. When no type attribute is specified, a free-style job will be generated. 

```groovy
job(type: maven) {
  name 'maven-job'
}
```

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

## Disable

```groovy
disabled(Boolean shouldDisable)
```

Provides ability to disable a job.

## Block build
```groovy
blockOn(String projectNames)
```

Block build if certain jobs are running, supported by the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Build+Blocker+Plugin">Build Blocker Plugin</a>. projectNames is comma separated.

## Build History
```groovy
logRotator(int daysToKeepInt = -1, int numToKeepInt = -1, int artifactDaysToKeepInt = -1, int artifactNumToKeepInt = -1)
```

Sets up the number of builds to keep.

## JDK
```groovy
jdk(String jdkStr)
```

Selects the JDK to be used for this project. The jdkStr must match the name of a JDK installation defined in the Jenkins system configuration. The default JDK will be used when the jdk method is omitted.

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

# Maven

The 'rootPOM', 'goals', 'mavenOpts', 'perModuleEmail', 'archivingDisabled' and 'runHeadless' methods can only be used in jobs with type 'maven'.

## Root POM
```groovy
rootPOM(String rootPOM)
```

To use a different 'pom.xml' in some other directory than the workspace root.

## Goals
```groovy
goals(String goals) 
```

The Maven goals to execute including other command line options. 

When specified multiple times, the goals and options will be concatenated, e.g.

```groovy
goals("clean") 
goals("install") 
goals("-DskipTests") 
```

is equivalent to

```groovy
goals("clean install -DskipTests") 
```

## MAVEN_OPTS
```groovy
mavenOpts(String mavenOpts) 
```

The JVM options to be used when starting Maven. When specified multiple times, the options will be concatenated.

## Email Per Module
```groovy
perModuleEmail(boolean shouldSendEmailPerModule)
```

Enable or disable email notifications for each Maven module.

## Disable Artifact Archiving
```groovy
archivingDisabled(boolean shouldDisableArchiving)
```

Disables automatic Maven artifact archiving. Artifact archiving is enabled by default.

## Run Headless
```groovy
runHeadless(boolean shouldRunHeadless)
```

Specifiy this to run the build in headless mode if desktop access is not required. Headless mode is not enabled by default.

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
svn(String svnUrl, String localDir='.', Closure configure = null)
```

Add Subversion source. 'svnUrl' is self explanatory. 'localDir' sets the <local> tag (which is set to '.' if you leave this arg out). The Configure block is handed a hudson.scm.SubversionSCM node.

You can use the Configure block to configure additional svn nodes such as a <browser> node. First a FishEyeSVN example:

```groovy
svn('http://svn.apache.org/repos/asf/xml/crimson/trunk/') { svnNode ->
    svnNode / browser(class:'hudson.scm.browsers.FishEyeSVN') {
        url 'http://mycompany.com/fisheye/repo_name'
        rootModule 'my_root_module'
    }
}
```

and now a ViewSVN example:
```groovy
svn('http://svn.apache.org/repos/asf/xml/crimson/trunk/') { svnNode ->
    svnNode / browser(class:'hudson.scm.browsers.ViewSVN') / url << 'http://mycompany.com/viewsvn/repo_name'
}
```

For more details on using the configure block syntax, see our [dedicated page](configure-block).

## Perforce
```groovy
p4(String viewspec, String user = 'rolem', String password = '', Closure configure = null)
```

Add Perforce SCM source. The user probably has to be specified. The password will be properly encrypted. Sets p4Client to builds-${JOB_NAME}. The configure block is handed a hudson.plugins.perforce.PerforceSCM. Perforce requires a few fields to be setup, so it's very likely that the configure block will be needed, especially for things like p4Port.

```groovy
p4('//depot/Tools/build') { node ->
    node / p4Port('perforce:1666')
    node / p4Tool('/usr/bin/p4')
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

## Snapshot Dependencies
```groovy
snapshotDependencies(boolean checkSnapshotDependencies)
```

When enabling the snapshot dependencies trigger, Jenkins will check the snapshot dependencies from the  '\<dependency\>', '\<plugin\>' and '\<extension\>' elements used in Maven POMs and setup a job relationship to the jobs building the snapshots. This can only be used in jobs with type 'maven'.

# Build Steps

Adds step block to contain an ordered list of build steps. Cannot be used for jobs with type 'maven'.

## Shell command
```groovy
shell(String commandStr)
```

Runs a shell command.

## Batch File
```groovy
batchFile(String commandStr)
```

Supports running a Windows batch file as a build step.

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

## Ant
```groovy
ant(String targetsArg = null, String buildFileArg = null, String antInstallation = '(Default)', Closure antClosure = null) {
    target(targetName)
    targets(Iterable<String> targets)
    prop(Object key, Object value)
    props(Map<String, String> map)
    buildFile(String buildFile)
    javaOpt(String opt)
    javaOpts(Iterable<String> opts)
    antInstallation(String antInstallationName)
}
```

Runs Apache Ant. Ant Closure block can be used for all configuration, and it's the only way to add Java Options and System
Properties.
* Target argument - Available as an argument or closure method, each call is cumulative. The target argument can be space or newline delimited.
* Properties - Available via closure method calls. All calls are cumulative. A Map is used to back it, which will enforce unique keys for the properties.
* Ant Installation - Available as an argument or closure method. Refers to the pull down box in the UI to select which installation of Ant to use, specify the exact string seen in the UI. The last call will be the one used.
* Build File - Available as an argument or closure method. Specifies which build.xml file to use, this should be relative to the workspace.
* Java Options - Available via closure method calls. Arguments to be passed directly to the JVM.

From the unit tests, it'll use the targets "build test publish deploy":

```groovy
steps {
    ant('build') {
        target 'test'
        targets(['publish', 'deploy'])
        prop 'logging', 'info'
        props 'test.threads': 10, 'input.status':'release'
        buildFile 'dir1/build.xml'
        javaOpt '-Xmx1g'
        javaOpts(['-Dprop2=value2', '-Dprop3=value3'])
        antInstallation 'Ant 1.8'
    }
}
```

## Copy Artifacts

```groovy
copyArtifacts(String jobName, String includeGlob, String targetPath = '', boolean flattenFiles = false, boolean optionalAllowed = false, Closure copyArtifactClosure = null) {
    upstreamBuild(boolean fallback = false) // Upstream build that triggered this job
    latestSuccessful() // Latest successful build
    latestSaved() // Latest saved build (marked "keep forever")
    permalink(String linkName) // Specified by permalink: lastBuild, lastStableBuild
    buildNumber(int buildNumber) // Specific Build
    workspace() // Copy from WORKSPACE of latest completed build
    buildParameter(String parameterName) // Specified by build parameter
}
```

Supports the Copy Artifact plugin. As per the plugin, the input glob is for files in the workspace. The methods in the closure are considered the selectors, of which only one can be used.

## System Groovy Scripts
```groovy
systemGroovyCommand(String commandStr, Closure systemGroovyClosure = null) {
    binding(String name, String value)
    classpath(String classpathEntry)
}
systemGroovyScriptFile(String fileName, Closure systemGroovyClosure = null) {
    binding(String name, String value)
    classpath(String classpathEntry)
}
```

Runs a system groovy script, which is executed inside the Jenkins master. Thus it will have access to all the internal objects of Jenkins and can be used to alter the state of Jenkins. The `systemGroovyCommand` method will run an inline script and the `systemGroovyScriptFile` will execute a script file from the generated job's workspace. The closure block can be used to add variable bindings and extra classpath entries for a script. The methods in the closure block can be called multiple times to add any number of bindings or classpath entries. The Groovy plugin must be installed to use these build steps.

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

## Archive JUnit
```groovy
archiveJunit(String glob, boolean retainLongStdout = false, boolean allowClaimingOfFailedTests = false, boolean publishTestAttachments = false)
```

Supports archiving JUNit results for each build.

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

## Jabber Publisher
```groovy
publishJabber(String target, String strategyName, String channelNotificationName, Closure jabberClosure) {
    strategyName 'ALL' // ALL, FAILURE_AND_FIXED, ANY_FAILURE, STATECHANGE_ONLY
    notifyOnBuildStart false
    notifySuspects false
    notifyCulprits false
    notifyFixers false
    notifyUpstreamCommitters false
    channelNotificationName 'Default' // Default, SummaryOnly, BuildParameters, PrintFailingTests
}
```

Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/Jabber+Plugin">Jabber Plugin</a>. A few arguments can be specified in the method call or in the closure.

## SCP Publisher
```groovy
publishScp(String site, Closure scpClosure) {
    entry(String source, String destination = '', boolean keepHierarchy = false)
}
```

Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/SCP+plugin">SCP Plugin</a>. First arg, site, is specified globally by the plugin. Each entry is
individually specified in the closure block, e.g. entry can be called multiple times.

## CloneWorkspace Publisher
```groovy
cloneWorkspace(String workspaceGlob, String workspaceExcludeGlob = '', String criteria = 'Any', String archiveMethod = 'TAR', boolean overrideDefaultExcludes = false, Closure cloneWorkspaceClosure = null) {}
```

Supports the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Clone+Workspace+SCM+Plugin">Clone Workspace SCM Plugin</a>.

Due to the simplicity of this publisher, the closure support is purely provided for creating very specific configs.  Usually the non-closure variants will suffice - the simplest purely requiring the workspaceGlob alone, and the other (equivalent to pressing the "Advanced" button in the Jenkins UI) provided all settings.

## Downstream
```groovy
downstream(String projectName, String thresholdName = 'SUCCESS')
```

Specifies a downstream job. The second arg, thresholdName, can one of four values: 'SUCCESS', 'UNSTABLE', 'FAILURE', 'ALWAYS'.

## Extended Downstream
```groovy
downstreamParameterized(Closure downstreamClosure) {
     trigger(String projects, String condition = 'SUCCESS', boolean triggerWithNoParameters = false, Closure downstreamTriggerClosure = null) {
        currentBuild() // Current build parameters
        propertiesFile(String propFile) // Parameters from properties file
        gitRevision(boolean combineQueuedCommits = false) // Pass-through Git commit that was built
        predefinedProp(String key, String value) // Predefined properties
        predefinedProps(Map<String, String> predefinedPropsMap)
        predefinedProps(String predefinedProps) // Newline separated
        matrixSubset(String groovyFilter) // Restrict matrix execution to a subset
        subversionRevision() // Subversion Revision
     }
}
```

Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/Downstream-Ext+Plugin">Downstream-Ext plugin</a>. The plugin is configured by adding triggers
to other projects, multiple triggers can be specified. The projects arg is a comma separated list of downstream projects. The condition arg is one of these
possible values: SUCCESS, UNSTABLE, UNSTABLE_OR_BETTER, UNSTABLE_OR_WORSE, FAILED.  The methods inside the downstreamTriggerClosure are optional, though it
makes the most sense to call at least one.  Each one is relatively self documenting, mapping directly to what is seen in the UI. The predefinedProp and
predefinedProps methods are used to accumulate properties, meaning that they can be called multiple times to build a superset of properties.


## Violations Plugin
```groovy
violations(int perFileDisplayLimit = 100, Closure violationsClosure = null) {
    sourcePathPattern(String sourcePathPattern)
    fauxProjectPath(String fauxProjectPath)
    perFileDisplayLimit(Integer perFileDisplayLimit)
    sourceEncoding(String encoding = 'default')
    *(Integer min = 10, Integer max = 999, Integer unstable = 999, String pattern = null)
}
```
Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/Violations">Violations Plugin</a>. The violationsClosure is crucial to configure this DSL method.
For each supported type, you specify it and a few option parameters, this is represented above with the asterisk (*). If a type isn't specified, then it uses
defaults. The following example wil

```groovy
violations(50) {
   sourcePathPattern 'source pattern'
   fauxProjectPath 'faux path'
   perFileDisplayLimit 51
   checkstyle(10, 11, 10, 'test-report/*.xml')
   findbugs(12, 13, 12)
}
```

## Chuck Norris

```groovy
chucknorris()
```

Enables the Cordell Walker plugin.

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
* Publish - TestNG
* Publish - Checkstyle, FindBugs, PMD, Cobertura, Emma, Analysis
* Publish - Junit
* Publish - Javadoc

@quidryan:
* Environment variables

@daspilker:
* Steps - groovy

## To Be Designed
* Parameterized Builds
* Publish - DeployPublisher
* Build - Python
* Report - MavenMailer
* Publish - BuildTrigger
* Publish - SVNTagPublisher
* Publish - RedeployPublisher
* Extend SCM (SVN and others?) to handle multiple Module Locations
* Publish - JoinTrigger