Notes on Constructors and member initializer lists

Constructor is a special non-static member function of a class that is used to initialize
objects of its class type. In the definition of a constructor of a class, _member initializer list_
specifies the initializers for direct and virtual bases and non-static data members. 
(Not to be confused with ```std::initializer_list```).

(_since C++ 20_) A constructor must not be a coroutine


Constructor Syntax

Constructors are declared using member function declarators of the following form:

  _class-name_ **(** _parameter-list_ (_optional_) **)** ```except-spec```(_optional_) attr(_optional_)


