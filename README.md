A data expression language (DEL)
================================

This package implements a simple language for representing data as
compact expressions.

Concept
-------

The language is designed to represent trees of data in a manner that is
human readable and human-accessible.

Each node object defines either an identifier name-space of its
children.  The names of the children may not be unique but will maintain
strict ordering.

Each leaf object has a single typed value and no children.

Syntax
------

### Leaf objects

A leaf object is constructed simply through its dedicated syntax.

All leaf objects are contained within an attribute. An attribute
associates an identifier as a name with one of the leaf objects.

The syntax for an attribute begins with a `@` followed by the name of
the attribute, an optional leaf node, and then the `;` object
terminator.

The following example would produce an attribute with the name
`attribute-name` and the identifier value `attribute-value`:

```
@attribute-name attribute-value;
```

##### Documentation comments

A comment is prefixed by a single `#` character and a space separator or
any number of adjacent `/` characters followed by a space separator and
is terminated with with a Unicode line or paragraph separator. Every
comment describes a `comment` object constructor.

Multi-line comments begin with a single `/` followed by any number of
`*` and a single space separator. Each line starts with some number of
spaces, a `*` character, then a single space separator. They end with
any number of  `*` character followed by a `/` character.

Every comment object is appended to the next object that is not a
comment object as an attribute with the name `doc` with the text
dedented and trimmed in a `comment` object.

#### Identifiers

An identifier is any combination and number of Unicode letters, numbers,
connectors, marks, and dashes that does not start with a Unicode number
character.

#### Paths

Paths are a sequence of member specifiers (a `.` followed by an
identifier) and indices (a token tree surrounded by `[` and `]`).

A member specifier will select the first node or attribute with a name
matching the specifier and an index will select the element at that
index of the item (wherein the token tree is reduced to an integer).

#### Text

Raw text is encapsulated within a pair of single quotes (`'`).

Escaped text is encapsulated within a pair of double quotes (`"`).

##### Escaping

Escaping simply uses a `\\` character.

##### Dedenting

Using three quotation markers instead of one at either end will remove
the leading white-space from the first line from all subsequent lines
and all trailing white-space that is not a line or paragraph separator.
There must be a line or paragraph separator after the opening quotation
marks.

#### Floats & integers

A series of decimal values and underscores is an integer. A pair of
these delimited by an adjacent period is a floating-point value.

The `0b` prefix exists for binary values, `0o` for octal, `0x` for
hexadecimal, and `0d` for decimal. The `0bu`, `0ou`, `0xu`, `0du`, and
`0u` prefixes exist for unsigned variants.

#### Tokens

Raw tokens will only ever appear in token-trees.

Tokens are either identifiers, integer tokens, floating-point tokens, or
single Unicode punctuation, symbols, separators, and other characters
appear as the bare character as a single token.

#### Token sequence

A token sequence is a sequence of raw tokens from the parser with
balanced Unicode brackets. The sequence starts with (but does not
include) a `(` and ends with a `)` after all internal brackets have been
balanced.

This allows for the embedded logic language.

### Node objects

A node object is constructed using the following sequence of items:

1. a constructor name (as an absolute path or identifier),
2. an optional item name (as an identifier), and finally
3. a sequence of objects, or
4. an object terminator.

There are two means of providing an object terminator. The first is to
simply use a `;` token and the second is to encapsulate a sequence of
objects within a pair of `{` and `}` where the closing brace also acts
as an object terminator.

As such, the most simple expression of such an object could appear as:

```
object root;
```

An object with many children could be represented using:

```
object root {
  child a {
    child x;
    child y;
    child z;
  }
  child b;
  child c;
}
```

Validation schemas
------------------

A validation schema may be written in DEL as follows.

### The schema of the schema

