# Introduction

The idea of this article is to introduce some C++ design patterns using finance concepts. We assume that the reader has knowledge about finance derivatives (bonds, stocks, options, trading strategies), experience developing in C++ and the reader wants to improve his/her designing skills.

This article will contain a lot of questions and answers that will guide the reader to understand how we think when we are dealing with object programming.

# Object oriented programming paradigm

Object oriented programming is a programming paradigm where we design our software through objects whose interact between themselves, define¡ing functionality and providing the information that we need in our application.

An object is an entity which contains data and functions. This data is contained in fields usually called attributes and the functions usually called methods. We use the object's fields to define the state of each object and the methods to define what the object can do.

The objects particularly are instances of classes, we define the fields and methods that the objects will have defining a class, when we instance this class and that particular instance (object) will be able to call any method defined in that class and this object will have also all the fields defined there.

Let us create a pretty simple example, a class that represents market stocks:

```python

class Stock:
    # fields
    string name; 
    float closeValue
    float ask
    float bid
    float variance
    long timestamp

    # methods
    float getSpread():
        return bid - ask
        
```

I hope that with this simple example you get an idea about what is an object. In the following sections we will see more sophisticated classes and design patterns those will help as to solve the designing problems that we meet when we are programming.

# Defining our first classes

We will start defining simple classes that represent stocks and vanilla put/call options. Let us define a base class called Instrument which contains all the shared attributes for any kind of instrument:

```C++
// Instrument.h
#include <string>

class Instrument {

public:
	Instrument(const std::string &name, const float &closeValue, const float &ask, const float &bid,
			   const float &variance, const long int &timestamp);

	virtual ~Instrument();
	virtual float getSpread();

	float getAsk() const;
	void setAsk(float ask);

	float getBid() const;
	void setBid(float bid);

	float getCloseValue() const;
	void setCloseValue(float closeValue);

	long int getTimestamp() const;
	void setTimestamp(const long int &timestamp);

	const std::string& getName() const;
	void setName(const std::string& name);

	float getVariance() const;
	void setVariance(float variance);

protected:
    std::string _name;
    float _closeValue;
    float _ask;
    float _bid;
    float _variance;
    long int _timestamp;
};


// Instrument.cpp

Instrument::Instrument(const std::string &name, const float &closeValue, const float &ask, const float &bid,
					const float &variance, const long int &timestamp) : _name(name), _closeValue(closeValue),
				_ask(ask), _bid(bid), _variance(variance), _timestamp(timestamp) {

}

float Instrument::getSpread() {
	return this->_bid - this->_ask;
}

// getters and setters
```

Now, we will implement the class Stock which inherits from Instrument.

```C++

// Stock.h
#include "Instrument.h"

class Stock: public Instrument {
public:
	Stock(const std::string &name, const float &closeValue, const float &ask, const float &bid,
				   const float &variance, const long int &timestamp);
};

// Stock.cpp
Stock::Stock(const std::string &name, const float &closeValue, const float &ask, const float &bid,
		   const float &variance, const long int &timestamp): Instrument(name, closeValue, ask, bid, variance, timestamp) {

}

```

As we can see the class Stock has the same fields and attributes as its parent class (Instrument), in this kind of cases we have to ask to ourselves if it worths to have this new class instead of just having the class Instrument to represent the stocks, this question will be answered later.

Now, we will implement a base option class which inherits from Instrument and contains the extra fields strike, stock and maturity related with vanilla options.

It also will contain five new methods (isATM, isITM, isOTM, getIntrinsicValue, canExercise) that will be implemented in each child class.

```C++
#include "Instrument.h"
#include "Stock.h"

class Option: public Instrument {
public:
	Option(const float strike, const Stock*& stock, const long int& maturity, const std::string &name, const float &closeValue, const float &ask, const float &bid,
			   const float &variance, const long int &timestamp);

	virtual bool isATM() const {
		return this->_strike == this->_stock->getAsk();
	}
	
	// american options
	virtual bool canExercise(const long int &currentTime) {
		return currentTime < this->_maturity;
	}
	
	virtual bool isOTM() const {
		return !this->isITM() && !this->isATM();
	}
	
	virtual bool isITM() const = 0;
	virtual bool getIntrinsicValue() const = 0;
	
	// new class's getters y setters
protected:
	float _strike;
	Stock* _stock;
	long int _maturity;
};
```

Let me stop here for a moment a let me tell you some basic criterias that I have considered to define these classes:

1. Do we need to have the class Stock?
No, we don't, but having the Stock class will allow us in the future the possibility to add stock methods or attributes that I couldn't see at the beginning.

