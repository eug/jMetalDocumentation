# Measures

A novelty in jMetal 5.0 is the inclusion of measures, which allow to obtain algorithm-specific information during its execution. The current implementation supports two types of measures: a `PullMeasure` provides the value of the measure on demand (synchronous), while a `PushMeasure` allows to register listeners to receive the value of the measure when it is produced (asynchronous).

## What are measures?

Measures are designed to access specific properties of a running algorithm. For instance, one could know the size of the current population in a genetic algorithm, the current velocity of the particles in a particle swarm optimization, the current iteration, etc. In order to deal properly with the many properties that an algorithm could have, two types of measures are provided.

Usually, properties are accessed by getters, like `getPopulationSize()` or `getCurrentIteration()`. These properties are accessed in a synchronous way, which means that we obtain (and process) them on the user's demand. Each property of this kind can be accessed through a `PullMeasure`, as shows the `PullMeasure.get()` method:
```
public interface PullMeasure<Value> extends Measure<Value> {
	public Value get();
}
```

At the opposite, a `PushMeasure` allows to obtain the value of a property in an asynchronous way, which means that the value is provided upon generation, the user having no control on when such a value will be provided to him. This is achieved by using an [Observer Design Pattern](https://en.wikipedia.org/wiki/Observer_pattern), such that the user registers one or several `MeasureListener` instances to receive and process the value when it is produced:
```
public interface PushMeasure<Value> extends Measure<Value> {
	public void register(MeasureListener<Value> listener);
	public void unregister(MeasureListener<Value> listener);
}

public interface MeasureListener<Value> {
	public void measureGenerated(Value value);
}
```

Some people could wonder why these interfaces only require *reading* methods: the `PullMeasure` does not provide a `set(value)` method to assign its value, and the `PushMeasure` does not provide some kind of `push(value)` method neither. This is because these interfaces are designed from a user perspective, not an algorithm designer perspective, and the user can only read these values, not write them. One could say that it makes the thing harder for the designer, but actually the designer needs to know exactly which implementation to use, because he is the one who know how the values will be provided. So if he uses an existing implementation, he should of course know the specific methods of this implementation, which can include methods to set/push his values. Only the user does not need to know about the specific implementation used by the measure, so he is the one who will have to deal with generic `PullMeasure` and `PushMeasure`. Moreover, while it is natural to think about these set and push methods, it is actually not the only way to implement these measures. For instance, the algorithm designer could use a member variable to store the value of a property, and the update of this property will be done through this variable. This way, no need for a set method, and the designer can implement an anonymous class which simply read this variable when `get()` is called. Such an example is provided in the section [*How to use measures?*](#designer-perspective-implement-an-algorithm-with-measures).

One could notice that both the measure interfaces extend `Measure`. This one is an empty interface which is used as a root interface to both `PullMeasure` and `PushMeasure`. By having it, we give the possibility for programers to manage measures in a generic way, without having to consider both types of measures separately nor to use `Object` to have them together. Thus, it is only here to simplify the use of measures for some highly generic cases in jMetal. No algorithm designer nor user should normally need it.

Additionally to the measures themselves, jMetal 5 also provides the notion of `MeasureManager`, which deals with measures management features:
```
public interface MeasureManager {
	public Collection<Object> getMeasureKeys();
	public <T> PullMeasure<T> getPullMeasure(Object key);
	public <T> PushMeasure<T> getPushMeasure(Object key);
}
```
You can notice that we provide a method for each type rather than a generic method returning a `Measure`. This is because, as mentionned before, the `Measure` interface is empty, thus has no practical value. Moreover, when the user want to exploit a measure, he should know which type of measure is implemented to know how to interact with it. Rather than providing a `Measure` and arbitrarily cast it, we prefered to provide methods which directly provide the right type of measure. Notice that a measure instance can implement both interfaces (or two instances can be made available for the same property), so both methods can return an instance (same or not) for the same key, giving to the user the choice of using one way or another. This key identifies the specific property to measure and is algorithm-dependent, justifying the presence of an additional `getMeasureKeys()` for the user to retrieve them.

Finally, in order for an algorithm to explicit that it provides measures, it should implement the `Measurable` interface, which simply require to provide a `Measure Manager`:
```
public interface Measurable {
	public MeasureManager getMeasureManager();
}
```

One could wonder why we introduced the intermediary concept of `MeasureManager` rather than putting directly its methods into the `Measurable` interface. Indeed, by construction, the designer could simply decide to implement both the interfaces on the algorithm and implement `getMeasureManager()` by simply returning `this`, showing that we can artificially ignore the intermediary concept. Also, in the case where we actually reduce to the `Measurable` interface, the designer could add an intermediary objects which implements `Measurable` (so the methods of `MeasureManager`), and implement it also for the algorithm, but using his custom object in background. So technically, we see that the introducing an intermediary notion of `MeasureManager` does not impose any constraint, justifying that we use the most simple design (no intermediary concept). Anyway, we made this design choice for the following reason: while the `MeasureManager` should focus on the measure management strategy, a `Measurable` should focus on which measures to provide. For instance, a `MeasureManager` could implement an advanced management strategy which, for measures implementing only one of the measure interfaces, would automatically instantiate a `PullMeasure`/`PushMeasure` in order to have all the features for each measure, with some smart inspection management for `PushMeasures` corresponding to `PullMeasures`. On the other hand, a `Measurable` instance, typically an algorithm, would focus on choosing which measures to provide and feeding them, delegating any measure management process to a dedicated `MeasureManager` implementation.

## Why using measures?

Usually, properties are accessed by getters, like `getPopulationSize()` or `getCurrentIteration()`. The advantage of this design is that it gives a simple way to access these properties. However, we quickly face the limitations of this design when we want to read properties which evolve in time, like the current iteration. Indeed, such a design imposes to read these properties on a regular basis, sometime on a frequent basis to see all the updates. Such a design becomes particularly cumbersome when we want to access short-term values, like which individuals have been generated at a given iteration of a genetic algorithm, which is usually forgotten at the next iteration. Accessing such properties at runtime would require whether to consume a lot of computation time to continuously inspect some `getGeneratedIndividuals()` method, or to consume a lot of space to store all the data generated and access it through some `getGeneratedIndividuals(int iteration)` method.

The design we choose for jMetal 5 is based on the notion of measure, which further split into two categories: `PullMeasure` and `PushMeasure`. The simplest one, the `PullMeasure`, is designed for the first kind of property, which can be easily accessed (pulled) from a getter. While one could simply use a getter, the `PullMeasure` provides a more generic access to the property, allowing to apply generic evaluations on the algorithm (like generic experiments, which are not yet available in jMetal 5). The other type of measure, the `PushMeasure`, is designed for the other kind of properties, which are not easily managed through getters and need to be provided (pushed) in real-time by the algorithm. By using such a measure, the values can be received and processed without risking to loose any update, and without the need to continuously inspect the property.

Another advantage of having both these measures is that it can be easily integrated to an algorithm without requiring significant additional resources: indeed, for cases where a value have to be stored into a variable for the algorithm to use it, like an iteration counter to stop after N iterations, such a value can be covered (or replaced) by a `PullMeasure`. At the opposite, when a value is generated but not stored during the running of the algorithm, a `PushMeasure` can be used to push the value to any potential listener and forget about it, letting the listeners decide whether or not this value should be stored or further processed. The additional resources consumed are only:
- the stored measures, which are often lightweight and less numerous than the solutions to store
- the calls to the listeners of the `PushMeasures`, which are negligible if no listener is registered, otherwise fully controlled by the user

## How to use measures?

This section is particularly developped, because there is several kinds of *use*. First, we take the perspective of the *algorithm user*, who uses an algorithm providing measures, because it is important to know what a user can do for the algorithm designer to choose the right measures to exploit. Then we consider the perspective of an *algorithm designer*, who can face again two different situations: having an *existing algorithm* to adapt in order to exploit measures, or implementing a *new algorithm* with measures in mind from the start.

### User Perspective: Using Measures Provided by an Algorithm

...

It is worth noting that a `PushMeasure` can be trivially transformed into a `PullMeasure` by storing the pushed value into a variable, allowing its retrieval on demand. This is what makes `MeasureFactory.createPullFromPush(pushMeasure, initialValue)`, and this is why it is often more interesting for the user to have a `PushMeasure` rather than a `PullMeasure`. Indeed, a `PushMeasure` provides an increased facility for processing updates, and the user can easily create an equivalent `PullMeasure` in the case where he is not interested in a fine monitoring of these updates. Consequently, if a property is prone to evolve during a run, it is recommended to use a `PushMeasure` rather than a `PullMeasure`, both of them needing anyway to be updated at the right time.

In the case where only a `PullMeasure` is available but the user wants to know when the value is updated, he can use `MeasureFactory.createPushFromPull(pullMeasure, period)` to create a `PushMeasure` which regularly inspect the `PullMeasure` (as specified by the `period`) and notifies the registered listeners when the obtained value is different from the previous one. Because such a computation can be particularly costly, and the `period` parameter need to be finely tuned, this solution should be reserved for cases where the user have no other way to be notified about updates. For instance, if another `PushMeasure` is available and can be used to know when the first measure is updated, it is more reasonable to register a listener to this second measure in order to read the first measure at the right time. All these difficulties are another reason why an algorithm designer should prefer to use a `PushMeasure` when it is possible.

### Designer Perspective: Adapt an Algorithm to Exploit Measures

In this case, properties are generally already well identified, and methods are there for accessing them, particularly with getters (`getXxx()`). In the second case, one intents to identify the relevant properties to use as `PullMeasures` and `PushMeasures` in order to adapt its algorithm consequently before to implement it.

For instance, one could have a code which looks like this:
```
public class MyAlgorithm {
}
```

...
(speak about the use of `MeasureFactory.createPullsFromGetters(object)`)
(speak about the use of `MeasureFactory.createPullsFromFields(object)`)
(show example of anonymous class implementing `PullMeasure` which simply read a member variable when calling its `get()` method)
(speak about implementing `Measurable` and existing implementations of `MeasureManager`)

### Designer Perspective: Implement an Algorithm with Measures

...
One can transform a pull measure into a push one (e.g. with a period of refresh) as well as a push measure into a pull one (e.g. by storing the value for future demands), thus choosing one or the other is more a design choice depending on how the algorithm manage the measured values. For instance, if the value is always available in the algorithm (e.g. population size), then it is usual to use a pull measure to retrieve it on demand, while a value generated on the fly (e.g. solution evaluation) is prone to be managed by a push measure. % to avoid storing it.









```
public class NSGAIIMeasures extends NSGAII {
  private CountingMeasure iterations ;
  private BasicMeasure<List<Solution>> lastEvaluatedPopulation ;

  ...
  
  iterations = new CountingMeasure(0) ;
  lastEvaluatedPopulation = new BasicMeasure<>() ;

  public NSGAIIMeasures(...) {
    ...
    measureManager = new SimpleMeasureManager() ;
    measureManager.setPullMeasure("currentIteration", 
                                  iterations);
    measureManager.setPushMeasure("lastEvaluatedPopulation", 
                                  lastEvaluatedPopulation);
  }
  
  @Override protected void initProgress() {
    iterations.reset(1);
  }

  @Override protected void updateProgress() {
    iterations.increment();
    lastEvaluatedPopulation.set(getPopulation());
  }
  
  ...
}
```


In this example, our algorithm declares a `CountingMeasure` (a measure intended to count events) called `iterations` and aiming at counting how many iterations has been made since the starting of the algorithm.
Another one is a `BasicMeasure` (simply stores a value) called `lastEvaluatedPopulation` and aiming at providing the population considered at the evaluation time, thus giving a way to know how the population evolve between each round.
The point with push measures is when to effectively push the value to the listeners, thus identifying the precise location where the value changes.
Here, the iteration counting is reset at every call of `initProgress()`, which corresponds to a restart of the algorithm, and both measures are updated in `updateProgress()`, which is called at the end of each iteration (notice that these two methods are required by the evolutionary algorithm template defined in Section~\ref{subsection:templates}).
Regarding the `MeasureManager`, we register both the measures as pull or push measures depending on how we want them to be used out of the algorithm.
The client code can make use of these measures as follows:

```
...
 Algorithm<List<Solution>> algorithm;
 ...
  MeasureManager measureManager = 
     algorithm.getMeasureManager() ;
    
  CountingMeasure iteration =
    (CountingMeasure) measureManager.getPullMeasure(
                              "currentIteration");
    BasicMeasure<List<Solution>> lastEvaluatedPopulationMeasure
        = (BasicMeasure) measureManager.getPushMeasure(
                              "lastEvaluatedPopulation");
        
    lastEvaluatedPopulationMeasure(new Listener());
    
    Thread algorithmThread = new Thread(algorithm) ;
    algorithmThread.start();

    while(iteration.get() < maxIterations) {
      TimeUnit.SECONDS.sleep(5);
      System.out.println("Iteration:  " + iteration.get()) ;
    }

    algorithmThread.join();
}
```

In this piece of code, the measure manager is obtained from the algorithm, so the measures are now accessible. In the case of the pull measures, the algorithm is executed in a thread and the client code prints every five seconds the current iteration while the algorithm is concurrently running.

We can observe that a listener is registered to use the push measure. An example listener class is shown next:

```
public class Listener
                    implements MeasureListener<List<Solution>> {
  private int counter = 0 ;

  @Override synchronized public void 
           measureGenerated(List<Solution> solutions) {
   if ((counter % 10 == 0)) {
     // Do whatever action with the solution list (population).
   }
   counter ++ ;
  }
}
```

In this code, the `measureGenerated()` method is called whenever a push operation is invoked on the push measure, and an action is performed every 10 invocations. A typical action could be to write the current Pareto front approximation in a file for experimental evaluation.

The benefit of using a push measure in this example is that there is no need of modifying the algorithm code; all the logic is in the client side. The approach has also a minimum impact on performance: if no listeners are registered, the push operations only will make a check in an empty list.

In general, pull measures are useful when the client program needs to get information in a particular moment about the current state of the algorithm execution. On the contrary, pull measures are more useful when they are intended to inform the client code about relevant events that are produced in an asynchronous way. In the former example, that event is evaluating the population after an algorithm iteration, but more complex scenarios could be considered, e.g., the population has not changed in the last $N$ iterations, or the population does not contain infeasible solutions in case of solving a constrained problem.
