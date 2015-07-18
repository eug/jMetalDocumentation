<div id='introduccion'/>
##Introduction

The jMetal project started in 2006 as the result of our need of having a easy to use, flexible, extensible and portable multi-objctive optimization framework with metaheuristics. Since 2008 it has been hosted in http://jmetal.sourceforge.net. Since 2014 the current development is located in https://github.com/jMetal/jMetal.

After nine years since the first release of jMetal, we have decided it's time to make a deep redesign of the software. Some of the ideas we have taking in to cosideration are:

* Architecture redesign to provide a simpler design while keeping the same functionality.
* Maven is used as the tool for development, testing, packaging and deployment.
* Promote code reusing by providing algorithm templates
* Improve code quality:
  * Application of unit testing
  * Better use of Java features (e.g, generics)
  * Design patterns (singleton, builder, factory, observer
  * Application of clean code guidelines - “Clean code: A Handbook of Agile Software Craftsmanship" (Robert C. Martin)
* Parallelism support
* Introducing measures to get information of the algorithms in runtime
