# Formula
handle mathematical expressions at [haxe-runtime](https://haxe.org).  

This tool has its roots in [old C symbolic math stuff](https://github.com/maitag/lyapunov-c)  

It can do simplification, forming derivative and  
handling parameters to connect Formulas together.  


### Installation
```
haxelib git Formula https://github.com/maitag/formula.git
```

### Documentation
__Formula__ class is an [haxe-abstract](https://haxe.org/manual/types-abstract.html) to support operator-overloading from the underlaying __TermNode__ class,  
so prefer this one for instantiation:
```
var f:Formula;
```

Set up a math expression as String with new or using the "="-operator:
```
f = new Formula("1+2*3");
f = "1+2*3";
f = 7;  // supports Float to
```


__Math expressions:__

two side operators:  
`+`, `-`, `*`, `/`, `^`, `%`  

mathmatical functions:  
`log(a, b)`, `ln(a)`, `abs(a)`, `max(a,b)`, `min(a,b)`  
`sin(a)`, `cos(a)`, `tan(a)`, `cot(a)`, `asin(a)`, `acos(a)`, `atan(a)`, `atan2(a,b)`  

constants: `e()` and `pi()`  


__Naming formulas:__

To be known to 'others', you can give a formula object a name:
```
f.name = "f";
```

or alternatively name it at first position while definition (separated by a colon):
```
f = "f: 1+2*3";
```


__Parameter binding:__

Bind Formulas together by using custom literals (like variable names):
```
f = "sin(b)";  // other formula can be bind to 'b' later
```

Now define another Formula object `x` to connect to variable `b` with the 'bind()' method:
```
var x:Formula = 0;
f.bind( ["b" => x] ); 
```

Formula `x` does not necessarily has to be same named as variable inside `f`,  
but if Formula `x` has same name, its more easy:
```
x.name = "b";
f.bind(x);
```

To bind more than one variable at once, proceed like this: `f.bind( ["b" => x, "c" => c] );`  


__Output formulas:__

In a String context Formula will return the full dissolved mathmatical expression (includes all bindings):
```
trace(f); // sin(0)
```

To dissolve only at a certain level of subterms, use the `toString` method:
```
trace( f.toString(0) ); // sin(b)
trace( f.toString(1) ); // sin(0)
```

Or print out all binding levels in order with the `debug()` method:
```
f.name = "f";
f.debug(); // f = sin(b) -> sin(0)
```


__Calculating results:__

The result of a formula expression can be calculate with the `result()` method.  
Use this if no unbinded variables are left:
```
trace( f.result() ); // 0
```


__Unbinding of parameters:__
```
f.unbind(x);
// or f.unbind("b");

// unbind more than one with array usage:
f.unbind( [x, y] );
// or f.unbind( ["b", "c"] );

// unbind all with:
f.unbindAll();
trace(f); // "sin(b)"
```


### Formula API
```
new(s:String, ?params:Dynamic)
	creates an Formula object based on the string s

set(a:Formula)
	this Formula gets the same properties as a

bind(params:Dynamic):Formula
	link a variable inside of this Formula to another Formula

unbind(params:Dynamic)
	delete the connection between a variable and the linked Formula

unbindAll()
	deletes the connection between all variables of the Formula and all linked Formulas

copy()
	returns a copy of this Formula

derivate(p:String)
	returns the derivate of this mathmatical expression

toString(?depth:Null<Int>)
	returns the mathmatical expression in form of a string
	depth specifies how deep variables should be replaced by their corresponding Formulas

simplify()
	tries various ways to make the term appear simpler
	and also normalizes it
	(use with caution because this process is not trivial and can be changed in later versions)

debug()
	debugging output to see all bindings

expandAll()
	expands the term
```


### Examples
```
var x:Formula, a:Formula, b:Formula, c:Formula, f:Formula;

x = 7.0;
a = "a: 1+2*3";  // a has a defined name

f = "2.5 * sin(x-a)^2";

// change name of Formula
x.name = "x";

// bind Formulas as parameter to other Formula
f.bind(["x" => x, "a" => a]);

trace( f.toString(0) ); // 2.5*(sin(x-a)^2)

// fast calculation at runtime
trace( f.result );      // 0

// derivation           // 2.5*((sin(x-a)^2)*(2*(cos(x-a)/sin(x-a))))
trace( f.derivate("x").simplify().toString(0) );

// change value (keeps parameter bindings)
x.set("atan(a)");
x.bind(a);              // a has a defined name to bind to

trace( f.toString(0) ); // 2.5*(sin(x-a)^2)
trace( f.toString(1) ); // 2.5*(sin(atan(a)-(1+(2*3)))^2)
trace( f.toString(2) ); // 2.5*(sin(atan((1+(2*3)))-(1+(2*3)))^2)

// unbind parameter
x.unbind("a");
f.unbind(["a", "x"]); // unbind accepts array of param-names
f.unbind(a);          // unbind accepts Formula
f.unbind([x => "x", a => "x"]);  // unbind accepts Map<Formula,String>
f.unbind([a , x]);    // unbind accepts array of Formulas
f.unbindAll();        // or unbind all params

trace( f );  // 2.5*(sin(x-a)^2)

// operations with Formulas
a = "a: 1-2"; 
x = "x = 3*4";
c = 5;

f = a + x / c;
f.name = "f";

// show parameters
// c has no name, so operation will not generate param for f
trace( f.params() ); // [ "a", "x" ]

// debugging Formulas
f.debug(); // f = a+(x/5) -> (1-2)+((3*4)/5)

// simplify reduce operations with values
a.simplify();
trace( f );             // -1+((3*4)/5)

// using math functions
f = Formula.sin(c * a) + Formula.max(f, 3);
f.name = "F";
f.debug(); // F = sin(5*a)+max(f,3) -> sin(5*-1)+max((a+(x/5)),3) -> sin(5*-1)+max((-1+((3*4)/5)),3)
```


### Todo

- position of error while thrown string-parsing-error
- math expression output for programming languages (to use in glsl)
- more useful unit tests
- more ways to simplify and transform terms
- comparing terms for equality
- remove unnecessary parentheses
- !-operator


### Possible tasks in future

- handling other datatypes for values (integer, fixed-point numbers, vectors, matrices, complex numbers)
- handle recursive parameter bindings (something like x(n+1) = x(n) ...)
- definite integrals (or even indefinite later on)
- gpu-optimization for parallel calculations
