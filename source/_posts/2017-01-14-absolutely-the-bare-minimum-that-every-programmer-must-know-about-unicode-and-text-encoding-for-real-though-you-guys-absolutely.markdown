---
layout: post
title: Absolutely The Bare Minimum That Every Programmer Must Know About Unicode and Text Encoding For Real Though You Guys (Absolutely)
date: 2017-01-14 13:11:58 -0800
comments: true
categories: Unicode, Encoding, ASCII, UTF Encodings, UTF-8, UTF-16, Ruby
---

Until recently, character encoding had always been an elusive topic -- I knew
enough about it to get by 99% of the time, but lacked a rich enough mental
schema to reason about it intuitively when things went wrong.

Truly wrapping my head around it in a deep way always seemed a bit
prohibiting, and I think there's a reason for this: it's because very programs
that we as programmers spend our time write are _themselves_ composed of encoded
text.  And so getting to the bottom of encoding feels a bit like
pulling up the floor boards to see how the ground we stand on is constructed.

But alas, there arose a bug in the software I write at work which
forced me down into the conceptual subbasement where
encoding lives.  And so in this post, I'm going to tell you about what I
found down there.

The title of this post is a wink/nod toward two widely-read
posts on this topic, which helped me enormously along the way:

- https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/
- http://kunststube.net/encoding/

Both are excellent articles which I totally recommend reading.

The first, by Joel Spolsky (of stack overflow, trello, and general
internet blogging/punditship fame), is probably the most well-known, in
addition to being the most abstract and assuming the most upfront
knowledge on the part of the reader.

The second, by David C. Zentgraf (stack overflow), is more concrete,
including a fair amount of technical detail, and walking you through
definitions and nomenclature and that sort of thing.  It's also written
with a slight bent toward PHP.

What I'm hoping to accompish with this post is utter cement.  Let's not
assume anything, fill in as much detail as possible, and really get to
the bottom of this otherwise scary topic.  Most of this stuff if
generalizable, but I'm coming at this from a Ruby background, so I'll
also include a section on encoding in Ruby as well.

Ready? Ok.

In the Beginning, There Was Binary
---------------------------------

Before we can talk about encoding, we need to put some notational tools
in our respective toolbelts.

At the lowest level of abstraction, all of the information stored on a
computer is represented as a sequence of zeros and ones (or more technically, as a series of [memory cells](https://en.wikipedia.org/wiki/Memory_cell_(binary)
in one of two states of voltage, which by convention we map onto zero
and one).  This information stored in this format is commonly referred to as 'binary' or 'raw binary'.

A single digit of binary (corresponding to a memory cell) is referred to
as a 'bit'.  Bits are usually grouped in chunks of 8 called 'bytes'.
Remember this; we will come back to it later.

Binary can be used to represent numbers.  To understand how, we first have to
understand that decimal notation, the way we learn to write and think
about numbers as children and use for most of our everyday lives, is just one of
many notational systems for numbers.  Decimal notation is a base-10 numbering
system, meaning each 'position' (sometimes refered to as a 'place', e.g.
'tens place', 'hundreds place') can be inhabited by one of ten symbols, 0-9,
and positions themselves represent powers of ten starting with 10 ^ 0 at the
right-most position and increasing by one power of ten with each step
to the left.

For example, in decimal notation you can think of 365 as:

> 3(10 ^ 2) + 6(10 ^ 1) + 5(10 ^ 0)

or equivalently,

> 3(100) + 6(10) + 5(1)

or equivalently

> "3 in the **hundreds place**, 6 in the **tens place**, 5 in the **ones place**"

Because we're so used to viewing numbers this way, it can seem a bit
pedantic to write this out.  We take for granted that numbers are written and interpreted
like this in base-10, but it's also possible to represent numbers in
notational systems with other bases.  When we use
binary to represent numbers on a computer for example, since we have only two possible
symbols (zero or one) at our disposal, it's more convenient to switch to
a base-2 notational system or 'binary notation'.
In binary notation, positions represent powers of 2 (rather than powers
of 10), starting with 2 ^ 0 at the right-most position and increasing by
one power of two with each step to the left.

For example, in binary notation you can think of 101 as:

> 1(2 ^ 2) + 0(2 ^ 1) + 1(2 ^ 0)

or equivalently,

> 1(4) + 0(2) + 1(1)

or equivalently,

> "1 in the **fours place**, 0 in the **twos place**, 1 in the **ones place**

or translated back to familiar old decimal notation,

> 5

So, to represent the decimal number '5', a computer would store the
binary sequence '101'.

**A quick note on notation**: From this point forward, in order to avoid
confusing numbers in binary notation with those in standard decimal
notation, I will follow the convention used by many programming
languages (including Ruby) of prefixing binary numbers with '0b'.  Using
the previous example to illustrate this more concretely, we represent
binary five ('101') as '0b101', so that we don't confuse it with decimal
one-hundred and one ('101').

Representing Bytes
------------------

As we mentioned earlier, binary bits are usually stored/interpreted in 8-digit
chunks called 'bytes'.

When a computer stores a number in a byte, at the least abstract level
it always uses binary.  By contrast, when we humans *talk about* a value
stored in a byte, we have a variety of notational options at our
disposal.

We could choose to use binary notation like a computer and
represent a byte as a value between 0b00000000 and 0b11111111.  That
approach has the advantage being isomorphic with our hardware -- it
gives us a nice little pictograph of the 8 bits that compose the byte, each
of them in one of two states.  However, it has the disadvantage of
being a bit opaque to the human brain when interpreted as a numberic
value since we're used to dealing with decimal notation.
(E.g. 0b10110001 is harder for me to interpret as a numeric value than
its decimal equivalent, 177.)  It is also less economical in terms of
typographic space -- we use 8 digits in binary to represent numbers that
would require only 3 digits in decimal.

So, what if we chose to to represent a byte in decimal notation as a
value between 0 and 255.  Well, as we said that has the advantage of
being easy for the human brain to read and reason about and
typographically concise, but we loose the isomorphism or mapping between
digits and bits stored on the computer, and the conversion from decimal
back to binary is (for a human) relatively expensive.

Hexadecimal Notation
--------------------
Enter hexadecimal notation, or 'hex'.  Hexadecimal notation is base-16, meaning
that there are sixteen symbols that can inhabit each position, and that
and positions themselves represent powers of 16 starting with 16 ^ 0 at the
right-most position and increasing by one power of 16 with each step
to the left.  In the arabic numbering system, we only have 10 symbols --
0-9.  So in order to pull off using a base-16, we need to scrounge up
six additional symbols.  The solution is to just use the first six
letters of the alphabet, A-F. A stands for decimal 11, B stand for
decimal 12, etc.

NOTE: number formatting rules
