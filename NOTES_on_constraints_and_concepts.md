Notes on constraints and concepts (_since C++20_)

_Class templates_, _function templates_, and _non-template functions_ (typically members of class
templates) may be associated with a _constraint_, which specifies the requirements on template
arguments, which can be used to select the most appropriate function overloads and template
specializations.

Named sets of such requirements are called _concepts_. Each concept is a predicate, evaluated at
compile time, and becomes a part of the interface of a template where it is used as a constraint:


```cpp
#include <string>
#include <cstddef>
#include <concepts>

// Declaration of the concept "Hashable", which is satisfied by any type 'T' such that for values
// 'a' of type 'T', the expression std::hash<T>{}(a) compiles and its result is convertible
// to std::size_t
template<typename T>
concept Hashable = requires(T a) {
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};

struct meow {};
void f(T) {}
// Alternative ways to apply the same constraint:
// template<typename T>
//      requires Hashable<T>
// void f(T) {}
// 
// template<typename T>
// void f(T) requires Hashable<T> {}

int main() {
  using std::operator""s;
  f("abc"s);    // OK, std::string satisifes Hashable
  // f(meow{}); // Error: meow does not satisfy Hashable
}
```

Violations of constraints are detected at compile time, early in the template instantiation
process, which leads to easy to follow error messages:

```cpp
std::list<int> l = {3, -1, 10};
std::sort(l.begin(), l.end());
// Typical compiler diagnostic without concepts:
//   invalid operands to binary expression ('std::_List_iterator<int>' and 
//    'std::_List_iterator<int>')
//                             std::__lg(__last - __first) * 2);
//                                       ~~~~~~ ^ ~~~~~~~
//  ... 50 lines of output ...
//
// Typical compiler diagnostic with concepts:
//   error: cannot call std::sort with std::_List_iterator<int>
//   note: concept RandomAccessIterator<std::_List_iterator<int>> was not satisified
```

The intent of concepts is to model semantic categories (Number, Range, RegularFunction) rather than 
syntactic restrictions (HasPlus, Array). 

Concepts

A concept is a named set of requirements. The definition of a concept must appear at namespace scope.
The definition of a concept has the form

**template <** _template-paramter-list_ **>**
**concept** _concept-name_ **=** _constraint-expression_**;**

```cpp
// concept
template <class T, class U>
concept Derived = std::is_base_of<U, T>::value;
```

Concepts cannot recursively refer to themselves and cannot be constrained:

```cpp
template <typename T>
concept V = V<T*>; // error: recursive concept

template<class T> concept C1 = true;
template<C1 T>
concept Error1 = true; // Error: C1 T attempts to constrain a concept definition
template<class T> requires C1<T>
concept Error2 = true; // Error: the requires-clause attempts to constrain a concept
```


