# IR Persistence

## Introduction

Our GSoc project on IR Persistence lead to several interesting discoveries.  One discovery was that the overhead of parsing generic grammars can be quite a bit more expensive than we thought.  This document is talking about a new proposed grammar along with some hand-waving arguments why I think they will be quicker.

Another quick statement about one our goals.  We really want this format to be readable so we can
- Read it (duh)
- Make test cases by hand (like writing assembly)
- Easier to debug

## An instruction by any other name

We basically have two choices for encoding instructions.  Parameterizing all state of an instruction, but leaving the instruction name as part of the serialization format. We can also take those parameters and encode them into 'special' names for that particular type of instruction.  Let's consider these two trivially:

```text Parameterizing
%v_1 = Call(self, functional, "require", [%v_2], %cl_1)
1    2 3   45   6 7         8 9        0 12   34 5    6   Tokens
1                             1           2      3        Allocs
```

And instruction encoding:

```text Instruction encoding
%v_1 = sfcall&("require", [%v_2], %cl_1)
1    2 3      45        6 78   90 1    2  Tokens (same allocs)
```

One thing falls out of this right away.  We are generating less tokens when we encode parameter information into the instruction.  Is this good or bad?  I suspect it is good since I think the lexer will differentiate between these instruction strings faster than we can bounce around in an LALR parser after several yynext() calls.  This is pure speculation on my part based on the idea that a lexer can make a near optimal speed match of simple permutations of strings.  A parser might be the same speed at the cost of extra tokens (Note: Is this a crazy assumption?).

## Variability means allocation

If we accept encoding information into the instruction is useful we can then encode more information into it to remove the need to do stuff like:

```text 
// ListNode:word_list - collection of words (e.g. %W{a b c}) with out delimeters [!null]
word_list      : /# none #/ {
                   $$ = new ArrayNode(lexer.getPosition());
	       }
	       | word_list word ' ' {
                   $$ = $1.add($2 instanceof EvStrNode ? new DStrNode($1.getPosition()).add($2) : $2);
	       }
```

Besides n extra production transitions we are also standing on an arbitrary list data structure.  This can be surprisingly costly when you care about cold performance (Note: language parsers on the JVM care about cold perf).

Let's consider an instruction set which encodes most common arities into a call to remove variability:

```text N-arity examples
%v_1 = sf1call("require", %v_2)
%v_1 = sf1call&("require", %v_2, %cl_1)
%v_1 = sf2call("+", %v_2, %v_3)
%v_1 = sfncall&("require", %v_1, %v_3, %cl_1)
```
Because we can anchor productions off of the encoded instruction token we can know exactly how many arguments will be past it and stuff those directly into the instruction OR in the case where we do want a list after parsing we know the lists exact size.

Another thing you will notice is that [] is not in those examples.  Since we know the arity we no longer need grammar to delineate a list anymore.

Ah there is one caveat above I should have mentioned.  We cannot encode all arities possible in the language this way so we still need an 'n' version.  This version needs to dynamically create a list like shown above.  This will be a rarity though and should impact performance except extreme cases.

## What is whitespace after all

I argue that most of this extra syntax is unneeded sugar and is not needed by the grammar to parse this output.  I am correct :)  The obvious defect of not having extra syntax is not having nice visual cues.  There is a simple way to avoid the cost of having this extra syntax...Designate it as whitespace.  So:

```text
%v_1 = sfncall&("require", %v_1, %v_3, %cl_1)
```

is exactly the same as:

```text
%v_1 sfncall& "require" %v_1 %v_3 %cl_1
1    2        3         4    5    6 Tokens
```

So our persister can just dump the extra syntax to make it more readable but the lexer will strip it out as part of whitespace detection and never pass it to the parser as tokens.

There is one big design issue with this idea.  You need to make absolutely sure those characters can be treated as whitespace.  If anything else in the language needs them then you need to start using contexts in the lexer to be able to provide those.  I don't consider this a big issue for our project.

## Removing interning and knowing more about variability

When we persist a scope in IR we actually know how many string literals exist and what they are.  We could dump those out as a prologue to the actual instructions.  When loading that prologue we will only intern() those values and never bother to intern after that point.  In our Ruby parser we are stuck calling intern over and over because we cannot know whether the string has already been interned (in a way which would not defeat the performance penalty of intern'ing the world).

Also if we know we have n local variables we can record that and then pre-allocate a single array for it.  This means no growing.  It means no bounds checking at the cost of reading a numeric value in the prologue.

The main detriment to this is the cost of parsing this prologue info.  This cure could be worse than the disease, but I think this can be added after our first swipe as a potential set of optimizations.

## Use it when you need it

There is some amount of parsing which always needs to be done when loading a .rb file.  However, if you consider most loaded methods in a class are never used then we should consider ways of being lazier and delay the amount of parsing we do.  A hard scope boundary like a IRMethod can just save an offset to the section of a persisted file and load it on-demand when that method is first referenced.  The details of this will highly change the actual format used for IR persistence.  We can do a simple parse to find end boundary and not actually process anything or we can have pre-calculated offsets in an index and design the format around being able to seek around in the file.

Pre-calculated offsets really pushes this towards a binary format.  A human-readable format is nice for debugging purposes and for generating unit tests, but any character additions/subtractions will mess up all offsets.  If we have a binary format, then it makes sense to have an assembler and disassembler.

## Binary Format/Deferred Notes
### constant pool(s)

Extra notes on intern'ing.  If we defer execution of methods we may need to have n segmented constant pools.  One for mandatory section and m additional pools for each method?  This will potentially mean extra interning since each method might use the same variable names, but that will be much less than what happens in current AST parser.

If all entries in constant pool are fixed width (with data pool for variable length values) then we can have something like

```text
   offset 0
   data pool
   offset n
   constant pool (all fixed width for all things)
   offset m
   mandatory instrs
   offset p
   optional method1
...

A single constant pool for all literals since we need type + data.  A backing table on load will know if a constant has been loaded or not based on offset index.  If not then it needs to reify the constant and store it (all Operand types).  This means only the actual constants referred to will get stood up only interning exactly what is used.

### Tagged bits for constant pool

instr arg_count arg1 ... argn

Tagged arg values will know to draw actual values from constant pool.  If an arg does not fit into word size it should be stored in constant pool with a tagged index.  The other decision to make is word-size.
