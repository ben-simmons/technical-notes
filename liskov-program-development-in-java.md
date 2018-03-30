Program Development in Java: Abstraction, Specification, and Object-Oriented Design

* Barbara Liskov
* 2000


## Ch 1. Introduction

### 1.1. Decomposition and Abstraction

### 1.2. Abstraction

* **Procedures**: separate procedure definition from invocation.
	* Uses abstraction by parameterization and abstraction by specification.
* **Abstraction by parameterization**: abstracts from the identity of the data by replacing them with parameters.
	* Generalizes modules so that they can be used in more situations. 
* **Abstraction by specification**: abstracts from the implementation details to the behavior users can depend on.
	* Allows modules to change independently.
	* We only require that a module's implementation supports the behavior being relied on.
* Kinds of abstraction:
	* **Procedure abstraction** allows us to introduce new operations.
	* **Data abstraction** allows us to introduce new types of data objects.
	* **Iteration abstraction** allows us to iterate over items in a collection without revealing details of how the items are obtained.
	* **Type hierarchy** allows us to abstract from individual data types to families of related types.

#### 1.2.2. Abstraction by Specification

* **precondition** (or **requires assertion**) specifies something that is assumed to be true on entry to the procedure.
* **postcondition** (or **effects assertion**) specifies something that's supposed to be true at the completion of any invocation of the procedure for which the precondition was satisfied.


## Ch 5. Data Abstraction

* **Data abstraction** allows us to abstract from the details of how data objects are implemented to how the objects behave.
* We achieve abstraction by specification by making the operations part of the type.
	* If we didn't have operations:
		* All that is needed to implement the type is to select a storage representation for the objects.
		* But then if the representation ever changes, or if its interpretation ever changes, all programs that use the type must be changed.
	* I.e. data abstraction is useful whenever we have to *change* the program.
* **IMPORTANT** Data abstraction allows us to defer decisions about data structures until the uses of the data are fully understood.
	* At the beginning, in absense of info, the chosen structure may lack needed information or be organized in an inefficient way.
	* We start by introducing the abstract type.
	* Decisions about how to implement the abstract type are made later.

### 5.1. Specifications for Data Abstractions

### 5.2. Using Data Abstractions

### 5.3. Implementing Data Abstractions

### 5.4. Additional Methods

### 5.5. Aids to Understanding Implementations

* The *abstraction function* captures the designer's intent in choosing a particular representation.
	* It describes the decision you made about what the instance variables represent.
* The *rep invariant* captures the common assumptions on which the implementations are based. Allows you to consider the implementation of each operation in isolation of the others.
	* **HOW??** Because you can assume that the rep invariant holds at the beginning of each operation. As long as all operations conform to the rep invariant, then this is a valid assumption. You just have to prove the rep invariant holds for each operation, but you can prove that for each one independently.

#### 5.5.1. The Abstraction Function

#### 5.5.2. The Representation Invariant

* Frequently, not all objects of a class are legitimate representations of abstract objects.

#### 5.5.3. Implementing the Abstraction Function and Rep Invariant

* The abstraction function explains the interpretation of the rep.
	* It maps the state of each legal representation object to the abstract object it is intended to represent.
* The representation invariant defines all the common assumptions that underlie the implementations of type's operations.
	* It defines which representations are legal by mapping each representation object to either true (legal) or false (illegal).
	* The abstraction function says nothing about the behavior of illegal representations. That's the whole purpose of the rep invariant -- to ensure that illegal representations can't exist.

#### 5.5.4. Discussion

* Rep invariants always hold for the reps, even when an object is used outside its implementation.
	* I.e. Even if the rep object does not map to a valid abstract object.
* Rep invariant can be violated while executing one of the type's operations, but must hold whenever operations return to their callers.
	* I.e. the violation cannot be observed.
* All operations must be implemented in a way that preserves the rep invariant.

### 5.6. Properties of Data Abstraction Implementations

#### 5.6.1. Benevolent Side Effects

* A mutable abstraction must have a mutable rep.
* An immutable abstraction may have an immutable or mutable rep.
	* A mutable rep is not a problem *so long as modifications made to the rep cannot be observed by the abstraction's users*.
