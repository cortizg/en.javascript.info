# Conjuntos y rangos [...]

Varios caracteres o clases de caracteres entre corchetes `[…]` significa "buscar cualquier carácter entre los dados".

## Conjuntos

Por ejemplo, `pattern:[eao]` significa cualquiera de los 3 caracteres: `'a'`, `'e'`, or `'o'`.

A esto se le dice *conjunto*. Los conjuntos se pueden usar en una expresión regular junto con los caracteres normales:

```js run
// encontrar [t ó m], y luego "op"
alert( "Mop top".match(/[tm]op/gi) ); // "Mop", "top"
```

Tenga en cuenta que aunque hay varios caracteres en el conjunto, corresponden exactamente a un carácter en la coincidencia.

Entonces, el siguiente ejemplo no da coincidencias:

```js run
// encuentra "V", luego [o ó i], luego "la"
alert( "Voila".match(/V[oi]la/) ); // null, sin coincidencias
```

El patrón busca:

- `pattern:V`,
- después *una* de las letras `pattern:[oi]`,
- después `pattern:la`.

Entonces habría una coincidencia para `match:Vola` o `match:Vila`.

## Rangos

Los corchetes también pueden contener *rangos de caracteres*.

Por ejemplo, `pattern:[a-z]` es un carácter en el rango de `a` a `z`, y `pattern:[0-5]` es un dígito de `0` a `5`.

En el ejemplo a continuación, estamos buscando `"x"` seguido de dos dígitos o letras de `A` a `F`:

```js run
alert( "Excepción 0xAF".match(/x[0-9A-F][0-9A-F]/g) ); // xAF
```

Aquí `pattern:[0-9A-F]` tiene dos rangos: busca un carácter que sea un dígito de `0` a `9` o una letra de `A` a `F`.

Si también queremos buscar letras minúsculas, podemos agregar el rango `a-f`: `pattern:[0-9A-Fa-f]`. O agrega la bandera `pattern:i`.

También podemos usar clases de caracteres dentro de los `[…]`.

Por ejemplo, si quisiéramos buscar un carácter de palabra `pattern:\w` o un guión `pattern:-`, entonces el conjunto es `pattern:[\w-]`.

También es posible combinar varias clases, p.ej.: `pattern:[\s\d]` significa "un carácter de espacio o un dígito".

```smart header="Character classes are shorthands for certain character sets"
For instance:

- **\d** -- is the same as `pattern:[0-9]`,
- **\w** -- is the same as `pattern:[a-zA-Z0-9_]`,
- **\s** -- is the same as `pattern:[\t\n\v\f\r ]`, plus few other rare unicode space characters.
```

### Example: multi-language \w

As the character class `pattern:\w` is a shorthand for `pattern:[a-zA-Z0-9_]`, it can't find Chinese hieroglyphs, Cyrillic letters, etc.

We can write a more universal pattern, that looks for wordly characters in any language. That's easy with unicode properties: `pattern:[\p{Alpha}\p{M}\p{Nd}\p{Pc}\p{Join_C}]`.

Let's decipher it. Similar to `pattern:\w`, we're making a set of our own that includes characters with following unicode properties:

- `Alphabetic` (`Alpha`) - for letters,
- `Mark` (`M`) - for accents,
- `Decimal_Number` (`Nd`) - for digits,
- `Connector_Punctuation` (`Pc`) - for the underscore `'_'` and similar characters,
- `Join_Control` (`Join_C`) - two special codes `200c` and `200d`, used in ligatures, e.g. in Arabic.

An example of use:

```js run
let regexp = /[\p{Alpha}\p{M}\p{Nd}\p{Pc}\p{Join_C}]/gu;

let str = `Hi 你好 12`;

// finds all letters and digits:
alert( str.match(regexp) ); // H,i,你,好,1,2
```

Of course, we can edit this pattern: add unicode properties or remove them. Unicode properties are covered in more details in the article <info:regexp-unicode>.

