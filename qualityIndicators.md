## Quality indicators

Quality indicators are considered in jMetal 5 as components of the core package (`jmetal-core`). As many other components, there is a generic interface and an `impl` package containing the provided implementations of that interface.

The `QualityIndicator` interface is very simple:

```java
package org.uma.jmetal.qualityindicator;

/**
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 *
 * @param <Evaluate> Entity to evaluate
 * @param <Result> Result of the evaluation
 */
public interface QualityIndicator<Evaluate, Result> extends DescribedEntity {
  public Result evaluate(Evaluate evaluate) ;
  public String getName() ;
}
```

The idea is than every quality indicator is applied to some entity (`Evaluate`) to be evaluated, and it returns a `Result`. The use of generics allows to represent indicators returning anything, from a double value (the most usual return type) to a pair of values as in the case of our implementation of Set Coverage. The quality indicators also has an associated name.

### Auxiliary classes
Before describing how quality indicators are implemented, we must comment before a number of auxiliary classes that are used:
* The [`Front`](https://github.com/jMetal/jMetal/blob/master/jmetal-core/src/main/java/org/uma/jmetal/util/front/Front.java) interface and [`ArrayFront`](https://github.com/jMetal/jMetal/blob/master/jmetal-core/src/main/java/org/uma/jmetal/util/front/imp/ArrayFront.java) class: Frequently, a reference Pareto front is stored in a file containing the objective values of a number of solutions. A `Front` is an entity intended to store the contents of these files; in the case of the `ArrayFront` class, it stores the front into an array of points.
* The [`FrontNormalizer`](https://github.com/jMetal/jMetal/blob/master/jmetal-core/src/main/java/org/uma/jmetal/util/front/util/FrontNormalizer.java) class: many indicators normalize the list of solutions to be evaluated, and this class is intended to do this: given a reference front or the maximum and minimum values, it returns a normalized list of solutions.

### An example of indicator: Epsilon

To illustrate a quality indicator in jMetal 5, we describe next the code of the Epsilon indicator.

The declaration of the Epsilon class is included in this piece of code: 
```java
public class Epsilon<Evaluate extends List<? extends Solution<?>>>
    extends SimpleDescribedEntity
    implements QualityIndicator<Evaluate,Double> {
  ...    

```
Although at a first glance it seems a very complex declaration, it simply states that `Evaluate` must be a List of any kind of jMetal `Solution`, so any attempt to use the indicator with a non compatible object will be detected at compiling time. 

Our approach to implement most of indicators is to consider that most of them require a reference front to be computed, so that front must be incorporated as a parameter of the class constructor: 

```java
  private Front referenceParetoFront ;

  /**
   * Constructor
   *
   * @param referenceParetoFrontFile
   * @throws FileNotFoundException
   */
  public Epsilon(String referenceParetoFrontFile) throws FileNotFoundException {
    super("EP", "Epsilon quality indicator") ;
    if (referenceParetoFrontFile == null) {
      throw new JMetalException("The reference pareto front is null");
    }

    Front front = new ArrayFront(referenceParetoFrontFile);
    referenceParetoFront = front ;
  }
...
```

Then, the `evaluate` method computes the indicator value by using the reference front:
```java
  /**
   * Evaluate() method
   *
   * @param solutionList
   * @return
   */
  @Override public Double evaluate(Evaluate solutionList) {
    if (solutionList == null) {
      throw new JMetalException("The pareto front approximation list is null") ;
    }

    return epsilon(new ArrayFront(solutionList), referenceParetoFront);
  }
```

Readers interested in how the Epsilon is computed can find all the code [here]( https://github.com/jMetal/jMetal/blob/master/jmetal-core/src/main/java/org/uma/jmetal/qualityindicator/impl/Epsilon.java)

### About normalization
An important issue to take into account is that quality indicators do not normalize the solution list to be evaluated. Instead, the user can choose if the fronts are normalized or not before using them.

This piece of code shows an example of how reading a reference from a file and how to get a `FrontNormalized` from it:
```java
Front referenceFront = new ArrayFront("fileName");
FrontNormalizer frontNormalizer = new FrontNormalizer(referenceFront) ;
```
Then, the front normalizer can be use to a normalized reference front:
```java
Front normalizedReferenceFront = frontNormalizer.normalize(referenceFront) ;
```
And then, given any solution list to be normalized, it can be done this way:
``` java
List<Solution> population ;
...
Front normalizedFront = frontNormalizer.normalize(new ArrayFront(population)) ;
```

###Using quality indicators
One we have decided about normalization, we can create a quality indicator and use it. We select the Hypervolume as an example:
```java
Hypervolume<List<? extends Solution<?>>> hypervolume ;
hypervolume = new Hypervolume<List<? extends Solution<?>>>(referenceFront) ;

double hvValue = hypervolume.evaluate(population) ;
```

###Discussion
Leaving the normalization up to the user can be error prone, but there is a performance advantage: if the same indicator has to be applied to many solution lists, the normalization of the reference front is carried out only once. This is the case, for example, when some indicator-based algorithms have to find the solution contributing the least to the Hypervolume.

### Computing quality indicators from the command line
If you need to compute the value of a given quality indicator of a front of solutions from the command line you can use the [`CommandLineIndicatorRunner`](https://github.com/jMetal/jMetal/blob/master/jmetal-exec/src/main/java/org/uma/jmetal/qualityIndicator/CommandLineIndicatorRunner.java) class.

The usage of this program is:
```
java org.uma.jmetal.qualityIndicator.CommandLineIndicatorRunner indicatorName referenceFront frontToEvaluate TRUE | FALSE
```

where indicator name can be:
* `GD`: Generational distance
* `IGD`: Inverted generational distance
* `IGD+`: Inverted generational distance plus
* `HV`: Hypervolume
* `SPREAD`: Spread (two objectives)
* `GSPREAD`: Generalized spread (more than two objectives
* `ER`: Error ratio
* `ALL`: Select all the available indicators

The last parameter is used to indicate whether the fronts are to be normalized or not computing the quality indicators.

TO BE COMPLETED
