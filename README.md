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

ECMAScript regular expressions have slowly improved over the years to adopt new functionality commonly present in other languages, including:

- Unicode Support
- Named Capture Groups
- Match Indices

However, a large majority of other languages and libraries have a common set of features that ECMAScript regular expressions currently lack.
Some of these features improve performance in degenerative cases such as backtracking in complex patterns. Some of these features introduce
new tools for developers to write more powerful regular expressions.

As a result, ECMAScript developers wishing to leverage these capabilities are left with few options, relying on native bindings to third-party
libraries in environments such as NodeJS, or server-side evaluation.

There are numerous applications for extending the ECMAScript regular expression feature set, including:

- In-browser support for TextMate grammars for web based editors/IDEs.
- Improved performance for expressions through possessive quantifiers and backtracking control.
- RegExp-based parsers that can support balanced brackets/parens.
- Documenting complex patterns *in the pattern itself*.
- Improved readability through the use of multi-line patterns and insignificant whitespace.

<!--#endregion:motivations-->

<!--#region:prior-art-->
<!--
# Prior Art

- Language: [Feature](#todo)

-->
<!--#endregion:prior-art-->

<!--#region:syntax-->
# Syntax

This proposal seeks to investiage multiple additions to the ECMAScript regular expression syntax based on features commonly available in
other languages and engines. This work is based on the research at https://rbuckton.github.io/regexp-features/, which is an ongoing effort
to document the commonalities and differences of various features in popular regular expression engines. This proposal does not seek to 
implement *all* of the proposed syntax, but to investigate each feature to determine its applicability to ECMAScript. Where possible,
we will indicate whether the syntax described should be considered *definitive* (i.e., the specific syntax is not subject to change should
the feature be adopted), or *proposed* (i.e., the specific syntax is open for debate). 

<dfn id="definitive-syntax">Definitive syntax</dfn> is that which is generally-consistent with all engines that implement the functionality, 
such that a change to the syntax would have a net-negative effect when considering compatibility with other engines (such as would be the 
case with TextMate grammars, patterns commonly used in documentation to describe a valid input, etc.).

<dfn id="proposed-syntax">Proposed syntax</dfn> is that which is inconsistent between the various engines that implement similar 
functionality, such that a change to the syntax to fit ECMAScript requirements would not likely be a compatiblity concern.

## Flags

### Explicit capture mode (`n`)

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, .NET ([feature comparison](https://rbuckton.github.io/regexp-features/features/flags.html))

The explicit capture mode (`n`) flag affects capturing behavior, such that normal capture groups (i.e., `()`) are treated as non-capturing groups. Only named capture groups are returned.

> NOTE: The `n`-mode flag can be used inside of a [Modifier](#modifiers).

#### API

- `RegExp.prototype.explicitCapture` (Boolean) &mdash; Indicates whether the `n`-mode flag is set.

### Extended mode (`x`)

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, .NET, Oniguruma, Hyperscan, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/flags.html))

The extended mode (`x`) flag treats unescaped whitespace characters as insignificant, allowing for multi-line regular expressions. It also enables [Line Comments](#line-comments).

> NOTE: The `x`-mode flag can be used inside of a [Modifier](#modifiers)

> NOTE: While the `x`-mode flag can be used in a _RegularExpressionLiteral_, it does not permit the use of _LineTerminator_ in _RegularExpressonLiteral_. For multi-line
> regular expressions you would need to use the `RegExp` constructor.

> NOTE: Perl's original `x`-mode treated whitespace as insignificant anywhere within a pattern *except* for within character classes. Perl v5.26 introduced the `xx` flag which
> also ignores non-escaped SPACE and TAB characters. Should we chose to adopt the `x`-mode flag, we could opt to treat it as Perl's `xx` mode at the outset.

#### API

- `RegExp.prototype.extended` (Boolean) &mdash; Indicates the `x`-mode flag is set.

## Modifiers

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, .NET, Oniguruma, Hyperscan, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/modifiers.html))

Modifiers allow you to change the currently active RegExp flags within a subexpression.

- `(?imnsux-imnsux)` &mdash; Sets or unsets (using `-`) the specified RegExp flags starting at the current position until the next closing `)` or the end of the pattern.
- `(?imnsux-imnsux:subexpression)` &mdash; Sets or unsets (using `-`) the specified RegExp flags for the subexpression.

> NOTE: Certain flags cannot be modified mid-expression. These currently include `g` (global), `y` (sticky), and `d` (hasIndices).

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

#### Example

```js
const re1 = /^(?i)[a-z](?-i)[a-z]$/;
re1.test("ab"); // true
re1.test("Ab"); // true
re1.test("aB"); // false

const re2 = /^(?i:[a-z](?-i:[a-z]))$/;
re2.test("ab"); // true
re2.test("Ab"); // true
re2.test("aB"); // false
```

## Comments

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, .NET, Oniguruma, Hyperscan, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/comments.html))

