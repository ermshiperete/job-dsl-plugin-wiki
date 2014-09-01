This is the in-depth documentation of the methods available on inside the _view_ part of the DSL.

## List View

```groovy
view(type: ListView) {  // since 1.21
    // common options
    name(String name)
    description(String description)
    filterBuildQueue(boolean filterBuildQueue = true)
    filterExecutors(boolean filterExecutors = true)
    configure(Closure configureBlock)

    // list view options
    statusFilter(StatusFilter filter)
    jobs {
        name(String jobName)
        names(String... jobNames)
        regex(String regex)
    }
    columns {
        status()
        weather()
        name()
        lastSuccess()
        lastFailure()
        lastDuration()
        buildButton()
    }
}
```

Create a view which shows jobs in a list format. Details about the options can be found below. Similar to jobs, the view DSL can be extended using a [[configure block|The Configure Block]].

```groovy
view(type: ListView) {
    name('project-A')
    description('All jobs for project A')
    filterBuildQueue()
    filterExecutors()
    jobs {
        name('release-projectA')
        regex('project-A-.+')
    }
    columns {
        status()
        weather()
        name()
        lastSuccess()
        lastFailure()
        lastDuration()
        buildButton()
    }
}
```

## Build Pipeline View

```groovy
view(type: BuildPipelineView) {  // since 1.21
    // common options
    name(String name)
    description(String description)
    filterBuildQueue(boolean filterBuildQueue = true)
    filterExecutors(boolean filterExecutors = true)
    configure(Closure configureBlock)

    // build pipeline view options
    displayedBuilds(int displayedBuilds)
    title(String title)
    // consoleOutputLinkStyle and customCssUrl available in version 1.4.2 of the Build Pipeline Plugin
    consoleOutputLinkStyle(OutputStyle consoleOutputLinkStyle)
    customCssUrl(String cssUrl)
    selectedJob(String selectedJob)
    triggerOnlyLatestJob(boolean triggerOnlyLatestJob = true)
    alwaysAllowManualTrigger(boolean alwaysAllowManualTrigger = true)
    showPipelineParameters(boolean showPipelineParameters = true)
    showPipelineParametersInHeaders(boolean showPipelineParametersInHeaders = true)
    refreshFrequency(int seconds)
    showPipelineDefinitionHeader(boolean showPipelineDefinitionHeader = true)
}
```

Create a view of upstream and downstream connected jobs. Details about the options can be found below. Requires the [Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin).

```groovy
view(type: BuildPipelineView) {
    name('project-A')
    filterBuildQueue()
    filterExecutors()
    title('Project A CI Pipeline')
    displayedBuilds(5)
    selectedJob('project-A-compile')
    alwaysAllowManualTrigger()
    showPipelineParameters()
    refreshFrequency(60)
}
```

## Sectioned View

```groovy
view(type: SectionedView) {  // since 1.25
    // common options
    name(String name)
    description(String description)
    filterBuildQueue(boolean filterBuildQueue = true)
    filterExecutors(boolean filterExecutors = true)
    configure(Closure configureBlock)

    // sections view options
    sections {
        listView(Closure listViewSectionClosure)
    }
}
```

Create a view that can be divided into sections. Details about the options can be found below. Requires the
[Sectioned View Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Sectioned+View+Plugin).

```groovy
view(type: SectionedView) {
    name('project-summary')
    filterBuildQueue()
    filterExecutors()
    sections {
        listView {
            name('Project A')
            jobs {
                regex('project-A-.*')
            }
            columns {
                status()
                weather()
                name()
                lastSuccess()
                lastFailure()
            }
        }
        listView {
            name('Project B')
            jobs {
                regex('project-B-.*')
            }
            columns {
                status()
                weather()
                name()
                lastSuccess()
                lastFailure()
            }
        }
    }
}
```

## Nested View

```groovy
view(type: NestedView) {  // since 1.25
    // common options
    name(String name)
    description(String description)
    filterBuildQueue(boolean filterBuildQueue = true)
    filterExecutors(boolean filterExecutors = true)
    configure(Closure configureBlock)

    // sections view options
    views {
        view(Map<String, Object> arguments = [:], Closure viewClosure)
    }
    columns {
        status()
        weather()
    }
}
```

Create a view that allows grouping views into multiple levels. Details about the options can be found below. Requires
the [Nested View Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Nested+View+Plugin).

```groovy
view(type: NestedView) {
    name('project-a')
    views {
        view {
            name('overview')
            jobs {
                regex('project-A-.*')
            }
            columns {
                status()
                weather()
                name()
                lastSuccess()
                lastFailure()
            }
        }
        view(type: BuildPipelineView) {
            name('pipeline')
            selectedJob('project-a-compile')
        }
    }
}
```

## Common View Options

### Name

```groovy
name(String viewName)
```

The name of the view, **required**.

The view name is treated as absolute to the Jenkins root by default, but the seed job can be configured to interpret
names relative to the seed job. (since 1.24)

```groovy
name('project-A')
```

### Description

```groovy
description(String description)
```

Sets description of the view, optional.

```groovy
description('lorem ipsum')
```

### Filter Build Queue

```groovy
filterBuildQueue(boolean filterBuildQueue = true)
```

If set to `true`. only jobs in this view will be shown in the build queue. Defaults to `false` if omitted.

```groovy
filterBuildQueue()
```

### Filter Executors

```groovy
filterExecutors(boolean filterExecutors = true)
```

