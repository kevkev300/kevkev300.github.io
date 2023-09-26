---
layout: single
title:  Cheatsheet "Practical Object Oriented Design in Ruby"
date:   2023-09-26
categories: books cheatsheets code
permalink: /:year/:month/:day/:title
---

Hello everyone,
I took some notes during reading "Practical Object Oriented Design in Ruby" by *Sandi Metz*. It's an amazing book about basic code design & code architecture. So, I am providing my notes here (also for me to come back to). However, I strongly recommend everyone to read this book, you will get a lot out of it!

## Chapter 2 - Designing Classes with a Single Responsibility
- always Single Responsibility
   - one class - one thing
   - one method - one thing
   - isolate extra responsibility in another class
- Determine: ask questions to the class and see if it makes sense
   > Please Mr. Gear, what is your ratio?
- Decide: future cost of doing nothing today > cost of doing today?
- Depend on behavior, not data: one place to access data
   - always use accessor methods
   - Hide data structures: accessing specific parts of a data structure (e.g. `array[0]`) depends on the structure. Use `Struct` or other objects instead
> Don't decide, preserve your ability to make a decision *later*.

## Chapter 3 - Managing Dependencies
> Knowing creates dependency.
- 4 kinds of dependencies
   - know name of class
   - know method you want to send
   - know arguments that a message requires
   - know order of arguments
- Law of Demeter violation: method chaining with other classes
  > object that knows another that knows another that knows something
- use dependency injection
  ```ruby
  # instead of
  def x(y, z)
    Foo.new(bar, foo) * something
  end

  # do
  def x(foo)
    foo * something
  end
  ```
- isolating dependency into its own method -> DRY and dependency of single method is reduced
  ```ruby
  def x(y, z)
    foo(y, z) * something
  end

  def foo(y, z)
    Foo.new(y, z)
  end
  ```
- isolate vulnerable methods, methods are called on a dependency -> DRY
- use verbose hash arguments instead of positional
- write explicit defaults in initializer
  - use `args.fetch(key, default_value)` instead of `args[key] || default_value`
  - more complex: use default method `args.merge(defaults)`
  - even more complex: use a wrapper method, a "factory"
- manage direction of dependencies
  >  depend on things that change less often than you do
  - consider: depentents vs likelihood of requirement change


## Chapter 4 - Creating Flexible Interfaces
> an app is made out of classes but defined by messages

- class exposes too many methods and knows too much of its neighbors -> difficult to reuse
- plugable, component like objects that reveal little and know little -> reusable
- do not think about objects that you need but messages that you need to exist and then figure our to whom to send them
- interfaces
  - public: show responsibility, don't change often
    - be explicit
    - rather be about *what* than *how*
    - have names that are unlikely to be changed
    - use hash parameters
  - private/protected: implementation details, might change more often
    - "I know more than people in the future do"
    - "I need to protect future coders from using those methods"
    - unsafe to depend on -> do not te
- context
  - trust other objects to know how to do their job
  - asking *what* instead telling another *how* (object vs procedural)
  - think about *what I want* instead of *how it is done* which could be in the real of another object
  - > I know what I want and I trust you to do your part
- **Law of Demeter**
  - "Only talk to your next neighbor"
  - no method chaining
  - might be ok if each method in method chain returns the same class
  - might be ok if just accessing an attribute
  - not okay if changing an instance with this
  - delegation might be a way out of this but might also be bad as it can mask the Law of Demeter
- Think about the purpose of a method, reduce context


## Chapter 5 - Reducing Costs with Duck Typing
- overlooking a duck type results in lots of changes and that a method might need to know way too much
- duck types also have a public interface/contract
- its a role that multiple objects can play
- every object that plays that role must implement the duck types interface so that the method can get called upon every single object
- the method that calls the duck type method trusts that the object implements the duck type interface
- duck types bring flexibility, reduce context and allow easy adding of new objects
- think about duck types especially when there are `case`, `is_a?`, `responds_to?` statements. Checking them would mean
  > I know who you are and because of that I know what you do
- virtual types that define the type by what it does not who it is
- basically, a duck type is just an agreement of how a public interface should look like
> Flexible applications are build on objects that trust; it is your job to make your objects trustworthy.



## Chapter 6 - Acquiring Behavior Through Inheritance
- share a role
- great when having multiple specializations from one stable
- anti-pattern: define different types in the same class which have different behavior
  - need to check for the type before executing the behavior -> dependency! *I know who you are and therefore know what you do*
