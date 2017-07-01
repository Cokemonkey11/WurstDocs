---
title: Manual
---
# Enums

In Wurst, _Enums_ can be used to set up collections of named (int) constants.
These Constants can then be accessed via the Enum's name:

    enum State
        FLYING
        GROUND
        WATER

    init
        State s = State.GROUND

You can also use enums inside of classes

    class C
        State currentState

        construct( State state )
            currentState = state

To check the current value of an enum, you can use the switch statement.
Note that all Enummembers have to be checked (or a defaut).

    switch currentState
        case State.FLYING
            print("flying")
        case State.GROUND
            print("ground")
        case State.WATER
            print("water")

In switch statements and variable assignments the qualifier can be ommited so you can also write:

    switch currentState
        case FLYING
            print("flying")
        case GROUND
            print("ground")
        case WATER
            print("water")


# Tuple Types

With _tuple_ types you can group several variables into one bundle. This can be used to return more than one value from a function, to create custom types and of course for better readability.

Note that tuples are not like classes. There are some important differences:
- You do not destroy tuple values.
- When you assign a tuple to a different variable or pass it to a function you create a copy of the tuple. Changes to this copy will not affect the original tuple.
- Tuple types cannot be bound to type parameters, so you can not have a List{vec} if vec is a tuple type.
- As tuple types are not created on the heap you have no performance overhead compared to using single variables.


        // Example 1:

        // define a tuple
        tuple vec(real x, real y, real z)

        init
            // create a new tuple value
            vec v = vec(1,2,3)
            // change parts of the tuple
            v.x = 4
            // create a copy of v and call it u
            vec u = v
            u.y = 5
            if v.x == 4 and v.y == 2 and u.y == 5
                testSuccess()


        // Example 2:

        tuple pair(real x, real y)
        init
            pair p = pair(1,2)
            // swap the values of p.x and p.y
            p = pair(p.y, p.x)
            if p.x == 2 and p.y == 1
                testSuccess()


Because tuples don't have any functions themselves, you can add extension
functions to an existing tuple type in order to achieve class-like
functionality.
Remember that you can't modify the value of a tuple in it's extension function
- so you have to return a new tuple every time if you wan't to change something.
Look at the Vector package in the Standard Library for some tuple usage
examples. (Math/Vectors.wurst)



# Extension Functions

Extension functions enable you to "add" functions to existing types without
creating a new derived type, recompiling, or otherwise modifying the original
type.
Extension functions are a special kind of static function, but they are called
as if they were instance functions of the extended type.

## Declaration

    public function TYPE.EXTFUNCTIONNAME(PARAMETERS) returns ...
        BODY
        // The keyword "this" inside the body refers to the instance of the extended type

## Examples


    // Declaration
    public function unit.getX() returns real
        return GetUnitX(this)

    // Works with any type
    public function real.half() returns real
        return this/2

    // Parameters
    public function int.add( int value )
        return this + value

    // Usage
    unit u = CreateUnit(...)
    ...
    print( u.getX().half() )

    // Also classes, e.g. setter and getter for private vars
    public function BlubClass.getPrivateMember() returns real
        return this.privateMember

    // And tuples as mentioned above
    public function vec2.lengthSquared returns real
        return this.x*this.x+this.y*this.y


# Lambda expressions and Closures

A lambda expression (also called anonymous function) is a lightweight way to provide an implementation
of a functional interface or abstract class (To keep the text simple, the following
explanations are all referring to interfaces, but abstract classes can be used in the same way).

A *functional interface* is an interface which has only one method.
Here is an example:

    // the functional interface:
    interface Predicate<T>
        function isTrueFor(T t) returns bool

    // a simple implementation
    class IsEven implements Predicate<int>
        function isTrueFor(int x) returns bool
            return x mod 2 == 0

    // and then we can use it like so:
    let x = 3
    Predicate<int> pred = new IsEven()
    if pred.isTrueFor(x)
        print("x is even")
    else
        print("x is odd")
    destroy pred


When using lambda expressions, it is not necessary to define a new class
implementing the functional interface. Instead the only function of the
functional interface can be implemented where it is used, like so:

    let x = 3
    // Predicate is defined here:
    Predicate<int> pred = (int x) -> x mod 2 == 0
    if pred.isTrueFor(x)
        print("x is even")
    else
        print("x is odd")
    destroy pred

The important part is:

    (int x) -> x mod 2 == 0

