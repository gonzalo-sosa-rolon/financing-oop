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
For example, the method _isITM_ could be implemented in the following way:

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
we may need to change our interfaces to provide to the user new constructors for the new kind of options creating a mess in both sides of the project (client/server). The same problem occurs when we are implementing the method _canExercise_ and we have to deal with different types of options such us american, europeans, bermudians, etc..

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

*Problems again* with this particular design we will have problems to deals with different combinations, for example, here we has created a _BaseOption_ class that works as an american option, and using inheritance we defined our call and put options. But if we want to create an european call/put option using this design we will have to add at least two new classes where we overrided the method _canExcercise_ and have the same logic to calculate if that option is ITM, therefore, our work here is not done, we have to come with a better design and this is the idea of this article: get a design, find the design´s problems, think how we can solve it and redesign it. In the following sections we will see specific problems and we will try to find the best to design to that particular problem.
