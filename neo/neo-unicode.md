# Neo-Unicode

Neo Unicode is a repository with experimental, proof of concept code, that aims to improve
[Pharo](http://www.pharo.org)'s [Unicode](https://en.wikipedia.org/wiki/Unicode) support. 
This is a work in progress that is not yet suitable for public consumption.

The project consists of 2 packages, Neo-Unicode-Core and Neo-Unicode-Tests
in the http://mc.stfx.eu/Neo repository. There is no Metacello configuration.

_Sven Van Caekenberghe (December 2015)_


## The current situation

In Pharo we have Unicode strings as a sequence of Unicode characters, 
each defined/identified by a code point (out of 10.000s covering all languages).

To encode Unicode for external representation as bytes, we use UTF-8 like the rest of the modern world. 

You can read more about the current situation in the first part of the [Character Encoding and Resource Meta Description](http://files.pharo.org/books/enterprisepharo/book/Zinc-Encoding-Meta/Zinc-Encoding-Meta.html) chapter of the Enterprise Pharo book.

All in all, the current situation is OK for day to day practical use.


## What is the problem ? What is missing ?

The world is a complex place and the [Unicode](http://www.unicode.org) standard 
tries to cover all possible languages, situations, usages and interpretations, combined with backward compatibility. 
For 1000s of pages, all kinds of exceptions and special rules are described.
Obviously we do not have enough support for all of this.

We do not need all of Unicode, nor do we need to implement all edge cases, but we need more. 
In particular, we need to support normalization, casing and collation/sorting.


## The Unicode Character Database

A first step for better Unicode support is to work with the actual data defined by the standard.
The main element is the Unicode Character Database, which lists details for each of approximately 30.000 code points.

**NeoUnicodeCharacterData** holds this information per code point.
The database is loaded in Pharo from an external URL, on demand, and then cached.

    NeoUnicodeCharacterData database
    
An extension method provides access to this data from a character.

    $é unicodeCharacterData.
    $e unicodeCharacterData.
    $1 unicodeCharacterData.

A GT Inspector extension helps in interpreting this data, 
as it is stored in a way that favours compactness more than readability.

![Image](https://raw.githubusercontent.com/svenvc/docs/master/neo/custom-ucd-inspector.png)

See the class and method comments, as well as the unit tests for more information.
Obviously, correctly interpreting all these properties is very complex.


## Normalization

Some characters or letters can be written in more than one way.

The simplest example in a common language (the French letter é) is

    LATIN SMALL LETTER E WITH ACUTE [U+00E9]

which can also be written as

    LATIN SMALL LETTER E [U+0065] followed by COMBINING ACUTE ACCENT [U+0301]

The former being a composed normal form, the latter a decomposed normal form. 

Not only diacritical marks (accents) lead to two representations, various special characters might do too.
For example, the ellipsis character of 3 dots can be considered, under a certain interpretation, 
to be equivalent to 3 separate dots in a row.

    HORIZONTAL ELLIPSIS [U+2026]
    FULL STOP [U+002E] followed by FULL STOP [U+002E] followed by FULL STOP [U+002E]

Some characters have more than 2 representations. The Ångström symbol Å can be written in 3 ways !

    ANGSTROM SIGN [U+212B]
    LATIN CAPITAL LETTER A WITH RING ABOVE [U+00C5]
    LATIN CAPITAL LETTER A [U+0041] followed by COMBINING RING ABOVE [U+030A]

Normalization is needed to properly compare strings.
There are different normalization forms, NFC, NFD, NFKC and NFKD, each with specific rules.

**NeoUnicodeNormalizer** is a first and simplified implementation of this operation,
using the Unicode Character Database.

    NeoUnicodeNormalizer new decomposeString: 'les élèves Français'.
    NeoUnicodeNormalizer new composeString: 'les e´le`ves Franc ̧is'.
    NeoUnicodeNormalizer new decomposeString: 'Düsseldorf Königsallee'.
    NeoUnicodeNormalizer new composeString: 'Du¨sseldorf Ko¨nigsallee'.

(Note: copy/paste of the above decomposed strings won't work,
generate the decomposed strings yourself in Pharo using #decomposeString:)


## Casing

Recognizing lower, upper and title case and converting between them is defined by Unicode as well.
The Unicode Character Database can be used for these operations.

**NeoUnicodeCaser** is a simple tool to do these conversions,
using the Unicode Character Database.

    NeoUnicodeCaser new case: #uppercase string: 'abc'.
    NeoUnicodeCaser new case: #lowercase string: 'ABC'.
    NeoUnicodeCaser new case: #titlecase string: 'abc'.


## Collation/Sorting

Nothing yet.


## Legacy Notes

The current **Unicode** class seems to be capable of reading an older version of UnicodeData.txt 
but uses it only for casing.

The current **CombinedChar** class seems to be capable of reading an older version of UnicodeData.txt 
but uses it only combining composable characters.

Neither holds on to the database as it is pretty big, but they do hold onto quite some data. 
This could probably be optimized in the future.


## References

Some links to useful documents:
- http://www.unicode.org/Public/UNIDATA/
- http://www.unicode.org/reports/tr15/
- http://www.unicode.org/versions/Unicode8.0.0/ch03.pdf (3.11)
- ftp://ftp.unicode.org/Public/3.0-Update/UnicodeCharacterDatabase-3.0.0.html
- http://unicode.org/faq/normalization.html
- http://www.unicode.org/reports/tr44/
- http://blog.golang.org/strings
- http://blog.golang.org/normalization
- http://docs.oracle.com/javase/7/docs/api/java/text/Normalizer.html
- https://developer.apple.com/swift/blog/?id=30
- http://useyourloaf.com/blog/swift-string-cheat-sheet.html
- https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html
- https://www.mikeash.com/pyblog/friday-qa-2015-11-06-why-is-swifts-string-api-so-hard.html
- http://graphemica.com/
- http://www.charbase.com
- http://www.fileformat.info/info/unicode/index.htm
- https://www.azabani.com/pages/gbu/



