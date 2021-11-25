# Twig certification preparation - Concepts

Twig is **fast** (compiles templates down to plain optimized PHP code), **secure** (comes with a sandbox mode, user can safely modify the template design) and **flexible** (flexible lexer and parser, define custom tags and filters).

## Basic Concepts

A template contains **variables** or **expressions**, which get replaced with values when the template is evaluated, and **tags**, which control the template's logic.

There are two kinds of delimiters:

- `{% ... %}`: executes statements, such as for-loops
- `{{ ... }}`: outputs the result of an expression

### Escaping

Twig supports both, automatic escaping or manual escaping of variables. The default escaping strategy is `html`.

Manual escaping is done by using the `escape` or `e` filter: `{{ user.username|e }}` or with an other strategy: `{{ user.username|e('js') }}`. There are the following strategies: html, js, css, url, html_attr, name or filename, PHP callback.

Outputting `{{` as raw string in a template is possible by using a variable expression: `{{ '{{' }}`. For bigger sections it makes sense to mark a block `verbatim`.

### Whitespace Control

Except the first newline after a template tag, no further whitespace is modified by the template engine, so each whitespace is returned unchanged.

Two modifiers are supported:

- whitespace trimming via `-`: Removes all whitespace (including newlines)
- line whitespace trimming via `~`: Removes all whitespace (excluding newlines). Using it on the right disables the default removal of the first newline inherited from PHP.

Modifiers can either be used on one side or on both side of statements and expressions:

```
{% set value = 'no spaces' %}
{#- No leading/trailing whitespace -#}
{%- if true -%}
    {{- value -}}
{%- endif -%}
{# output 'no spaces' #}

<li>
    {{ value }}    </li>
{# outputs '<li>\n    no spaces    </li>' #}

<li>
    {{- value }}    </li>
{# outputs '<li>no spaces    </li>' #}

<li>
    {{~ value }}    </li>
{# outputs '<li>\nno spaces    </li>' #}
```

The `spaceless` filter can be used in addition to the whitespace modifiers to remove whitespace between HTML tags, not whitespace within HTML tags or whitespace in plain text.

## Variables and Expressions

The application passes variables to the templates for manipulation in the template. Use a dot (`.`) to access attributes of a variable: 
`{{ foo.bar }}`

__Note:__ The curly braces are not part of the variable but the print statement. When accessing variables inside tags, don't put the braces around them.

Accessing the attribute bar of foo (`{{ foo.bar }}`) follows this order:

- `foo['bar']`
- `foo->bar`
- `foo->bar()`
- `foo->getBar()`
- `foo->isBar()`

You will receive a `null` value, if `strict_variables` option is set to `false`. With `strict_variables` option set to `true`, Twig will throw an error.

A specific syntax for accessing items on PHP arrays is `{{ foo['bar'] }}`.

__Note:__ If you want to access dynamic attributes of a variable or the attribute's name contains special characters (like `-` that would be interpreted as a minus operator), use the `attribute` function instead:

- dynamic attribute: `{{ attribute(object, method, arguments) }}`
- special character name: `{{ attribute(foo, 'data-foo') }}`
- array: `{{ attribute(array, item) }}`

These global variables are always available in templates: `_self` (references the current template), `_context` (references the current context), `_charset` (references the current charset).

### Expressions

Operator precedence (lowest first): `?:` (ternary operator), `b-and`, `b-xor`, `b-or`, `or`, `and`, `==`, `!=`, `<`, `>`, `>=`, `<=`, `in`, `matches`, `starts with`, `ends with`, `..`, `+`, `-`, `~`, `*`, `/`, `//`, `%`, `is` (tests), `**`, `??`, `|` (filters), `[]`, `.`

- `{{ 'Hello' ~ 'Jan'|lower }}   {# Hello jan #}`
- `{{ ('Hello' ~ 'Jan')|lower }}   {# Hello Jan #}`

**Literals:** 

The simplest form of a expression are **Literals**. They represent PHP types such as strings, numbers and arrays.

- `"Hello World"` or `'It\'s good'` - strings
- `42/42.23` - numbers (int and float)
- `["foo", "bar"]` - arrays
- `{"foo", "bar"}` - hashes

The keys of hashes can be as follows:

