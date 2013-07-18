Not all of these have been asked a lot, but they're good things to know.

1. Q: **Is there a Java Binding for Job-DSL?** - No, not at the moment. The DSL relies heavily of both closures for contexts  and methodMissing/propertyMissing for XML generation. There's not currently a good Java equivalent of either of those. It's not to say that it couldn't be done, it'd just be ugly.