This is the in-depth documentation of the methods available on inside the _view_ part of the DSL.

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
filterBuildQueue(boolean filterBuildQueue)
```

If set to `true`. only jobs in this view will be shown in the build queue. Defaults to `false` if omitted.

```groovy
filterBuildQueue(true)
```

## Filter Executors

```groovy
filterExecutors(boolean filterExecutors)
```

If set to `true`, only those build executors will be shown that could execute the jobs in this view.  Defaults to `false` if omitted.

```groovy
filterExecutors(true)
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