```del
/// The schema for the DEL schema.
.del.schema del {
  /// A schema definition.
  .del.node schema {
    /// A schema must provide a name.
    @name required;

    .del.children {
      /// A node constructor may be used here.
      @node .del.node;
      /// A schema may contain further schemas.
      @node .del.schema;
    }
  }

  /// A node object descriptor.
  .del.node node {
    /// Every node must have a name.
    @name required;

    /// A valid child in the node.
    .del.attribute name {
      @value required;
      @value optional;
      @value dissalowed;
    }

    .del.ordered-children {
      @attribute name;
    }
    .del.children {
      @node .del.attribute;
    }
    .del.children {
      @node .del.children;
      @node .del.any-child;
    }
  }

  /// An attribute descriptor.
  .del.node attribute {
    /// Every attribute must be named.
    @name required;

    /// The attribute may be left empty.
    .del.attribute no-value {
      @no-value;
    }
    /// The attribute may have a text value.
    .del.attribute text-value {
      @no-value;
    }
    /// The attribute may have a signed integer value.
    .del.attribute signed-value {
      @no-value;
    }
    /// The attribute may have an unsigned integer value.
    .del.attribute unsigned-value {
      @no-value;
    }
    /// The attribute may have a floating-point value.
    .del.attribute float-value {
      @no-value;
    }
    /// The attribute may have any identifier as a value.
    .del.attribute identifier-value {
      @no-value;
    }
    /// The attribute may have a path as a value.
    .del.attribute path-value {
      @no-value;
    }
    /// The attribute may have a token sequence as a value.
    .del.attribute token-value {
      @no-value;
    }
    /// The attribute may have a value matching an exact token.
    .del.attribute value {
      @identifier-value;
    }
    /// The attribute may have any possible value.
    .del.attribute any-value {
      @no-value;
    }

    /// An attribute may allow for mutliple different kinds of values to
    /// be accepted.
    .del.children {
      @maximum 1;
      @attribute any-value;
      .del.children {
        @attribute no-value;
        @attribute text-value;
        @attribute signed-value;
        @attribute unsigned-value;
        @attribute float-value;
        @attribute identifier-value;
        @attribute path-value;
        @attribute token-value;
        @attribute value;
      }
    }
  }

  /// A reference to a particular item as an attribute.
  .del.node reference {
    @name required;

    /// Items constructed with a possible constructor that may be
    /// referenced.
    ///
    /// This item must exist in the schema namespace.
    /// Either an identifer must be provided, which will match the first
    /// and most local item with that name or a path relative to the
    /// root.
    .del.reference item {
      @item .del.node;
    }

    .del.children {
      @reference item;
    }
  }

  /// Children that a node may have in no particular order.
  .del.node children {
    @name disallowed;

    /// The minimum number of matching children.
    .del.attribute minimum {
      @integer-value;
    }
    /// The maximum number of matching children.
    .del.attribute maximum {
      @integer-value;
    }

    /// The name of a node that may appear in the element.
    .del.reference node {
      @item .del.node;
    }
    /// The name of an attribute that may appear in the element.
    .del.reference attribute {
      @item .del.attribute;
    }
    /// The name of a reference that may appear in the element.
    .del.reference reference {
      @item .del.reference;
    }

    .del.children {
      @maximum 1;
      @attribute minimum;
    }
    .del.children {
      @maximum 1;
      @attribute maximum;
    }
    .del.children {
      @node .del.ordered-children;
      @node .del.children;
      @node .del.any-child;
      @reference attribute;
      @reference reference;
    }
  }

  /// Children appearing in a particular order.
  .del.node ordered-children {
    @name disallowed;

    /// The name of a node that may appear in the element.
    .del.reference node {
      @item .del.node;
    }
    /// The name of an attribute that may appear in the element.
    .del.reference attribute {
      @item .del.attribute;
    }
    /// The name of a reference that may appear in the element.
    .del.reference reference {
      @item .del.reference;
    }

    .del.children {
      @node .del.children;
      @node .del.any-child;
      @reference node;
      @reference attribute;
      @reference reference;
    }
  }

  /// Match any node that has been defined in a known schema.
  .del.node any-child {
    @name disallowed;
  }
}
```

