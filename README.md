# formula
to play with mathematical formulas at haxe-runtime

This work is >in progress<  
to recode old [symbolic Math Stuff](https://github.com/maitag/lyapunov-c)  
in haxe language.  

### How to use
```
	var x:Term = Term.newValue(7);
	var a:Term = Term.fromString("1 + 2 * 3");
	var f:Term = Term.fromString("2.5 * sin(x - a) ^ 2", ["x"=>x, "a"=>a] );
	trace( "f=" + f.toString(0) + " -> " + f.toString(1) + " -> " + f.result );
	// -> f=2.5*(sin(x-a)^2) -> 2.5*(sin(7-(1+(2*3)))^2) -> 0
```

### Todo

- trim and derivation functions
- abstract Formula class around Term to type more easy