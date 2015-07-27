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

We describe next the `NSGAIIRunner` class. The Javadoc comment indicate the program parameters: the first one is the class of the problem to solve and, optionally, a reference front file name can be given. In that case, the reference front will be used to compute all the quality indicators available:
```java
public class NSGAIIRunner extends AbstractAlgorithmRunner {
  /**
   * @param args Command line arguments.
   * @throws JMetalException
   * @throws FileNotFoundException
   * Invoking command: java org.uma.jmetal.runner.multiobjetive.NSGAIIRunner problemName [referenceFront]
   */
  public static void main(String[] args) throws JMetalException, FileNotFoundException {
```
The first part of the `main` method is for declaring the types of the problem to solve (a problem dealing with `DoubleSolution` individuals) and the operators. The `referenceParetoFront` is used to indicate the name of the optional reference front:
```jaava
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
Next the problem is loaded from its class name:
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
The last step is to run the algorithm and to write the obtained solutions into two files and optionally to print the values of all the available quality indicators:
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

### Running an algorithm from an IDE
Once you have configured your algorithm you can use your favorite IDE to execute them. For example, in the case of IntellJ Idea you can select the runner class name and select the option "Run 'NSGAIIRunner.main()'" clicking with the left mouse button if you intend to run NSGA-II:
![Running with IntellJ Idea](https://github.com/jMetal/jMetalDocumentation/blob/master/figures/runningNSGAIIRunnerInIntelliJIdea.png)

As as result of the execution, the following messages are printed into the output console:
```
/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/bin/java -Didea.launcher.port=7532 "-Didea.launcher.bin.path=/Applications/IntelliJ IDEA 14 CE.app/Contents/bin" -Dfile.encoding=UTF-8 -classpath "/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/tools.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Users/ajnebro/Softw/jMetal/jMetal/jmetal-exec/target/classes:/Users/ajnebro/Softw/jMetal/jMetal/jmetal-core/target/classes:/Users/ajnebro/.m2/repository/org/apache/maven/reporting/maven-reporting-api/3.0/maven-reporting-api-3.0.jar:/Users/ajnebro/.m2/repository/org/apache/maven/doxia/doxia-sink-api/1.0/doxia-sink-api-1.0.jar:/Users/ajnebro/.m2/repository/org/apache/commons/commons-lang3/3.3.2/commons-lang3-3.3.2.jar:/Users/ajnebro/.m2/repository/org/apache/commons/commons-math3/3.3/commons-math3-3.3.jar:/Users/ajnebro/.m2/repository/org/hamcrest/hamcrest-all/1.3/hamcrest-all-1.3.jar:/Users/ajnebro/Softw/jMetal/jMetal/jmetal-algorithm/target/classes:/Users/ajnebro/Softw/jMetal/jMetal/jmetal-problem/target/classes:/Applications/IntelliJ IDEA 14 CE.app/Contents/lib/idea_rt.jar" com.intellij.rt.execution.application.AppMain org.uma.jmetal.runner.multiobjective.NSGAIIRunner
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.multiobjective.NSGAIIRunner main
INFORMACIÓN: Total execution time: 1147ms
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Random seed: 1438006918503
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Objectives values have been written to file FUN.tsv
jul 27, 2015 4:21:59 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printFinalSolutionSet
INFORMACIÓN: Variables values have been written to file VAR.tsv
jul 27, 2015 4:22:00 PM org.uma.jmetal.runner.AbstractAlgorithmRunner printQualityIndicators
INFORMACIÓN: 
Hypervolume (N) : 0.6594334269577787
Hypervolume     : 0.6594334269577787
Epsilon (N)     : 0.012122558511198256
Epsilon         : 0.012122558511198256
GD (N)          : 2.054388435747992E-4
GD              : 2.054388435747992E-4
IGD (N)         : 1.8304524180524584E-4
IGD             : 1.8304524180524584E-4
IGD+ (N)        : 0.003808931172199927
IGD+            : 0.003808931172199927
Spread (N)      : 0.34070732976112383
Spread          : 0.34070732976112383
R2 (N)          : 0.13179198315493879
R2              : 0.13179198315493879
Error ratio     : 1.0
```
