# Introduction

The idea of this article is to introduce some C++ design patterns using finance concepts. We assume that the reader has knowledge about finance derivatives (bonds, stocks, options, trading strategies) and experience developing in C++ and he want to improve its designing skills.

# Object oriented programming paradigm

Object oriented programming is a programming paradigm where we design our software through objects whose interact between themselves, define functionality and provide the information that we need in our application.

An object is an entity which contains data and functions. This data is contained in fields usually called attributes and the object functions are usually called methods. In other words, we use the fields to define the state of the object and the methods to define what the object can do.

The objects particularly are instances of classes, we define the fields and methods that the object will have defining a class, then we instance this class and that particular instance (object) will be able to call any method defined in the class and it will have memory to store each field defined in that class.

Let us create a pretty simple example, we want to create a class that represents Stocks.
We have to get used to think any entity that we will use as an object, a Stock´s object could be represented as the following class:

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


As we can imagine, in the Stock fields we will store the different properties of each object, and we will be able to calculate the spread of the each stock through the method getSpread.

Defining our first classes

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

As we can see the class Stock has the same fields and attributes as its parent class (Instrument), in this kind of cases we have to ask ourselves if it worths to have this new class instead of just using the class Instrument to represent stocks. 

This question will be answered later.

Now, we will implement a base option class which inherits from Instrument and contains the extra fields strike, stock and maturity.
It also will contain four new abstract methods (isATM, isITM, isOTM, getIntrinsicValue) that will be implemented in each child class.

```C++
#include "Instrument.h"
#include "Stock.h"

class Option: public Instrument {
public:
	Option(const float strike, const Stock*& stock, const long int& maturity, const std::string &name, const float &closeValue, const float &ask, const float &bid,
			   const float &variance, const long int &timestamp);

	virtual bool isATM() const = 0;
	virtual bool isITM() const = 0;
	virtual bool isOTM() const = 0;
	virtual bool getIntrinsicValue() const = 0;
	
	// new class getters y setters
protected:
	float _strike;
	Stock* _stock;
	long int _maturity;
};
```