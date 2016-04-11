This is my summary of the "Refactoring: Improving the Design of Existing Code" by Martin Fowler. I use it while learning and as quick reference. It is not intended to be an standalone substitution of the book so if you really want to learn the concepts here presented, buy and read the book and use this repository as a reference and guide.

If you are the publisher and think this repository should not be public, just write me an email at hugomatilla [at] gmail [dot] com and I will make it private.
#3 Bad Smells in code
### Duplicated code
Same code structure in more than one place.
### Long Method
The longer a procedure is the more difficult is to understand.
### Large Classes
When a class is trying to do too much, duplicated code cannot be far behind.
### Long Parameter List
The are hard to understand, inconsistent and difficult to use.
### Divergent Change
When one class is commonly changed in different ways for different reasons.
### Shotgun Surgery
When every time you make a kind of change, you have to make a lot of little changes to a lot of different classes.
### Feature Envy
A method that seems more interested in a class other than the one it actually is in.
### Data Clumps
Bunches of data(fields, parameters...) that hang around together.
### Primitive Obsession
Using primitives types instead of small objects.
### Switch Statements
The same switch statement scattered about a program in different places. Use polymorphism.
### Parallel Inheritance Hierarchies
Every time you make a subclass of one class, you also have to make a subclass of another.
### Lazy Class
A class that isn't doing enough to pay for itself should be eliminated.
### Speculative Generality
All sorts of hooks and special cases to handle things that aren't required.
### Temporary Field
An instance variable that is set only in certain circumstances.
### Message Chain
When a client asks one object for another object, which the client then asks for yet another object...
### Middle Man
When an object delegates much of its functionality.
### Inappropriate Intimacy
When classes access to much to another classes.
### Alternative Classes with Different Interfaces 
Classes with methods that look to similar.
### Incomplete Library Class
When we need extra features in libraries.
### Data Class
Don't allow manipulation in Data Classes. Use encapsulation and immutability.
### Refused Bequest
Subclasses that don't make uses of parents methods.
### Comments
Not all comments but the ones that are there because the code is bad.

#6 Composing methods
##1 Extract Method

You have a code fragment that can be grouped together.

```java

	void printOwing(double amount) {
		printBanner();
		//print details
		System.out.println ("name:" + _name);
		System.out.println ("amount" + amount);
	}
```

to 

```java

	void printOwing(double amount) {
		printBanner();
		printDetails(amount);
	}
	
	void printDetails (double amount) {
		System.out.println ("name:" + _name);
	System.out.println ("amount" + amount);
	}
```

**Motivation**

* Increases the chances that other methods can use a method
* Allows the higher-level methods to read more like a series of comments

**Mechanics**

* Create a new method, and name it after the intention of the method
* Copy the extracted code into the new method
* Scan for references to any variables local in scope
* If temporary variables are only in the extracted code set them as temporary variables.
* If temporary variables are modified, return its modified value. If this is not possible try
	* [6 Split Temporary Variable]()
	* [4 Replace Temp with Query]()
* If variables are read, pass them in the constructor
* Replace
* Compile and test.

```java

	void printOwing(double previousAmount) {
		Enumeration e = _orders.elements();
		double outstanding = previousAmount * 1.2;
		printBanner();
		
		// calculate outstanding
		while (e.hasMoreElements()) {
			Order each = (Order) e.nextElement();
			outstanding += each.getAmount();
		}
		printDetails(outstanding);
	}
```	

to

```java

	void printOwing(double previousAmount) {
		printBanner();
		double outstanding = getOutstanding(previousAmount * 1.2);
		printDetails(outstanding);
	}

	double getOutstanding(double initialValue) {
		double result = initialValue;
		Enumeration e = _orders.elements();
		
		while (e.hasMoreElements()) {
			Order each = (Order) e.nextElement();
			result += each.getAmount();
		}
		return result;
	}
```

##2 Inline Method
A method's body is just as clear as its name.