This is a lambda expression. It consists of two parts and an arrow symbol *->*
between the two parts. The left hand side of the arrow is a list of formal parameters,
as you know them from function definitions. On the right hand side there
is an expression, which is the implementation. The implementation consists only
of a single expressions, because lambda expressions are typically small and used
in one line. But if one expression is not enough there is the begin-end expression.

Remember that, because closures are just like normal objects, you also have to destroy them
like normal objects. And you can do all the other stuff you can do with
other objects like putting them in a list or into a table.


## begin-end expression

Sometimes one expression is not enough for a closure. In this case, the begin-end
expression can be used. It allows to have statements inside an expression. The
begin keyword has to be followed by a newline and an increase in indentation.
The rule that newlines are ignored inside parenthesis is ignored for the begin-end
expression, so that it is possible to have multiple lines of statements within:

    doLater(10.0, () -> begin
        KillUnit(u)
        createNiceExplosion()
        doMoreStuff()
    end)

It is also possible to have a return statement inside a begin-end expression
but only the very last statement can be a return.


## Capturing of Variables


The really cool feature with lambda expressions is, that they create a *closure*.
This means that they can close over local variables outside their scope
and capture them.
Here is a very simple example:

    let min = 10
    let max = 50
    // remove all elements not between min and max:
    myList.removeWhen((int x) ->  x < min or x > max)

In this example the lambda expression captured the local variables min and max.

It is important to know, that variables are captured by value. When a closure
is created the value is copied into the closure and the closure only works on that copy.
The variable can still be changed in the environment or in the closure, but this will have no effect
on the respective other copy of the variable.

This can be observed when a variable is changed after the closure is created:

    var s = "Hello!"
    CallbackFunc f = () -> begin
        print(s)
        s = s + "!"
    end
    s = "Bye!"
    f.run()  // will print "Hello!"
    f.run()  // will print "Hello!!"
    print(s) // will print "Bye!"


## Behind the scenes

The compiler will just create a new class for every lambda expression in your code.
This class implements the interface which is given by the context in which
the lambda expression is used.
The generated class has fields for all local variables which are captured.
Whenever the lambda expression is evaluated, a new object of the class is created
and the fields are set.

So the "Hello!" example above is roughly equivalent to the following code:

    // (the interface was not shown in the above code, but it is the same):
    interface CallbackFunc
        function run()

    // compiler creates this closure class implementing the interface:
    class Closure implements CallbackFunc
        // a field for each captured variable:
        string s

        function run()
            // body of the lambda expression == body of the function
            print(s)
            s = s + "!"

    var s = "Hello!"
    CallbackFunc f = new Closure()
    // captured fields are set
    f.s = s
    s = "Bye!"
    f.run()  // will print "Hello!"
    f.run()  // will print "Hello!!"
    print(s) // will print "Bye!"

## Function types

A lambda expression has a special type which captures the type of the parameter
and the return type. This type is called a *function type*. Here are some examples with their type:

    () -> 1
        // type: () -> integer

    (real r) -> 2*r
        // type: (real) -> real

    (int x, string s) -> s + I2S(x)
        // type: (int,string) -> string


While function types are part of the type system, Wurst has no way to write down
a function type. There are no variables of type "(int,string) -> string".
Because of this, a lambda expression can only be used in places where
a concrete interface or class type is known.
This can be an assignment where the type of the variable is given.

    Predicate<int> pred = (int x) -> x mod 2 == 0

However it is not possible to use lambda expressions if the type of the variable is only inferred:

    // will not compile, error "Could not get super class for closure"
    let pred = (int x) -> x mod 2 == 0

## Lambda expressions as code-type

Lambda expressions can also be used where an expression of type `code` is expected.
The prerequisite for this is, that the lambda expression does not have any parameters
and does not capture any variables. For example the following code is _not_ valid,
because the local variable `x` is captured.


    let t = getTimer()
    let x = 3
    t.start(3.0, () -> doSomething(x)) // error: Cannot capture local variable 'x'


This can be fixed by attaching the data to the timer manually:

    let t = getTimer()
    let x = 3
    t.setData(x)
    t.start(3.0, () -> doSomething(GetExpiredTimer().getData()))

If a lambda expression is used as `code`, there is no new object created and
thus there is no object which has to be destroyed. The lambda expression will just
be translated to a normal Jass function, so there is no performance overhead when
using lambda expressions in this way.

# Advanced Concepts

## Function Overloading

Function overloading allows you to have several functions with the same name.
The compiler will then decide which function to call based on the static type
of the arguments.

Wurst uses a very simple form of overloading. If there is exactly one function in
scope which is feasible for the given arguments, then this function will be used.
If there is more than one feasible function the compiler will give an error.