A comment is a sequence of characters that is ignored by pattern matching and can be used to document a pattern.

- `(?#comment)` &mdash; The entire expression is removed from the pattern. The text of *comment* may not contain other `(` or `)` characters.

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

#### Example

```js
const re = /foo(?#comment)bar/;
re.test("foobar"); // true
```

## Line Comments

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, .NET, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/line-comments.html))

A Line Comment is a sequence of characters starting with `#` and ending with `\n` (or the end of the pattern) that is ignored by pattern matching and can be used to document a pattern.

- `# comment` &mdash; A line comment in a multi-line RegExp

> NOTE: Requires the `x`-mode [flag](#extended-mode-x).

> NOTE: Inside of `x`-mode, the `#` character must be escaped (using `\#`) outside of a character class.

#### Example

```js
const re = new RegExp(String.raw`
    # match ASCII alpha-numerics
    [a-zA-Z0-9]
`, "x");
```

## Buffer Boundaries

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, .NET, Oniguruma, Hyperscan, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/buffer-boundaries.html))  

Buffer boundaries are similar to the `^` and `$` anchors, except that they are not affected by the `m` (multiline) flag:

- `\A` &mdash; Matches the start of the input.
- `\z` &mdash; Matches the end of the input.
- `\Z` &mdash; A zero-width assertion consisting of an optional newline at the end of the buffer. Equivalent to (?=\n?\z).

> NOTE: Requires the `u` flag, as `\A`, `\z`, and `\Z` are currently just escapes for `A`, `z` and `Z` without the `u` flag. 

> NOTE: Not supported inside of a character class.


## Line Endings Escape

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, Oniguruma, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/line-endings-escape.html))

- `\R` &mdash; Matches any [line ending character sequence per UTS#18](https://unicode.org/reports/tr18/#Line_Boundaries). Equivalent to: `(?>\r\n?|[\x0A-\x0C\x85\u{2028}\u{2029}])` (see [Atomic Groups](#atomic-groups))

> NOTE: Requires the `u` flag, as `\R` is currently just an escape for `R` without the `u` flag.

> NOTE: Not supported inside of a character class.

## Possessive Quantifiers

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, Oniguruma, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/possessive-quantifiers.html))

Possessive quantifiers are like normal (a.k.a. "greedy") quantifiers, but do not backtrack if the rest of the pattern to the right fails to match. Possessive quantifiers are often used as a performance tweak to avoid expensive backtracking in a complex pattern.

- `*+` &mdash; Match zero or more instances of the preceding atom without backtracking.
- `++` &mdash; Match one or more instances of the preceding atom without backtracking.
- `?+` &mdash; Match zero or one instances of the preceding atom without backtracking.
- `{n,}+` &mdash; Where _n_ is an integer. Matches the preceding atom at-least _n_ times without backtracking.
- `{n,m}+` &mdash; Where _n_ and _m_ are integers, and _m_ >= _n_. Matches the preceding atom at-least _n_ times and at-most _m_ times without backtracking.

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

## Atomic Groups

**Status:** [Definitive][]

**Prior Art:** Perl, PCRE, Boost.Regex, .NET, Oniguruma, ICU, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/non-backtracking-expressions.html))

An Atomic Group is a non-backtracking expression which is matched independent of neighboring patterns, and will not backtrack in the event of a failed match. This is often used to improve performance.

- `(?>pattern)` &mdash; Matches the provided pattern, but no backtracking is performed if the match fails.

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

#### Example

```js
// NOTE: x-mode flag used to illustrate difference
// without atomic groups:
const re1 = /\((      [^()]+   | \([^()]*\))+ \)/x;
re1.test("((()aaaaaaaaaaaaaaaaaaaaaaaaaaaaa"); // can take several seconds to fail

// with atomic groups
const re2 = /\((  (?> [^()]+ ) | \([^()]*\))+ \)/x;
re2.test("((()aaaaaaaaaaaaaaaaaaaaaaaaaaaaa"); // significantly faster as less backtracking is involved
```

## Conditional Expressions

**Status:** [Definitive][]/[Proposed][] (depending on condition, see below)

**Prior Art:** Perl, PCRE, Boost.Regex, .NET, Oniguruma, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/conditional-expressions.html))

A Conditional Expression checks a condition and evaluates its first alternative if the condition is true; otherwise, it evaluates its second alternative.

- `(?(condition)yes-pattern|no-pattern)` &mdash; Matches yes-pattern if condition is true; otherwise, matches no-pattern.
- `(?(condition)yes-pattern)` &mdash; Matches yes-pattern if condition is true; otherwise, matches the empty string.

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

### Conditions

The following conditions are proposed:

- `(?=test-pattern)` &mdash; Evaluates to **true** if a positive lookahead for test-pattern matches; Otherwise, evaluates to **false**.
  - **Status**: [Definitive][]
