## The `Algorithm` interface

A metaheuristic or algorithm in jMetal 5 is an entity that implements the `Algorithm` interface:
```java
package org.uma.jmetal.algorithm;

/**
 * Interface representing an algorithm
 * @author Antonio J. Nebro
 * @version 0.1
 * @param <Result> Result
 */
public interface Algorithm<Result> extends Runnable {
  void run() ;
  Result getResult() ;
}

```
This interface is very generic: it specifies that an algorithm must have a `run()` method and return a result by the `getResult()` method. As it extends `Runnable`, any algorithm can be executed in a thread.
 
The symplicity of `Algorithm` offers plenty of freedom to implement a metaheuristic according to your favorite preferences. However, as jMetal 5 is a framework, it includes a set of resources and strategies to help in the implementation of algorithms with the idea of promoting good design, code reusing, and flexibility. The key components are the use of the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) and [algorithm templates](https://github.com/jMetal/jMetalDocumentation/blob/master/algorithmTemplates.md). In the next section we give a detailed description of how the well-known NSGA-II algorithm is implemented, configured, and extended.

### Case study: NSGA-II


TO BE COMPLETED
