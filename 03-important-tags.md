# Twig 1.x Certification Preparation - Important Tags

## `apply`-tag

To apply a filter on a section of code, use the `apply` tag:

```
{% apply lower|escape('html') %}
    <strong>SOME TEXT</strong>
{% endapply %}
```

## `autoescape`-tag

Whether automic escaping is enabled or not you can escape a section of a template by using the `autoescape` tag.

```
{% autoescape %}
    Everything will be automatically escaped in this block
    using the HTML strategy
{% endautoescape %}

{% autoescape 'js' %}
    Everything will be automatically escaped in this block
    using the js escaping strategy
{% endautoescape %}

{% autoescape false %}
    Everything will be outputted as is in this block
{% endautoescape %}
```

You can mark values as "safe" and disable escaping by using the `raw` filter.

```
{% autoescape %}
    {{ safe_value|raw }}
{% endautoescape %}
```

__Note:__ Already escaped values by the `escape` filter, are not escaped again.

__Note:__ Twig does not escape static expressions.

__Warning:__ The autoescape tag has no effect on included files.

## `block`-tag

Used for inheritance and act as placeholders and replacements at the same time. Block names must consist of:

- alphanumeric characters and underscores
- first character can't be a digit
- dashes are not allowed

You can't define multiple `block` tags with the same name in the same template.

If you want to print a block multiple times you can use the `block()` function:

```
<title>{% block title %}{% endblock %}</title>
<h1>{{ block('title') }}</h1>
{% block body %}{% endblock %}
```

Block End-Tags can have a name. Needs to be the same name, which was used to open the block:

```
{% block sidebar %}
    {% block inner_sidebar %}
        ...
    {% endblock inner_sidebar %}
{% endblock sidebar %}
```

Blocks have access to variables from outer scopes.

Block shortcut:

```
{% block title %}
    {{ page_title|title }}
{% endblock %}

{% block title page_title|title %}
```

## `embed`-tag

The `embed` tag combines the behavior of `include` and `extends`. Think of an embedded template as a "micro layout skeleton":

```
{% embed "teasers_skeleton.twig" %}
    {# These blocks are defined in "teasers_skeleton.twig" #}
    {# and we override them right here:                    #}
    {% block left_teaser %}
        Some content for the left teaser box
    {% endblock %}
    {% block right_teaser %}
        Some content for the right teaser box
    {% endblock %}
{% endembed %}
```

The `embed` tag takes the exact same arguments as the `include` tag:

```
{% embed "base" with {'foo': 'bar'} %}
    ...
{% endembed %}

{% embed "base" with {'foo': 'bar'} only %}
    ...
{% endembed %}

{% embed "base" ignore missing %}
    ...
{% endembed %}
```

__Warning:__ As embedded templates do not have "names", auto-escaping strategies based on the template name won't work as expected if you change the context (for instance, if you embed a CSS/JavaScript template into an HTML one). In that case, explicitly set the default auto-escaping strategy with the autoescape tag.

## `extends`-tag

Used to extend a template from another one. Multiple inheritance is not supported. Should be the first tag in the template.

Dynamic inheritance:

```
{% extends some_var %}

{% extends ['layout.html', 'base_layout.html'] %}
```

Conditional inheritance:

```
{% extends standalone ? "minimum.html" : "base.html" %}
```

## `for`-tag

Used to loop over a sequence. A sequence can be either an array or an object implementing the Travversable interface:

```
{% for user in users %} ... {% endfor %}

{% for i in 0..10 %} ... {% endfor %}

{% for letter in 'a'..'z' %} ... {% endfor %}

{% for letter in 'a'|upper..'z'|upper %} ... {% endfor %}
```

**The loop variable:**

- loop.index  The current iteration of the loop. (1 indexed)
- loop.index0 The current iteration of the loop. (0 indexed)
- loop.revindex   The number of iterations from the end of the loop (1 indexed)
- loop.revindex0  The number of iterations from the end of the loop (0 indexed)
- oop.first  True if first iteration
- loop.last   True if last iteration
- loop.length The number of items in the sequence
- loop.parent The parent context

__Note:__ The `loop.length`, `loop.revindex`, `loop.revindex0` and `loop.last` variables are only available for PHP arrays, or objects that implement the Countable interface. They are also not available when looping with a condition.

It is not possible to `break` or `continue` a loop, but you can use a filter to skip over certain items: `{% for user in users if user.active %} ... {% endfor %}`

__Note:__ Using the `loop` variable within the condition is not recommended, as it will probably not be doing what you expect it to do.

If no iteration took place (empty sequence) you can render a replacement: 

```
{% for user in users %}
    ...
{% else %}
    ...
{% endfor %}
```

**Iterating:**

```
{# iterating over keys #}
{% for key in users|keys %} ... {% endfor %}

{# iterating over keys and values #}
{% for key, user in users %} ... {% endfor %}

{# iterating over a subset #}
{% for user in users|slice(0, 10) %} ... {% endfor %}
```

## `if`-tag

Used to test if an expression evaluates to `true`.

```
{% if online == false %} ... {% endif %}

{# array not empty #}
{% if users %} ... {% endif %} 

{# variable defined #}
{% if users is defined %} ... {% endif %}

{# not to check evaluation to false #}
{% if not user.subscribed %} ... {% endif %}

{# conditions like and and or #}
{% if temperature > 18 and temperature < 27 %} ... {% endif %}

{# multiple branches like in PHP with elseif and else #}
{% if product.stock > 10 %}
   Available
{% elseif product.stock > 0 %}
   Only {{ product.stock }} left!
{% else %}
   Sold-out!
{% endif %}
```

## `set`-tag

The `set` tag is used to assign variables inside code blocks:

```
{% set foo = 'foo' %}
{% set foo = [1, 2] %}
{% set foo = {'foo': 'bar'} %}
```

The assigned value can be any valid Twig expression: `{% set foo = 'foo' ~ 'bar' %}`. Several variabls can be assigned in one block: `{% set foo, bar = ''foo, 'bar' %}`. Same like `{% set foo = 'foo' %}` and `{% set bar = 'bar' %}`.

The `set` tag can also capture chunks of text:

```
{% set foo %}
    <div id="pagination">
        ...
    </div>
{% endset %}
```

__Note:__ loops are scoped in Twig. Therefore a variable declared inside a `for` loop is not accessible outside a loop itself. If you want access, just declare it before the loop.

## `use`-tag

The `use` statement tells Twig to import the block in the defined template:

```
{% extends "base.html" %}

{% use "blocks.html" %}

{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

```
{# blocks.html #}

{% block sidebar %}{% endblock %}
```

__Note:__ The `use` tag only imports a template if it does not extend another template, if it does not define macros, and if the body is empty. But it can use other templates.

__Note:__ Because use statements are resolved independently of the context passed to the template, the template reference cannot be an expression.

The main template can also override any imported block. If the template already defines the sidebar block, then the one defined in blocks.html is ignored. To avoid name conflicts, you can rename imported blocks:

```
{% extends "base.html" %}

{% use "blocks.html" with sidebar as base_sidebar, title as base_title %}

{% block sidebar %}{% endblock %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

__Note:__ You can use as many use statements as you want in any given template. If two imported templates define the same block, the latest one wins.

## `verbatim`-tag

Marks a section of a template as being raw text that should not be parsed.

```
{% verbatim %}
    <ul>
    {% for item in seq %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>
{% endverbatim %}
```

This tag was named `raw` before and has been renamed to avoid confusion with the `raw` filter.