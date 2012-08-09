# Supported commands
All supported commands exist in a 'job' block.  We currently only support free-form projects.

Many commands listed here provide some "common use case" scenarios as simple method calls, but sometimes you'll need to access additional properties not provided in the DSL. 

In these cases the the DSL additionally supports the passing of a Closure as the last parameter. This gives native access to the Job config XML, which is typically very straight forward to understand. 

(Note: The full XML can be found for any job by taking the Jenkins URL and appending "/config.xml" to it. We find that creating a job the way you like it, then viewing the XML is the best way to learn what fields you need.)

## Name
```
name(String jobName)
```

The Name of the job, **required**. This could be a static name but given the power of Groovy you could get very fancy with the these.

## Using
```
using(String templateName)
```

Refers to a template Job as the basis for this Scripted job. These are loaded before any configure blocks or DSL commands.  Template Jobs are just standard Jenkins Jobs which are used for their underlying config.xml

## Security
```
authorization {
    permission(String)
    permission(Permissions perm, String user)
    permission(String permEnumName, String user)
}
```

A helper Enum called Permissions is available: javaposse.jobdsl.dsl.helpers.Permissions. Import one of these to get some basic type checking, and to learn your options. It looks like this:

```
Enum Permissions { 
    ItemConfigure, ItemWorkspace, ItemDelete, ItemBuild, ItemRead, ItemRelease, ItemExtendedRead
    RunDelete, RunUpdate, etc.
```

## SCM
Only one SCM at a time is supported by Jenkins Jobs, so any existing SCM will be replaced. In the case of multiple scm calls in a DSL script, the last one evaluated will win. 

(Note: We currently only support git, but will be supporting Perforce and Subversion in the future. Please file a defect if you would like them sooner rather than later.)

```groovy
scm {
    git(String url, String branch = null, Closure configure = null)
    perforce(String viewspec, Closure configure)
    subverison(String url, Closure configure)
}

// Examples of what can be accessed from the closure block
git { // Is hudson.plugins.git.GitSCM
    gitConfigName
    gitConfigEmail
}

subversion { // Is hudson.scm.SubversionSCM
    excludedRegions ''
    includedRegions ''
    excludedUsers ''
    excludedRevprop ''
    excludedCommitMessages ''
}

perforce { // Is hudson.plugins.perforce.PerforceSCM
    // Uses PerforceSCM almost directly
    p4User // Default: rolem
    p4Passwd
    p4Port // Default: performce:1666
    p4Client
    projectPath // Client Spec
    projectOptions // Default: noallwrite clobber nocompress unlocked nomodtime rmdir
    p4Exe // Default: p4
    p4SysDrive
    p4SysRoot
    useClientSpec // Default: false
    forceSync // Default: false
    alwaysForceSync // Default: false
    dontUpdateServer // Default: false
    disableAutoSync // Default: false
    disableSyncOnly // Default: false
    useOldClientName // Default: false
    updateView // Default: true
    dontRenameClient // Default: false
    updateCounterValue // Default: false
    dontUpdateClient // Default: false
    exposeP4Passwd // Default: false
    wipeBeforeBuild // Default: true
    wipeRepoBeforeBuild // Default: false
    firstChange // Default: -1
    slaveClientNameFormat // Default: ${basename}-${nodename}
    lineEndValue
    useViewMask // Default: false
    useViewMaskForPolling // Default: false
    useViewMaskForSyncing // Default: false
    pollOnlyOnMaster // Default: true
}
```

## Triggers
```
triggers {
    scm(String cronString)
    cron(String cronString)
    // TODO, what are the others?
}
```

## Build Steps
```
step {
    shell(String command)
    maven(String targetsArg = null, String pomArg = null, Closure configure = null) 
    gradle(String tasksArg = null, String switchesArg = null, Boolean useWrapperArg = true, Closure configure = null)
    // TODO
    groovy(String command)
    grails(String version)
    ant(String version, String targets, String buildFile, String properties, String javaOptions, Closure configure)
}

ant {
    String version
    String targets
    String buildFile
    String props // Don't want to name properties
    String javaOptions
}

gradle {
    boolean useWrapper
    // TODO
}

maven { // Is hudson.tasks.Maven
    targets
    mavenName
    pom
    usePrivateRepository
}

grails { // Is com.g2one.hudson.grails.GrailsBuilder
    name
    grailsWorkDir
    projectWorkDir
    projectBaseDir
    serverPort
    properties
    forceUpgrade
    nonInteractive
}
```
##  Configure
_This is primarily defined in the [configure block] page. This is a short overview._

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

```
archive(String include, String exclude)
enum DownstreamTrigger { Succeeds, Unstable, Fails }
downstream(String projects, DownstreamTrigger trigger)
```

## To Be Designed

* Checkstyle, FindBugs, PMD, Cobertura
* Publish
** Junit
** Javadoc
** xUnit
** TestNG
* Email
* Parameterized Builds