```warn header="Unicode properties aren't supported in Edge and Firefox"
Unicode properties `pattern:p{…}` are not yet implemented in Edge and Firefox. If we really need them, we can use library [XRegExp](http://xregexp.com/).

Or just use ranges of characters in a language that interests us, e.g.  `pattern:[а-я]` for Cyrillic letters.
```

## Excluding ranges

Besides normal ranges, there are "excluding" ranges that look like `pattern:[^…]`.

They are denoted by a caret character `^` at the start and match any character *except the given ones*.

For instance:

- `pattern:[^aeyo]` -- any character except  `'a'`, `'e'`, `'y'` or `'o'`.
- `pattern:[^0-9]` -- any character except a digit, the same as `pattern:\D`.
- `pattern:[^\s]` -- any non-space character, same as `\S`.

The example below looks for any characters except letters, digits and spaces:

```js run
alert( "alice15@gmail.com".match(/[^\d\sA-Z]/gi) ); // @ and .
```

## Escaping in […]

Usually when we want to find exactly a special character, we need to escape it like `pattern:\.`. And if we need a backslash, then we use `pattern:\\`, and so on.

In square brackets we can use the vast majority of special characters without escaping:

- Symbols `pattern:. + ( )` never need escaping.
- A hyphen `pattern:-` is not escaped in the beginning or the end (where it does not define a range).
- A caret `pattern:^` is only escaped in the beginning (where it means exclusion).
- The closing square bracket `pattern:]` is always escaped (if we need to look for that symbol).

In other words, all special characters are allowed without escaping, except when they mean something for square brackets.

A dot `.` inside square brackets means just a dot. The pattern `pattern:[.,]` would look for one of characters: either a dot or a comma.

In the example below the regexp `pattern:[-().^+]` looks for one of the characters `-().^+`:

```js run
// No need to escape
let regexp = /[-().^+]/g;

alert( "1 + 2 - 3".match(regexp) ); // Matches +, -
```

...But if you decide to escape them "just in case", then there would be no harm:

```js run
// Escaped everything
let regexp = /[\-\(\)\.\^\+]/g;

alert( "1 + 2 - 3".match(regexp) ); // also works: +, -
```

## Ranges and flag "u"

If there are surrogate pairs in the set, flag `pattern:u` is required for them to work correctly.

For instance, let's look for `pattern:[𝒳𝒴]` in the string `subject:𝒳`:

```js run
alert( '𝒳'.match(/[𝒳𝒴]/) ); // shows a strange character, like [?]
// (the search was performed incorrectly, half-character returned)
```

The result is incorrect, because by default regular expressions "don't know" about surrogate pairs.

The regular expression engine thinks that `[𝒳𝒴]` -- are not two, but four characters:
1. left half of `𝒳` `(1)`,
2. right half of `𝒳` `(2)`,
3. left half of `𝒴` `(3)`,
4. right half of `𝒴` `(4)`.

We can see their codes like this:

```js run
for(let i=0; i<'𝒳𝒴'.length; i++) {
  alert('𝒳𝒴'.charCodeAt(i)); // 55349, 56499, 55349, 56500
};
```

So, the example above finds and shows the left half of `𝒳`.

If we add flag `pattern:u`, then the behavior will be correct:

```js run
alert( '𝒳'.match(/[𝒳𝒴]/u) ); // 𝒳
```

The similar situation occurs when looking for a range, such as `[𝒳-𝒴]`.

If we forget to add flag `pattern:u`, there will be an error:

```js run
'𝒳'.match(/[𝒳-𝒴]/); // Error: Invalid regular expression
```

The reason is that without flag `pattern:u` surrogate pairs are perceived as two characters, so `[𝒳-𝒴]` is interpreted as `[<55349><56499>-<55349><56500>]` (every surrogate pair is replaced with its codes). Now it's easy to see that the range `56499-55349` is invalid: its starting code `56499` is greater than the end `55349`. That's the formal reason for the error.

With the flag `pattern:u` the pattern works correctly:

```js run
// look for characters from 𝒳 to 𝒵
alert( '𝒴'.match(/[𝒳-𝒵]/u) ); // 𝒴
```
