ArchRole - Role/trait support for Objective-C
=============================================

ArchRole brings support for roles (sometimes called traits) to Objective-C.

What's a role?
--------------

A role is a group of methods that can be added to any class that meets the 
prerequisites.  You can think of it as being like a protocol with definitions 
for some of the methods, or a category that can be applied to several different 
classes.

In `ArchRole`, a role defines a set of required methods and properties that the 
adopting class must implement, either directly or by adopting other roles that 
satisfy them.  The role also provides methods and properties to the classes 
that adopt them.  When `+initialize` is called on the adopting class, all of 
the methods in the roles it adopts are copied into it.

Unlike mixins, two conflicting roles cannot both be adopted by a single class; 
the application will throw an exception upon launch unless the conflict is 
resolved.

What does a role look like in code?
-----------------------------------

A role is composed of a matching protocol and class.

The protocol is applied to any class that adopts the role.  Methods that must 
be implemented by those classes are marked `@required`; methods supplied by the 
role are marked `@optional`.  The protocol must also conform to the `ArchRole` 
protocol; it may also conform to other roles' protocols, but `ArchRole` must be 
explicitly listed (otherwise the role machinery will assume it's an ordinary 
protocol).

    @protocol MyRole <AnotherRole, AThirdRole, ArchRole>
    
    - (void)requiredMethod;
    
    @optional
    
    - (void)providedMethod1;
    - (void)providedMethod2;
    
    @end

The role's class must have the same name as the role, must inherit directly 
from the `ArchRole` class, and must implement all of the optional methods 
listed in the protocol.  All methods implemented in the class--even ones not 
listed in the protocol--will be added to classes that do the role.  The role's 
class cannot be instantiated; it is merely a container for the methods that the 
role implements.

    @interface MyRole : ArchRole
    - (void)providedMethod1;
    - (void)providedMethod2;
    @end
    
    @interface MyRole (ConvenienceDeclarations) <AnotherRole, AThirdRole>
    - (void)requiredMethod;
    @end

To implement your provided methods, just use an `@implementation` section as 
normal.  If your role uses any other roles, make sure you use 
`INITIALIZE_DECLARED_ROLES` or `+[YourClass composeDeclaredRoles]` as described 
in "Adopting a defined role" below.

The objc-rolec preprocessor
---------------------------

The easiest way to achieve this protocol/class structure is to use the 
`objc-rolec` preprocessor.  This accepts Objective-C files with an .rh 
extension and syntax like:

    @role MyRole <AnotherRole, AThirdRole>
    
    - (void)requiredMethod;
    
    @provides
    
    - (void)providedMethod1;
    - (void)providedMethod2;
    
    @end

And creates a counterpart .h file with standard Objective-C syntax.  Note that 
`objc-rolec` does not really understand Objective-C syntax, and will attempt to 
convert @role keywords in comments, preprocessor directives, and string 
literals.  Please be careful.

Adopting a defined role
-----------------------

To adopt a role in your class, adopt its protocol with the angle bracket syntax 
and arrange for `+[YourClass composeDeclaredRoles]` to be called.  The easiest 
way to do this is to add `INITIALIZE_DECLARED_ROLES` at the top of your 
`@implementation` section, which will insert an `+initialize` method for you. 
If you have your own `+initialize` method, call `+composeDeclaredRoles` there 
instead.

    @interface MyClass <MyRole>
    
    @end
    
    @implementation MyClass
    
    INITIALIZE_DECLARED_ROLES
    
    - (void)requiredMethod {
        ...
    }
    
    @end

Role conflicts
--------------

Role composition is careful to avoid conflicts between different roles.

If a role provides a method that's already defined in the adopting class, the 
class's implementation is always the one used; a role method will never 
override a class method.

If a class adopts two roles which both provide implementations of the same 
method, ArchRoles throws an exception.  This occurs during role composition, 
which usually occurs the first time your class is used.

A class can resolve the conflict by choosing not to compose a version of the 
method that it doesn't want.  To do this, override the 
`+shouldComposeInstanceMethod:fromRole:` or `+shouldComposeClassMethod:fromRole:` 
method and return NO for the versions you don't want.

Note that *all* methods from the roles are added during composition, not just 
the ones that are publicly declared in the protocol, and so two roles with 
identically-named private methods will conflict.  We recommend prefixing any 
private role methods to avoid conflicts.

Method renaming isn't currently supported.  Perhaps it should be.

Notes on the current release
----------------------------

Many, *many* things are untried at this stage, even very simple things like:

1. Adopting several roles in one class.
2. A role adopting another role.
3. Any of the stuff relating to conflicts between roles.

Roles cannot currently define ivars, either directly or by synthesis; ArchRole 
will throw an exception at application launch if a role's class tries to define 
any.  I don't know if fragile ivar support will allow this to be added--I need 
to do some research to find out.

Authors
-------

Original version by Brent Royal-Gordon of Architechies 
(<brent@architechies.com>, <http://architechies.com>).

If you use this library in your code, I'd love to hear about it--please drop 
me an e-mail.

License
-------

This software is licensed under the MIT license:

Copyright (c) 2011 Architechies.

Permission is hereby granted, free of charge, to any person obtaining a copy of 
this software and associated documentation files (the "Software"), to deal in 
the Software without restriction, including without limitation the rights to 
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies 
of the Software, and to permit persons to whom the Software is furnished to do 
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all 
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE 
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, 
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE 
SOFTWARE.