```java

	int getRating() {
		return (moreThanFiveLateDeliveries()) ? 2 : 1;
	}

	boolean moreThanFiveLateDeliveries() {
		return _numberOfLateDeliveries > 5;
	}
```

to

```java

	int getRating() {
		return (_numberOfLateDeliveries > 5) ? 2 : 1;
	}
```
**Motivation**

* When Indirection is needless (simple delegation) becomes irritating.
* If group of methods are badly factored and grouping them makes it clearer


**Mechanics**

* Check that the method is not polymorphic.
* Find all calls to the method.
* Replace each call with the method body.
* Compile and test.
* Remove the method definition

##3 Inline Temp
You have a temp that is assigned to once with a simple expression, and the temp is getting in the way of other refactorings.

```java

	double basePrice = anOrder.basePrice();
	return (basePrice > 1000)
```

to 

```java
	
	return (anOrder.basePrice() > 1000)
```	
**Motivation**

* Use it with [4 Replace Temp with Query]()

**Mechanics**

* Declare the temp as final if it isn't already, and compile.
* Find all references to the temp and replace them with the right-hand side of the assignment.
* Compile and test after each change.
* Remove the declaration

##4 Replace Temp with Query
You are using a temporary variable to hold the result of an expression.
```java
	
	double basePrice = _quantity * _itemPrice;
	if (basePrice > 1000)
		return basePrice * 0.95;
	else
		return basePrice * 0.98;
```
to

```java

	if (basePrice() > 1000)
		return basePrice() * 0.95;
	else
		return basePrice() * 0.98;
	...
	double basePrice() {
		return _quantity * _itemPrice;
	}
```

**Motivation**

* Replacing the temp with a query method, any method in the class can get at the information.
* Is a vital step before [1 Extract Method]()

**Mechanics**

* Look for a temporary variable that is assigned to once. For more than [6 Split Temporary Variable]()
* Declare the temp as final.
* Compile.
* Extract the right-hand side of the assignment into a method.
	* Initially mark the method as private
	* Ensure there are no side effects (does not modify any object) or use [Separate Query from Modifier]()
* Compile and test
* [3 Inline Temp]() on the temp.

##5 Introduce Explaining Variable
You have a complicated expression
```java
	
	if ( (platform.toUpperCase().indexOf("MAC") > -1) &&
		(browser.toUpperCase().indexOf("IE") > -1) &&
		wasInitialized() && resize > 0 )
	{
		// do something
	}
```
to

```java
	
	final boolean isMacOs = platform.toUpperCase().indexOf("MAC") >-1;
	final boolean isIEBrowser = browser.toUpperCase().indexOf("IE") >-1;
	final boolean wasResized = resize > 0;
	if (isMacOs && isIEBrowser && wasInitialized() && wasResized) {
		// do something
	}

```	

**Motivation**

* When expressions are hard to read

**Mechanics**

* Declare a final temporary variable, and set it to the result of part of the complex expression.
* Replace the result part of the expression with the value of the temp.
* Compile and test.
* Repeat for other parts of the expression.

If using [1 Extract Method]() cost same effort use it, it will make available this methods out from the method scope. 
##6 Split Temporary Variable
You have a temporary variable assigned to more than once, but is not a loop variable nor a collecting temporary variable.

```java

	double temp = 2 * (_height + _width);
	System.out.println (temp);
	temp = _height * _width;
	System.out.println (temp);
```
to
```java

	final double perimeter = 2 * (_height + _width);
	System.out.println (perimeter);
	final double area = _height * _width;
	System.out.println (area);
```

**Motivation**
 
* Variables should not have more than one responsibility.
* Using a temp for two different things is very confusing for the reader.

**Mechanics**

* Change the name of a temp at its declaration and its first assignment. (No if  is of the  `i = i + x`)
* Declare the new temp as final.
* Change all references of the temp up to its second assignment.
* Declare the temp at its second assignment.
* Compile and test.
* Repeat in stages, each