* **Benevolent side effects** are modifications that are not visible outside the implementation.
	* Example of data type for rational numbers.
	* Rep stores numerator and denominator.
	* Rep may store it in non-reduced or reduced form.
	* `equals()` does the reduction and retains the reduced form, which mutates state, but doesn't matter because the mutation isn't visible to the users.
* Side effects are often performed for reasons of efficiency.
* Side effects are possible whenever the abstraction function is many-to-one.
	* Many rep objects then represent a particular abstract objects.

#### 5.6.2. Exposing the Rep

* We want the ability to do **local reasoning**
	* We want to be able to ensure that a class is correct just by examining the code of that class.
* If local reasoning is not supported, we say the implementation **exposes the rep**.
	* Exposing the rep means that the implementation makes mutable components of the rep (e.g. instance variables) accesible to code outside of the class.

### 5.7. Reasoning about Data Abstractions

#### 5.7.1. Preserving the Rep Invariant

* Must show that the rep invariant holds for all concrete objects.
* Can use **data type induction**
	* First step: Establish the property for the constructor(s).
	* Induction step: Establish the property for the methods.

#### 5.7.2. Reasoning about Operations

* In these proofs we can assume that the rep invariant holds on entry of each method.
* Note that we are able to reason about each operation independently, which is possible because of the rep invariant.

#### 5.7.3. Reasonging at the Abstract Level

* **Abstract invariant** is the abstract analog of the rep invariant.
	* E.g. for vectors and arrays we assume that their size is >= zero, and that all indexes that are >= zero and less than the size are in bounds.
* Can completely ignore observers in the proofs since they don't modify their objects.
* Note that we're reasoning at an abstract level, not an implemention level.
* Working at the abstract level can greatly simplify reasoning.
	* **QUESTION** How exactly? I think it simplifies the extra reasoning you have to do to establish a valid rep invariant.

#### 5.8.1. Design Issues

#### 5.8.2. Operation Categories

* **Creators**. Create objects from scratch.
	* All creators are constructors, most constructors are creators.
* **Producers**. Create objects from other objects.
	* May be either constructors or methods.
* **Mutators**. Modify objects of their type.
	* Only mutable types can have mutators.
* **Observers**. Take objects of their type as inputs and returns results of other types.
	* E.g. On `IntSet`: `size()` returns int, `isIn()` returns boolean.

**QUESTION** Would a method that returns `this` be a creator, producer, or an observer?

#### 5.8.3. Adequacy

* Data type is **adequate** if it provides enough operations so that everything users need to with its objects can be done both conveniently and efficiently.
	* It's a loose definition.
	* Type must be **fully populated**: between its creators, mutators, and producers, it must be possible to obtain every possible abstract object state.

### 5.9. Locality and Modifiability

* **Locality** is the ability to reason about a module by just looking at its code and not any other code.
	* Requires that a representation be modifiable only within its type's implementation.
	* If modifications can occur elsewhere, then we cannot establish correctness just by examining its code. Cannot guarantee locally that the rep invariant holds, and cannot use data type induction.
	* I.e. the module must not expose the rep.
* **Modifiability** is the ability to reimplement an abstraction without having to reimplement any other code.
	* If access to the representation occurs in some other module, we cannot replace the implementation without affecting that other module.
* I think these two points make clear that the implementation is not the same thing as the rep. The rep is also an abstract thing, even though it's a lower abstraction than the abstract object.
	* I guess you could say that a *correct* implementation maps one-to-one with the rep?
	* As opposed to an implementation that exposes the rep, for example.


## Ch 6. Iteration Abstraction

* An **iterator** is a special kind of procedure that causes the items we want to iterate over to be produced incrementally.
	* Iterators here are a generalization of the iteration mechanisms available in programming languages.
* Iterators are frequently needed for *adequacy*.

### 6.1. Iteration in Java

* An iterator returns a special kind of object called a **generator**.
	* A generator keeps track of the state of an iteration in its rep.
		* Has `hasNext()` and `next()` methods.
	* Remember that iterator is the procedure; generator is the object.
* All Java generators belong to types that are subtypes of the `Iterator` interface.
	* Guess is should have been called the `Generator` interface?

### 6.2. Specifying Iterators

