A data exrepssion language (DEL)
================================

This package implements a simple language for representing data as
compact expressions.

Concept
-------

The language is designed to represent trees of data in a manner that is
human readable and human-accessible.

Each node object defines either an identifier namespace of its children.
The names of the children may not be unique but will maintain strict
ordering.

Each leaf object has a single typed value and no children.

Syntax
------

### Leaf objects

A leaf object is constructed simmply through its dedicated syntax.

All leaf objects are contained within an attribute. An attribute
associates an identifier as a name with one of the leaf objects.

The syntax for an attribute begins with a `@` followed by the name of
the attribute, a leaf node, and then the `;` object terminator.

The following example would produce an attribute with the name
`attribute-name` and the identifier value `attribute-value`:

```
@attribute-name attribute-value;
```

##### Documentation comments

A comment is prefixed by a single `#` character and a space separator or
any nummber of adjacent `/` characters followed by a space separator and
is terminated with with a unicode line or paragraph separator. Every
comment describes a `comment` object constructor.

Multiline comments begin with a single `/` followed by any numer of `*`
and a single space separator. Each line starts with some number of
spaces, a `*` character, then a single space separator. They end with
any number of  `*` character followed by a `/` character.

Every comment object is appended to the next object that is not a
comment object as an attribute with the name `doc` with the text
dedented and trimmed in a `comment` object.

#### Identifiers

An identifier is any combination and number of unicode letters, numbers,
connectors, marks, and dashes that does not start with a unicode number
character.

#### Paths

Paths begin with a `::` and comprise a sequence of identifiers separated
with the `::`.

#### Text

Raw text is encapsulated within a pair of single quotes (`'`).

Escaped text is encapsulated within a pair of double quotes (`"`).

##### Escaping

Escaping simply uses a `\\` character.

##### Dedenting

Using three quotation markers instead of one at either end will remove
the leading whitespace from the first line from all subsequent lines.
and all trailing non-newline whitespace. There must be a newline after
the opening quotation marks.

#### Floats & integers

A series of decimal values and underscores is an integer. A pair of
these delimited by an adjacent period is a floting-point value.

The `0b` prefix exists for binary values, `0o` for octal, `0x` for
hexadecimal, and `0d` for decimal. The `0bu`, `0ou`, `0xu`, `0du`, and
`0u` prefixes exist for unsigned variants.

#### Tokens

Raw tokens will only ever appear in token-trees.

Tokens are either identifiers, integer tokens, floating-point tokens, or
single unicode punctuation, symbols, separators, and other characters
appear as the bare character as a single token.

#### Token sequence

A token sequence is a sequence of raw tokens from the parser with
balanced unicode brackets. The sequence starts with (but does not
include) a `(` and ends with a `)` after all internal brackets have been
balanced.

This allowes for the embedded logic language.

### Node objects

A node object is constructed using the following sequence of items:

1. a constructor name (as an identifier),
2. the item name (as an identifier),
3. a series of objects, then finally
4. an object terminator.

There are two means of providing an object terminator. The first is to
simply use a `;` token and the second is to encapsulate a sequence of
objects within a pair of `{` and `}` where the closing brace also acts
as an object terminator.

As such, the most simple expression of such an object could appear as:

```
object;
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
schema del-schema {
  /// A schema definition.
  node schema {
    /// A schema does not require a name but it may provide one.
    @name required;

    children {
      /// A node constructor may be used here.
      @node ::del-schema::node;
    }
  }

  /// A node object descriptor.
  node node {
    /// Every node must have a name.
    @name required;

    /// A valid child in the node.
    attribute name {
      @value required;
      @value optional;
      @value dissalowed;
    }

    ordered-children {
      @attribute name;
    }
    children {
      @node ::del-schema::attribute;
    }
    children {
      @node ::del-schema::children;
    }
  }

  /// An attribute descriptor.
  node attribute {
    /// Every attribute must be named.
    @name required;

    /// The attribute may be left empty.
    attribute no-value {
      @no-value;
    }
    /// The attribute may have a text value.
    attribute text-value {
      @no-value;
    }
    /// The attribute may have an integer value.
    attribute integer-value {
      @no-value;
    }
    /// The attribute may have a floating-point value.
    attribute float-value {
      @no-value;
    }
    /// The attribute may have any identifier as a value.
    attribute identifier-value {
      @no-value;
    }
    /// The attribute may have a path as a value.
    attribute path-value {
      @no-value;
    }
    /// The attribute may have a token sequence as a value.
    attribute token-value {
      @no-value;
    }
    /// The attribute may have a value matching an exact token.
    attribute value {
      @identifier-value;
    }

    /// An attribute may allow for mutliple different kinds of values to
    /// be accepted.
    children {
      @attribute no-value;
      @attribute text-value;
      @attribute integer-value;
      @attribute float-value;
      @attribute identifier-value;
      @attribute path-value;
      /// Generic token values and explict tokens can't be mixed for an
      /// attribute.
      children {
        @minimum 1;
        @maximum 1;
        @attribute token-value;
        @attribute value-value;
      }
    }
  }

  /// Children that a node may have in no particular order.
  node children {
    @name disallowed;

    /// The minimum number of matching children.
    attribute minimum {
      @integer-value;
    }
    /// The maximum number of matching children.
    attribute maximum {
      @integer-value;
    }

    /// The name of a node that may appear in the element.
    attribute node {
      @path-value;
    }
    /// The name of an attribute that may appear in the element.
    attribute attribute {
      @identifier-value;
    }

    children {
      @maximum 1;
      @attribute minimum;
    }
    children {
      @maximum 1;
      @attribute maximum;
    }
    children {
      @minimum 1;
      @maximum 1;
      @attribute exactly;
      @attribute at-least;
      @attribute optional;
      @attribute default;
    }
    children {
      @node ::del-schema::children;
      @node ::del-schema::ordered-children;
      @attribute node;
      @attribute attribute;
    }
  }

  /// Children appearing in a particular order.
  node ordered-children {
    @name disallowed;

    /// The name of a node that may appear in the element.
    attribute node {
      @path-value;
    }
    /// The name of an attribute that may appear in the element.
    attribute attribute {
      @identifier-value;
    }

    children {
      @node ::del-schema::children;
      @attribute node;
      @attribute attribute;
    }
  }
}
```
