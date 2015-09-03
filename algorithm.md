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
 
The symplicity of `Algorithm` offers plenty of freedom to implement a metaheuristic according to your favorite preferences. However, as jMetal 5 is a framework it includes a set of resources and strategies to help in the implementation of algorithms with the idea of promoting good design, code reusing, and flexibility. The key components are the use of the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) and [algorithm templates](https://github.com/jMetal/jMetalDocumentation/blob/master/algorithmTemplates.md). In the next section we give a detailed description of how the well-known NSGA-II algorithm is implemented, configured, and extended.

### Case study: NSGA-II

NSGA-II is a genetic algorithm (GA), i.e. it belongs to the evolutionary algorithms (EAs) family. The implementation of NSGA-II provided in jMetal 5.0 follows the evolutionary algorithm template described in the [algorithm templates](https://github.com/jMetal/jMetalDocumentation/blob/master/algorithmTemplates.md) section of the documentation. This means that it must define the methods of the [`AbstractEvolutionaryAlgorithm`](https://github.com/jMetal/jMetal/blob/jmetal-5.0/jmetal-core/src/main/java/org/uma/jmetal/algorithm/impl/AbstractEvolutionaryAlgorithm.java) class. According to this template, the flow control of the algorithm is defined in the `run()` method:
```java
@Override public void run() {
    List<S> offspringPopulation;
    List<S> matingPopulation;

    population = createInitialPopulation();
    population = evaluatePopulation(population);
    initProgress();
    while (!isStoppingConditionReached()) {
      matingPopulation = selection(population);
      offspringPopulation = reproduction(matingPopulation);
      offspringPopulation = evaluatePopulation(offspringPopulation);
      population = replacement(population, offspringPopulation);
      updateProgress();
    }
  }
  ```

We describe next how these methods are implemented in the [NSGAII](https://github.com/jMetal/jMetal/blob/jmetal-5.0/jmetal-algorithm/src/main/java/org/uma/jmetal/algorithm/multiobjective/nsgaii/NSGAII.java) class.

The class declaration is as follows:
```java
public class NSGAII<S extends Solution<?>> extends AbstractGeneticAlgorithm<S, List<S>> {
   ...
}
```
which indicates that it extends the [`AbstractGeneticAlgorithm`](https://github.com/jMetal/jMetal/blob/jmetal-5.0/jmetal-core/src/main/java/org/uma/jmetal/algorithm/impl/AbstractGeneticAlgorithm.java). The generic `S` allows to specify the encoding of the solutions that will manipulate the algorithm, and it will determine the kind of problems that can be solved as well as the operators that can be used. This is exemplified in the class constructor:

```java
 public NSGAII(Problem<S> problem, int maxIterations, int populationSize,
      CrossoverOperator<S> crossoverOperator, MutationOperator<S> mutationOperator,
      SelectionOperator<List<S>, S> selectionOperator, SolutionListEvaluator<S> evaluator) {
    super() ;
    this.problem = problem;
    this.maxIterations = maxIterations;
    this.populationSize = populationSize;

    this.crossoverOperator = crossoverOperator;
    this.mutationOperator = mutationOperator;
    this.selectionOperator = selectionOperator;

    this.evaluator = evaluator;
  }
```
The constructor parameter contains:

* The problem to solve.
* The main algorithm parameters: the population size and the maximum number of iterations.
* The genetic operators: crossover, mutation, and selection.
* An [evaluator](https://github.com/jMetal/jMetalDocumentation/blob/master/evaluators.md) object to evaluate the solutions in the population.

We can observe that all the parameters depend on `S`. This way, if `S` is instantiated e.g. to [`DoubleSolution`](https://github.com/jMetal/jMetal/blob/jmetal-5.0/jmetal-core/src/main/java/org/uma/jmetal/solution/DoubleSolution.java), then the problems to be solved must extend `Problem<DoubleSolution>` and all the operators must manipulate `DoubleSolution` objects. 

...

TO BE COMPLETED
