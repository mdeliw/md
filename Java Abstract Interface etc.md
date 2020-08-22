[toc]

This article is about practical use of *Abstract*  and *Interface* Class in Java.  



Java is an Object Oriented Language and allows concepts like Inheritance and Composition to define a relationship between concepts. The composition concept is a better representation of the real world compared to the inheritance concept. So understand Inheritance, but don’t expect to use it in real world. That said, inheritance has value in some types of uses.



> Inheritance provides a ==“IS A"== relation, while composition provides a ==“CAN BE”== or a ==“HAS A” relation.  



Example:

- *Ostrich is a bird, but can’t fly.* The flying attribute is a composition and not inheritance.
- *Most animals have 4 legs. Kangaroo is a animal, but has 5 legs. Some animals can survive with 3 legs*.
- Most birds can’t speak human words, but some parrots can. 
- Truck and Car is a Vehicle. Car can be roofless. Truck can have more than 4 wheels.

## Inheritance

Uses are limited to tightly linked concepts that share a lot of common behaviors and can be extended into a simple structure of parent - child concepts. In a complex world where parent - child hierachy is very long (child has a child that also has a child etc), inheritance can quickly become confusing to understand how a child is inheriting  its attributes from which parent.  This is called *“exploding classes”* problem.

### How to use?

In Java, a *“concrete”* class is a concept that is implemented  using variables and methods. The implementation is termed as *”how the class behaves”*.

Let’s say you have a concept that fits into a class definition. You have three choices to develop the class:

- If you know *all the behaviors* of the concept, you can create a *“concrete class”*. A concrete class can allow further modification in behaviors via inheritance, or it can be locked to not allow inheritance. 
- If you *don’t know all the behaviors* of the concept, you can create an *“abstract class”* where some behaviors are defined (is concrete), while remaining behaviors are left as abstract, to the inheritance pattern to allow one or more child classes to make them concrete. Subsequently you can have childrens that can define some more behaviors, and leave some behaviors abstract - these childrens are also called *”abstract”*. A child that has all behaviors defined (some defined via inheritance, and some define in this class), is called a *”concrete”* class.
- You can also have a class where no behaviors are known - meaning every behavior is left abstract. This is of no use though.

### Know the concepts

- *Abstract class*
- *Concrete class*
- *Super  class* - A parent class is called a super class in the context of its child class.
- *Base class* - This is the parent class - which can be a abtract or a concrete class.
- *Derived class* - This is a child class - which can be a abtract or a concrete class.
- Java doesn’t allow a class to inherit from multiple parents, only a single parent. 

## Interface

With recent enhancements in the language, an interface allows abstraction, polymorphism, and multiple inheritance. These recent changes remove the need for use of inheritance by the use of of *“extends”* 

An interface supports:

- *Abstract method* - these can be overridden
- *Constant variable*
- *Concrete method* (called default methods) - these can be overridden
- *Static method*
- *Inheritance* - a concrete class can inherit multiple interfaces. Multiple inheritance is same as Composition.
- Also a child interface can inherit from a parent interface class - but this pattern is not common in real world.
- *Composition* - similar to multiple inheritance
- *Polymorphism* - a property by which an instance of a clas (object) can behave differently based on need. For example, a single object called *Shape* can behave like a *Square* or a *Circle* 
- An abstract class (from a inheritance concept) can implement an interface

