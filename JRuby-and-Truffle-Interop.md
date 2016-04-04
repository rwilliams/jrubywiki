JRuby supports standard Truffle interop messages. This document explains what it does when it receives them, how to get it to explicitly send them, how to get it to send them using more idiomatic Ruby, and how what messages it sends for normal Ruby operations on foreign objects.

Interop ignores visibility entirely.

## How JRuby responds to messages

### `IS_EXECUTABLE`

Returns true only for instances of `Method` and `Proc`.

### `EXECUTE`

Calls either a `Method` or `Proc`, passing the arguments as you'd expect. Doesn't pass a block.

### `INVOKE`

Calls the method with the name you provided, passing the arguments as you'd expect. Doesn't pass a block.

### `HAS_SIZE`

Returns true only for instances of `Array`, `Hash` and `String`.

### `GET_SIZE`

Call `size` on the object.

### `IS_BOXED`

Returns true only for instances of `String` with a length of 1, which allows them to be unboxed as a character.

### `UNBOX`

For a `String`, returns the first character. Unboxing empty strings is not supported and will cause an `UnsupportedMessageException`. For all other objects returns the object.

### `IS_NULL`

Returns true only for the `nil` object.

### `READ`

The label must be a Java `int` or `String`, or a Ruby `String` or `Symbol`.

If the receiver is a Ruby `String` and the label is an integer, read a byte from the string, ignoring the encoding. If the index is out of range you'll get 0.

For the following conditions the label cannot be a Java `int`.

Otherwise, if the label starts with `@`, read it as an instance variable.

Otherwise, if there isn't a method defined on the object with the same name as the label, and there is a method defined on the object called `[]`, call `[]` with the label as the argument.

Otherwise, perform a method call using the label as the called method name.

In all cases where a call is made no block is passed.

### `WRITE`

The label must be a Java `String`, or a Ruby `String` or `Symbol`.

If the label starts with `@`, write it as an instance variable.

Otherwise, if there isn't a method defined on the object with the same name as the label, and there is a method defined on the object called `[]=`, call `[]=` with the label and value as the two arguments.

Otherwise, perform a method call using the label appended with `=` as the called method name, and the value as the argument.

In all cases where a call is made no block is passed.

## How to explicitly send messages from JRuby

### `IS_EXECUTABLE`

`Truffle::Interop.executable?(value)`

### `EXECUTE`

`Truffle::Interop.execute(receiver, *args)`

### `INVOKE`

Not supported.

### `HAS_SIZE`

`Truffle::Interop.size?(value)`

### `GET_SIZE`

`Truffle::Interop.size(value)`

### `IS_BOXED`

`Truffle::Interop.boxed?(value)`

### `UNBOX`

`Truffle::Interop.unbox(value)`

### `IS_NULL`

`Truffle::Interop.null?(value)`

### `READ`

Todo

### `WRITE`

Todo

## How to send messages using idiomatic Ruby

### `IS_EXECUTABLE`

Todo

### `EXECUTE`

Todo

### `INVOKE`

Todo

### `HAS_SIZE`

Todo

### `GET_SIZE`

Todo

### `IS_BOXED`

Todo

### `UNBOX`

Todo

### `IS_NULL`

Todo

### `READ`

Todo

### `WRITE`

Todo

## What messages are sent for Ruby syntax on foreign objects

Todo