* Note that the iterator specification includes a requirement *on the code using the generator*.
	* E.g. User can't call remove() before iteration is complete.
	* Add a requires clause (at end of Iterator procedure instead of beginning) that `this` must not be modified while the generator is in use.

### 6.5. Rep Invariants and Abstraction Functions for Generators

* All generators have the same abstract state: a sequence of items that remain to be generated.
	* The abstraction function needs to map the rep to this sequence.


## Ch 7. Type Hierarchy

* This chapter discusses a way to enhance the utility of data abstraction by defining families of related types.
* A type family is defined by a **type hierarchy** (i.e. inheritance).
* Type families are used in two different ways:
	* Multiple implementations of a type, where subtypes do not add any new behavior, except that each of them has its own constructors.
		* E.g. sparse and dense polynomials
	* But usually subtypes extend the behavior of their supertypes.
* The **substitution principle** is the property that the supertype's behavior must be supported by the subtypes: subtype objects can be substituted for supertype objects without affecting the behavior of the using code.
	* **IMPORTANT** THIS IS THE LISKOV SUBSTITUTION PRINCIPLE.
	* Interesting that Liskov herself defined it in the "common" sense, not Bob Martin.
	* It allows us to abstract from the differences among the subtypes to the commonalities, which are captured in the supertype specification.

### 7.1. Assignment and Dispatching

* The utility of type hierarchy rests on a loosening of the rules governing assignment and argument passing and on the way calls are dispatched to code.

#### 7.1.1. Assignment

* A variable declared to belong to one type can actually refer to an object belonging to some subtype of that type.
	* Means that the type of object referred to by a variable is not necessarily the same as what is declared for the variable.
	* We distinguish them as the object's **apparent type** and its **actual type**.
* The compiler does type checking based on the information available to it: it uses the apparent types, not the actual types.

#### 7.1.2. Dispatching

* Calling the right method is achieved by a runtime mechanism called **dispatching**.
	* **IMPORTANT** The compiler does not generate code to call the method directly. Instead, it generates code to find the method's code and then branch to it.
* One of the ways to implement dispatching:
	* Have objects contain, in addition to their instance variables, a pointer to a **dispatch vector**, which contains pointers to the implementations of the object's methods.

### 7.2. Defining a Type Hierarchy

* First step is to define the type at the top.
	* This type's specification may be incomplete -- e.g. lacking constructors.
* Specifications of subtypes are given relative to those of their supertype.
	* Only limited to changes to the behavior of the supertype methods are allowed (see section 7.9).

### 7.3. Defining Hierarchies in Java

* Supertypes in Java are defined by both classes and interfaces
	* Note that classes are NOT the same thing as types.
	* **Interfaces** only define a specification, it does not contain any code implementing the supertype.
	* **Concrete classes** provide a full implementation of the type.
	* **Abstract classes** provide at most a partial implementation of the type.
* **IMPORTANT** It's best if subclasses can interact with superclasses entirely via the public interface since this preserves full abstraction and allows the superclass to be reimplemented without affecting the implementations of its subclasses.
	* Me: sort of like the opposite of the Template Method pattern. Subclasses are the framework instead of the superclass being the framework.
	* But that interface may be inadequate to permit efficient subclasses.
	* In that case, the superclass can define **protected** methods, constructors, and instance variables that are visible to subclasses.
	* Note that protected members are also package visible: that modifier not a strict mapping to member usage with type hierarchies.

### 7.4. A Simple Example

* `MaxIntSet` subtype of `IntSet`
	* Does not modify IntSet behavior, only extends with a method to get the max element.
* In this case, the abstraction function is defined in terms the supertype abstraction function.
	* In fact, here it completely reuses the supertype abstraction function. 
	* The fact that the two abstraction functions are the same reflects the fact that `MaxIntSet` relies on `IntSet` to store the set elements.
* In this case, the rep invariant is NOT defined in terms of the rep invariant for `IntSet`.
	* Preserving the `IntSet` rep invariant is the job of `IntSet`'s implementation, and there is no way that the implementation of `MaxIntSet` can interfere since it has only public access to the `IntSet` part of its rep.
	* Though, repOk() should always check the invariant for the superclass since the subclass rep cannot be correct if the superclass part of the rep is not correct.
	* **IMPORTANT** Remember that type defines the rep invariant, but the rep applies to the object, not the type.
