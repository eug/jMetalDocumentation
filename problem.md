## The `Problem` interface
To include a problem in jMetal, it must implemente the `Problem` interface:

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

TO BE COMPLETED