Logic extension
---------------

The logic module and filter defines a set of items that allow for
programmatic generation of a structure.

The `emit` item will evaluate an expression that builds a sequence of
items and then substitute itself for the items that were generated.

The `let` item will define a function with any number of arguments and a
expression that is evaluated with the function is invoked.

The `gen` item will define a function that takes a number of arguments
and produces a function that appends all of its internal items to the
sequence passed to it.

```del
.del.schema logic {
  .del.node module {
    @name required;

    .del.children {
      @node .logic.module;
      @node .logic.let;
    }
  }

  /// Emit takes an expression that produces a function from a sequence
  /// to a sequence and passes it the empty sequence. It then emits each
  /// of the items from the sequence in order.
  /// 
  /// Emit will reverse the order of items in the expression so that
  /// items will be returned in the emitted that they were added.
  .del.node emit {
    @name disallowed;

    .del.attribute sequence {
      @token-value;
    }

    .del.ordered-children {
      @attribute expression;
    }
  }

  /// A generator is a function that appends items to a sequence of
  /// items.
  .del.node gen {
    @name required;

    /// Bind a value to a name when the generator is invoked.
    .del.attribute argument {
      @identifier-value;
    }

    .del.children {
      @node .logic.let;
      @node .logic.emit;
      @node .logic.item;
    }
  }
  
  .del.node item {
    .del.any-child;
  }

  /// A function that may be used in logic expressions.
  .del.node let {
    @name required;

    /// Arguments may be leaf values or other functions.
    .del.attribute argument {
      @identifier-value;
    }
    /// The body is an expression.
    .del.attribute body {
      @token-value;
    }

    .del.children {
      @attribute argument;
    }
    .del.children {
      @node .logic.let;
    }
    .del.ordered-children {
      @attribute body;
    }
  }
}
```

The logic defines an additional `gen` node for defining functions with
inferred and recursive typing that produce a sequence of nodes and
attributes.

The logic also defines an additional `let` node for defining 

The logic represents sequences of items as function that will apply the
first item to its only argument and the remainder of its items to the
result.

The logic package also provides the following items:

```del
.logic.module logic {
  /// Boolean logic
  .logic.module boolean {
    /// true :: forall a. forall b. a -> b -> a
    .logic.let true {
      @argument true;
      @argument false;
      @body (true);
    }

    /// false :: forall a. forall b. a -> b -> b
    .logic.let false {
      @argument true;
      @argument false;
      @body (false);
    }

    /// if :: forall a. (a -> a -> a) -> a -> a -> a
    .logic.let if {
      @argument cond;
      @argument if-true;
      @argument if-false;
      @body (cond if-true if-false);
    }
  }

  /// Sequence operations
  ///
  /// A sequence is defined as
  /// `[a] :: rec s. forall b. (a -> s -> b) -> b`.
  .logic.module sequence {
    /// head :: forall a. [a] -> a
    .logic.let head {
      @argument item;
      @body (item true);
    }

    /// tail :: forall a. [a] -> [a]
    .logic.let tail {
      @argument item;
      @body (item false);
    }

    /// empty is a built-in to determine if a sequence is empty.
    /// empty :: forall a. forall b. [a] -> (b -> b -> b)

    /// nil :: rec s. forall a. forall b. (a -> s -> b) -> b
    .logic.let nil {
      @argument op;
      @body (error "Cannot destructure an empty list");
    }

    /// cons :: forall a. forall b. a -> [a] -> (a -> [a] -> b) -> b
    .logic.let cons {
      @argument item;
      @argument sequence;
      @argument op;
      @body (op item sequence);
    }

    /// map :: forall a. forall b. (a -> b) -> [a] -> [b]
    .logic.let map {
      @argument op;
      @argument sequence;
      @body (
        if (empty sequence)
          nil
          (cons (op $ head sequence) (map op $ tail sequence))
      );
    }

    /// filter :: forall a. forall b.
    ///   (a -> ([a] -> [a] -> [a])) -> [a] -> [a]
    .logic.let filter {
      @argument cond;
      @argument sequence;
      @body (
        if (empty sequence)
          nil
          (if (cond $ head sequence)
            (cons (head sequence) $ filter $ tail sequence)
            (filter $ tail sequence))
      );
    }

    /// fold :: forall a. forall b. (b -> a -> b) -> b -> [a] -> b
    .logic.let fold {
      @argument op;
      @argument accumulator;
      @argument sequence;
      @body (
        if (empty sequence)
          accumulator
          (fold ((op accumulator) $ head sequence) $ tail sequence)
      );
    }

    /// len :: forall a. [a] -> uint
    .logic.let len {
      @argument sequence;

      .logic.let inc {
        @argument acc;
        @argument item;
        @body (acc + 1);
      }

      @body (fold inc 0 sequence);
    }

    /// take :: forall a. uint -> [a] -> [a]
    .logic.let take {
      @argument count;
      @argument sequence;

      /// unwrap-seq :: forall a. forall b. a -> b -> a
      .logic.let unwrap-sequence {
        @body (true); 
      }

      /// unwrap-count :: forall a. forall b. a -> b -> b
      .logic.let unwrap-count {
        @body (false); 
      }

      /// wrap :: forall a. forall b. forall c.
      ///   a -> b -> (a -> b -> c) -> c
      .logic.let wrap {
        @argument sequence;
        @argument count;
        @argument unwrap;
        @body (unwrap sequence count);
      }

      /// op :: forall a. forall b. ([a], int) -> a -> ([a], int)
      .logic.let op {
        @argument pair;
        @argument sequence;
        @body (
          if ((unwrap-count pair) == 0)
            pair
            (wrap
              (cons (head sequence) (unwrap-sequence pair))
              ((unwrap-count pair) - 1))
        );
      }

      @body (unwrap-sequence $ fold op (wrap nil count) sequence)
    }
  }

  /// Alterantive values.
  .logic.module alternative {
    /// left :: forall a. forall b. forall c.
    ///   a -> (a -> c) -> (b -> c) -> c
    .logic.let left {
      @argument left;
      @argument if-left;
      @argument if-right;
      @body (if-left left);
    }

    /// right :: forall a. forall b. forall c.
    ///   b -> (a -> c) -> (b -> c) -> c
    .logic.let left {
      @argument right;
      @argument if-left;
      @argument if-right;
      @body (if-right right);
    }

    /// case :: forall a. forall b. forall c.
    ///   ((a -> c) -> (b -> c) -> c) -> (a -> c) -> (b -> c) -> c
    .logic.let case {
      @argument choice;
      @argument if-left;
      @argument if-right;
      @body (choice if-left if-right);
    }
  }

  /// Optional values
  .logic.module option {
    /// some :: forall a. forall b. a -> (a -> b) -> b -> b
    .logic.let some {
      @argument item;
      @argument if-some;
      @argument if-none;
      @body (if-some item);
    }

    /// none :: forall a. forall b. (a -> b) -> b -> b
    .logic.let none {
      @argument item;
      @argument if-some;
      @argument if-none;
      @body (if-none);
    }

    /// with-some :: forall a. forall b.
    ///   ((a -> b) -> b -> b) -> (a -> b) -> b -> b
    .logic.let with-some {
      @argument option;
      @argument if-some;
      @argument if-none;
      @body (option if-some if-none);
    }
  }
}
```

Static analysis of the logic expressions can be used to determine if it
is possible for the logic to produce an output that would fail to
validate. For the purposes of this, miniumum and maximum repititions are
ignored so `.del.children` will match any number of repeated elements.