* If `MaxIntSet` required access to the `IntSet` rep (e.g. for efficiency), then the rep invariant of `MaxIntSet` must *include* the rep invariant of `IntSet`.

### 7.6. Abstract Classes

* Abstract class constructors cannot be called by its users, but the constructors can be used by subclasses to initialize the superclass's part of the rep.
	* Example: An abstract class that has state visible to its subclasses, like `protected int size` in `IntSet`
	* **Question** Are interfaces allowed to have constructors? IIRC they can define variables, just not method bodies.
* The **template pattern** is where the abstract class provides a nonabstract method that makes use of the abstract methods, which allows the superclass to define the generic part of the implmentation, with the subclasses filling in the details.
	* Me: compare to subclass using the public interface of the superclass. In that case, the subclasses are depending on the superclass to fill in the details.
* Typical for an abstract class to have no abstraction function, since the real implementations are provided by the subclasses.
* It's better to avoid use of protected members.
	* 1. In Java, protected members are also package visible.
	* 2. The superclass can be reimplemented without affecting the implementation of any subclass.
	* Though they can sometimes be useful to enable efficient implementations of subclasses.

### 7.7. Interfaces

* A class is used to define a type and also to provide a complete or partial implementation.
* An interface defines only a type.
	* Provides only nonstatic, public methods, and all of its methods are abstract.
* A class can only extend one class, but it can implement multiple interfaces.

### 7.8. Multiple Implementations

* Multiple implementations can be thought of as defining a very constrained type family.

#### 7.8.1. Lists

* Each implementation is more efficient than would be possible without multiple implementations, if you can avoid conditional tests that would be required in a single implementation, e.g. EmptyIntList doesn't need to do an isEmpty check, compared to IntList.
* Don't need to override `equals()` in the subtypes. They are invisible to user except for creation. Recall that they are meant only to change implementation, not behavior.

#### 7.8.2. Polynomials

* Keep the degree as an instance variable of superclass `Poly`, since this is useful information for all `Poly` subclasses.
* Cool trick: after mutation, decide if it's more efficient to be another implementation, and convert to that subclass.
	* E.g. SparsePoly to DensePoly, and vice versa.
	* Make these constructors package visible (not protected), because the implementation subclasses need to have access to them, even though they don't directly depend on each other.

### 7.9. The Meaning of Subtypes

* **IMPORTANT** Subtypes must satisfy the substitution principle so that users can write and reason about code just using the supertype specification.
* Three properties must be supported:
	* **Signature Rule**: The subtype objects must have all the methods of the supertype, and the signatures of the subtype methods must be compatible with the signature of the corresponding supertype methods.
		* Guarantees that every call that is **type-correct** according to the supertype's definition is also type correct for the subtype.
		* In Java, this is enforced by the compiler.
		* Java allows the subtype to have fewer exceptions than supertype method.
			* Code written in terms of the supertype can function correctly even if those exceptions are not thrown.
		* Java's notion of compatibility is stricter than necessary: Java requires that the return type of the sub- and super-method be identical, when it only need check if the subtype method returned a subtype.
			* Because, if subtypes conform to the substitution principle, they are always substitutable for the supertype!
			* **NOTE**: This was written in 2000 before covariance was added to Java in version 5.0.
	* **Methods Rule**: Calls of these subtype methods must "behave like" calls to the corresponding supertype methods.
	* **Properties Rule**: The subtype must preserve all properties that can be proved about supertype objects.
* All of these behaviors concern *only* specifications.
	* We are interested in whether the supertype and subtype specifications are sufficiently similar that the substitution principle is satisfied.
* Methods and Properties rules guarantee that subtype objects behave enough like supertype objects that code written in terms of the supertype won't notice the difference.
	* **IMPORTANT** These requirements CANNOT be checked by a compiler since they require reasoning about the meaning of specifications.
		* **QUESTION** I wonder if that statement is a provable thing about compilers or if it really relies on it being incredibly difficult to imbue computers with human reasoning.

#### 7.9.1. The Methods Rule

