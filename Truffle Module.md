# Truffle Module

The `Truffle` module is only available when running in Truffle mode (`-X+T`).

## Primitives

* `Truffle::Primitive.binding_of_caller` returns the `Binding` of the caller of the method you call this method in

## Debug

* `Truffle::Debug.dump_call_stack` print the current call stack
* `Truffle::Debug.java_class_of(object)` return the name of the Java class used to implement an object
* `Truffle::Debug.panic` print Ruby and Java call stacks, Ruby local variables, AST traces, and exit
* `Truffle::Debug.tree` return a string of the compact AST of the current method
* `Truffle::Debug.full_tree` return a string of the full AST of the current method
* `Truffle::Debug.parse_tree` return a string of the JRuby AST
* `Truffle::Debug.flush_stdout` flush standard output
