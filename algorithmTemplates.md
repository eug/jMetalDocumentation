## Algorithm templates

Most of metaheuristic families are characterized by a common behaviour which is shared by all the algorithms belonging to the family. This behaviour can be expressed as a template that can be instatiated to implement a particular algorithm. From a software engineering point of view, an algorithm whose behavior falls in a basic template would only require to implement some specific methods for the new technique; the common behavior would not be needed to be programmed, therefore resulting in less code replication. This inversion of control if a characteristic of [software frameworks](https://en.wikipedia.org/wiki/Software_framework), as is the case of jMetal.

### The Evolutionary Algorithm Template
Many papers about evolutionary algorithms (EAs) include a pseudo-code to describe them similar to this one:  
```
P(0) ← GenerateInitialSolutions() 
t ← 0
Evaluate(P(0))
while not StoppingCriterion() do
  P'(t) ← selection(P(t))
  P''(t) ← Variation(P'(t)) 
  Evaluate(P''(t))
  P (t + 1) ← Update(P (t), P''(t)) 
  t←t+1
end while
```

To mimic this pseudo-code, jMetal 5 incorporates a template in the form of an abstract class named [`AbstractEvolutionaryAlgorithm`](https://github.com/jMetal/jMetal/blob/master/jmetal-core/src/main/java/org/uma/jmetal/algorithm/impl/AbstractEvolutionaryAlgorithm.java), which contains the following code:
```java
package org.uma.jmetal.algorithm.impl;

/**
 * Created by Antonio J. Nebro on 26/10/14.
 * @param <S> Solution
 * @param <R> Result
 */
public abstract class AbstractEvolutionaryAlgorithm<S extends Solution<?>, R>  implements Algorithm<R>{
  private List<S> population;
  public List<S> getPopulation() {
    return population;
  }
  public void setPopulation(List<S> population) {
    this.population = population;
  }

  protected abstract void initProgress();
  protected abstract void updateProgress();
  protected abstract boolean isStoppingConditionReached();
  protected abstract List<S> createInitialPopulation();
  protected abstract List<S> evaluatePopulation(List<S> population);
  protected abstract List<S> selection(List<S> population);
  protected abstract List<S> reproduction(List<S> population);
  protected abstract List<S> replacement(List<S> population, List<S> offspringPopulation);
  @Override public abstract R getResult();

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
}  
```

TO BE COMPLETED
