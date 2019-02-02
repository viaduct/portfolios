Destruction and memory free can be ignored in a programming logic
==============================================================================

Object and value construction, referring, modifications are all part 
of a programming logic. Actually almost every part of code will be a logic 
for the software what the code consists. But there's one exception, object 
destruction and meory free. 

Allocation and construction, which are in the oppisite side, are considered 
as parts of a logic because it determines when an object becomes valid. 
For example, *value* in instance of ``class Obj { public: int value = 0; };`` 
will not be accessible by ``Obj* a;`` before it's properly initialized with 
valid object. Allocation and construction explicitly guarantee that an object 
is initialized and will do its work. 

While, destruction and memory free are in quite different situation. They 
are done when an object will not be used anymore. Using an object requires 
construction, but not using it does not requires destruction: it can be 
removed in any time. If a memory space is infinite, there might be no need 
to destruct an object. 

Due to this reason, many modern programming languages adopted garbage 
collection to let programmers do not care about destruction which can be 
ignored in a software logic. One exception is RAII pattern, which requires 
a pair of constructor and destructor must be called in time. 

This aspect implies that writing a software logic should consider memory 
management and object destruction in different level from ordinary 
logic. Current memory mangement logics proof this: ``shared_ptr``, 
``unique_ptr`` in C++, and garbage collector run in language-level, 
not programmer-defined-logic-level. 