- `(?<=test-pattern)` &mdash; Evaluates to **true** if a positive lookbehind for test-pattern matches; Otherwise, evaluates to **false**.
  - **Status**: [Proposed][] (backtracking not supported in all engines)
- `(?!test-pattern)` &mdash; Evaluates to **true** if a negative lookahead for test-pattern matches; Otherwise, evaluates to **false**.
  - **Status**: [Definitive][]
- `(?<!test-pattern)` &mdash; Evaluates to **true** if a negative lookbehind for test-pattern matches; Otherwise, evaluates to **false**.
  - **Status**: [Proposed][] (backtracking not supported in all engines)
- `(n)` &mdash; Evaluates to **true** if the capture group at offset _n_ was successfully matched; Otherwise, evaluates to **false**.
  - **Status**: [Definitive][]
- `(<name>)` &mdash; Evaluates to **true** if the named capture group with the provided _name_ was successfully matched; Otherwise, evaluates to **false**.
  - **Status**: [Definitive][]
- `('name')` &mdash; Evaluates to **true** if the named capture group with the provided _name_ was successfully matched; Otherwise, evaluates to **false**.
  - **Status**: [Definitive][]
- `(R)` &mdash; Evaluates to **true** if inside a recursive expression; Otherwise, evaluates to **false**.
  - **Status**: [Proposed][] ([recursion](#recursion) not supported in all engines)
- `(Rn)` &mdash; Evaluates to **true** if inside a recursive expression for the capture group at offset n; Otherwise, evaluates to **false**.
  - **Status**: [Proposed][] ([recursion](#recursion) not supported in all engines)
- `(R&name)` &mdash; Evaluates to **true** if inside a recursive expression for the named capture group with the provided name; Otherwise, evaluates to **false**.
  - **Status**: [Proposed][] ([recursion](#recursion) not supported in all engines)
- `(DEFINE)` &mdash; Always evaluates to **false**. This allows you to define Subroutines.
  - **Status**: [Proposed][] ([subroutines](#subroutines) not supported in all engines)

#### Example

```js
// conditional using lookahead:
const re1 = /^(?(?=\{)\{[0-9a-f]+\}|[0-9a-f]{4})$/
re1.test("0000"); // true
re1.test("{0}"); // true
re1.test("{00000000}"); // true

// match optional brackets
const re2 = /(?<open-bracket>\[)?(?<content>[^\]]+)(?(<open-bracket>)\]))/;
re1.test("abc"); // true
re1.test("[abc]"); // true
re1.test("[abc"); // false
```

## Subroutines

**Status:** [Proposed][] (some engines use differing syntax)

**Prior Art:** Perl, PCRE, Boost.Regex, Oniguruma, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/subroutines.html))

A Subroutine is a pre-defined capture group or named capture group that can be reused in multiple places within the pattern to re-evaluate the subexpression from the referenced group.

- `(?n)` &mdash; Where _n_ is an integer >= 1. Evaluates the capture group whose offset is _n_.
- `(?-n)` &mdash; Where _n_ is an integer >= 1. Evaluates the capture group whose offset is the _n_<sup>th</sup> capture group declared to the left of the current atom.
- `(?+n)` &mdash; Where _n_ is an integer >= 1. Evaluates the capture group whose offset is the _n_<sup>th</sup> capture group declared to the right of the current atom.
- `(?&name)` &mdash; Evaluates the named capture group with the provided name.

> NOTE: Subroutines also allow [Recursion](#recursion).

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

#### Example

```js
const iso8601DateRegExp = new RegExp(String.raw`
  (?(DEFINE)
    (?<Year>\d{4}|[+-]\d{5,})
    (?<Month>0[1-9]|1[0-2])
    (?<Day>0[1-9]|2[0-9]|3[01])
  )
  (?<Date>(?&Year)-(?&Month)-(?&Day)|(?&Year)(?&Month)(?&Day))
`, "x");
```

## Recursion

**Status:** [Proposed][] (some engines use differing syntax)

**Prior Art:** Perl, PCRE, Boost.Regex, Oniguruma, Glib/GRegex ([feature comparison](https://rbuckton.github.io/regexp-features/features/recursion.html))

A Recursive Expression provides a mechanism for re-evaluating a capture group inside of itself, to handle cases such as matching balanced parenthesis or brackets, etc.

- `(?R)`, `(?0)` &mdash; Reevaluates the entire pattern starting at the current position.

> NOTE: This has no conflicts with existing syntax, as ECMAScript currently produces an error for this syntax in both `u` and non-`u` modes.

<!--#endregion:syntax-->

<!--#region:semantics-->
<!-- 
# Semantics

-->

<!--#endregion:semantics-->

<!--#region:api-->
<!-- 
# API

-->
<!--#endregion:api-->

<!--#region:examples-->
<!--
# Examples

-->
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
[Definitive]: #definitive-syntax
[Proposed]: #proposed-syntax
