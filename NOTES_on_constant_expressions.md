Constant expressions and ```constexpr``` specifier 

Defines an expression that can be evaluated at compile time.
Such expressions can be used as non-type template arguments, array sizes, and in 
other contexts that require constant expressions, e.g.

```cpp
int n = 1;
std::array<int, n> a1; // error: n is not a constant expression
const int cn = 2;
std::array<int, cn> a2; // OK: cn is a constant expression
```

Core constant expressions

A _core constant expression_ is any expression whose evaluation _would not_ evaluate any 
one of the following:

1. the ```this``` pointer, except in a ```constexpr``` function that is being evaluated
as part of the expression
2. a function call expression that calls a function (or a constructor) that is not 
declared ```constexpr```
```cpp
constexpr int n = std::numeric_limits<int>::max();  // OK: max() is constexpr
constexpr int m = std::time(nullptr);  // Error: std::time() is not constexpr
```
3. a function call to a ```constexpr``` function which is declared, but not defined
4. a function call to a ```constexpr``` function/constructor template instantiation 
where the instantiation fails to satisfy ```constexpr``` function/constructor requirements.
5. (_since C++20_) a  



```constexpr``` specifier (since C++ 11)

specifies that the value of a variable or function can appear in constant expressions

Explanation
The _constexpr_ specifier declares that it is possible to evaluate the value of the function
or variable at compile time. Such variables and functions can then be used where only compile 
time constant expressions are allowed (provided that appropriate function arguments are given).

A ```constexpr``` specifier used in an object declaration or non-static member function 
(_until C++ 14_) implies ```const```. A _constexpr_ specifier used in a function or static member
 variable (_since C++ 17_) declaration implies _inline_. If any declaration of a function or 
a function template has a ```constexpr``` specifier, then every declaration must contain that
specifier.

A ```constexpr``` variable must satisfy the following requirements:
* its type must be a literal type
* it must be immediatelly initialized
* the full-expression of its initialization, including all implicit conversions, constructor calls,
etc, must be a constant expression
* (_since C++ 20_) it must have constant destruction i.e. either:
   * it is not of class type nor (possibly multi-dimensional) array thereof, or
   * it is of class type or (possibly multi-dimenisional) array thereof, that class
     type has a constexpr destructor, and for a hypothetical expression ```e``` whose only effect
     is to destroy the object, ```e``` would be a core constant expression if the lifetime of the 
     object and its non-mutable subobjects (but not its mutable subobjects) were considered to 
     start within ```e```.
* (_since C++ 20_) if a ```constexpr``` variable is not translation-unit-local, it should not be initialized to point
  to, or refer to, or have a (possibly recursive) subobject that points to or referes to, a 
  translation-unit-local entity that is usable in constant expressions. Such initialization is 
  disallowed in a module interface unit (outside its private-module-fragment, if any) or a module
  partition, and is deprecated in any other context. 

A ```constepxr``` function must satisfy the following requirements:
* it must not be virtual (_until C++ 20_) 
* it must not be a coroutine (_since C++ 20_)
* its return type (if any) must be a literal type
* each of its parameters must be a literal type
* for constructor (and destructor _since C++ 20_), the class must have no virtual base classes
* there exists at least one set of argument values such that an invocation of the function could be an
evaluated subexpression of a core constant expression (for constructors, use in a constant initializer
is sufficient (_since C++ 14_). No diagnostic is required for a violation of this bullet. 
* its function body must not be a function-try-block (_until C++20__)  


_Function-try-block_

Establishes an exception handler around the body of a function


_Function-try-block_ Syntax

The _function-try-block_ is one of the alternative syntax forms for _function-body_, which is a part of
function  defintion.

**try** _ctor-initializer (optional)_ _compound-statement_ _handler-sequence_

_ctor-initializer_ - member initializer list, only allowed in constructors

_compound-statement_ - the brace-enclosed sequence of statements that constitues the body of a function

_handler-sequence_ - sequence of one or more catch-clauses

A _function-try-block_ associates a sequence of catch clauses with the entire function body, and with
the member initializer list (if used in a constructor) as well. Every exception thrown from any statement
in the function body, or (for constructors) from any member or base constructor, or (for destructors) from
any member or base destructor, transfer constrol to the _handler-sequence_ the same way an exception thrown
in a regular try block would. 


_Function-try-block_ example

```cpp
#include <string>
struct S {
    std::string m;
    S(const std::string& str, int idx) try : m(str, idx) {

    } catch (const std::exception& e) {

       std::cout << "S(" << str << ", " << idx << ") failed: " << e.what() << '\n';
    } // implicit "throw;" here
};

int main() {
   S s1{"ABC", 1}; // does not throw (index is in bounds)
   try {
      S s2{"ABC", 4}; // throws (out of bounds)
   } catch (std::exception& e) {
      std::cout << "S s2... raised an exception: " << e.what() << '\n';
   }
}
```
Before any catch clauses of a function-try-block on a constructor are entered, a 
fully-constructed members and bases have already been destroyed.


(_since C++ 11_) if the _function-try-block_ is on a delegating constructor
