## Quality indicators

Quality indicators are considered in jMetal 5 as components of the core package (`jmetal-core`). As many other components, there is a generic interface and an `impl` package containing the provided implementations of that interface.

The `QualityIndicator` interface is very simple:

```java
package org.uma.jmetal.qualityindicator;

/**
 * @author Antonio J. Nebro <antonio@lcc.uma.es>

 * @param <Evaluate> Entity to evaluate
 * @param <Result> Result of the evaluation
 */
public interface QualityIndicator<Evaluate, Result> extends DescribedEntity {
  public Result evaluate(Evaluate evaluate) ;
  public String getName() ;
}
```

