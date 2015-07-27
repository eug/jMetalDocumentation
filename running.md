## Running algorihtms
To run an algorithm in jMetal you have two choices: using an IDE or using the command line. We explain both methods in this section. We comment first how configure a metaheuristic to solve a problem.

### Configuring algorithms
In jMetal 5 the only to configure an algorithm is to write a class for that purpose; we refer to those classes as runners. Doing that from an external configuration file is a issue belonging to the todo list of the next release.

We provide at least a runner algorithm. They can be found in the `jmetal-exec` module, in folder https://github.com/jMetal/jMetal/tree/master/jmetal-exec/src/main/java/org/uma/jmetal/runner/multiobjective. 

For example, in the case of NSGA-II we include up to 5 runners:
* `NSGAIIRunner`: configuration of the standard NSGA-II to solve continuous problems.
* `NSGAIIInteger`: configuration to solve integer problems.
* `NSGAIIBinary`: configuration to solve binary problems.
* `NSGAIIMeasures`: similar to `NSGAIIRunner`, but it includes examples of how to use measures.
* `ParallelNSGAII`: as `NSGAIIRunner` but configured to use threads to evaluate the populations in parallel.

We describe next the `NSGAIIRunner` class. The first part of the class is for declaring the types of the problem to solve (a problem dealing with `DoubleSolution` individuals) and the operators. The `referenceParetoFront` is used to indicate optionally the name of a reference front:
```java
public class NSGAIIRunner extends AbstractAlgorithmRunner {
  public static void main(String[] args) throws JMetalException, FileNotFoundException {
    Problem<DoubleSolution> problem;
    Algorithm<List<DoubleSolution>> algorithm;
    CrossoverOperator<DoubleSolution> crossover;
    MutationOperator<DoubleSolution> mutation;
    SelectionOperator<List<DoubleSolution>, DoubleSolution> selection;
    String referenceParetoFront = "" ;
```
The next group of sentences are for processing the program arguments. A benchmark problem is solved by default:
``` java
    String problemName ;
    if (args.length == 1) {
      problemName = args[0];
    } else if (args.length == 2) {
      problemName = args[0] ;
      referenceParetoFront = args[1] ;
    } else {
      problemName = "org.uma.jmetal.problem.multiobjective.zdt.ZDT1";
      referenceParetoFront = "jmetal-problem/src/test/resources/pareto_fronts/ZDT1.pf" ;
    }
```
Next the problem is loaded from its name:

```java
    problem = ProblemUtils.<DoubleSolution> loadProblem(problemName);
```
Then the parameters and the algorithm are configured:
```java 
    double crossoverProbability = 0.9 ;
    double crossoverDistributionIndex = 20.0 ;
    crossover = new SBXCrossover(crossoverProbability, crossoverDistributionIndex) ;

    double mutationProbability = 1.0 / problem.getNumberOfVariables() ;
    double mutationDistributionIndex = 20.0 ;
    mutation = new PolynomialMutation(mutationProbability, mutationDistributionIndex) ;

    selection = new BinaryTournamentSelection<DoubleSolution>(new RankingAndCrowdingDistanceComparator<DoubleSolution>());

    algorithm = new NSGAIIBuilder<DoubleSolution>(problem, crossover, mutation)
        .setSelectionOperator(selection)
        .setMaxIterations(250)
        .setPopulationSize(100)
        .build() ;
```

The last step is to run the algorithm and to print the obtained solutions and optionally the values of all the available quality indicators:
```java
    AlgorithmRunner algorithmRunner = new AlgorithmRunner.Executor(algorithm)
        .execute() ;

    List<DoubleSolution> population = algorithm.getResult() ;
    long computingTime = algorithmRunner.getComputingTime() ;

    JMetalLogger.logger.info("Total execution time: " + computingTime + "ms");

    printFinalSolutionSet(population);
    if (!referenceParetoFront.equals("")) {
      printQualityIndicators(population, referenceParetoFront) ;
    }
  }
```


### Configuring and running an algorithm from an IDE
This is the easiest way to proceed. The first step is to 