##7 Remove Assignments to Parameters
The code assign to a parameter
```java
	
	int discount (int inputVal, int quantity, int yearToDate) {
		if (inputVal > 50) inputVal -= 2;
```
to
```java
	
	int discount (int inputVal, int quantity, int yearToDate) {
		int result = inputVal;
		if (inputVal > 50) result -= 2;
```

**Motivation**

* You can change the internals of  object is passed but do not point to another object.
* Use only the parameter to represent what has been passed.

**Mechanics**

* Create a temporary variable for the parameter.
* Replace all references to the parameter, made after the assignment, to the temporary variable.
* Change the assignment to assign to the temporary variable.
* Compile and test.

##8 Replace Method with Method Object
You have a long method that uses local variables in such a way that you cannot apply Extract Method.

```java

	class Order...
		double price() {
			double primaryBasePrice;
			double secondaryBasePrice;
			double tertiaryBasePrice;
			// long computation;
			...
		}
```
to
```java

	class Order...
		double price(){
			return new PriceCalculator(this).compute()
		}
	}	

	class PriceCalculato...
	compute(){
		double primaryBasePrice;
		double secondaryBasePrice;
		double tertiaryBasePrice;
		// long computation;	
		return ...
	}

```

**Motivation**

* When a method has lot of local variables and applying decomposition is not possible

**Mechanics**

* Create a new class, name it after the method.
*  Give the new class a final field for the object that hosted the original method (the source object) and a field for each temporary variable and each parameter in the method.
* Give the new class a constructor that takes the source object and each parameter.
* Give the new class a method named "compute."
* Copy the body of the original method into compute. Use the source object field for any
invocations of methods on the original object.
* Compile.
* Replace the old method with one that creates the new object and calls compute.

_This sample does not really needs this refactoring, but shows the way to do it._
```java

	Class Account
		int gamma (int inputVal, int quantity, int yearToDate) {
			int importantValue1 = (inputVal * quantity) + delta();
			int importantValue2 = (inputVal * yearToDate) + 100;
			if ((yearToDate - importantValue1) > 100)
			importantValue2 -= 20;
			int importantValue3 = importantValue2 * 7;
			// and so on.
			return importantValue3 - 2 * importantValue1;
		}
	}	
```
to
```java

	class Gamma...
		private final Account _account;
		private int inputVal;
		private int quantity;
		private int yearToDate;
		private int importantValue1;
		private int importantValue2;
		private int importantValue3;

		Gamma (Account source, int inputValArg, int quantityArg, int yearToDateArg) {
			_account = source;
			inputVal = inputValArg;
			quantity = quantityArg;
			yearToDate = yearToDateArg;
		}

		int compute () {
			importantValue1 = (inputVal * quantity) + _account.delta();
			importantValue2 = (inputVal * yearToDate) + 100;
			if ((yearToDate - importantValue1) > 100)
			importantValue2 -= 20;
			int importantValue3 = importantValue2 * 7;
			// and so on.
			return importantValue3 - 2 * importantValue1;
		}

		int gamma (int inputVal, int quantity, int yearToDate) {
			return new Gamma(this, inputVal, quantity,yearToDate).compute();
		}
```

##9 Substitute Algorithm
You want to replace an algorithm with one that is clearer.

```java

	String foundPerson(String[] people){
		for (int i = 0; i < people.length; i++) {
			if (people[i].equals ("Don")){
				return "Don";
			}
			if (people[i].equals ("John")){
				return "John";
			}
			if (people[i].equals ("Kent")){
				return "Kent";
			}
		}
		return "";
	}
```
to
```java
	
	String foundPerson(String[] people){
		List candidates = Arrays.asList(new String[] {"Don", "John","Kent"});
	for (int i = 0; i<people.length; i++)
		if (candidates.contains(people[i]))
			return people[i];
	return "";
	}
```

**Motivation**
 
* Break down something complex into simpler pieces.
* Makes easier apply changes to the algorithm.
* Substituting a large, complex algorithm is very difficult; making it simple can make the substitution tractable.

**Mechanics**

* Prepare your alternative algorithm.
* Run the new algorithm against your tests until it passes.

