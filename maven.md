##jMetal as a Maven project

jMetal 5 is a Maven project which is partitioned into four packages:

* `jmetal-core`: Classes of the core architecture plus some utilities, including quality indicators.
* `jmetal-algorithm`: Implementations of metaheuristics.
* `jmetal-problem`: Implementations of problems.
* `jmetal-exec`: Executable programs to configure and
run the algorithms.

These packages can be found in the Maven Central Repository, where the Maven dependencies can be obtained. For
example, in the case of needing the jmetal-core package,
the current dependence is:

```
<dependency>
<groupId>org.uma.jmetal</groupId>
<artifactId>jmetal-core</artifactId>
<version>5.0</version>
</dependency>
```

