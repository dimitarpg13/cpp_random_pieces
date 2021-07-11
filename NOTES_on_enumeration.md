Notes on Enumeration declaration

An _enumeration_ is a distinct type whose value is restricted to a range of values, which
may include several explicitly named constants ("_enumerators_"). The values of the 
constants are values of an integral type known as the _underlying type_ of the enumeration.

An enumeration is defined using the following syntax:


_enum-key_ _attr_(optional) _enum-name_(optional) _enum-base_(optional)(_C++11_)
{ _enumerator-list_(optional) }

this is _enum-specified_, which appears in _decl-specifier-seq_ of the declaration syntax:
defines the enumeration type and its enumerators


_enum-key_ _attr_(optional) _enum-name_ _enum-base_(optional) ; (_since C++11_)

 