- keys can be strings: `{ 'foo': 'foo', 'bar': 'bar' }`
- keys can be names: `{ foo: 'foo', bar: 'bar' }`
- keys can be integers: `{ 2: 'foo', 4: 'bar' }`
- keys can be omitted if they are the same as the variable name: 

```
{ foo }
{# is equivalent to the following #}
{ 'foo': foo }
```

- keys can be expressions, but must be enclosed into parentheses: 

```
{% set foo = 'foo' %}
{ (foo): 'foo', (1 + 1): 'bar', (foo ~ 'b'): 'baz' }
```

**Math operators:** `+`, `-`, `/`, `%`, `//` (divide two integer numbers and return the floored result), `*`, `**`

**Logic operators:** `and`, `or`, `not`, `(expr)`

__Note:__ Operators are case sensitive!

**Comparison operators:** `==`, `!=`, `<`, `>`, `<=`, `>=`, `starts with`, `ends with`, `matches`

**Containment operators:** `in`: `{{ 1 in [1, 2, 3] }}` or `{{ 'cd' in 'abcde' }}` Can be used on strings, arrays, or objects implementing the Traversable interface. To perform a negative test, use the `not in` operator: `{% if 1 not in [1, 2, 3] %}` which is equivalent to `{% if not (1 in [1, 2, 3]) %}`

**Test operator:** The `is` operator performs test. The right operand is the name of the test: `{{ name is odd }}`. Test can be negated by using `is not` operator: `{% if name is not odd %}` which is equivalent to `{% if not (name is odd) %}`.

**Other operators:** `|`, `..`, `~`, `.` or `[]`, `?:` (ternary), `??` (null-coalescing)

**String interpolation:** `#{expression}` allows any valid expression to appear in a double-quoted string: `{{ "foo #{bar} baz" }}` or `{{ "foo #{1 + 2} baz" }}`.


## Filters

Variables can be modified by filters. Filters are seperated by `|`. Filters can be chained:`{{ name|strptags|title }}`. Filters can have arguments: `{{ list|join(', ') }}`.

To apply a filter on a section of code, use the `apply` tag:

```
{% apply lower|escape('html') %}
    <strong>SOME TEXT</strong>
{% endapply %}
```

## Functions

Functions generate content and my have arguments: `{% range(0, 3) %)`.

## Named Arguments

Filter and Functions support named arguments: 

- `{{ "now"|date(timezone="Europe/Paris") }}`
- `{% range(low=1, high=10, step=2) %}`

Both positional and named arguments can be used in one call, in which case positional arguments must always come before named arguments: 
`{{ "now"|date('d/m/Y H:i', timezone="Europe/Paris") }}`

## Control Structure

Control structures appear inside  `{% ... %}` blocks, like a for loop:

```
{% for user in users %}
	...
{% endfor %}
```

Conditionals, like `if/elseif/else` can be used to test an expression:

```
{% if users|length > 0 %}
	...
{% endif %}
```

## Including other Templates

The `include` function is useful to include a template and return the rendered content ino the current one: `{{ include('sidebar.html') }}`.

By default, included templates have access to the same context as the template which includes them: any variable defined in the main template will be available in the included template too.

## Template Inheritance

Template inheritance allows you to build a base "skeleton" template that contains all the common elements of your site and defines **blocks** that child templates can override.

A `block` tag tells the template engine that a child template may override those portions of the template.

The `extends` tag tells the template engine that this template "extends" another template.

It's possible to render the contents of the parent block by using the `parent()` function. It gives you the result of the parent block.

__Note:__ Horizontal reuse is possible with the help of the `use` tag. 

## Twig Internals

The rendering of a Twig template can be summarized into four key steps:

1. Load the template: if it is already compiled, load it and go to the evaluation step, otherwise:

- **Lexer** tokenizes the template source code into smaller pieces for easier processing
- **Parser** converts the token stream into a meaningful tree of nodes (AST)
- **Compiler** transforms the AST into PHP code

2. Evaluate the template: calling the `display()` method of the compiled template and passing it the context.

```
// Lexer
$twig->tokenize(new \Twig\Source($source, $identifier));

// Parser
$nodes = $twig->parse($stream);

// Compiler
$php = $twig->compile($nodes);
```

- Token Stream -> AST -> PHP
- Token objects -> Node objects -> PHP strings
- Lexer -> Parser -> Compiler
