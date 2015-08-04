## The `Problem` interface
To include a problem in jMetal, it must implement the `Problem` interface:

```java
package org.uma.jmetal.problem;

/**
 * Interface representing a multi-objective optimization problem
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 *
 * @param <S> Encoding
 */
public interface Problem<S extends Solution<?>> extends Serializable {
  /* Getters */
  public int getNumberOfVariables() ;
  public int getNumberOfObjectives() ;
  public int getNumberOfConstraints() ;
  public String getName() ;

  /* Methods */
  public void evaluate(S solution) ;
  public S createSolution() ;
```

Every problem is characterized by the number of decision variables, the number of objective functions and the number of constraints, so getter methods for returning those values have to be defined. The genetic type `S` allows to determine the encoding of the solutions of the problem. This way, a problem must include a method for evaluating any solution of class `S` as well as providing a `createSolution()` method for creating a new solution. 

The `Solution` interface is generic, so jMetal 5 has a number of interfaces extending it to represent double (i.e., continous) problems, binary problems, etc. This way, the [`DoubleProblem`](https://github.com/jMetal/jMetal/blob/jmetal-5.0/jmetal-core/src/main/java/org/uma/jmetal/problem/DoubleProblem.java) interface is defined in this way:

```java
package org.uma.jmetal.problem;

/**
 * Interface representing continuous problems
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public interface DoubleProblem extends Problem<DoubleSolution> {
  Double getLowerBound(int index) ;
  Double getUpperBound(int index) ;
}
```
Problems implementing `DoubleProblem` only accept `DoubleSolution` objects, and methods for getting the lower and upper limits of each variable have to be implemented.

TO BE COMPLETED