If set to `true`, only those build executors will be shown that could execute the jobs in this view.  Defaults to `false` if omitted.

```groovy
filterExecutors()
```

## List View Options

### Status Filter

```groovy
statusFilter(filter)
```

Filter the job list by enabled/disabled status. Valid values are `ALL` (default), `ENABLED` and `DISABLED`.

```groovy
statusFilter(ENABLED)
```

### Jobs

```groovy
jobs {
    name(String jobName)
    names(String... jobNames)
    regex(String regex)
}
```

Adds jobs to the view. `name` and `names` can be called multiple times to added more jobs, but only the last `regex` call will be used.

```groovy
jobs {
    name('build')
    name('test')
}
```

```groovy
jobs {
    names('build', 'test')
}
```

```groovy
jobs {
    regex('project-A-.+')
}
```

### Columns

```groovy
columns {
    status()
    weather()
    name()
    lastSuccess()
    lastFailure()
    lastDuration()
    buildButton()
    lastBuildConsole() // since 1.23, requires the Extra Columns Plugin
}
```

Adds columns to the views. The view will have no columns by default. Some column types require the
[Extra Columns Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Extra+Columns+Plugin).

## Build Pipeline View Options

### Displayed Builds

```groovy
displayedBuilds(int builds)
```

Sets number of displayed builds. Optional, defaults to 1, must be greater than zero.

```groovy
displayedBuilds(5)
```

### Title

```groovy
title(String title)
```

Sets a title for the pipeline. Optional.

```groovy
title('Project A CI Pipeline')
```

### Console Output Link Style

```groovy
consoleOutputLinkStyle(OutputStyle outputStyle)
```

Defines the console output style. Options are Lightbox, NewWindow and ThisWindow.

```groovy
consoleOutputLinkStyle(OutputStyle.Lightbox)
```

### CSS URL

```groovy
customCssUrl(String cssUrl)
```

Sets a URL for custom CSS files.

```groovy
customCssUrl('url/to/custom/css/files')
```

### Selected Job

```groovy
selectedJob(String jobName)
```

Defines the first job in the pipeline.

```groovy
selectedJob('project-a-compile')
```

### Trigger Only Latest Job

```groovy
triggerOnlyLatestJob(boolean triggerOnlyLatestJob = true)
```

Use this method to restrict the display of a trigger button to only the most recent successful build pipelines. This option will also limit retries to just unsuccessful builds of the most recent build pipelines. Optional, defaults to `false`.

```groovy
triggerOnlyLatestJob()
```

### Always Allow Manual Trigger

```groovy
alwaysAllowManualTrigger(boolean alwaysAllowManualTrigger = true)
```

Use this method if you want to be able to execute a successful pipeline step again. Optional, defaults to `false`.

```groovy
alwaysAllowManualTrigger()
```

### Show Pipeline Parameters

```groovy
showPipelineParameters(boolean showPipelineParameters = true)
```

Use this method if you want to display the parameters used to run the first job in each pipeline's revision box. Optional, defaults to `false`.

```groovy
showPipelineParameters()
```

### Show Pipeline Parameters In Headers

```groovy
showPipelineParametersInHeaders(boolean showPipelineParametersInHeaders = true)
```

Use this method if you want to display the parameters used to run the latest successful job in the pipeline's project headers. Optional, defaults to `false`.

```groovy
showPipelineParametersInHeaders()
```

### Refresh Frequency

```groovy
refreshFrequency(int refreshFrequency)
```

Frequency at which the Build Pipeline Plugin updates the build cards in seconds. Optional, defaults to `3`.

```groovy
refreshFrequency(60)
```

### Show Pipeline Definition Header

```groovy
showPipelineDefinitionHeader(boolean showPipelineDefinitionHeader = true)
```

Use this method if you want to show the pipeline definition header in the pipeline view. Optional, defaults to `false`.

```groovy
showPipelineDefinitionHeader()
```

## Sectioned View Options

### List View Section

```groovy
view(type: SectionedView) {
    sections {
        listView {
            name(String name)               // name of the section
            width(String width)             // either FULL, HALF, THIRD or TWO_THIRDS
            alignment(String alignment)     // either CENTER, LEFT or RIGHT
            jobs(Closure jobClosure)        // see the jobs closure for list views above
            columns(Closure columnsClosure) // see the columns closure for list views above
        }
    }
}
```

Creates a section containing a list of jobs. Width defaults to `FULL` and alignment defaults to `CENTER` if not
specified.

```groovy
view(type: SectionedView) {
    sections {
        listView {
            name('project-A')
            width('HALF')
            alignment('LEFT')
            jobs {
                regex('project-A-.*')
            }
            columns {
                status()
                weather()
                name()
                lastSuccess()
                lastFailure()
            }
        }
    }
}
```

## Nested View Options

### Views

```groovy
view(type: NestedView) {
    views {
        view(Map<String, Object> arguments = [:], Closure viewClosure)
    }
}
```

Creates the nested views. The view methods works like the top-level view method.

```groovy
view(type: NestedView) {
    views {
        view {
            name('overview')
            jobs {
                regex('project-A-.*')
            }
            columns {
                status()
                name()
            }
        }
        view(type: BuildPipelineView) {
            name('pipeline')
            selectedJob('project-a-compile')
        }
    }
}
```

### Columns

```groovy
view(type: NestedView) {
    columns {
        status()
        weather()
    }
}
```

Adds columns to the view. Only the status and weather column are supported.