* When objects belong to subtypes, their method calls actually go to the code provided by the implementation of the subtype.
* The methods rule says that we can still reason about the meanings of these calls using the supertype specification, even though the subtype code is running.
	* Ex: For any IntSet, if we call y.insert(x), we know x is in the set when the call returns.
	* Ex: For any Poly, if a call p.coeff(3) returns 6, we know that the degree of p is at least 3. (the third term has a non-zero coefficient).
* New behavior is "acceptable" if it takes advantage of the nondeterminism of the supertype's specification.
	* When we give new specifications for supertype methods in subtypes, we often take advantage of nondeterminism like this.
	* Me: but that might also mean that you have to strenghten your specification, depending on if that new behavior makes sense.
* **IMPORTANT** A subtype method can weaken the precondition and strengthen the postcondition.
	* `=>` here means logical implication
	* **Precondition Rule**: `pre_super => pre_sub`
		* The subtype requires less of its caller than the supertype method does.
		* Must accept just as much as the supertype method does, but can also accept more.
		* If a call doesn't satisfy the precondition, then anything could happen. In the subtype, we have simply decided on a particular thing.
		* **QUESTION** I assume it's ok because the rep invariant must still be held. The subtype has to figure out how to do that with more liberal input.
	* **Postcondition Rule**: `(pre_super && post_sub) => post_super`
		* When it returns, everything that supertype method would provide is assured, and maybe some additional effects as well.
		* Exception example:
			* Rule violated:
				* It's legal by the compiler for subtype method to throw fewer exceptions than supertype method.
				* But it's not legal by the supertype specification, if the spec requires throwing the exception as an effect! (e.g. IntList subclass add() that doesn't throw DuplicateException)
			* Rule satisfied:
				* Iterator throws NoSuchElementException
				* But it's legal by spec for allPrimes generator to not ever throw that exception, because there's always a greater prime.
		* Example: int vs long
			* ints are 32 bits, longs are 64 bits
			* They have different behavior in certain cases
			* Rule violated: if adding two ints results in an overflow, the overflow will not happen if the same two values are longs.
			* **QUESTION** This is tricky. Not totally grasping it. Longs will still overflow (same sort of behavior), but not in the same cases as int.
			* **QUESTION** Would it still be a violation if we changed the spec? What if spec was to throw OverflowException if the parameterized value of max was breached, instead of a concrete value of max (e.g. concrete values being 2^31-1 for 32-bit, 2^63-1 64-bit)

#### 7.9.2. The Properties Rule

* We also reason about properties of objects, not just the effects of individual calls.
* Some properties are **invariants**: they are always true for objects of the type.
* Some properties are **evolution properites**: they involve reasoning about how objects evolve over time.
	* If a Poly has degree six, then it will always have degree six, because Poly is immutable.
	* Me: This example is NOT the same as invariant, because, while the Poly degree could be said to be "invariant" for the object itself, that instance variable is not shared by all objects of the type. If we had a SixDegreePoly class, then degree six would be an invariant property.
* To show that a subtype satisfies the **properties rule**, we must prove that it preserves each property of the supertype.
	* Proof of an invariant property can use datatype induction.
		* Creators and producers establish, the invariant, and all methods of the subtype must preserve the invariant.
		* Including the "extra" methods in the subtype, and the subtype constructors. ALL methods must preserve the invariant.
	* Proof of an evolution property involves showing that every method preserves it.
* The properties must be defined in the *supertype* definition.
* The invariant properties come from the abstract model.
	* Ex: IntSets are modeled as mathematical sets
		* Invariants: must have a size >= zero, must not contain duplicate elements.
	* Ex: OrderedIntLists are modeled as sequences that are sorted in ascending order.
		* Invariants: their elements appear in sorted order.
* Evolution property examples:
	* Poly: immutability
	* SimpleSet: objects can grow over time but not shrink
* An immutable type can have mutable subtypes provided that their mutations cannot be observed using supertype methods.
	* The mutations *MAY* be observed by subtype methods.

* **IMPORTANT** Recap of the substitution principle rules:
	* The **signature rule** ensures that if a program is type-correct based on the supertype specification, it is also type-correct with respect to the subtype specification.
	* The **methods rule** ensures that reasoning about calls of supertype methods is valid even though the calls actually go to code that implements a subtype.
	* The **properties rule** ensures that reasoning about properties of objects based on the supertype specification is still valid when objects belong to a subtype. The properties must be stated in the overview section of the supertype specification.

