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
 


TO BE COMPLETED
