# Supported commands

## Job

```
name(String jobName)
```

Name of the job, required. This could be a static name but given the power of Groovy you could get very fancy with the name.

```
using(String templateName)
```

Refers to a template job as the basis for this job.


## Security

```
Enum Permissions { 
    ItemConfigure, ItemWorkspace, ItemDelete, ItemBuild, ItemRead, ItemRelease, ItemExtendedRead
    RunDelete, RunUpdate, etc
```
```
authorization {
    permission(String)
    permission(Permissions perm, String user)
}
```

## SCM

Only one scm is supported, so any existing SCM will be replaced. In the case of multiple scm calls in the DSL, the last one will win.
```groovy
scm {
    git(String url, Closure configure) // configure is optional
    perforce(Closure configure)
    subverison(Closure configure)
}

git {
    String url
    String branch
    // TODO Pull all fields from GitSCM
}

subversion {
    String url
    // Strategy
    // TODO Pull all fields from SubversionSCM
}

perforce {
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
    // TODO
}
```

## Build Step
```
step {
    shell(String command)
    groovy(String command)
    ant(String version, String targets, String buildFile, String properties, String javaOptions)
    ant(Closure configure)
    gradle(Closure configure)
    maven(Closure configure)
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

maven {
    String pomLocation
    // TODO
}
```
## Job Settings

Most are very easy to access with configure directly:
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
