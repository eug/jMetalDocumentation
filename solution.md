## The `Solution` interface

One of the first decisions that have to be taken when using metaheuristics is to define how to encode
or represent the tentative solutions of the problem to solve. Representation strongly depends on the
problem and determines the operations (e.g., recombination with other solutions, local search procedures,
etc.) that can be applied. Thus, selecting a specific representation has a great impact on the behavior
of metaheuristics and, hence, in the quality of the obtained results.

The following figure depicts the basic components that are used for representing solutions in jMetal 5:
![The Solution Interface](https://github.com/jMetal/jMetalDocumentation/blob/master/figures/jMetal5SolutionClassDiagram.png)
where three representations are included: binary, real, and integer. By using this approach, many implementations can be provided for the same encoding, adding an extra degree of flexibility. The use of generics also allows that an attempt to incorrectly assign the value of a variable results in a compilation error, e.g., trying to assign to an int variable the variable value of a `DoubleSolution`.

The code of the `Solution` interface is shown next:
```java
package org.uma.jmetal.solution;

public interface Solution<T> extends Serializable {
  public void setObjective(int index, double value) ;
  public double getObjective(int index) ;

  public T getVariableValue(int index) ;
  public void setVariableValue(int index, T value) ;
  public String getVariableValueString(int index) ;

  public int getNumberOfVariables() ;
  public int getNumberOfObjectives() ;

  public Solution<T> copy() ;

  public void setAttribute(Object id, Object value) ;
  public Object getAttribute(Object id) ;
}
```

The interface has methods for accessing both the variables and the objectives of a solution, a copy method, and to methods for accessing solution atributes. Defining a particular encoding implies implementing or extending this interface. This way, the interfaces for solutions having a list of double and integer variables are defined as follows:
```java
package org.uma.jmetal.solution;

/**
 * Interface representing a double solutions
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public interface DoubleSolution extends Solution<Double> {
  public Double getLowerBound(int index) ;
  public Double getUpperBound(int index) ;
}
```
```java 
package org.uma.jmetal.solution;

/**
 * Interface representing a integer solutions
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public interface IntegerSolution extends Solution<Integer> {
  public Integer getLowerBound(int index) ;
  public Integer getUpperBound(int index) ;
}
``` 
These interfaces extend `Solution` with methods for getting the lower and upper bounds of the double and integer variables. The way of setting those values are left to the implementation classes.

In the case of a binary solution, the interface is: 
```java
import org.uma.jmetal.util.binarySet.BinarySet;

/**
 * Interface representing a binary (bitset) solutions
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public interface BinarySolution extends Solution<BinarySet> {
  public int getNumberOfBits(int index) ;
  public int getTotalNumberOfBits() ;
}
```

assuming that this interface represents a list of binary variables.

The adopted approach allows to define encodings having mixed variables. For exammple, this interfce defines solutions composed of lists of double and integer values:
```java
package org.uma.jmetal.solution;

/**
 * Interface representing a solution composed of integers and real values
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public interface IntegerDoubleSolution extends Solution<Number> {
  public Number getLowerBound(int index) ;
  public Number getUpperBound(int index) ;
  public int getNumberOfIntegerVariables() ;
  public int getNumberOfDoubleVariables() ;
}
```


### Solution attributes
The idea of incorporating attributes is to allow algorithm to add specific fields to solutions. For example, NSGA-II requires to rank the solutions and assign them the value of the crowding distance. 

To avoid having to manage directly the solution attributes, we include this utility interface:
```java
package org.uma.jmetal.util.solutionattribute;
/**
 * Attributes allows to extend the {@link Solution} classes to incorporate data required by
 * operators or algorithms manipulating them.
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public interface SolutionAttribute <S extends Solution<?>, V> {
  public void setAttribute(S solution, V value) ;
  public V getAttribute(S solution) ;
  public Object getAttributeID() ;
}
```

and a defafult implementation:
```java
package org.uma.jmetal.util.solutionattribute.impl;

/**
 * Generic class for implementing {@link SolutionAttribute} classes
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 */
public class GenericSolutionAttribute <S extends Solution<?>, V> implements SolutionAttribute<S, V>{

  @SuppressWarnings("unchecked")
  @Override
  public V getAttribute(S solution) {
    return (V)solution.getAttribute(getAttributeID());
  }

  @Override
  public void setAttribute(S solution, V value) {
     solution.setAttribute(getAttributeID(), value);
  }

  @Override
  public Object getAttributeID() {
    return this.getClass() ;
  }
}
```
Note that in the current implementation the `getAttributedID()` returns a class identifier. This means that we cannot have two different attributes of the same class. 

### Example of attribute: constraints

### Example of attribute: ranking

The use of solution attributes can be encapsulated. As an
example, we have defined the following interface to assign a
rank to a solution (i.e, NSGA-II’s ranking):
public interface Ranking<S extends Solution>
extends SolutionAttribute<S, Integer>{
public Ranking computeRanking(List<S> solutionList) ;
public List<S> getSubfront(int rank) ;
public int getNumberOfSubfronts() ;
}
so a client class (e.g., the NSGAII class) can merely use:
Ranking ranking = computeRanking(jointPopulation);
This way, the solution attribute is managed internally by
the class implementing the ranking and is hidden to the
metaheuristic.
