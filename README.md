<!--#region:intro-->
# ECMAScript Regular Expression Language Features

This seeks to investigate and introduce new features to the ECMAScript `RegExp` object based on features available commonly in other languages.

<!--#endregion:intro-->

<!--#region:status-->
## Status

**Stage:** 0  
**Champion:** Ron Buckton (@rbuckton)  

_For detailed status of this proposal see [TODO](#todo), below._
<!--#endregion:status-->

<!--#region:authors-->
## Authors

* Ron Buckton ([@rbuckton](https://github.com/rbuckton))
<!--#endregion:authors-->

<!--#region:motivations-->
# Motivations

TBD

<!--#endregion:motivations-->

<!--#region:prior-art-->
<!--
# Prior Art

- Language: [Feature](#todo)

-->
<!--#endregion:prior-art-->

<!--#region:syntax-->
# Syntax

## Buffer Boundaries

Buffer boundaries are similar to the `^` and `$` anchors, except that they aren't affected by the `m` flag:

- `\A` &mdash; Matches the start of the input.
- `\z` &mdash; Matches the end of the input.
- `\Z` &mdash; A zero-width assertion consisting of an optional newline at the end of the buffer. Equivalent to (?=\n?\z).

Requires the `u` flag. Not supported inside of a character class.

[Prior Art](https://rbuckton.github.io/regexp-features/features/buffer-boundaries.html)

## Line Endings Escape

- `\R` &mdash; Matches any line ending character sequence.

Requires the `u` flag. Not supported inside of a character class.

[Prior Art](https://rbuckton.github.io/regexp-features/features/line-endings-escape.html)

## Possessive Quantifiers

Possessive quantifiers are like normal (a.k.a. "greedy") quantifiers, but do not backtrack.

- `*+` &mdash; Match zero or more instancs of the preceding atom without backtracking.
- `++` &mdash; Match one or more instancs of the preceding atom without backtracking.
- `?+` &mdash; Match zero or one instancs of the preceding atom without backtracking.
- `{n,}+` &mdash; Where _n_ is an integer. Matches the preceding atom at-least _n_ times without backtracking.
- `{n,m}+` &mdash; Where _n_ and _m_ are integers, and _m_ >= _n_. Matches the preceding atom at-least _n_ times and at-most _m_ times without backtracking.

[Prior Art](https://rbuckton.github.io/regexp-features/features/possessive-quantifiers.html)

## Flags

- `n` &mdash; Non-capturing mode. Only named capture groups are captured. Normal capture groups (i.e., `()`) are treated as non-capturing groups.
- `x` &mdash; Extended mode. Whitespace characters within the pattern are ignored and [Line Comments](#line-comments) are enabled.

[Prior Art](https://rbuckton.github.io/regexp-features/features/flags.html)

## Modifiers

Modifiers allow you to change the currently active RegExp flags within a subexpression.

- `(?imsux-imsux)` &mdash; Sets or unsets (using `-`) the specified RegExp flags starting at the current position until the next closing `)` or the end of the pattern.
- `(?imsux-imsux:subexpression)` &mdash; Sets or unsets (using `-`) the specified RegExp flags for the subexpression.

[Prior Art](https://rbuckton.github.io/regexp-features/features/modifiers.html)

## Comments

A Comment is a sequence of characters that is ignored by pattern matching and can be used to document a pattern.

- `(?# comment )` &mdash; The entire expression is removed from the pattern. A comment may not contain other `(` or `)` characters.

[Prior Art](https://rbuckton.github.io/regexp-features/features/comments.html)

## Line Comments

A Line Comment is a sequence of characters starting with `#` and ending with `\n` (or the end of the pattern) that is ignored by pattern matching and can be used to document a pattern.

- `# comment` &mdash;

Requires the `x` flag.

[Prior Art](https://rbuckton.github.io/regexp-features/features/line-comments.html)

## Atomic Groups

An Atomic Group is a non-backtracking expression which is matched independent of neighboring patterns, and will not backtrack in the event of a failed match. This is often used to improve performance.

- `(?>pattern)` &mdash; Matches the provided pattern, but no backtracking is performed if the match fails.

[Prior Art](https://rbuckton.github.io/regexp-features/features/non-backtracking-expressions.html)

## Conditional Expressions

A Conditional Expression checks a condition and evaluates its first alternative if the condition is true; otherwise, it evaluates its second alternative.

- `(?(condition)yes-pattern|no-pattern)` &mdash; Matches yes-pattern if condition is true; otherwise, matches no-pattern.
- `(?(condition)yes-pattern)` &mdash; Matches yes-pattern if condition is true; otherwise, matches the empty string.

[Prior Art](https://rbuckton.github.io/regexp-features/features/conditional-expressions.html)

### Conditions

The following conditions are proposed:

- `(?=test-pattern)` &mdash; Evaluates to true if a positive lookahead for test-pattern matches; Otherwise, evaluates to false.
- `(?<=test-pattern)` &mdash; Evaluates to true if a positive lookbehind for test-pattern matches; Otherwise, evaluates to false.
- `(?!test-pattern)` &mdash; Evaluates to true if a negative lookahead for test-pattern matches; Otherwise, evaluates to false.
- `(?<!test-pattern)` &mdash; Evaluates to true if a negative lookbehind for test-pattern matches; Otherwise, evaluates to false.
- `(n)` &mdash; Evaluates to true if the capture group at offset _n_ was successfully matched; Otherwise, evaluates to false.
- `(<name>)` &mdash; Evaluates to true if the named capture group with the provided _name_ was successfully matched; Otherwise, evaluates to false.
- `('name')` &mdash; Evaluates to true if the named capture group with the provided _name_ was successfully matched; Otherwise, evaluates to false.
- `(R)` &mdash; Evaluates to true if inside a recursive expression; Otherwise, evaluates to false.
- `(Rn)` &mdash; Evaluates to true if inside a recursive expression for the capture group at offset n; Otherwise, evaluates to false.
- `(R&name)` &mdash; Evaluates to true if inside a recursive expression for the named capture group with the provided name; Otherwise, evaluates to false.
- `(DEFINE)` &mdash; Always evaluates to false. This allows you to define Subroutines.

## Subroutines

A Subroutine is a pre-defined capture group or named capture group that can be reused in multiple places within the pattern to re-evaluate the subexpression from the referenced group.

- `(?n)` &mdash; Where _n_ is an integer >= 1. Evaluates the capture group whose offset is _n_.
- `(?-n)` &mdash; Where _n_ is an integer >= 1. Evaluates the capture group whose offset is the _n_<sup>th</sup> capture group declared to the left of the current atom.
- `(?+n)` &mdash; Where _n_ is an integer >= 1. Evaluates the capture group whose offset is the _n_<sup>th</sup> capture group declared to the right of the current atom.
- `(?&name)` &mdash; Evaluates the named capture group with the provided name.

Subroutines also allow [Recursion](#recursion).

[Prior Art](https://rbuckton.github.io/regexp-features/features/subroutines.html)

## Recursion

A Recursive Expression provides a mechanism for re-evaluating a capture group inside of itself, to handle cases such as matching balanced parenthesis or brackets, etc.

- `(?R)`, `(?0)` &mdash; Reevaluates the entire pattern starting at the current position.

[Prior Art](https://rbuckton.github.io/regexp-features/features/recursion.html)

<!--#endregion:syntax-->

<!--#region:semantics-->
# Semantics

TBD

<!--#endregion:semantics-->

<!--#region:api-->
# API

TBD

```ts
interface RegExpConstructor {
    new (pattern: RegExp | string, flags?: string): RegExp;
    (pattern: RegExp | string, flags?: string): RegExp;

    tag(array: TemplateStringsArray, ...args: any[]): RegExpTaggedConstructor;
}

interface RegExpTaggedConstructor {
    new (flags?: string): RegExp;
    (flags?: string): RegExp;
}
```

<!--#endregion:api-->

<!--#region:examples-->
# Examples

```js

const iso8601DateRegExp = new RegExp.tag`
  (?(DEFINE)
    (?<Year>\d{4}|[+-]\d{5,})
    (?<Month>0[1-9]|1[0-2])
    (?<Day>0[1-9]|2[0-9]|3[01])
  )
  (?<Date>(?&Year)-(?&Month)-(?&Day)|(?&Year)(?&Month)(?&Day))
`("x");

```

<!--#endregion:examples-->

<!--#region:grammar-->
<!--
# Grammar

> TODO: Provide the grammar for the proposal. Please use [grammarkdown][Grammarkdown] syntax in
> fenced code blocks as grammarkdown is the grammar format used by ecmarkup.

```grammarkdown
```
-->
<!--#endregion:grammar-->

<!--#region:references-->
# References

- [Regular Expression Feature Comparison](https://rbuckton.github.io/regexp-features/)

<!--#endregion:references-->

<!--#region:prior-discussion-->
<!--
# Prior Discussion

> TODO: Provide links to prior discussion topics on https://esdiscuss.org.

* [Subject](https://esdiscuss.org)
-->
<!--#endregion:prior-discussion-->

<!--#region:todo-->
# TODO

The following is a high-level list of tasks to progress through each stage of the [TC39 proposal process](https://tc39.github.io/process-document/):

### Stage 1 Entrance Criteria

* [x] Identified a "[champion][Champion]" who will advance the addition.  
* [ ] [Prose][Prose] outlining the problem or need and the general shape of a solution.  
* [x] Illustrative [examples][Examples] of usage.  
* [x] High-level [API][API].  

### Stage 2 Entrance Criteria

* [ ] [Initial specification text][Specification].  
* [ ] [Transpiler support][Transpiler] (_Optional_).  

### Stage 3 Entrance Criteria

* [ ] [Complete specification text][Specification].  
* [ ] Designated reviewers have [signed off][Stage3ReviewerSignOff] on the current spec text.  
* [ ] The ECMAScript editor has [signed off][Stage3EditorSignOff] on the current spec text.  

### Stage 4 Entrance Criteria

* [ ] [Test262](https://github.com/tc39/test262) acceptance tests have been written for mainline usage scenarios and [merged][Test262PullRequest].  
* [ ] Two compatible implementations which pass the acceptance tests:  
  * [ ] [_Pending_][Implementation1]  
  * [ ] [_Pending_][Implementation2]  
* [ ] A [pull request][Ecma262PullRequest] has been sent to tc39/ecma262 with the integrated spec text.  
* [ ] The ECMAScript editor has signed off on the [pull request][Ecma262PullRequest].  
<!--#endregion:todo-->

[Process]: https://tc39.github.io/process-document/
[Proposals]: https://github.com/tc39/proposals/
[Grammarkdown]: http://github.com/rbuckton/grammarkdown#readme
[Champion]: #status
[Prose]: #motivations
[Examples]: #examples
[API]: #api
[Specification]: #todo
[Transpiler]: #todo
[Stage3ReviewerSignOff]: #todo
[Stage3EditorSignOff]: #todo
[Test262PullRequest]: #todo
[Implementation1]: #todo
[Implementation2]: #todo
[Ecma262PullRequest]: #todo
