---
layout: post
title: "c++11 functional"
date: 2013-04-24 17:15
comments: true
categories: C++11
---

##c++11 functional

###std::function
  _function_ is a template. As with other templates we've used, we must specify return type and argument types  when we create a _function_ type.
e.g. function<int(int, int)>

* __C++98 Function pointers and member function pointers__
  1. Exact parameter/return types and ex. specs. must be specified.  
  2. Can’t point to nonstatic member functions.  
  3. Can’t point to function objects.  
* __boost::function __
  1. Using for function, member functions and function objects.  
  2. Useful for callback function.
* __C++11 std::function__
  1. Using for callable entities that can be called like a function.
     * _Functions, function points, function references._  
     * _Object  implicitly convertible to one of those._  
     * _Function object._  
  2. Useful to be able to refer to any callable entity compatible with a given calling interface.  

_example:_
{% codeblock lang:objc %}
#include <functional>

int add(int i, int j)
{
  return i +j;
}

int (*addPoint)(int, int) = add;

auto mod = [](int i, int j) {return i % j;};

class Div {
public:
  int operator () (int i, int j) {
    return i / j;
  }
};

class Operation{
public:
   int minus(int num1, int num2) { return num1 - num2; }
   //int information = 99;
};
int main() {
std::function<int(int, int)> f1 = add;                          //user-defined function
std::function<int(int, int)> f2 = addPoint;                     //function pointer

std::function<int(int, int)> f3 = Div();                         //user-defined class object with overloadding the fuction-call operator.
std::function<int(int, int)> f4 = std::minus<int>();             //library function object

std::function<int(int, int)> f5 = [](int i, int j){return i*j;}; //unamed lambda
std::function<int(int, int)> f6 = mod;                           //named lambda object

Operation operation;
std::function<int(int, int)> f7 = std::bind(&Operation::minus,
                                            operation,
                                            std::placeholders::_1,
                                            std::placeholders::_2);//a call of std::bind
std::function<int(Operation&, int, int)> f8 =  &Operation::minus;  //member function
std::function<int(Operation*, int, int)> f9 =  &Operation::minus;  //member function
}
std::cerr<<f1(4,2)<<std::endl;
std::cerr<<f2(4,2)<<std::endl;
std::cerr<<f3(4,2)<<std::endl;
std::cerr<<f4(4,2)<<std::endl;
std::cerr<<f5(4,2)<<std::endl;
std::cerr<<f6(4,2)<<std::endl;
std::cerr<<f7(4,2)<<std::endl;
std::cerr<<f8(operation,4,2)<<std::endl;
std::cerr<<f9(&operation,4,2)<<std::endl;
{% endcodeblock %}

  std::bad_function_call is the type of the exception thrown by std::function::operator() if the function wrapper has no target.

_example:_
{% codeblock lang:objc %}
std::function<int()> f = nullptr;
try {
    f();
} catch(const std::bad_function_call& e) {
    std::cout << e.what() << '\n';
}
{% endcodeblock %}

