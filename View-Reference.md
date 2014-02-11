This is the in-depth documentation of the methods available on inside the _view_ part of the DSL.

## ListView

```groovy
view(type: ListView) {  // since 1.21
    name(String name)
    description(String description)
    filterBuildQueue(boolean filterBuildQueue)
    filterExecutors(boolean filterExecutors)
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
    configure(Closure configureBlock)
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

## Name

```groovy
name(String viewName)
```

The name of the view, **required**.

```groovy
name('project-A')
```

## Description

```groovy
description(String description)
```

Sets description of the view, optional.

```groovy
description('lorem ipsum')
```

## Filter Build Queue

```groovy
filterBuildQueue(boolean filterBuildQueue = true)
```

If set to `true`. only jobs in this view will be shown in the build queue. Defaults to `false` if omitted.

```groovy
filterBuildQueue()
```

## Filter Executors

```groovy
filterExecutors(boolean filterExecutors = true)
```

If set to `true`, only those build executors will be shown that could execute the jobs in this view.  Defaults to `false` if omitted.

```groovy
filterExecutors()
```

## Status Filter

```groovy
statusFilter(filter)
```

Filter the job list by enabled/disabled status. Valid values are `ALL` (default), `ENABLED` and `DISABLED`.

```groovy
statusFilter(ENABLED)
```

## Jobs

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

# Columns

```groovy
columns {
    status()
    weather()
    name()
    lastSuccess()
    lastFailure()
    lastDuration()
    buildButton()
}
```

Adds columns to the views. The view will have no columns by default.