In this case, the BaseOption class has a Stock pointer instead of having a pointer to an Instrument, the client will have a restriccion and only will be able to create stock´s options (let us imagine that we want it in that way), adding this limitation without the Stock class wouldn´t be impossible. My point here is "we don´t have to abuse and create hundries of classes, but eventually adding classes could help us to keep our code clean and safe)". Anytime that you are defining your classes take a time to think if you really need that class.

2. Do we need to have a base option class or I can use a simple property that determines if the option is a put or call?

In this first touch, we are just implementing vanilla put and call options, so, we can add a class attribute that tells if the instance is a put or a call, and then adding some case logic we can implement each option´s method.
For example, the method __isITM__ could be implemented in the following way:

```C++
	
	// isCall : true when the option is a call option, false when the option is a put option.
	bool isITM() const {
		bool result = this->stock->getAsk() > this->_strike;
		
		if (!this->isCall) {
			result = !result;
		}
		
		return result;
	}

```

But then if we add new option classes (barriers for example) we will have to reimplement our base methods, and even worst
we may need to change our interfaces to provide to the user new constructors for the new kind of options creating a mess in both sides of the project (client/server). The same problem occurs when we are implementing the method __canExercise__ and we have to deal with different types of options such us american, europeans, bermudians, etc..

Let's finish our first classes. Now, using this base option class we inherit to have the put option and the call option classes.

```C++
// CallOption.h
#include "Instrument.h"
#include "Stock.h"

class CallOption: public Option {
public:
	// we just need to implement these methods.
	virtual bool isITM() const;
	virtual bool getIntrinsicValue() const;
}

// CallOption.cpp

bool CallOption::isITM() {
	return this->_stock->getAsk() > this->_strike;	
}

bool CallOption::getIntrinsicValue() {
	float result = 0.0;
	
	if (this->isITM()) {
		result = this->_stock->getAsk() - this->_strike;
	}
	
	return result;
}
```

Implementing the class *PutOption* should be easy using these classes, I will not add that class here, but think just for a moment and the solution will come to your mind =).

**Problems again** with this particular design we will have problems to deals with different combinations, for example, here we has created a __BaseOption__ class that works as an american option, and using inheritance we defined our call and put options. But if we want to create an european call/put option using this design we will have to add at least two new classes where we overrided the method __canExcercise__ and have the same logic to calculate if that option is ITM, therefore, our work here is not done, we have to come with a better design and this is the idea of this article: get a design, find the design´s problems, think how we can solve it and redesign it. In the following sections we will see specific problems and we will try to find the best design to that particular problem.

## A better design for options

The problem that we have found in our option class implementation is that we will need to have a lot of different classes with replicated code to solve the behavior of the different option types. If we want to have european put and call options we will need to extend again from **BaseOption**, override the method __canExcersice__, and then extend again to implement a class for call options, and another class for put options. We are replicating a lot of code, we are having a class for each combination that we need, these are guidelines that show us that we didn´t really choose a good design. We will use the delegation pattern to solve this issue.

### Delegation pattern

In this pattern the face object delegates some tasks to its associated objects. It delegates the responsibility to perform that particular task to some internal object that knows how to performs it. Let me show you a pretty simple example:

```C++
class Foo {
public:
	void m1() {
		// m1 implementation
	}
};

class Bar {
public:
	void m1() {
		// Bar delegates the task to Foo
		this->_foo.m1();
	}
private:
	Foo _foo;
};
```

### Strategy Pattern

Strategy pattern allows us to have different behaviors selected at runtime. The easiest way to understand it is with a simple example. We have a class Person and an unsorted list of people (Person´s objects) and we want to sort this list by different attributes (name, age, lastname, etc). We can define a base class where we define our strategy interface, and then implement each child comparator:

```C++
class Person {
public:
	// getters and setters
private:
	int _age;
	std::string _name;
};

class ComparatorStrategy {
public:
	int compare(const Person &p1, const Person &p2) const = 0;
};

class ComparatorByNameStrategy: public ComparatorStrategy {
public:
	
	int compare(const Person &p1, const Person &p2) const {
		if (p1.getName() == p2.getName()) {
			return 0;
		} else {
			return p1.getName() > p2.getName()? 1: -1;
		}
	}
};
```

Then we decide which comparator we want to use, providing the comparator to the list´s sort method:

```C++
List<Person> people;

people.sort(new ComparatorByNameStrategy());
```

The sort method will use the passed comparator to compare the list´s objects. 

