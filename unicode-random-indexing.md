title: Unicode strings: you probably don't want random indexing by code point
date: 2014-12-21

Whenever the topic of Unicode string encoding comes up, someone always disparages variable-width encodings like UTF-8 and UTF-16, saying that it's important to be able to get the nth code point in constant time. *This is almost never what you really want,* and people should stop saying this unless they have very exotic needs. A lot of what I'm going to say here is pretty obvious to people who are experienced with Unicode string handling, so if you're one of these people then please forgive me for belaboring a point -- there are a lot of misconceptions going around.

## A quick review of how Unicode works

Unicode represents text in various human languages, like "dog" or "陽明山國家公園", or even "(╯°□°）╯︵ ┻━┻". The symbols that you see, like "d" or "한" or "ȃ͜͡", are called [grapheme clusters](http://www.unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries), because they're made up of one or more graphemes -- the components that you see, like the letter a and the circle over it in "å".

Grapheme clusters are represented as sequences of one or more [code points](http://en.wikipedia.org/wiki/Code_point). These correspond to things like the letters "a" and "ξ", or to more obscure things like [combining characters](http://en.wikipedia.org/wiki/Combining_character) -- if you stick a [combining acute accent (code point U+0301)](http://www.fileformat.info/info/unicode/char/0301/index.htm) after a [lowercase e (code point U+0065)](http://www.fileformat.info/info/unicode/char/0065/index.htm), then it looks like "é". Incidentally, Unicode also happens to have a [code point U+00E9](http://www.fileformat.info/info/unicode/char/e9/index.htm) that looks exactly the same -- é. These are the same grapheme cluster as far as human eyes are concerned, and users will perceive them both exactly the same, and if the user types in one of those and presses backspace then the grapheme cluster should be deleted whether it's one code point or two. And if the user asks how many characters there are in "kamelåså", then the answer should be 8, even if those å graphemes happen to have more than one code point in them.

Finally, those Unicode code points are represented as bytes using some encoding. The simplest, [UTF-32](http://en.wikipedia.org/wiki/UTF-32), just represents each code point as a 32-bit integer. My favorite, [UTF-8](http://en.wikipedia.org/wiki/UTF-8), uses a variable-width encoding to represent each code point using a sequence of 1-4 bytes. The strangely popular [UTF-16](http://en.wikipedia.org/wiki/UTF-16) is also variable-width, using either 2 or 4 bytes.

## Let's look at some common tasks

Suppose that you find yourself with a string containing a bunch of people's names, separated by newlines. Like this:

```
Catherine the Great
藤原定家
Thorin II Oakenshield, son of Thráin, son of Thrór, King under the Mountain
```

The first obvious thing you'd want to do with this is to split it -- find the substrings, like "Catherine the Great", that contain the individual names. You can do this by going through the code points and splitting on line feeds, U+000A -- our good old friend `'\n'`. So far, easy.

Another thing you could do is a substring search -- maybe the user wants to locate the string "Thrór". At this point things get trickier -- you'll probably want to treat the one-code-point and two-code-point representations of the letter ó as equivalent. There are Unicode text-handling libraries that handle this sort of thing, though, so assume you've got one. The substring search will tell you that "Thrór" starts at code point 70 (or 71, depending on how the á in Thráin is represented), or at UTF-8 byte position 79, or at UTF-16 offset 71. Notice that if you have the byte position in an encoded representation of the string, you can index by *that* in constant time, no matter how variable-width the encoding might be.

The same thing applies to stuff like regular expression searches. Suppose you want to find all those "son of X" things with the regular expression `/son of (\S+)/`. Your regular expression matcher can iterate over the string by code point -- easy, no matter what encoding you're using -- and keep track of the byte offsets for its matches. It'll find its first match, "Thráin", at code points 55-62, or UTF-8 byte positions 63-71. Grabbing this substring from the match location is easy and fast -- just remember where the match was in the encoded representation, rather than where it was in the sequence of code points.

For our final act, let's count characters -- or more precisely, grapheme clusters. "Catherine" is 9 characters, and 9 code points, and 9 bytes in UTF-8. "藤原定家" is 4 characters, and 4 code points, and 12 bytes in UTF-8. "Thráin" is 6 characters, and 6 or 7 code points (depending on how you represent the á), and 6 or 7 bytes in UTF-8. If you assume that the number of code points is the same as the number of characters, then you'll sometimes be wrong.

## What do *you* need to do with Unicode strings?

I can't really think of any task for which constant-time string indexing by Unicode code point would be helpful. People use this as an argument against variable-width text encodings *all the time,* but it just doesn't seem that useful.

(If you really, *really* need fast indexing by [any form of text segmentation](http://www.unicode.org/reports/tr29/) on potentially very large strings, you can make this (and almost every other conceivable string operation) decently fast using some sort of augmented B+ tree with gap buffers at the leaves. Probably unnecessary, but it could be fun to write.)
