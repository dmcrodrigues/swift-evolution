# Invert the order of pattern match operator

* Proposal: [SE-NNNN](NNNN-invert-order-of-pattern-match-operator.md)
* Author(s): [David Rodrigues](http://github.com/dmcrodrigues)
* Status: Awaiting review
* Review manager: TBD

## Introduction

This proposal seeks to improve the intent and expressivity of pattern match operator with the inversion of its order.

## Motivation

Pattern match operator, `~=`, performs a match between a value and a certain pattern, e.g. checking if an integer value is contained in a range of integers, and is fundamentally used to support expression patterns in `switch` statement case labels.

```swift
let point = (2, 4)
switch point {
case (0, 0):
    print("The point is at the origin")
case (0...4, 0...4):
    print("The point is in the subregion")
default:
    break
}
```

The second `switch` case can be represented alternatively using the pattern match operator.

```swift
if 0...4 ~= point.x && 0...4 ~= point.y {
    print("The point is in the subregion")
}
```

The operator may be unknown for many but it can be very handy in a few situations as, for example, in the value-binding pattern.

```swift
let point = (2, 4)
switch point {
case (let x, let y) where 0...4 ~= x && 0...4 ~= y:
    print("The point is in the subregion")
default:
    break
}
```

The current syntax, despite being entirely correct, it may not be the most expressive and clear way to represent our intent. For example, let's consider that we want to validate if a integer is in a certain interval.

```swift
let x = 5
0...8 ~= x
```

By analyzing the code we can see that in reality what we're doing is validating if a certain interval contains a certain integer which is exactly the opposite of what we state above. This may be debatable but most of the time we validate certain conditions around our value and not the other way around.

```swift
x > 0 && x < 10
// vs 
0 < x && 10 > x

x == 10
// vs
10 == x
```

Currently we cannot perform validations using the pattern match operator around our value which has an impact in clarity and expressiveness of our code since we are forced to state something like "*if blue is the ocean*" instead of "*if the ocean is blue*".

### Pattern matching in other languages

This concept can also be found in other languages but there's a incoherence between the syntax adopted in Swift with the other operators/concepts already available.

##### Ruby

Ruby has the same operator (is actually inverted) to match a string against a regular expression with a small but important difference, the operator is 
commutative.

```ruby
/needle/ =~ 'haystack'
'haystack' =~ /needle/
```

##### Perl

Perl has the binding operator with the same behavior found in Ruby, it matches a string against a regular expression.

```perl
$haystack =~ /needle/
```

##### Databases

Pattern matching can also be found in relational databases using the traditional SQL `LIKE` operator.

```SQL
haystack LIKE 'needle'
```

### Good Practices

##### Yoda conditions

The current syntax is aligned with the so called Yoda conditions which places the literal value of the expression on the left side and the variable on the right side. They're a good practice to achieve code that's more safe in certain extent, e.g. detects accidental assignments, solves unsafe null behaviour. But nothing comes free, in order to achieve more safety we are compromising code readability.

However, Swift does not benefit in adopting this kind of programming style mainly because of his design:

- An assignment does not return the value assigned;
- Optionals prevents unsafe null behaviour (unless you force-unwrap which you should not).

Summing up, the current syntax could be relevant for languages like C/C++ or Java but in Swift there isn't a clear gain in adopting it with the penalty of compromising the expressivity and legibility of the code.

## Proposed Solution

By reversing the order of the operator, we can, in a certain way, become more aligned with our logical thought by thinking if a value can be matched in a certain pattern or not, and consequently promote a deeper expressivity of the code. With this change we may also be promoting the goals declared in the [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/#use-terminology-well) like the clarity at the point of use and sticking to the established meaning.

```swift
let x = 5
x =~ 0...8 // Proposed
// vs
0...8 ~= x // Current

case (let x, let y) where x =~ 0...4 && y =~ 0...4: // Proposed
// vs
case (let x, let y) where 0...4 ~= x && 0...4 ~= y: // Current
```

## Detailed design

> Ongoing

## Impact on existing code

Any code that is using the operator will be affected but since we are just changing the order and inverting the operator, it should be easy to automate and include in Swift migrator.
