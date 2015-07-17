<div id='id-architecture'/>
##Architecture

The architecture of jMetal 5.0 relies on four interfaces: 

![jMetal architecture](https://github.com/jMetal/jMetalDocumentation/blob/master/figures/jMetal5CoreClassDiagram.png)

This diagram captures the typical functionality provided by jMetal: an `Algorithm` solves a `Problem` by manipulating a set of potential `Solution` objects through the use of several `Operators`. We can observe the use of parametrized types to model the use of Java generics, which are now widely applied.

The use of generics in the architecture allows to align all the components in metaheuristic so that the compiler can check that everything fit. For example, this code represents all the elements to configure an genetic algorithm to solve a continuous problem:

```java
    Problem<DoubleSolution> problem;
    Algorithm<List<DoubleSolution>> algorithm;
    CrossoverOperator<DoubleSolution> crossover;
    MutationOperator<DoubleSolution> mutation;
    SelectionOperator<List<DoubleSolution>, DoubleSolution> selection;
```
