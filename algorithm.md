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
 
The symplicity of `Algorithm` offers plenty of freedom to implement a metaheuristic according to your favorite preferences. However, as jMetal 5 is a framework, it includes a set of resources and strategies to help in the implementation of algorithms with the idea of promoting good design, code reusing, and flexibility. The key components are the use of the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) and algorithm templates. In the next section we give a detailed description of how the well-known NSGA-II algorithm is implemented, configured, and extended.

### Algorithm templates: Evolutionary and genetic algorithms
Most of metaheuristic families are characterized by a common behaviour which is shared by all the algorithms belonging to the family. This behaviour can be expressed as a template that can be instatiated to implement a particular algorithm. From a software engineering point of view, an algorithm whose behavior falls in a basic template would only require to implement some specific methods for the new technique; the common behavior would not be needed to be programmed, therefore resulting in less code replication. 

### Case study: NSGA-II


TO BE COMPLETED
