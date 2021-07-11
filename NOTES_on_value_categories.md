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

* _xvalue_ (an "eXpriing" value) is a glvalue that denotes an object or bit-field whose resources
  can be reused;

* an _lvalue_ (so-called, historically, because lvalues could appear on the left-hand side of an 
  assignment expression) is a glvalue that is not an xvalue;

* an _rvalue_ (so-called, historically, because rvalues could appear on the right-hand side of an 
  assignment expression) is a prvalue or an xvalue.


Primary categories

**lvalue**

The following expressions are _lvalue expressions_:

* the name of a variable, a function, a template parameter object (_since C++20_), or a data
  member, regardless of type, such as ```std::cin``` or ```std::endl```. Even if the variable's
  type is rvalue reference, the expression consisting of its name is an lvalue expression;

* a function call or an overloaded operator expression, whose return type is lvalue reference,
  such as ```std::getline(std::cin, str)```, ```std::cout << 1```, ```str1 = str2```, or ```++it```;

* ```a = b```, ```a += b```, ```a %= b```, and all other built-in _assignment_ and _compound assignment_
  expressions;

* ```++a``` and ```--a```, the built-in _pre-increment_ and _pre-decrement_ expressions;

* ```*p```, the built-in _indirection_ expression;

* ```a[n]``` and ```p[n]```, the built-in _subscript_ expressions, where one operand in ```a[n]``` is an
array lvalue (_since C++11_);

* ```a.m```, the _member of object_ expression, except where ```m``` is a member enumerator or
  non-static member function, or where ```a``` is an rvalue and ```m``` is a non-static data member
  of object type

* ```p->m```, the built-in _member of pointer_ expression, except where ```m``` is a member enumerator or
  a non-static member function;

* ```a.*mp```, the _pointer to member of object_ expression, where ```a``` is an lvalue and ```mp``` is a 
  pointer to data member;

* ```p->*mp```, the built-in _pointer to member of pointer_ expression, where ```mp``` is a pointer to data
  member;

* ```a, b```, the built-in _comma_ expression, where ```b``` is an lvalue;

* ```a ? b : c```, the _ternary conditional_ expression for certain ```b``` and ```c``` (e.g. when both are
  lvalues of the same type, but see definition for detail);

* a _string literal_, such as ```"Hello World!"```;

* a cast expression to lvalue reference type, such as ```static_cast<int&>(x)```;

* (_since C++11_) a function call or an overloaded operator expression, whose return type is rvalue reference
  to function;

* (_since C++11_) a cast expression to rvalue reference to function type, such as 
  ```static_cast<void (&&)(int)>(x)```. 


Properties:

* same as _glvalue_ (see below):
   * A _glvalue_ may be implicitly converted to a _prvalue_ with lvalue-to-rvalue, array-to-pointer, or 
     function-to-pointer implicit conversion
   * A _glvalue_ may be _polymorphic_: the _dynamic type_ of the object it identifies is not necessarily
     the static type of the expression
   * A _glvalue_ can have incomplete type, where permitted by the expression

* address of an lvalue may be taken by built-in address of operator: ```&++i``` and ```&std::endl```
  are valid expressions

* a modifiable lvalue may be used as the left-handed operand of the built-in assignment and compound
  assignment operators

* an lvalue may be used to initialize an lvalue reference; this associates a new name with the object 
  identified by the expression


**prvalue**

The following expressions are _prvalue expressions_:

* a literal (except for string literal), such as ```42```, ```true```, or ```nullptr```;

* a function call or an overloaded operator expression, whose return type is non-reference,
  such as ```str.substr(1, 2)```, ```str1 + str2```, or ```it++```;

* ```a++``` and ```a--```, the built-in _post-increment_ and _post-decrement_ expressions;

* ```a + b```, ```a % b```, ```a & b```, ```a << b```, and all other built-in arithmetic expressions;

* ```a && b```, ```a || b```, ```!a```, the built-in _logical_ expressions;

* ```a < b```, ```a == b```, ```a >= b```, and all other built-in _comparison_ expressions;

* ```&a```, the built-in _address-of_ expression;

* ```a.m```, the _member of object_ expression, where ```m``` is a member enumerator or a non-static
  member function, or where ```a``` is an rvalue and ```m``` is a non-static data member of non-
  reference type (_until C++11_);

* ```p->m```, the built-in _member of pointer_ expression, where ```m``` is a member enumerator or
  a non-static member function;

* ```a.*mp```, the _pointer to member of object_ expression, where ```mp``` is a pointer to member
  function, or where ```a``` is an rvalue and ```mp``` is a pointer to data member (_until C++11_);

* ```p->*mp```, the built-in _pointer to member pointer_ expression   

* ```a, b```, the built-in _comma_ expression, where ```b``` is an rvalue;

* ```a ? b : c```, the _ternary conditional_ expression for certain ```b``` and ```c```;

* a cast expression to non-reference type such as ```static_cast<double>(x)```, ```std::string```,
  or ```(int)42```;

* the ```this``` pointer;

* an _enumerator_;
 

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


Array to pointer conversion

An _lvalue_ or _rvalue_ of type "array of _N_ ```T```" or "array of unknown bound of ```T```" can be
implicitly converted to a prvalue of type "pointer to ```T```". If the array is a _prvalue_, _temporary
materialization_ occurs (_since C++17_). The resulting pointer refers to the first element of the array.  


Function to pointer


An _lvalue_ of function type T can be implicitly converted to a prvalue pointer to that function.
This does not apply to non-static member functions because lvalues that refer to non-static
member functions do not exist.  
