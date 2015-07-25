## The `Solution` interface

One of the first decisions that have to be taken when using metaheuristics is to define how to encode
or represent the tentative solutions of the problem to solve. Representation strongly depends on the
problem and determines the operations (e.g., recombination with other solutions, local search procedures,
etc.) that can be applied. Thus, selecting a specific representation has a great impact on the behavior
of metaheuristics and, hence, in the quality of the obtained results.

The following figure depicts the basic components that are used for representing solutions in jMetal 5:
![The Solution Interface](https://github.com/jMetal/jMetalDocumentation/blob/master/figures/jMetal5SolutionClassDiagram.png)
where three representations are included: binary, real, and integer. By using this approach, many implementations can be provided for the same encoding, adding an extra degree of flexibility.

The code of the `Solution` interface is shown next:
```java
public interface Solution<T> extends Serializable {
  public void setObjective(int index, double value) ;
  public double getObjective(int index) ;

  public T getVariableValue(int index) ;
  public void setVariableValue(int index, T value) ;
  public String getVariableValueString(int index) ;

  public int getNumberOfVariables() ;
  public int getNumberOfObjectives() ;

  public double getOverallConstraintViolationDegree() ;
  public void setOverallConstraintViolationDegree(double violationDegree) ;

  public int getNumberOfViolatedConstraints() ;
  public void setNumberOfViolatedConstraints(int numberOfViolatedConstraints) ;

  public Solution<T> copy() ;

  public void setAttribute(Object id, Object value) ;
  public Object getAttribute(Object id) ;
}
```



