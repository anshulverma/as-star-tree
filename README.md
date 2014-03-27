Multidimensional Abstract Syntax Tree or AS*-Tree
=======

_A proposal for a multidimensional Abstract Syntax Tree implementation
applied for building syntax for an expression._

Syntax tree represent the syntactic structure of source code. The
abstract property is due to the fact that it makes grouping and ordering
implicit based on the structure of the tree. For instance, grouping
parantheses are implicit in the tree structure. Based on this
definition, a `AS*-Tree` is nothing but a AST. The difference becomes
aparent in the following section when we introduce multidimentionality
principle.

## Binary Expression Tree

From [wikipedia](http://en.wikipedia.org/wiki/Binary_expression_tree):

> A __binary expression tree__ is a specific implementation of a binary
> tree to evaluate certain expressions.

Here is an example of a binary tree from the same wikipedia article:
![example of a binary tree]
(http://upload.wikimedia.org/wikipedia/commons/5/55/Exp-tree-ex-12.svg
 "Binary tree example")

This tree serializes to an algebraic expression: 
`((5 + z) / -8) * (4 ^ 2)`

Looking at this example we can see that it makes perfect sense to
structure the tree the way it is. So, you may wonder what is the need of
a completely new type of tree? Can't we use binary tree to represent any
sort of expression? Yes, a binary expression tree is more than sufficient
to represent any sort of expression (algebraic, logical etc.). The need
of `AS*-tree` stems from the fact that a binary expression tree is
redundant by nature for operators that satisfy the associative law. This
is explained in much detail in the following section.

Also, I should note, even though the tree above looks balanced,
a expression tree (or a syntax tree) inherently is not balanced. To
understand this just modify the previous expression slightly to:
`(4 ^ 2) * ((5 + z) / -8)`. This will not be solved by `AS*-Tree`.


### Associative property of operators

> A binary operation is associative, then pairs of parantheses can be
> inserted at any place without affecting the outcome of the expression
> as a whole.

To demonstrate this, consider this example of the multiplication operator
`*` applied on a set _S_ (set of numbers):

```text
(x * y) * z = x * (y * z) for all x, y, z ∈ S
```

## Redundancy problems of binary expression tree

We are applying this data structure to the use case of building
expression trees. We are also aiming to remove the redundancy inherent
to a regular expression tree by using the multidimensionality feature of
`AS*-tree`. Before we explain multidimentiality, lets understand the
redundancy problem of binary expression trees.

### Redundancy of operators in binary expression tree

Consider this algebraic expression: `a * b * c * d` (please note `*` is
associative so adding parantheses does not affect the outcome of this
expression). This expression can be represented by the following binary
expression tree:

![Example of redundant binary expression tree](https://raw.githubusercontent.com/anshulverma/as-star-tree/master/images/redundant-binary-expression-tree.jpeg "Redundant nature of binary expression tree")

There are implementations that may generate a binary expression tree in
slightly modified form. That being said, the end result will always have
multiple operator nodes (i.e. nodes representing `*`).

The important thing to take away from this case is the fact that there
are several operator nodes (`*`). There is no way to group the
operations under one operator node since we are restricted by the fact
that we are dealing with a __binary__ tree.

### Redundancy of parent-child relationship in binary expression tree

Consider a binary expression `a * (b * c)`. This binary expression can
be represented as a binary expression tree as:

![Example of Parent-Child redundancy](https://raw.githubusercontent.com/anshulverma/as-star-tree/master/images/parent-child-redundancy.jpeg "Example of parent-child redundant expression tree")

Again, this is just one of the ways to represent the example expression
in a expression tree. The main takeaway from this is the fact that we
still can't reduce this tree to two levels (operator and operands)
because of the __binary__ nature of this tree.

## Motivation

The question to ask is: _So what if a tree has redundant operator nodes, it still does the job by
clearly representing an expression?_

We cannot answer away this question by merely tying the redundancy
problem with space consumption. Even though space is wasted, we may not
be concerned by it since, on average expression trees are not too big to
process in memory.

The problem becomes more aparant when you try to represent this data
structure in any language or serialize/deserialize it. Due to redundancy
there will be a lot of nesting which may make the data hard to parse.
Lets try to serialize a binary expression tree in JSON form.

Assuming a node in the tree is represented as:

````json
{
    "operator"    : "if operator node, this will contain the operator type",
    "data"        : "if operand node, this will contain the operand value",
    "left-child"  : { "optional, may contain another node" },
    "right-child" : { "optional, may contain another node" }
}
````

If we try to serialize the algebraic expression from the first example
(`a * b * c * d`) into JSON form using above structure, we will end up
with this:

````json
{
    "operator": "*",
    "left-child": {
        "operator": "*",
        "left-child": {
            "operator": "*",
            "left-child": {
                "data": "b"
            }
        },
        "right-child": {
            "data": "c"
        }
    },
    "right-child": {
        "data": "d"
    }
}
````

As you can imagine from the example above, the level of nesting can
increase pretty fast as the size of expression increases. The main
motivations of `AS*-Tree` are as follows:

- To help a developer who is looking to develop a service to
read/process/write expression tree. If we can simplify the structure of
a expression tree and provide function to apply reduction
transformations to remove redundancy, it would go a long way to simplify
the development process.

- For an untrained eye, a deeply nested tree representation can become
unreadable quickly. To simplify such a tree is to increase readablity of
representation.

## Multidimensionality

Looking at the tree above, you may notice the redundancy of the `*`
node. The reason for multiple operator nodes is the binary nature of
this tree. How about we remove that limitiation and represent this tree
as:

![Example of AS*-Tree](https://raw.githubusercontent.com/anshulverma/as-star-tree/master/images/optimized-as-star-tree.jpeg "Optimized AS*-Tree example")

The multidimentionality feature allows us to reduce this tree to one
level by replacing the `*` node at 2nd level (sibling of `a`) with nodes
`b` and `c` (which results in three nodes at the first level. The
transformed tree looks like this:

![Optimized Parent-Child AS*-Tree](https://raw.githubusercontent.com/anshulverma/as-star-tree/master/images/parent-child-optimized.jpeg "Example of Parent-Child redundancy reduction")

Also, it must be noted that this transformation cannot be applied to any
tree. As mentioned before, the parent operator and grandparent operator
must be same for the applicability of this transformation. For instance,
nothing can be done to this tree (representing `a * (b + c)`):

![Non reducable tree](https://raw.githubusercontent.com/anshulverma/as-star-tree/master/images/parent-child-non-redundancy.jpeg "Example of Non-Reducable binary expression tree")