#### 7.9.3. Equality

* Meaning of the `equals` method:
	* See chapter 5.
	* If two objects are `equals`, it will never be possible to distinguish them in the future using methods of their type.
	* For mutable types, this means objects are `equals` only if they are the very same object.
	* For immutable types, this means objects are `equals` if they have the same state.
		* Subtype objects of an immutable type might be distinguishable (can have extra methods that are mutators and more state), but code that uses them via the supertype interface cannot distinguish them.
		* Example: Point2 vs Point3
		* Point3 subtype of Point2
		* **IMPORTANT** Point3 must provide its own extra `equals` method, AND it must also override `equals` for Point2 and Object.
		* Needed so `equals` works property regardless of object's apparent type.
		* **QUESTION** does this mean you always have to override the equals method for *every type* in the hierarchy above your subtype??
			* Brittle, as practical experience is that types are often inserted into the middle of the hierarchy.

### 7.10. Discussion of Type Hierarchy

* Three different kinds of supertypes:
	* **Incomplete supertypes**: they serve just to establish constraints on the behavior of subtypes.
		* Using code is typically not written in terms of them.
		* Example: collection supertype defines observers and mutators, even though the mutators will not be used by immutable subtypes.
	* **Complete supertypes** provide entire data abstractions, with useful specifications for all the methods.
		* Example: Reader interface.
	* **Snippets** provide just a few methods, not enough to qualify as an entire data abstraction.
		* Example: Cloneable, it indicate only that subtypes have a `clone` method.
		* Snippets are always defined by interfaces.
* **IMPORTANT** The substitution principle precludes the use of inheritance as a *simple code-sharing mechanism*, because it requires that the subtype be similar to the supertype.

### 7.11. Summary

* Benefits of hierarchy:
	* Can be used to define the relationship among a group of types, making it easier to understand the group as a whole.
	* Allows code to be written in terms of a supertype, yet work for many types -- all the subtypes of that supertype.
	* Hierarchy provides extensibility: code can be written in terms of a supertype, yet continue to work when subtypes are defined later.
	* These benefits can *only* be obtained if subtypes obey the substitution principle.


## Ch 8. Polymorphic Abstraction

* **Polymorphic abstractions** work for many types.
	* The abstraction might be polymorphic with respect to the type of elements its objects contain
		* `Vector` is polymorphic w.r.t. its element type.
	* In Java, polymorphism is expressed through hierarchy.

## 8.1. Polymorphic Data Abstractions

* Example of a specification for shallow copy instead of deep copy
* Does not expose the rep because the state of set consists only of the identities of its elements and not their states.

## 8.2. Using Polymorphic Data Abstractions

* Collections contain Objects (this was written before generics)

## 8.3. Equality Revisited

* Contents of an object of the collection type depends on how `equals` is implemented for the elements.
	* Ex: Vector `equals` returns true if the two vectors have the same state, not just if they're the identical object
	* Can't just use `==` because that approach won't work for immutable types (where state *does* determine equality).
	* Solution is to wrap the vector in container objects when you intend to distinguish distinct vector objects.
	* The Container `equals` uses `==` instead of Vector `equals`.

## 8.4. Additional Methods

* `Comparable` interface
	* Adds `compareTo()`

## 8.5. More Flexibility

* Preplanning how the supertype is defined to account for subtypes is not always possible.
	* Me: Understatement of the century!
* In that case, can add a new interface, with a subtype corresponding to each element subtype.
	* Ex: `Adder` interface with `PolyAdder`

## 8.6. Summary

* Sometimes more methods are needed than the ones on `Object`.
* In this case, polymorphic abstraction makes use of an interface to define the needed methods.
	* Two different ways to define the interface:
		* **Element subtype** approach: Use an interface that is a supertype of the element types.
			* Ex: `Comparable` interface.
			* Problem with this approach is that is requires preplanning.
		* **Related subtype** approach: Use an interface that is a supertype of types that are related to the element types.
			* Ex: `Adder` interface.


# Ch 9. Specifications

### 9.1. Specifications and Specificand Sets

