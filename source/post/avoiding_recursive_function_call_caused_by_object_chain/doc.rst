Avoiding recursive function call caused by object chain 
====================================================================

Background 
--------------- 
In programming, a case that a function calls itself is called recursive 
function call. This can be done directly or indirectly: if function *A* 
calls another function *B* which may call *A* again, this can be called 
an indirect recursive function call. 

Though recursive function is widely used in modern programming due to its 
usability, sometimes it need to be avoided. A recursive function may take 
a long time to be returned when it has too high recurrence level. This may 
cause function stack overflow, and block of event in event-driven 
programming. 

Practical case 
------------------
There's an object type, *Obj*. Obj can have other Objs as children and 
it can be parent of them. An Obj cannot be both parent and child of an 
Obj: it means any parent-child relationship can make recurrent. 
An Obj can have multiple parent and children. 
An Obj has 2 states, *Valid* 
and *Invalid*. An Obj which is explicitly set as Valid or has any of 
Valid parent is Valid state, otherwise it's invalid. 

If an Obj becomes Invalid, its children may become Invalid. State of 
children 
should be checked by iterating them one by one. This is a case where 
recursive 
function call occurs due to object chain. 

.. code-block:: cpp 

	class Obj 
	{ 
	public: 
		/* Set the Obj Invalid. */
		void setInvalid(); 
		
		/* Set the Obj Valid. */
		void setValid(); 

		/* Check whether the Obj is Valid or Invalid, 
		   and call setInvalid() or setValid() when needed. */
		void check(); 

		/* Get current state. */ 
		bool isValid(); 

		/* Iteratable container of children. */
		Objs children(); 
	}; 
	void Obj::check() 
	{ 
		if ( needToSetValid() ) 
		{
			setValid(); 
		} 
		else if ( needToSetInvalid() ) 
		{ 
			setInvalid(); 
		} 
	} 
	void Obj::setInvalid() 
	{ 
		// Do some invalidating code. 

		for ( Obj* child : children() ) 
		{ 
			// Recursive function call occurs! 
			child->check(); 
		} 
	} 

	void function_from_client() 
	{ 
		auto obj = getObj(); 
		obj->setInvalid(); 
	} 

The code above is the usual case of recursive function due to object chain. 
Whenever ``Obj::setInvalid()`` is called, it may call other Objs' 
``Obj::setInvalid()`` as well. 

Solution 
-------------- 
One solution is using "mark and skip" way. Instead of calling children's 
``check()`` method directly, just let a function in upper stack know that 
``check()`` of children must be called and return the current scope right 
away. 

.. code-block:: cpp 

	class Obj 
	{ 
	public: 
		/* Previous code */
		using SideEffects = list<function<void ()>>; 
		void setInvalid(SideEffects* outSideEffect); 

		void setInvalidWithoutRecur(); 
	}; 

	void Obj::setInvalid(SideEffects* outSideEffects) 
	{ 
		// Do some invalidating code. 

		for ( Obj* child : children() ) 
		{ 
			outSideEffects->push_back([&]{child->check();}); 
		} 
	} 
	void Obj::setInvalidWithoutRecur() 
	{ 
		SideEffects callbacks; 

		setInvalid(&callbacks); 
		while ( callbacks.empty() == false ) 
		{ 
			auto callback = callbacks.back(); 
			callbacks.pop_back(); 
			callback(); 
		} 
	} 
	void function_from_client() 
	{ 
		auto obj = getObj(); 
		obj->setInvalidWithoutRecur(); 
	} 

