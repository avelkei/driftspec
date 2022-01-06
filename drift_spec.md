# Drift language specification
v0.2 (2021-06-17)

## TL;DR

- Rust-like syntax with some differences
- using semicolons the end of lines is optional
- UTF-8 characters allowed in string and char literals, ASCII everywhere else

## Contents

- [Introduction](#introduction)
- [Keywords](#keywords)
- [Grammar notation](#grammar-notation)
- [Lexical rules](#lexical-rules)
- [Syntax](#syntax)

## Introduction

```c++
// Variables
name = ""
weight = 1.6

// Type declaration
// Required only if the type can't be inferred 
f: Float = 3.0

// Built-in primitive types:
// Int, Integer, Char, String, Bool, Float, Double
a: Char = 'A'
b: Bool = true

// Constant variables
const x = 10
// x = 12
// ^^^ Causes compile time error

// Operators
print(2 + 3)  // 5
print(2 - 3)  // -1
print(2 * 3)  // 6
print(2 / 3)  // 0.6666667
// Exponent
print(2 ^ 3)  // 8
// Modulo
print(2 % 3)  // 2
// Bitwise AND
print(2 and 3)  // 3
// Bitwise OR
print(2 or 3)   // 3
// Bitwise XOR
print(2 xor 3)  // 1
// Bitwise NOT
print(not 2)    // 1
// Bitwise Shift Right
print(2 shiftr 1) // 1
// Bitwise Shift Left
print(2 shiftl 1) // 4

// Compound types:
// List
primes = [2, 3, 5, 7]
print(primes[2])
// Output:
// 5

// List ranges
hundredNumbers = [0..100]
// Output:
// [0, 1, 2, 3, ... 97, 98, 99]

// HashMap
htmlStatusCodes = {
  404 = "Not Found",
  501 = "Not Implemented"
}
print(htmlStatusCodes[501])
// Output:
// Not Implemented

// Tuple
// Must have zero or at least two elements, a single expression enclosed
// in parentheses is treated as an expression, not a tuple
tuple = (21, 'T', "Green")
print(tuple[2])
// Output:
// Green

// Set
diceRolls = Set(3, 1, 5, 5, 4) // Same as Set(3, 1, 5, 4)

// Option type
// Holds either Nothing or some value of the given type
maybeAnInt: Option[Int] = Just(5)
maybeEmpty = Nothing

// Functions
// Type annotations are required for parameters and return type
// after the arrow (if the function returns any value)

fn square(x: Int): Int {
	return x*x
}
// The function body's last expression is implicitly treated as the return value,
// so the last return keyword is optional

// Function calls
fiveSquared = square(5)

// Function composition
squaredTwice = | square | square
// | squaredTwice([1, 2, 3, 4]) | print
// [1, 8, 27, 64]

// Use the '_' underscore character to indicate where the input is excepted to go
// in a composition.
| read("data.csv") | addTimestampToData(_, date.now()) | write("data.csv")

// Passing functions as objects
squaredNumbers = map(square, [1, 2, 3, 4, 5])
// Output:
// [1, 4, 9, 16, 25]

// Anonymous functions
tripledNumbers = map(fn (x) { return 3*x }, [1, 2, 3, 4, 5])
// Output:
// [3, 6, 9, 12, 15]

// Flow control

// Conditions
if x == null {
	print("Invalid value for calculation")
} else if x > 0 && square(x) < 0 {
	print("Something went wrong")
} else {
	print("Probably working as intended")
}

// Loops
for x in [1..10] {
	workWith(x)
}

while x < 100 {
	x *= 2
}

// Pattern matching
match number {
	Just(n) => print(n),
	Nothing => print("No value")
}

// Structs
struct Point {
	x: Int,
	y: Int
}

p = Point(4, 7)
print(p.y)
// Output: 
// 7

// Enumerations
enum Color {
	Red,
	Green,
	Blue,
	Grey[Int]
}

// Modules
// If no modules are defined in a file, the whole file is treated 
// as a single module, named with the filename, minus the file extension.
// For example: the public contents of a 'util.df' file can be imported
// using 'import util'
// Explicit example:
module util {
    pub fn upper(s: String) {
    	// ...
    }
}

// In other files:
import util {upper}
upper("uppercase") | print
// Output:
// UPPERCASE

// File input/output
contents = read("prices.csv")
write(data, "updated_prices.csv")

// Complete example

// Command line arguments are always available via the 'args' variable,
// even if it isn't included in the definition of 'main'
// Complete main function signature: 
// main(args: List[String])
fn main() {
	// args contains the space-separated command 
	// line arguments in a String list
	if args.length < 2 {
		print("Too few arguments")
		// Exit with an error code
		exit(1)
	}
	
}

// Typeclasses declare capabilities on types and their objects
// For example the 'print' functions expects values 
// which derive from the Show typeclass
typeclass Num derives [Show, Eq]

// Default implementation
impl Num {
	fn add(n1: Num, n2: Num): Num {
		return n1 + n2
	}
	// ...
}

// Implementing for a specific type
impl Show for Point {
	fn show(p: Point): String {
		return format("({p.x}, {p.y})")
	}
}
```

## Keywords

break,
const,
continue,
derives,
do,
else,
enum,
false,
for,
fn,
if,
import,
impl,
in,
match,
null,
return,
struct,
true,
typeclass,
while,

Bool,
Char,
Double,
Float,
Int,
Integer,
String

### Reserved words

class

## Grammar notation
EBNF variation as used in W3C's XML Specification \
https://www.w3.org/TR/xml/#sec-notation

## Lexical rules

### Literal expressions

```
UnicodeChar ::= [#x0-#x10FFFF]
AsciiLetter ::= [a-zA-Z]
Digit ::= [0-9]
Whitespace ::= (#x9 | #xA | #xD | #x20)+
```

```
StringLiteral ::= '"' UnicodeChar* '"'
NumberLiteral ::= Digit+ | Digit+ '.' Digit+
CharLiteral ::= "'" UnicodeChar? "'"
BoolLiteral ::= 'true' | 'false'
```

### Comments

```
CommentLine ::= '//' UnicodeChar*
```

### Identifiers

```
Identifier ::= (AsciiLetter | '_') (AsciiLetter | Digit | '_')*
```

## Syntax

### Type names

```
Type ::= 
    Identifier
    | Identifier '[' Type ']'
    | 'Int'
    | 'Integer'
    | 'Char'
    | 'String'
    | 'Bool'
    | 'Float'
    | 'Double'
```

### Variables

```
TypeAnnotation ::= ':' Type
StatementEnd ::= ';' | '\n'
VarDefStatement ::= 'pub'? 'const'? Identifier TypeAnnotation? '=' Expression

ElementAccessExpression ::= VariableExpression '[' ValueExpression ']'
MemberAccessExpression ::= VariableExpression '.' VariableExpression ('.' VariableExpression)*
VariableExpression ::= 
    Identifier
    | ElementAccessExpression
    | MemberAccessExpression

AssignmentStatement ::= 
    VariableExpression '=' Expression
    | VariableExpression '+=' Expression
    | VariableExpression '-=' Expression
    | VariableExpression '*=' Expression
    | VariableExpression '/=' Expression
    | VariableExpression '%=' Expression
```

### Expressions

```
Expression ::= 
    ValueExpression
    | ListExpression
    | RangeExpression
    | HashMapExpression
    | TupleExpression
    | CompositionExpression
    | AnonymousFunction
    | '(' Expression ')'

ValueExpression ::= 
    VariableExpression
    | LiteralExpression
    | OperatorExpression
    | FunctionCallExpression
    | '(' ValueExpression ')'
    | 'null'

LiteralExpression ::= 
    StringLiteral
    | NumberLiteral
    | CharLiteral
    | BoolLiteral

OperatorExpression ::=
    UnaryOperatorExpression
    | BinaryOperatorExpression
    | LogicalOperatorExpression

UnaryOperatorExpression ::=
    'not' ValueExpression
    | '-' ValueExpression
    | '!' ValueExpression

BinaryOperatorExpression ::=
    ValueExpression '+' ValueExpression
    | ValueExpression '-' ValueExpression
    | ValueExpression '*' ValueExpression
    | ValueExpression '/' ValueExpression
    | ValueExpression '%' ValueExpression
    | ValueExpression 'and' ValueExpression
    | ValueExpression 'or' ValueExpression
    | ValueExpression 'xor' ValueExpression
    | ValueExpression 'shiftl' ValueExpression
    | ValueExpression 'shiftr' ValueExpression

LogicalOperatorExpression ::=
    ValueExpression '&&' ValueExpression
    | ValueExpression '||' ValueExpression
    | ValueExpression '==' ValueExpression
    | ValueExpression '!=' ValueExpression
    | ValueExpression '<' ValueExpression
    | ValueExpression '<=' ValueExpression
    | ValueExpression '>' ValueExpression
    | ValueExpression '>=' ValueExpression

ListExpression ::= '[' Expression* ']'
RangeExpression ::= LiteralExpression '..' LiteralExpression
HashMapExpression ::= '{' HashMapEntries? '}'
HashMapEntries ::= HashMapEntry (',' HashMapEntry)*
HashMapEntry ::= ValueExpression '=' Expression
    
TupleExpression ::= '()' | '(' Expression ',' ValueList ')'
CompositionExpression ::= '|' Expression ('|' Expression)+
```

### Functions

```
FunctionCallExpression ::= VariableExpression '(' ValueList? ')'
ValueList ::= Expression (',' Expression)*

TypedDeclaration ::= 'pub'? Identifier TypeAnnotation
FunctionDefinition ::= 'pub'? 'fn' Indentifier '(' ParameterList? ')' TypeAnnotation? '{' StatementBlock ReturnStatement? '}'
ParameterList ::= TypedDeclaration (',' TypedDeclaration)*
ReturnStatement ::= 'return' Expression?

AnonymousFunction ::= 'fn' '(' ParameterList? ')' TypeAnnotation? '{' StatementBlock ReturnStatement? '}'
```

### Statements

```
StructDefinition ::= 'struct' Identifier ('derives' ListExpression)? '{' ParameterList '}'
EnumDefinition ::= 'enum' Identifier '{' EnumListing '}'
EnumListing ::= EnumMember (',' EnumMember)*
EnumMember ::= Identifier | Identifier '[' Type ']'

TypeclassDefinition ::= 'typeclass' Identifier ('derives' ListExpression)?
ImplStatement ::= 'impl' Identifier ('for' Identifier)? '{' FunctionDefinition* '}'

ForStatement ::= 'for' Identifier 'in' Expression '{' StatementBlock '}'
IfStatement ::= 
    'if' Expression '{' StatementBlock '}'
    | 'if' Expression '{' StatementBlock '}' 'else' ('if' Expression '{' StatementBlock '}')* '{' StatementBlock '}'
WhileStatement ::= 'while' Expression '{' StatementBlock '}'
MatchStatement ::= 'match' Expression '{' MatchBody '}'
MatchBody ::= MatchBranch (',' MatchBranch)*
MatchBranch ::= Expression '=>' Expression

ImportStatement ::= 'import' Identifier ('{' Identifier (',' Identifier)* '}')?
ModuleStatement ::= 'module' Identifier '{' StatementBlock '}'

// Same grammar as FunctionCallExpression, but treated as a statement if found 
// at the top level of its scope and it's followed by a StatementEnd token.
FunctionCallStatement ::= Identifier '(' ValueList? ')'

CompositionStatement ::= '|' Expression ('|' Expression)+


StatementBlock ::= Statement*
Statement ::=
    ( VarDefStatement
    | AssignmentStatement
    | FunctionDefinition
    | FunctionCallStatement
    | CompositionStatement
    | StructDefinition
    | EnumDefinition
    | TypeclassDefinition
    | ImplStatement
    | ForStatement
    | IfStatement
    | WhileStatement
    | MatchStatement 
    | ImportStatement
    | ReturnStatement
    | ModuleStatement ) StatementEnd
```

## Operator precedence

| Syntax | Associativity |
| ---- | ---- |
| Method calls | left-to-right |
| Function calls | left-to-right |
| `^` | left-to-right |
| Unary `!` `not` `-` | left-to-right |
| `*` `/` `%` | left-to-right |
| `+` `-` | left-to-right |
| `shiftl` `shiftr` | left-to-right |
| `and` | left-to-right |
| `xor` | left-to-right |
| `or` | left-to-right |
| `==` `!=` `<` `<=` `>` `>=` | left-to-right |
| `&&` | left-to-right |
| <code>&#124;&#124;</code> | left-to-right |
| <code>&#124;</code> | left-to-right |
| `=` `+=` `-=` `*=` `/=` `%=` | right-to-left |
