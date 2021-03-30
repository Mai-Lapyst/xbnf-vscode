# xbnf-vscode

VScode addon for Xbnf.

Xbnf is a bnf-like definition language for parsing and lexing.
It was written because there are to many variations of bnf, ebnf and abnf out there and I needed one that fit my needs.
Rather to spend eons researching wich i could use, i simply designed one myself, loesly based on ebnf and abnf.

## Writing Xbnf

First thing first, Xbnf allows comments:
```xbnf
# this is a comment

/*
 and this is a multiline comment
*/
```

A definition in Xbnf is the assosiaction of a symbol (name) with rules / syntax.
There are two kinds of definitions: 'Rules' and 'Tokens'.

### Rule

A rule is simply a definition of things that must occur in a specific order.
For example a variable definiton in a programming language can be covered by a rule:
```xbnf
VariableDefinitionRule = "var" Name ";"
```

The rule above would trigger by a string like this:
```
var i;
```

A important thing about rules is, that their parts can have any number of linear whitespace (spaces and tabulators) between them when it's atleast one whitespace apart.
Alternative string that would also trigger the rule:
```
var            i           ;
```

But this would'nt trigger the rule:
```
vari ;
```
and this neither:
```
var
i
;
```

#### Concatination-Operator

In order to allow no whitespace between parts of a rule, there is the concatination operator: `.`
When using this between two parts of a rule, the rule only triggers when there is no whitespace between the two parts:
```xbnf
NamespacedName = Name . "::" . Name;
```

Examples:
```
Hello::World        # triggers rule
Hello ::World       # dosnt triggers rule
```

Note: with the `?` specifier (see below) the concatination operator becomes optional, means that the between the two parts
of a rule can either be no whitespace at all or any number of whitespaces.

#### Or-Operator

You already know one operator: the concatination operator `.`.
But there is also the or-operator: `|` it's like the or in a regex:
```xbnf
TrueOrFalse = "true" | "false";
```
The above rule is triggered when either the string `true` or `false` occurs.

#### Grouping

Xbnf allows grouping of parts in a rule:
```xbnf
SomeRule = ("hello" "world") "somthing";
```

A group of parts itself behaves like a part, and can recievbed specifiers on it's own.
But apart of that they do not have any special features.

#### Specifiers

A specifier is used to specifiey how often a part can occur on its given place:
- `?` zero or one time (can also be applied to the concatination operator)
- `*` zero or unlimited times
- `+` one or unlimited times

This can be used for example in lists:
```xbnf
NameList = Name ("," Name)*;
```

Example input:
```
Car , Plane , Apple , Banana
```

A specifier is part of the rule-part its attached to, means that the concatination operator before the group/part is applied on the resultat of the specifier:

For example:
```xbnf
NameList1 = Name . ("," Name)*;
NameList2 = Name (. "," Name)*;
NameList3 = Name (.? "," Name)*;
```

Input:
```
Car , Plane , Apple , Banana    # NameList1 triggers
Car, Plane , Apple , Banana     # NameList3 triggers
Car, Plane, Apple, Banana       # NameList2 triggers
```

### Token

A token is a group of characters like `"true"`, thus strings are indirectly tokens.
But tokens can also behave like rules where all parts are concatinatet together, thus allowing no whitespaces between the parts.

To name a token you write its symbol all uppercase:
```xbnf
BOOLEAN = "true" | "false";

OTHERTOKEN = "abc" BOOLEAN;
```