- a form of automatic method delegation
- single inheritance: can only inherit from one superclass
- superclasses are abstract: they should never get instantiated and their sole purpose is to be inherited
- superclasses are generalizations while subclasses provide the specialization
- names imply inheritance (e.g.: Bike -> Mountain**Bike**)
- every subclass includes the **entire public interface** of its superclass and must respond to it
- subclasses of the same superclass are like related types that share **some** common behavior
- use the Template Pattern
  - push **everything** downstream and pull up what you need
  - don't use `super` as it can easily be forgotten + it creates a dependency that the subclass knows about the superclass
    - use local methods instead like `post_initialize` that gets called in the end of `initialize` instead of `super`
  - every method that the template uses needs to be implemented in the template (superclass)
    - either with a default value
    - or with an explicit error (`NotImplementedError`)
  - use hooks
    - e.g. local defaults in initializer to be implemented in the child: `@foo = args[:foo] || default_foo`
    - e.g. locals in any superclass method: `{food}.merge(local_foo)`


## Chapter 7 -Sharing Role Behavior with Modules
- share a role or multiple roles
- a form of automatic method delegation
- a role orthogonal to class, a role that an object plays that does not come with the class
- also use template pattern: every contract method (public interface) that a subclass implements should be implemented by the superclass
- superclass should not implement methods that only **some** subclasses need
- subclass should never overwrite a superclass' method to raise a `NotImplementedError` as this would be like "I am not this"
- don't write methods that need `super` -> subclass knows algorithm of superclass
- create **shallow hierarchies** -> otherwise the lookup path is too long and every single object in it could overwrite a method -> you don't know what is going on
- antipatterns -> if you see these, refactor
  - an object using a `type` variable to determine which message to send -> use classical inheritance
  - checking the `class` of an object to determine which message to send -> oversaw a duck type
- Honor the contract: Liskov Substitution Principle - subclasses can be used everywhere where the superclass is used

## Chapter 8 - Combining Objects with Composition
- composition: an object that consists of a lot of parts (e.g. a Bike consists of parts like chain...)
- *has_a* relationships
- use a factory to create `Parts` object and the different `Part`s.
  - The factory is the only place that knows about how to read the config and, therefore, how to create the `Part`s
  - ```ruby
    foo = MainObject.new(parts: PartsFactory.build(config))
    foo.parts # <Parts @parts: [#<Part>,...]>
    ```
  - want `foo.parts` to behave like an enumerable? Include `Enumerable`
- easy to create new kinds of the object as you only need to provide a new config
- if `Part` is only has instance variables and no behavior, use `OpenStruct`

### Deciding between Inheritance, Composition and Duck Types
| |Inheritance |Composition |Duck Types |
|-|------------|------------|-----------|
|relationship |*is-a* |*has-a* |*behaves-like*|
|when to use|specialization|behavior is more than the sum of its parts|roles spreading across objects that are outside of their main responsibility|
|con |arranging code in hierarchies, can no reuse existing code|no method delegation, need to be explicit|
|pro |free method delegation |many small, free, single responsibility objects with clear interface|
|reasonable|big behavior changes with small code changes|plugging in a new object does not require code change|
|usable|open-closed: new subclasses do not require code change|pluggable & interchangeable|
|exemplary|provides guidance for writing code| |
|transparent| |single, clear responsibility|
|nots|if inheritance is chosen where it shouldn't, all the positive aspects turn negative|be biased to use composition when possible|common interface or/and behavior, common behavior should be defined in **modules**

- a part within a composition can have specializations via inheritance, e.g.: a specific wheel as a part of a bike
- always ask yourself: "What happens when I am wrong?"

## Chapter 9 - Designing Cost-Effective-Tests
- main problem: people write too many
- test the *public interface* as this is the most stable thing
- kinds of tests/messages
  - tests of state:
    - assert the value the message returns
    - incoming message
    - no side effects
    - test for messages in own public interface
  - tests of command:
    - outgoing message
    - side effects
    - responsibility of sending object to test that they were properly sent
  - tests of queries:
    - outgoing message
    - no side effects
    - part of public interface of receiver that already implements it as test of state
    - do not test
- isolate objects under tests: use test doubles (stubs) to define a value for outgoing messages
  - test the test doubles to have the correct interface as what they stub (RSpec: verifying_doubles)
- Do not test private methods, or better: don't even write them but extract int public interface of another class
- use shared tests to test roles / duck types
- test inheritance
  - shared test for interface (`SuperclassInterfaceTest`)
  - shared test for subclass responsibilities e.g. what subclass **can** overwrite (`SuperclassSubclassTest`)
  - superclass test fo abstract superclass behavior like
    - method that forces subclass to implement method (e.g.: with `NotImplementedError`)
    - including `locals`
  - unique subclass behavior, e.g.: that `locals` have the correct value BUT not that it is included
