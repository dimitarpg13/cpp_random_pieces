Notes on Constructors and member initializer lists

Constructor is a special non-static member function of a class that is used to initialize
objects of its class type. In the definition of a constructor of a class, _member initializer list_
specifies the initializers for direct and virtual bases and non-static data members. 
(Not to be confused with ```std::initializer_list```).

(_since C++ 20_) A constructor must not be a coroutine


Constructor Syntax

Constructors are declared using member function declarators of the following form:

  _class-name_ **(** _parameter-list_ (_optional_) **)** ```except-spec```(_optional_) attr(_optional_)

where _class-name_ must name the current class (or current instantiation of a class template), or, when
declared at namespace scope or in a friend declaration, it must be a qualified class name.

The only specifiers allowed in the _decl-specifier-seq_ of a constructor declaration are ```friend```,
```inline```, ```constexpr``` (_since C++ 11_), ```consteval``` (_since C++ 20_), and ```explicit```
(in particular, no return type is allowed). Note that _cv-_ and _ref-qualifiers_ are    



Non-static member functions


A non-static member function is a function that is declared in a member specification of a class without
a static or friend specifier. 

```cpp
class S {
    int mf1(); // non-static member function declaration
    void mf2() volatile, mf3() &&; // can be cv-qualified and reference-qualified
         // the declaration above is equivalent to two separate declarations
         // void mf2() volatile;
         // void mf3() &&;

    int mf4() const { return data; } // can be defined inline
    virtual void mf5() final; // can be virtual, can use final/override
    S() : data(12) {} // consructors are member functions too
    int data;    
};
int S::mf1() { return 7; ] // if not defined inline, has to be defined at namespace
```

Any function declarations are allowed, with additional syntax elements that are only available for
non-static member functions: ```final``` and ```override``` specifiers, pure specifiers, cv-qualifiers,
ref-qualifiers, and member initialization lists. 


A non-static member function of class X may be called:
1) for an object of type X using the class member access operator
2) for an object of a class derived from X
3) directly from within the body of a member function of X
4) directly from within the body of a member function of a class derived from X

Calling a member function of class X on an object of any other type invokes undefined behavior


Within the body of a non-static member function of X, any id-expression E (e.g. an idenitifier) that
resolves to a non-type non-static member of X or of a base class of X, is transformed to a member access
expression ```(*this).E``` (unless it is already a part of a member access expression). This does not 
occur in template definition context, so a name may have to be prefixed with ```this->``` explicitly
to become dependent.

```cpp
struct S {
   int n;
   void f();
};
void S::f() {
   n = 1; // transformed to (*this).n = 1
}
int main() {
   S s1, s2;
   s1.f(); // changes s1.n
}
```


```const```- and ```volatile```-qualified member functions


A non-static member function can be declared with a ```const```, ```volatile```, or ```const volatile```
qualifier (this qualifier appears after the parameter list in the function declaration). Differently
```cv```-qualfied functions have different types and so may overload each other.


In the body of ```cv```-qualified function, ```*this``` is ```cv```-qualified, e.g. in a ```const``` 
member function, only other ```const``` member functions may be called normally. (A non-```const```
member function may still be called if ```const_cast``` is applied or through an access path that
does not involve ```this```).

```cpp
#include <vector>
struct Array {
    std::vector<int> data;
    Array(int sz) : data(sz) {}
    // const member function
    int operator[](int idx) const {
                            // this has type const Array*
         return data[idx];  // transformed to (*this).data[idx];
    }
    // non-const member function
    int& operator[](int idx) {
                            // this has type Array*
         return data[idx];  // transformed to (*this).data[idx];
    }
};

int main()
{
   Array a(10);
   a[1] = 1;  // OK: the type of a[1] is int&
   const Array ca(10);
   ca[1] = 2; // Error: the type of ca[1] is int
}
```