#7 Moving features between elements

##10 Move method
A method is, or will be, using or used by more features of another class than the class on which it is defined.  

_Create a new method with a similar body in the class it uses most. Either turn the old method into a simple delegation, or remove it altogether._  

```java

	class Class1 {
		aMethod()
	}

	class Class2 {	}
```
to


```java

	class Class1 {	}

	class Class2 {
		aMethod()
	}
```

**Motivation**

When classes do to much work or when collaborate too much and are highly coupled

**Mechanics**

* Examine all features used by the source method that are defined ont he source class. Consider whether they also should be moved
* Check the sub and super-classes for other declarations
*  Declare the method in the target class
* Copy the code to the target. Adapt it to work.
* Compile
* Determine how to reference the target from the source
* Turn the source method to a delegating one
* Compile and test
* Decide to remove or keep as delegate the source method
* If removed, replace all references to the target method
* Compile and test

#11 Move field 
A field is, or will be, used by another class more than the class on which it is defined.
_Create a new field in the target class, and change all its users._

```java

	class Class1 {
		aField
	}

	class Class2 {	}
```
to


```java

	class Class1 {	}

	class Class2 {
		aField
	}
```

**Motivation**

If a field is used mostly by outer classes and before [12 Extract Class]() 

**Mechanics**

* If the field is public, use [X Encapsulate Field]()
* Compile and test
* Create a field in the target (with accessors)
* Compile and test
* Determine how to reference it from the source
* Remove it on the source
* Replace all references on the source with the ones on the target
* Compile and test

#12 Extract Class
You have one class doing work that should be done by two.
_Create a new class and move the relevant fields and methods from the old class into the new class._
```java

	class Person {
		name,
		officeAreaCode,
		officeNumber,
		getTelephoneNumber()
	}
```
to
```java

	class Person {
		name,
		getTelephoneNumber()
	}

	class TelephoneNumber {
		areaCode,
		number,
		getTelephoneNumber()
	}
```

**Motivation**

Classes grow.
_Split them when_:

* subsets of methods seem to be together
* subsets of data usually change together or depend on each other

**Mechanics**

* Decide how to split the responsibilities of the class.
* Create a new class to express the split-off responsibilities.
* Make a link from the old to the new class.
* Use [11 Move Field]() on each field you wish to move.
* Compile and test.
* Use [10 Move Method]() to move methods over from old to new. Start with lower-level methods.
* Compile and test.
* Review and reduce the interfaces of each class.
* Decide whether to expose the new class. If you do expose the class, decide whether to expose it as a reference object or as an immutable value object.

#13 Inline Class
A class isn't doing very much.
_Move all its features into another class and delete it._
```java

	class Person {
		name,
		getTelephoneNumber()
	}

	class TelephoneNumber {
		areaCode,
		number,
		getTelephoneNumber()
	}
```
to
```java

	class Person {
		name,
		officeAreaCode,
		officeNumber,
		getTelephoneNumber()
	}
```


**Motivation**

After refactoring normally there are a bunch of responsibilities moved out of the class, letting the class with little left.

**Mechanics**

* Declare the public protocol of the source class onto the absorbing class. Delegate all these methods to the source class.
	* _If a separate interface makes sense for the source class methods, use Extract Interface before inlining._
* Change all references from the source class to the absorbing class.
* Compile and test.
* Use [10 Move Method]() and [11 Move Field]() to move features from the source class to the absorbing class.
* Hold a short, simple funeral service.

#14 Hide Delegate
A client is calling a delegate class of an object.  
_Create methods on the server to hide the delegate._
```java

	class ClientClass {
		//Dependencies
		Person person = new Person() 
		Department department = new Department()
		person.doSomething()
		department.doSomething()
	}
```
to
```java

	class ClientClass {
		Person person = new Person()
		person.doSomething()
	}

	class Person{
		Department department = new Department()
		department.doSomething()
	}
```

