---
title: "101 C Sharp"
date: 2022-04-29T16:41:07+02:00
draft: false
---

Fundamentals

.NET =

- CLR (commun language runtime) : space to run .NET code
- FCL ( framework class library ) : base class library

- Reference Type vs Value Type :
  always is passing by value (by coping the value/reference)
- string is a reference type, it's immutable (can't be changed)

- must add ref to pass by reference, ref = out (must be assigned)

-

## good to have in mind

- readonly : can be assigned only in initializer or constructor
- const : can be assigned only in initializer

- using (var ......)
  {

}
guarantee that the object will be disposed. (try finally statement) (file , socket ...)

- propfull : getter and setter

- readonly : can be assigned only in initializer or constructor => can be assigned dynamically via file , bdd ... | runtime constant | optionally static
- constant : can be assigned only in initializer => compile time instance | always static

## OOP

- Pillares :
  - Encapsulation : hide the implementation details
  - Inheritance : reuse code
  - Polymorphism : override methods , same type behave differently

## New

- case var d when d > 50
  result = "greater than 50"
  break;

-

https://www.udemy.com/course/csharp-advanced/?ranMID=39197&ranEAID=Qouy7GhEEFU&ranSiteID=Qouy7GhEEFU-.sNjYK9P8bctsXCp0qKv0Q&utm_source=aff-campaign&utm_medium=udemyads&LSNPUBID=Qouy7GhEEFU
https://www.udemy.com/course/programming-in-microsoft-c-exam-70-483/?utm_source=adwords&utm_medium=udemyads&utm_campaign=LongTail_la.EN_cc.ROW&utm_content=deal4584&utm_term=_._ag_77879423894_._ad_535397245857_._kw__._de_c_._dm__._pl__._ti_dsa-1007766171032_._li_1006094_._pd__._&matchtype=&gclid=Cj0KCQiA4OybBhCzARIsAIcfn9nmySNQ7Q-ZIuBi42fyj0u2etAu60eo_ye3IwiQNddTCpLzkAf-0cEaAjJmEALw_wcB
