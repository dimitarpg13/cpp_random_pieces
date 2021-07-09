Notes on Value categories


Each C++ expression (an operator with its operands, a literal, a variable name, etc) is characterized
by two independent properties: a type and a value category. Each expression has some non-reference 
type, and each expression belongs to exactly one of the three primary value categories:
_prvalue_, _xvalue_, and _lvalue_.

* _glvalue_ ("generalized" lvalue) is an expression whose evaluation determines the identity of an
object, bit-field or a fucntion;

* _prvalue_ ("pure" rvalue) is an expression whose evaluation either
   * computes the value of the operand of an operator or is a void expression (such prvalue has no 
     _result object_), or
   * initializes an object or a bit-field (such _prvalue_ is said to have a _result object_). With
     the exception of ```decltype```, all class and array _prvalue_s have a result object even if
     it is discarded. The result object may be a variable, an object created by _new-expression_, 
     a temporary created by temporary materialization or a member thereof;



Temporary materialization

A _prvalue_ of any complete type ```T``` can be converted to an _xvalue_ of the same type ```T```.
This conversion initializes a temporary object of type ```T``` from the prvalue by evaluating the 
_prvalue_ as its result object, and produces an _xvalue_ denoting the temporary object. If ```T```
is a class or array of class type, it must have accessible and non-deleted destructor.

```cpp
struct S { int m; };
int i = S().m;  // member access expects glvalue as of C++17;
                // S() prvalue is converted to xvalue
``` 

Temporary materialization occurs in the following situations:

* when binding a reference to a prvalue;
* when performing member access on class prvalue;
* when performing an array-to-pointer conversion or subscripting on an array prvalue;
* when initializing an object of type ```std::initializer_list<T>``` from _braced-init-list_;
* when ```typeid``` is applied to a _prvalue_ (this is part of an unevaluated expression);
* when ```sizeof``` is applied to a prvalue (this is part of an unevaluated expression);
* when a prvalue appears as a _discarded-value expression_

Note that temporary materialization does _not occur_ when initializing an object from a prvalue
of the same type (by _direct-initialization_ or _copy-initialization_): such object is initialized
directly from the initializer. This ensures "guaranteed copy elision".


Function to pointer


An _lvalue_ of function type T can be implicitly converted to a prvalue pointer to that function.
 
