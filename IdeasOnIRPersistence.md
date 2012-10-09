# IR Persistence

## Introduction

Our GSoc project on IR Persistence lead to several interesting discoveries.  The overhead of parsing generic grammars can be quite a bit more expensive than we thought.  This document is talking about a new proposed grammar along with some hand-waving arguments why I think they will be quicker.

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

## Removing interning and knowing more about variability

When we persist a scope in IR we actually know how many string literals exist and what they are.  We could dump those out as a prologue to the actual instructions.  When loading that prologue we will only intern() those values and never bother to intern after that point.  In our Ruby parser we are stuck calling intern over and over because we cannot know whether the string has already been interned (in a way which would not defeat the performance penalty of intern'ing the world).

Also if we know we have n local variables we can record that and then pre-allocate a single array for it.  This means no growing.  It means no bounds checking at the cost of reading a numeric value in the prologue.

The main detriment to this is the cost of parsing this prologue info.  This cure could be worse than the disease, but I think this can be added after our first swipe as a potential set of optimizations.
