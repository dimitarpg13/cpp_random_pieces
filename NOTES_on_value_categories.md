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
   * initializes an object or a bit-field (such _prvalue_ is said to have a _result object_). 
