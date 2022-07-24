# Comparing Strings in JavaScript

How do we compare strings in JavaScript?

- [Basic comparison](#using-basic-comparison-operators)
- [Support for localization](#using-a-localization-aware-comparison-function)
- [Unicode equivalence](#using-a-normalization-function-for-unicode-equivalence)

## Using basic comparison operators

Basic comparison operators include `<`, `>`, `===` and others.

This approach works well for strings containing only ASCII characters. But what
about the other strings? Let's find out with the sample code below followed by
an explanation.

```javascript
(function() {
    // several versions of the same string
    const s1 = 'ñ';
    const s2 = '\u00F1';
    const s3 = 'n\u0303';
    const s4 = '\u006E\u0303';
    console.log(s1, s2, s3, s4); // all versions share the same visual appearance

    // each version is equal to itself, this is logical, isn't it?
    console.log(s1 === s1, s2 === s2, s3 === s3, s4 === s4);

    // however, not all versions are equal...
    console.log(
        s1 === s2, s1 === s3, s1 === s4,
        s2 === s3, s2 === s4,
        s3 === s4
    );
})();
```

As you noticed, the strings in the example above represent the same information
but are not equal. This is because JavaScript compares strings based on their
Unicode escape sequence, which can be inferred, if necessary, from the
characters they contain. Thus, the use of basic comparison operators is reliable
only if one can guarantee that the same string will never be encoded using
different Unicode escape sequences, or if one does not care about Unicode
equivalence between strings.

Usually we don't have to worry about Unicode equivalence when comparing strings,
so using the basic comparison operators should often be your first choice.

## Using a localization-aware comparison function

One can use `String.prototype.localeCompare()` to enable language-sensitive
string comparison.

Note that according to ECMAScript 5.1 (2011) specification, `String.prototype.localeCompare()`
[must correctly handle strings that are canonically equivalent according to the
Unicode standard](https://262.ecma-international.org/5.1/#sec-15.5.4.9).

> *The actual return values are implementation-defined to permit implementers to
encode additional information in the value, but the function is required to
define a total ordering on all Strings and to return 0 when comparing Strings
that are considered canonically equivalent by the Unicode standard.*

## Using a normalization function for Unicode equivalence

When Unicode equivalence is required, strings can first be normalized using
`String.prototype.normalize()`. The normalized versions can then be compared
using the basic comparison operators mentioned above.

Finally, before reading the documentation of the said normalization function,
you might want to read this introductory article first:
[How to Check if Two Strings are Equal in JavaScript](https://www.javascripttutorial.net/string/javascript-string-equals/).
