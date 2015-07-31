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

### An example of indicator: Epsilon

To illustrate how most of the indicators are implemented in jMetal 5, we describe next the code of the Epsilon indicator.

The declaration of the Epsilon class is included in this piece of code: 
```java
public class Epsilon<Evaluate extends List<? extends Solution<?>>>
    extends SimpleDescribedEntity
    implements QualityIndicator<Evaluate,Double> {
  ...    
}
```
Although at a first glance it can seem a very complex declaration, it simply states that `Evaluate` must be a List of any kind of jMetal `Solution`, so any attempt to use the indicator with a non compatible object will be detected at compiling time. 