We have a lot of things to improve this implementation, for example, we don´t need to create a new instance of the comparator any time that we need to use it, but I just focused in showing you how this pattern works in the simplest way to understand the basic concept. From these lines of code, just get the idea about what resolves the Strategy Pattern and try to think how we will use it in our option implementations.

### The new option design

Now that we know these design patterns, we can use them to solve our combination issue. The first thing that we have to do is think how we can split our option class in different entities, in this case, the option objects will be the face objects those will delegate some methods to different strategies. 

Let us split the class Option into two new classes. We call __ExcerciseStrategy__ to the class that solves if an option could be exercised and __PayoffStrategy__ to the class that calculates the current payoff of an option.

These are the implementation of these base classes:

```C++
class ExcerciseStrategy {
public:
	virtual bool canExercise(const long int &currentTime) const = 0;

};

class PayoffStrategy {
public:
	virtual float getIntrinsicValue() const = 0;
};

```

And here we have some ExcerciseStrategy implementations:

```C++
class AmericanExcerciseStrategy {
public:
	AmericanExcerciseStrategy(const long int &maturity): _maturity(maturity) {}

	virtual bool canExercise(const long int &currentTime) const {
		return currentTime < this->_maturity;
	}
private:
	const long int _maturity;
};

class EuropeanExcerciseStrategy {
public:
	EuropeanExcerciseStrategy(const long int &maturity): _maturity(maturity) {}

	virtual bool canExercise(const long int &currentTime) const {
		return currentTime == this->_maturity;
	}
private:
	const long int _maturity;
};

class BermudaExcerciseStrategy {
public:
	EuropeanExcerciseStrategy(std::vector<long int> &maturities): _maturities(maturities) {}

	virtual bool canExercise(const long int &currentTime) const {
		// return true if currentTime is contained in _maturities
	}
private:
	std::vector<long int> _maturities;
};
```

Now, some payoff implementations:

```C++
class CallPayoffStrategy {
public:
	CallPayoffStrategy(const Stock* &stock, const float &strike): _stock(stock),
																  _strike(strike) {}
	virtual float getIntrinsicValue() const {
		return this->_stock->getAsk() - this->_strike;
	}
private: 
	Stock* _stock;
	float _strike;
};


class PutPayoffStrategy {
public:
	PutPayoffStrategy(const Stock* &stock, const float &strike): _stock(stock),
																  _strike(strike) {}
	virtual float getIntrinsicValue() const {
		return this->_strike - this->_stock->getAsk();
	}
private: 
	Stock* _stock;
	float _strike;
};
```

Now, using these internal classes we created a compounend option class:

```C++
class Option: public Instrument {
public:
	Option(const ExcerciseStrategy* &excerciseStrategy, const PayoffStrategy* &payoffStrategy, const float strike, const Stock*& stock, const long int& maturity, const std::string &name, const float &closeValue, const float &ask, const float &bid,
			   const float &variance, const long int &timestamp);

	virtual bool isATM() const {
		// I am comparing float points, don´t worry is just an example.
		return this->getIntrinsicValue() == 0.0;
	}

	virtual bool isITM() const {
		return this->getIntrinsicValue() > 0;
	}

	virtual bool isOTM() const {
		return !this->isITM() && !this->isATM();
	}

	virtual bool getIntrinsicValue() const {
		// delegation part
		return this->_payoffStrategy->getIntrinsicValue();
	}

	virtual bool canExercise(const long int &currentTime) const {
		// delegation part
		return this->_excerciseStrategy->canExercise(currentTime);
	}

protected:
	float _strike;
	Stock* _stock;
	long int _maturity;
	ExcerciseStrategy* _excerciseStrategy;
	PayoffStrategy* _payoffStrategy;
};
```

And of course, a simple piece of code showing the usage:

```C++
// create an american put option
	Option option(new AmericanExcerciseStrategy(maturity), PutPayoffStrategy(stock, strike), strike, stock, maturity, variance, timestamp);
	
	// create an european call option
	Option option(new EuropeanExcerciseStrategy(maturity), CallPayoffStrategy(stock, strike), strike, stock, maturity, variance, timestamp);
```

#### what is worse what is better?

Please, try to understand that I am showing you some design patterns, but that doesn´t mean that you have to overdesign all
your classes. Now, we have gained extensibility and we have solved the combination issue, but our new option class is more complex and that is not always better. We can also see that creating options is more complicated as well (we will see creational patterns, don´t worry too much), therefore, we have to be extra careful when we are designing, even if we are using design patterns.