* The **specificand set** is the set of all program modules that **satisfy** the **specification**.

### 9.2. Some Criteria for Specifications

#### 9.2.1. Restrictiveness

* A specification is **sufficiently restrictive** if it rules out all implementations that are unacceptable to an abstraction's users.
* A specification is **sufficiently general** if it does not preclude acceptable implementations.

#### 9.2.2. Generality

* A good specification should be general enough to ensure that few, if any, acceptable programs are precluded.
	* Ex: square root specification that requires `0 <= (rt*rt - sq) <= e`
	* Constraints the implementor to find approximations that are greater than or equal to the actual square root.
* Specification styles:
	* **Definitional style** explicitly lists properties that members of the specificand set are to exhibit.
	* **Operational style**, instead of describing the properties, gives a recipe for constructing them.
		* Pro: Are more easily implemented by programmers because tey more closely resemble programming.
		* Con: Often lead to over-specification (more restrictive than needed).

#### 9.2.3. Clarity

Two types of misunderstanding:

* Reader knows they misunderstood due to some uncertainty in the specification.
* Reader thinks they understood but misinterpreted.

### 9.3. Why Specifications?

* Goal is to write specifications that are both restrictive enough and general enough.


## Ch 10. Testing and Debugging

## Ch 11. Requirements Analysis

## Ch 12. Requirements Specifications

## Ch 13. Design

* "A particular decomposition cannot accomodate all changes with equal ease."
	* Axis of change. Talked about by Bob Martin. Designing for one often precludes another.
* Their design process:
	* Start a design with the requirements specification.
	* For each **target abstraction**:
		* Identify **helping abstractions**, or **helpers**, that will be useful in implementing the target and that facilitate decomposition of the problem.
		* Specify the behavior of each helper.
			* Pin down the details.
		* Sketch an implementation of the target.
			* Just enough of an implementation to convince yourself that an acceptably efficient and modular implementation can be constructed.
* For large programs, we usually don't know in advance what the structure of the program ought to be. The discovery of this structure is a main goal of the design.

### 13.2. The Design Notebook

#### 13.2.1. The Introductory Section

* Module M1 depends on module M2 if a change to M2's specification might cause a change in M1.
* Using arc
* Extension arc
	* Indicates a subtype relationship
	* 

## Ch 14. Between Design and Implementation

#### 14.1.2. Structure

* "A second cause of lack of coherence is hand optimization of programs. An eagle-eyed programmer may notice that some arbitrary group of statements appears several times. In an attempt to save space, the programmer may bundle these statements into a procedure. In the long run, however, such optimizations are generally counterproductive because they make the program harder to modify."
	* See this a lot
	* DRY gone too far -- sometimes it's better to just inline repeated code.

## Ch 15. Design Patterns

### 15.3. The Bridge Pattern

* We use hierarchy for two different purposes: to extend behavior and to provide multiple implementations. These two uses can interfere.
* The **bridge pattern** separates the implementation hierarchy from the subtype hierarchy.
* This pattern occurs naturally where the state pattern is used, but it can be used more generally.

### 15.4. Procedures Should Be Objects Too

* A **closure** is a procedure, some of whose formal arguments are already bound.
	* **IMPORTANT** Can effectively make closures in Java with single-method objects.
	* Constructor binds some of the arguments.
* The **strategy pattern** and **command pattern** make up for the lack of procedure objects.
	* Strategy pattern expects certain behavior.
	* Command pattern does not care about behavior, but does expect a certain interface.

### 15.6. The Power of Indirection

* The **adapter**, **proxy**, and **decorator** patterns all interpose an object between the using code and the original object.
* Proxy and Decorator both support the same interface as original object.
	* Proxy has identical behavior to original object.
	* Decorator extends the behavior of original object.

### 15.7. Publish/Subscribe

* The **observer** pattern is basically pub/sub without a separate messaging system.

#### 15.7.1. Abstracting Control

* The **mediator** pattern basically inserts a layer between subjects and observers. Observers register with the mediator, subject communicates with mediator, mediator forwards the message however it wants.
* Communication need not be asymmetric; mediator can be used by a group of "colleagues".
* The mediator pattern can be further generalized to a **white board**, where information is posted.