Note that this is different to many other languages like Java, where the
function with the most specific feasible type is chosen instead of giving an error.

    function unit.setPosition(vec2 pos)
        ...

    function unit.setPosition(vec3 pos)
        ...

    function real.add(real r)
        ...

    function real.add(real r1, real r2)
        ...

This works because the parameters are of different types or have a different amount of paramaters and the correct function can therefore be determined at compiletime.

    function real.add(real r1)
        ...

    function real.add(real r1) returns real

This does not work because only the returntype is different and the correct function cannot be determined.


    class A
    class B extends A

    function foo(A c)
        ...

    function foo(B c)
        ...

    // somewhere else:
        foo(new B)



This does not work either, because B is a subtype of A. If you would call the function foo
with a value of type B, both functions would be viable. Other languages just take the
"most specific type" but Wurst does not allow this. If A and B are incomparable types, the overloading is allowed.


## Operator Overloading

Operator Overloading allows you to change the behavior of internal operators +, -, \* and / for custom arguments.
A quick example from the standard library (Vectors.wurst):

    // Defining the "+" operator for the tupletype vec3
    public function vec3.op_plus( vec3 v ) returns vec3
        return vec3(this.x + v.x, this.y + v.y, this.z + v.z)

    // Usage example
    vec3 a = vec3(1.,1.,1.)
    vec3 b = vec3(1.,1.,1.)
    // Without Operator Overloading (the add function was replaced by it)
    vec3 c = a.add( b )
    // With operator Overloading
    vec3 c = a + b

You can overload operators for existing types via Extension-Functions or via class-functions for the specific classtype.
In order to define an overloading function it has to be named as following:

    +  "op_plus"
    -  "op_minus"
    *  "op_mult"
    /  "op_divReal"

## Object Editing

Creating Object-Editor Objects via Wurst code.

*NOTE:* Object Editing hardly works at the moment, so you should only use it for fun but not for profit.
Do not use it for a real project yet!

### Compiletime Functions

Compiletime Functions are functions, that are executed when compiling your script/map.
They mainly offer the possibility to create Object-Editor Objects via code.

A compiletime function is just a normal Wurst function annotated with @compiletime.

    @compiltetime function foo()

Compiltetime functions have no parameters and no return value.

In order to run compiletime functions you have to enable the checkbox in the Wurstpack Menu.
When you use compiletime functions to generate objects, Wurst will generate the object files
next to your map and you can import them into your map using the object editors normal import
function. Compared to ObjectMerger this has the advantage, that you can directly see your new
objects in the object editor.
You can also enable an option to directly inject the objects into the map file, though the changes will not be visible in the object-editor directly.

You can use the same code during runtime and compiletime.
The special constant `compiletime` can be used to distinguish the two.
The constant is `true` when the function was called at compiletime and `false` otherwise.
The following example shows how this could be useful:

    init
        doInit()

    @compiletime
    function doInit()
        for i = 1 to 100
            if compiletime
                // create item object
            else
                // place item on map


### Object Editing Natives

The standard library provides some functions to edit objects in compiletime functions.
You can find the corresponding natives and higher level libraries in the objediting folder of the standard library.

The package ObjEditingNatives contains natives to create and manipulate objects. If you are familiar with
the object format of Wc3 and know similar tools like [Lua Object Generation](http://www.hiveworkshop.com/forums/jass-ai-scripts-tutorials-280/lua-object-generation-191740/)
or the ObjectMerger from JNGP, you should have no problems in using them. If you run Wurst with compiletime functions enabled, it will generate
the object creation code for all the objects in your map. This code is saved in files named similar to "WurstExportedObjects_w3a.wurst.txt" and
can be found right next to your map file. You can use this code as a starting point if you want to use the natives.

Wurst also provides a higher level of abstraction. For example the package AbilityObjEditing provides many classes
for the different base abilities of Wc3 with readable method names. That way you do not have to look up the IDs.

The following example creates a new spell based on "Thunder Bolt". The created spell has the ID "A005".
In the next line the name of the spell is changed to "Test Spell".
Level specific properties are changed inside the loop.

    package Objects
    import AbilityObjEditing

    @compiletime function myThunderBolt()
        // create new spell based on thunder bolt from mountain king
        let a = new AbilityDefinitionMountainKingThunderBolt("A005")
        // change the name
        a.setName("Wurst Bolt")
        a.setTooltipLearn("The Wurstinator throws a Salami at the target.")
        for i=1 to 3
            // 400 damage, increase by 100 every level
            a.setDamage(i, 400. + i*100)
            // 10 seconds cooldown
            a.setCooldown(i, 10.)
            // 0 mana, because no magic is needed to master Wurst
            a.setManaCost(i, 0)
            // ... and so on


*NOTE* There are also packages for other object types, but those packages are even more WIP.



## Automated Unit Tests

You can add the annotation @test to a function. Then when you type "tests" into the Wurst Console all functions
annotated with @test are executed.

You have to import the Wurstunit package to use functions like assertEquals.

Example:

        package Test
        import Wurstunit

        @test public function a()
            12 .assertEquals (3*4)

        @test public function b()
            12 .assertEquals (5+8)

If you run this, you get the following output:

        > 1+1
        res = 2     // integer
        > tests
        1 tests OK, 1 tests failed
        function b (Test.wurst, line 8)
            test failed: expected 13 but was 12
            at stmtreturn  (.../lib/primitives/Integer.wurst, line 25)
        >

The first line is just to check whether the console is working ;)

