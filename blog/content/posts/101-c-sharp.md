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

- propfull



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