_Reference:_ [std::functionn]( http://en.cppreference.com/w/cpp/utility/functional/function)

###std::mem_fn

   To use _function_, we must supply the call signature of the member we want to call. We can, instead, let the compiler deduce the member's type
by using another library facility,*mem_fn*, whick like _function_, is defined in the _functional header_.Like _function_, *mem_fn* generates a callable
object from a pointer to member. Unlike _function_, *mem_fn* will deduce the type of callable from the type of the pointer to member.


* __C++98 std::mem_fun/mem_fun_ref__  
  1. Only can deal with member functions with one or no argument.
  2. You should pick between both depending on which you want to deal with pointer or reference for class object.
  3. Do not support smart pointer.
* __boost::mem_fn __  
  _boost::mem_fn_ is ageneralization of the _std::mem_fun/mem_fun_ref_. It support member function pointers with more than one argument, and support 
smart poiter.
* __C++11 std::mem_fn __    
 Similar with _boost::mem_fn_. The name is quite confusing!(mem_fun Vs mem_fn) 

{% codeblock lang:objc %}
Foo foo;
foo.data = 100;

// member function without parameter
auto f1 = std::mem_fn(&Foo::displayGreeting);
f1(foo);

// member function with parameter
auto f2 = std::mem_fn(&Foo::displayNumber);
f2(foo, 55);

// member varibale
auto data = std::mem_fn(&Foo::data);
std::cout << "data: " << data(foo) << '\n';

// member function that return int
auto f3 = std::mem_fn(&Foo::dataValue);
auto f4 = std::mem_fn<int()>(&Foo::dataValue);

std::cout << "data: " << f3(foo) << '\n';
std::cout << "data: " << f4(foo) << '\n';

{% endcodeblock %}

_Reference:_ [std::mem_fn](http://en.cppreference.com/w/cpp/utility/functional/mem_fn)

###std::bind

   The bind can be thought of as a general-purpose function adaptor. It takes a callable object and generates a new callable that "adapts" the
parameter list of the original object. The general form of a call to bind is:
    auto _newCallable_ = bind(_Callable_, _arg_list_);

* __C++98 std::bind1st/std::bind2nd__  
   bind1st(op, value) _op(value, param)_ and bind2nd(op, value) _op(param, value)_. _op_ is a binary functor, and _bind1st/bind2nd_ actually make the
binary functor to unary functor. _bind1st_ bind the fist parameter and _bind2nd_ bind the sencond one. Not flexible!
   1. Bind only first or second arguments.
   2. Bind only one argument at a time.
   3. Can't bind functions with reference parameters.
   4. Require adaptable function objects. i.e._ptr_fun_, _mem_fun_, and mem_fun_ref_.
* __boost::bind__  
   _boost:::bind_ is generalization of the c++98 _std::bin1st/std::bind2nd_. It supports arbitrary function objects, function, function pointer, and 
member function pointers, and is able to bind any argument to a specific value or route input arguments into arbitrary positions. _bind_ does not place
any requirements on the function object; in particular. Mostly it can [bind 9 parameters](http://www.boost.org/doc/libs/1_49_0/boost/bind/placeholders.hpp).
* __c+=11 std::bind__  
    Similar with the _boost::bind_.   
    1. _functionObject_ std::bind(_callableEntity, 1stArgBinding, 2ndArgBinding... nthArgBinding);  [There's no limit for binding parameters' number](http://en.cppreference.com/w/cpp/utility/functional/placeholders).
    2. _1 is in namespace std::placeholders.

_example:_
{% codeblock lang:objc %}

std::string lily = "Lily";

// like the lily and 160.6 is pass by value
// like std::placeholders::_1 pass by reference
std::bind<int>(info, std::placeholders::_1, 18, std::placeholders::_2, std::placeholders::_2)(lily, 160.6);

auto f1 = std::bind(info, std::placeholders::_1, 18, std::placeholders::_2, std::placeholders::_2);
f1(lily, 160.6);

// nested bind
auto f2 = std::bind(high, std::placeholders::_1)(160.6);
std::bind(info, lily,  std::placeholders::_1, f2, std::placeholders::_2)(18, 160.6);

// nested bind subexpressions share the placeholders
std::bind(info, lily,  std::placeholders::_1, std::placeholders::_2, std::bind(high, std::placeholders::_2))(18, 160.6);

//bind to a member function object
Foo foo;
auto f3 = std::bind(&Foo::displayNumber, foo, std::placeholders::_1);

// bind pointer
Foo* pfoo;
auto f4 = std::bind(&Foo::displayNumber, pfoo, std::placeholders::_1);

// bind smart pointer
std::shared_ptr<Foo> spfoo;
auto f5 = std::bind(&Foo::displayNumber, spfoo, std::placeholders::_1);

// bind uniqu pointer.(std::unique_ptr must be wrapped by std::ref when bound,
//                    because std::unique_ptr isn’t copyable.)
std::unique_ptr<Foo> upfoo;
auto f6 = std::bind(&Foo::displayNumber, std::ref(upfoo), std::placeholders::_1);

// bind member varibale
foo.data = 77;
auto f7 = std::bind(&Foo::data, std::placeholders::_1);

f3(100);
f4(100);
f5(100);
f6(100);
std::cout<< f7(foo) << std::endl;
{% endcodeblock %}