You can search the standard library for "@test" to get some more examples.

# Other Stuff

## Stacktraces

You can enable stacktraces in the the menu of WurstPack or with the commandline
switch `-stacktraces`. Each error will then be displayed with a stacktrace showing
the line number and filename of each function-call leading to the error. Errors
must be generated with the magic function `function error(string msg)`.

# Standard Library

Wurst comes with a library of some useful standard functions
You can find the generated HotDoc pages here: [The Wurstscript Standardlibrary](http://peeeq.de/hudson/job/Wurst/HotDoc_Standard_Library_Documentation/index.html)
However, the best way to learn about the library is still to look at the source code.

# Wurst Style

In this section we describe some style guidelines which you should follow when programming in Wurst.

## Rule 1: Write for readability

You should always write your code so that it can be read easily. Ideally your code should be readable like normal English text.
Use suitable names for functions and variables to make your code sound like normal text.

- Create functions which help you to express your intend on a higher level of abstraction.
- When a function or variable has type boolean, name it like a question. The name should sound natural when used inside an if statement.
- Use class functions and extension methods to make code readable from left to right.


Example:

    // not so readable:
    if GetUnitState(h, UNIT_STATE_LIFE) <= 0.405
        let t = NewTimer()
        t.setData(...)
        t.start(...)
        ...

    // better:
    if hero.isDead()
        hero.reviveAt(town, REVIVE_TIME)

    // that timer stuff is in a different small function


### Rule 1.1 Document your Code
### Rule 1.2 Keep functions short
### Rule 1.3 The Golden Path




## Rule 2: Write checkable code

Your code should be checkable by the compiler. The Wurst compiler provides you with some powerful typechecking tools. Use
them wisely to reduce the mistakes you can do in your program. You might also want to watch [this awesome talk by Yaron Minsky](http://vimeo.com/14313378) who summarizes it pretty nicely: "make invalid state unrepresentable". Of course Wurst is not as sophisticated as ML
but there are still a lot of things you can do.

- avoid the castTo expression whenever possible
- use high level libraries like HashMap instead of lower level libraries like Table or even then hashtable natives
- use tuple types to give meaning to values (see Vector for an example)
- use enums with switch statements

## Rule 3: DRY: Don't repeat yourself

When you find the same lines of code at several places of your project, try to put the common code into a function, class or module.

## Rule 4: KISS: Keep it simple, stupid!

Always try to choose the most simple solution to your problem. Avoid premature optimization.


## Rule 5: Use Object oriented programming

### Avoid instanceof

Using *instanceof* is usually a sign for bad use of object orientation. Often instanceof checks can
be replaced by using dynamic dispatch. Let's look at an example:

    if shape instanceof Circle
        Circle c = shape castTo Circle
        area = bj_PI * c.radius*c.radius
    else if shape instanceof Rect
        Rect r = shape castTo Rect
        area = r.width * r.height

It would be better to have one area function in Shape and then implement it for Circle and Rect.
That way you can just write:

    area = shape.area()

The right area method will then be automatically selected based on the type of shape.


## Using Wurst with legacy maps

The Wurstpack World Editor will compile and link your wurst packages into your map even if that map already has GUI/jass/vJass - Wurst will run in parallel to that.

For the more adventurous, Wurst also has a jass-like dialect called Jurst, and supports automatic extraction of vJass to Jurst.
This feature should be treated as beta.

### Jurst

Jurst is a dialect of Wurst, which has the same features as Wurst, but with a Syntax similar to vJass.
You can use Jurst to adapt vJass code, but there are still a few manual steps involved, because of the difference in supported features.

In general, Jurst is less strict than Wurst.
The following code fragments are each valid Jurst:

    library LooksLikeVjass initializer ini
        private function ini takes nothing returns nothing
            local trigger t = CreateTrigger()
            t = null
        endfunction
    endlibrary

and

    package NearlyTheSameAsWurst
        function act()
            print(GetTriggerUnit().getName() + " died.")
        end

        init
            CreateTrigger()..registerAnyUnitEvent(EVENT_PLAYER_UNIT_DEATH)
                           ..addAction(function act)
        end

    // Note: "end" intentionally not present here.

As you can see, Jurst is able to access wurst code including packages from the wurst standard library, such as `print`.
However, you should not try to access Jurst (or jass/vjass) code from Wurst packages.

The especially keen can read the [Jurst Language Definition](https://github.com/peq/WurstScript/blob/master/de.peeeq.wurstscript/parserspec/Jurst.g4) for an exact reference of legal Jurst.

### Automatic Extraction of vJass to Jurst

One useful way of working with legacy maps is by extracting existing vJass to Jurst automatically.
In Eclipse, you can right-click on a map and export all the custom text triggers into Jurst files.
The files will be stored in the `wurst/exported/` folder.

![Extract to Files](assets/images/extractToFiles.png)

After doing this you'll want to manually delete the members in the trigger editor.
Be warned that the extract-to-files feature is likely to produce at least some files with syntax errors, so be sure to back up your map before proceeding.

# Optimizer

The Wurstcompiler has a set of build-in scriptoptimizing tools which will, when enabled, optimize the generated Jass code in various ways.
Jass optimization got very important to provide playable framerates when using very enhanced and complex systems.
On the one hand the optimizer cleans the code, making it smaller in size and removing useless stuff in order to reduce RAM-usage.
On the other hand it also offers some optimizations to increase the speed of execution and performance of the code.

## Cleaning

Stuff that is being removed, changed or not even printed

* Comments
* Unneeded White-spaces
* Excessive parentheses
* Some useless Jassconstants replaced with "null"

## Name compression

Smaller names execute faster and take less space, so all names of functions and variables are compressed to the shortest name possible.

## Inlining

Inlining is not an easy task, but brings great performance boosts to systems which use many different functions.
It also makes coding easier and more readable, because you don't have to care about the performance loss
when splitting stuff into too many functions.
Also blizzard.j functions, such as BJs and Swaps, can get inlined.

In the current implementation, a function can be inlined when it has only one exit-point.
This is the case, when there is no return statement or when the only return statement is at the end of a function.

Whether a function actually gets inlined depends on some heuristics in the compiler.
The heuristic tries to balance execution speed and size of the mapscript, as inlining usually makes the code longer but faster.
A function is more likely to get inlined, when it is short.
A function is less likely to get inlined, when it is called in many different places.
It is best to not rely on more guarantees about the inliner, as the heuristics are changed from time to time.

Global variables that have a constant value get inlined as well as constant locals.

## Constant Folding and Constant propagation

Expressions containing only constants are calculated at compiletime.
Ifs with constant conditions are removed.
Both mechanics work together to remove unneeded and unreachable code.

# Errors and Warnings

Wurst provides some errors and warnings to help finding bugs early in the development process.
In this chapter some of these errors and warnings are explained, as well as some ways to fix them.

## Warnings

* The assignment to local variable x is never read

    This warning means that a value used in an assignment is never read in all possible executions.
    This often means that you forgot to use it, so it probably is a bug you have to fix.
    Sometimes it is just some unnecessary code as in the following example:

        int x = 0
        if someCondition
            x = 2
        else
            x = 3
        print(x.toString())

    The fix for this warning is straight forward in this case, just remove the unused value:

        int x
        if someCondition
            x = 2
        else
            x = 3
        print(x.toString())


* The variable x is never read.

    Usually this warning also means that you have some unnecessary code or that you forgot to use something.
    In some rare cases you need to give a parameter but do not want to use it.
    In these cases you can **suppress the warning** by starting the variable name with an underscore:

        function foo(int _x)
            return 5

* The assignment to ... probably has no effect

    This warning usually appears with assignments like the following:

        construct(int emeny)
            this.enemy = enemy

    Note that there is a typo in the parameter which causes the assignment to be wrong.

* The import .... is never used directly.

    Usually this means that you are importing a package and never using it, so that you can remove the import.
    There are some corner cases like implicit generic conversions, which are not detected correctly at the moment.

Some warnings are just there to ensure some common coding standards in Wurst:

* Function names should start with an lower case character.
* Variable names should start with a lower case character.
* Type names should start with upper case characters.
