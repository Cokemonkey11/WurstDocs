---
title: Introduction
sections:
- Philosophy
- Syntax
- Basics
---

### Philosophy

WurstScript aims to provide an easy and comfortable workflow to produce readable and maintainable code.
Ease of use and stress-free map development take higher priority than execution speed of the produced code.
Wurst is easy to use and learn, especially with prior knowledge of Jass or another programming language, while still staying
beginner-friendly and readable to non programmers.

While we know that WurstScript won't replace vJass in the WC3 mapping scene (one reason being tons of vJass scripts that can't be easily ported), we still hope it will  serve as a very good alternative, in particular for users that are trying to learn Jass.

> Note that this manual is not a beginner's tutorial and expects the reader to have prior knowledge in programming. [Click here for a beginner's guide.](start.md)

### Syntax

The WurstScript Syntax uses indention to define Blocks, instead of using curly
brackets (like Java) or keywords like 'endif' (like Jass). You can use either spaces or tabs for indentation, but mixing both will throw a warning.
In the following we use the word "tab" to refer to the tab character or to 4 space characters.

```wurst
// Wurst example. 'ifStatements' need to be indented
if condition
	ifStatements
nextStatements

// Other example using curly brackets (e.g. Java or Zinc)
if condition {
	ifStatements
}
nextStatements
// Other example using keywords (e.g. Jass)
if condition
	ifStatements
endif
nextStatements

```


In general newlines come at the end of a statement, except for the following cases:

- A newline is ignored if the line ends with `(` or `[`
- A newline is ignored before a line beginning with `)`, `]`,`.` or `..`
- A newline is ignored, if the line ends of begins with:
    `,`, `+`, `*`, `-`, `div`, `/`, `mod`, `%`, `and`, `or`, `->`

You can use this to break longer expressions or long parameter lists over several lines, or to chain method invocations:
```wurst
someFunction(param1, param2,
	param3, param4)

someUnit..setX(...)
	..setY(...)
	..setName(...)

new Object()..setName("Foo")..move()
```

WurstScript tries to avoid using excessive verbosity and symbols to stay concise and readable.


### Basics

Wurst code is organized into _packages_. All your wurst code has to be inside a _package_.
Packages can also _import_ other packages in order to use variables, functions, classes, etc. from the imported package.
Packages can have an _init_ block that is executed when the map is loaded.

```wurst
package HelloWurst
// to use resources from other packages you have to import them at the top
import PrintingHelper

// the init block of each package is called when the map starts
init
	/* calling the print function from the PrintingHelper package */
	print("Hello Wurst!")
```

For more information about packages, refer to the packages section.

You can still use normal Jass syntax/code outside of packages - the Wurstpack World Editor will parse jass (and vjass, if jasshelper is enabled), in addition to compiling your Wurst code.

For more information about using maps with mixed wurst/jass code, refer to the "Using Wurst with legacy maps" section.

###### Naming Conventions

Wurst encourages several naming conventions to create a common way of writing code and to provide general readability rules:

-  **Functions** should start with a **lowercase letter**
-  **Variables** should either start with a **lowercase letter** *or* be **all uppercase letters and "_"**
-  **Class/Module/Interface/Package** names should start with an **uppercase letter**
-  **Tuplenames** should start with a **lowercase letter**

Since Wurst comes with optimizing tools build-in, you should always choose descriptive names.



###### Functions

A _function_ definition consists of a name, a list of formal parameters and a return
type. The return type is declared after the formal parameters using the _returns_ keyword.
If the function does not return a value this part is omitted.
```wurst
// this function returns the maximum of two integers
function max(int a, int b) returns int
	if a > b
		return a
	else
		return b

// this function prints the maximum of two integers
function printMax(int a, int b)
	print(max(a,b).toString())

function foo() // parentheses instead of "takes", "returns nothing" obsolete.
	...

function foo2(unit u) // parameters
	RemoveUnit(u)

function bar(int i) returns int // "returns" [type]
	return i + 4

function blub() returns int // without parameters
	return someArray[5]

function foobar()
	int i // local variable
	i = i + 1 // variable assignment
	int i2 = i // support for locals anywhere inside a function
```

###### Variables

Global (local) variables can be declared anywhere in a package (function).
A constant value may be declared using the _constant_ or _let_ keyword.
Mutable variables are declared by using the _var_ keyword or by writing the type of the variable before its name.
```wurst
// declaring a constant - the type is inferred from the initial expression
constant x = 5
// The same but with explicit type
constant real x = 5
// declaring a variable - the inferring works the same as 'let', but the value can be changed
var y = 5
// declaring a variable with explicit type
int z = 7
// declaring an array
int array a

// inside a function
function getUnitInfo( unit u )
	player p = u.getOwner()
	var name = u.getName()
	print( name )
	real x = u.getX()
	real y = u.getY()
	let sum = x + y
```

With these basic concepts you should be able to do anything you already know for Jass.
The syntax is a little bit different of course, but this is covered in the next chapter.