_The solution_
```java

	class ClientClass{
		Server server = new Server()
		server.doSomething()
	}

	class Server{
		Delegate delegate = new Delegate()
		void doSomething(){
			delegate.doSomething()	
		}
	}
	// The delegate is hidden to the client class.
	// Changes won't be propagated to the client they will only affect the server
	class Delegate{ 
		void doSomething(){...}
	}

```

**Motivation**

Key of encapsulation. 
Classes should know as less as possible of other classes.

**Mechanics**

* For each method on the delegate, create a simple delegating method on the server.
* Adjust the client to call the server.
	* If the client is not in the same package as the server, consider changing the delegate method's access to package visibility.
* Compile and test
* If no client needs to access the delegate anymore, remove the server's accessor for the delegate.
* Compile and test.

`> manager = john.getDepartment().getManager();`
```java



	class Person {
		Department _department;
		public Department getDepartment() {
			return _department;
		}
		public void setDepartment(Department arg) {
			_department = arg;
			}
		}

	class Department {
		private String _chargeCode;
		private Person _manager;
		public Department (Person manager) {
			_manager = manager;
		}
		public Person getManager() {
			return _manager;
		}
		...
```
to

`> manager = john.getManager();`
```java
	
	class Person {
		...
		public Person getManager() {
			return _department.getManager();
		}
	}
```

#15 Remove Middle Man
A class is doing too much simple delegation.
_Get the client to call the delegate directly._
```java

	class ClientClass {
		Person person = new Person()
		person.doSomething()
	}

	class Person{
		Department department = new Department()
		department.doSomething()
	}
```
to

```java

	class ClientClass {
		//Dependencies
		Person person = new Person() 
		Department department = new Department()
		person.doSomething()
		department.doSomething()
	}
```


**Motivation**

When the "Middle man" (the server) does to much is time for the client to call the delegate directly.

**Mechanics**

* Create an accessor for the delegate.
* For each client use of a delegate method, remove the method from the server and replace the call in the client to call method on the delegate.
* Compile and test

#16 Introduce Foreign Method
A server class you are using needs an additional method, but you can't modify the class.  
_Create a method in the client class with an instance of the server class as its first argument._

```java
	
	Date newStart = new Date(previousEnd.getYear(),previousEnd.getMonth(),previousEnd.getDate()+1);
```
to
```java
	
	Date newStart = nextDay(previousEnd);

	private static Date nextDay(Date date){
		return new Date(date.getYear(),date.getMonth(),date.getDate()+1);
	}
```

**Motivation**

When there is a lack of a method in class that you use a lot and you can not change that class.

**Mechanics**

* Create a method in the client class that does what you need.
	* The method should not access any of the features of the client class. If it needs a value, send it in as a parameter.
* Make an instance of the server class the first parameter.
* Comment the method as "foreign method; should be in server."

#17 Introduce Local Extension
A server class you are using needs several additional methods, but you can't modify the class. 
_Create a new class that contains these extra methods. Make this extension class a subclass or a wrapper of the original._
```java

	class ClientClass(){
		
		Date date = new Date()
		nextDate = nextDay(date);

		private static Date nextDay(Date date){
			return new Date(date.getYear(),date.getMonth(),date.getDate()+1);
		}
	}
```
to
```java
	
	class ClientClass() {
		MfDate date = new MfDate()
		nextDate = nextDate(date)
	}
	class MfDate() extends Date {
		...
		private static Date nextDay(Date date){
			return new Date(date.getYear(),date.getMonth(),date.getDate()+1);
		}
	}
```

**Motivation**

When plenty of [foreign methods]() need to be added to a class.

**Mechanics**

* Create an extension class either as a subclass or a wrapper of the original.
* Add converting constructors to the extension.
	* A constructor takes the original as an argument. 
	* The subclass version calls an appropriate superclass constructor 
	* The wrapper version sets the delegate field to the argument.
* Add new features to the extension.
* Replace the original with the extension where needed.
* Move any foreign methods defined for this class onto the extension.

#X 
```java
```

to


```java
```

**Motivation**

**Mechanics**
