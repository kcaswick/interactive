# Formatting

This document contains notes on how formatters are specified, and#
some specific details of plain text and HTML formatting.

Formatting is invoked when values are displayed either implicitly or using `display`.

##  Registering preferred mime types

See the language-specific documentation on using `SetPreferredMimeTypeFor`.

```csharp
Formatter.SetPreferredMimeTypeFor(typeof(System.Guid), "text/plain");
```

##  Registering formatters

Formatters can be specified by using `Formatter.Register`, keyed by type.
See the language-specific documentation on using `Formatter.Register`.

For example:

```csharp
Formatter.Register<System.Type>(t => t.GUID.ToString());
```

Formatters can be specified by using a generic type definition as a key, for example:

```csharp
Formatter.Register(
    type: typeof(List<>),
    formatter: (obj: object, writer) =>
    {
        writer.Write("quack");
    }, mimeType);
```

Then 

```
var list = new List<int> { 1, 2, 3, 4, 5 };
```

displays as `quack`.  Reflection can also be used to operate on the object at its more specific type.

##  How the formatter is chosen

The applicable formatter is chosen for an object of type A as follows:

1. If no mime type is specified, determine one:

   - Choose the most-specific user-registered mime type preference relevant to A

   - If no user-registered mime-types are relevant, then choose a default mime type.

2. Next, determine a formatter:

   - Choose the most-specific user-registered formatter relevant to A

   - If no user-registered formatters are relevant, then choose a default formatter.

Here "most specific" is in terms of the class and interface hierarchy.   In the event of an exact tie in
ordering or some other conflict, more recent registerations are
preferred. Type-instantiations of generic types are preferred to generic
formatters when their type definition are the same.

The default sets of formatters for a mime type always include a formatter for `object`.

### Examples

For example:

* If you register a formatter for type `A` then it is used for all objects of type `A` (until alternative formatter for type A is later specified)

* If you register a formatter for `System.Object`, it is preferred over all other formatters except other user-defined formatters

* If you register a formatter for any sealed type, it is preferred over all other formatters (unless more formatters for that type are specified)

* If you register `List<>` and `List<int>` formatters the `List<int>` formatter is preferred for objects of type `List<int>`

* If you register a confusing conflicting mess of overlapping formatters incrementally, they should Formatters.Clear() or restart the kernel.

* If you register `text/plain` as the mime type for `object` then it is used as the mime type for everything (likewise any other mime type)


## Default Formatters

See `DefaultHtmlFormatters.cs`, `DefaultPlainTextFormatters.cs` and `DefaultJsonFormatters.cs` among others.

## User Configuration of Default Formatters

The following global settings can be set:

* `Formatter.RecursionLimit` = 20

  Gets or sets the limit to how many levels the formatter will recurse into an object graph.

* `Formatter.ListExpansionLimit` = 20

  Gets or sets the limit to the number of items that will be written out in detail from an IEnumerable sequence.

* `Formatter<T>.ListExpansionLimit` = (not set)

  An optional type-specific list expansion limit

* `PlainTextFormatter.MaxProperties` = 20

  Indicates the maximum number of properties to show in the default plaintext display of arbitrary objects.
  If set to zero no properties are shown.

* `HtmlFormatter.MaxProperties` = 20

  Indicates the maximum number of properties to show in HTML table displays of arbitrary objects.
   If set to zero no properties are shown.
```

## HTML Formatting


### The `CSS` function

The `CSS` function can be used to add CSS styling to the host HTML system.

Here are some examples:
```csharp
CSS("h3 { background: red; }");

CSS(".dni-plaintext { text-align: left; white-space: pre; font-family: monospace; });"
```


### CSS tags emitted

The bespoke CSS tags emitted for .NET Interactive content are as follows:

| tag | content|
|:------|:-----------|
| `dni-plaintext` |  In HTML displays of values, any content generated by formatting arbitrary embedded object values as plaintext |

In the future, additional tags will be added to this list.




