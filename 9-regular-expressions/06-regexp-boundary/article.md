# Límite de palabra: \b

Un límite de palabra `pattern:\b` es una prueba, al igual que `pattern:^` y `pattter:$`.

Cuando el motor regexp (módulo de programa que implementa la búsqueda de expresiones regulares) se encuentra con `pattern:\b`, comprueba que la posición en la cadena es un límite de palabra.

Hay tres posiciones diferentes que califican como límites de palabras:

- Al comienzo de la cadena, si el primer carácter de cadena es un carácter de palabra `pattern:\w`.
- Entre dos caracteres en la cadena, donde uno es un carácter de palabra `pattern:\w` y el otro no.
- Al final de la cadena, si el último carácter de la cadena es un carácter de palabra `pattern:\w`.

Por ejemplo, la expresión regular `pattern:\bJava\b` se encontrará en `subject:Hello, Java!`, Donde `subject:Java` es una palabra independiente, pero no en `subject:Hello, JavaScript!`.

```js run
alert( "Hola, Java!".match(/\bJava\b/) ); // Java
alert( "Hola, JavaScript!".match(/\bJava\b/) ); // null
```

En la cadena `subject:Hola, Java!` las siguientes posiciones corresponden a `pattern:\b`:

![](hello-java-boundaries.svg)

So, it matches the pattern `pattern:\bHello\b`, because:

1. At the beginning of the string matches the first test `pattern:\b`.
2. Then matches the word `pattern:Hello`.
3. Then the test `pattern:\b` matches again, as we're between `subject:o` and a space.

The pattern `pattern:\bHello\b` would also match. But not `pattern:\bHell\b` (because there's no word boundary after `l`) and not `Java!\b` (because the exclamation sign is not a wordly character `pattern:\w`, so there's no word boundary after it).

```js run
alert( "Hello, Java!".match(/\bHello\b/) ); // Hello
alert( "Hello, Java!".match(/\bJava\b/) );  // Java
alert( "Hello, Java!".match(/\bHell\b/) );  // null (no match)
alert( "Hello, Java!".match(/\bJava!\b/) ); // null (no match)
```

We can use `pattern:\b` not only with words, but with digits as well.

For example, the pattern `pattern:\b\d\d\b` looks for standalone 2-digit numbers. In other words, it looks for 2-digit numbers that are surrounded by characters different from `pattern:\w`, such as spaces or punctuation (or text start/end).

```js run
alert( "1 23 456 78".match(/\b\d\d\b/g) ); // 23,78
alert( "12,34,56".match(/\b\d\d\b/g) ); // 12,34,56
```

```warn header="Word boundary `pattern:\b` doesn't work for non-latin alphabets"
The word boundary test `pattern:\b` checks that there should be `pattern:\w` on the one side from the position and "not `pattern:\w`" - on the other side.

But `pattern:\w` means a latin letter `a-z` (or a digit or an underscore), so the test doesn't work for other characters, e.g. cyrillic letters or hieroglyphs.
```
