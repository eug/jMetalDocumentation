## The `Operator` interface
Metaheuristic techniques are based on modifying or generating new solutions from existing ones by
means of the application of different operators. For example, EAs make use of crossover, mutation, and
selection operators for modifying solutions. In jMetal, any operation altering or generating solutions (or
sets of them) immplemnts or extends the `Operator` interface:

``` java
package org.uma.jmetal.operator;

/**
 * Interface representing an operator
 *
 * @author Antonio J. Nebro <antonio@lcc.uma.es>
 * @version 0.1
 * @param <Source> Source Class of the object to be operated with
 * @param <Result> Result Class of the result obtained after applying the operator
 */
public interface Operator<Source, Result> {
  /**
   * @param source The data to process
   */
  public Result execute(Source source) ;
}

```
The generics in this interface are intended to indicate that an operator is applied to a `Source` object and returns as a result  `Result` object. 


The framework already incorporates a number of operators, which can be classiffed into four different
classes:
* Crossover. Represents the recombination or crossover operators used in EAs. Some of the included
operators are the simulated binary (SBX) crossover and the single-point crossover for real and
binary encodings, respectively.
* Mutation. Represents the mutation operator used in EAs. Examples of included operators are
polynomial mutation (real encoding) and bit-flip mutation (binary encoding).
* Selection. This kind of operator is used for performing the selection procedures in many metaheuristics. An
example of selection operator is the binary tournament.
* LocalSearch. This class is intended for representing local search procedures. It contains a  method for consulting how many evaluations have been performed after been applied.
