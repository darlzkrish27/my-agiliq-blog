---
layout: post
title:  "Understanding Python unicode, str, UnicodeEncodeError and UnicodeDecodeError"
date:   2014-12-08 15:43:30+05:30
tags: python
author: akshar
---
## Agenda

* Understanding the basics of Python 'unicode' and 'str' type
* Deliberately causing UnicodeEncodeError and UnicodeDecodeError and fixing it.
* A practical example showing how encoding issues can trip you.

<br/>
To follow along easily, it would help if you understand concept of unicode, encoding and decoding in general. Please refer to our <a href="http://agiliq.com/blog/2014/11/character-encoding-and-unicode/" target="_blank">last blog</a> to understand the basics of unicode and encoding.

This post assumes you use Python 2.7 and this will not be useful if you are using Python 3.

## Basics

Make sure your terminal encoding is set to utf-8.

As discussed in last post, Unicode is just a standard which gives codepoint for different characters. You cannot store codepoint of a character on disk. Codepoint of the character must be encoded using some encoding scheme before it can be stored in a file.

Codepoints are integers. eg: Codepoint of character 'a' is U+0061 which is integer 97. This codepoint has a different binary representation in different encoding schemes. Or other way of saying it is, this codepoint has different byte sequence in different encoding schemes. And the byte sequence gets written to disk when we write 'a' to a file.

Codepoint of 'ä' is U+00E4, which is integer 228. This codepoint has a different binary representation, or byte sequence, in different encoding schemes.

Usually binary representation will not be shown to you. The binary representation would be converted to a hexadecimal number in the output. eg: In 'utf-8' encoding, 'ä' is represented by '11000011 10100100'. But most of the times you will see it's hexadecimal equivalent which is 'c3a4', written as '\xc3\xa4'.

Python has two different datatypes. One is 'unicode' and other is 'str'. Type 'unicode' is meant for working with **codepoints** of characters. Type 'str' is meant for working with **encoded binary representation** of characters.

A 'unicode' object needs to be converted to 'str' object before Python can write the character to a file. A 'unicode' object needs to be converted to 'str' object for the character to be printed.

## Python 'unicode' and 'str' type

We will use a character which has different binary representation in different encoding schemes. **ä** is one such character. This character is called 'LATIN SMALL LETTER A WITH DIAERESIS'.

Codepoint for this character is U+00E4. You can check it at <a href="http://www.utf8-chartable.de/">http://www.utf8-chartable.de/</a>

The way to define a Unicode codepoint is:

	>>> uni_latin_a = u'\u00e4'

Check it's type:

	>>> type(uni_latin_a)
	<type 'unicode'>

A unicode starts with 'u' followed by quote and the codepoint has to be preceded by '\u'.

Let's define a 'str'.

	>>> str_normal_a = 'a'

Check it's type:

	>>> type(str_normal_a)
	<type 'str'>

## UnicodeEncodeError

Let's try to convert 'unicode' to 'str'

	>>> str_latin_a = uni_latin_a.encode()
	Traceback (most recent call last):
	  File "<ipython-input-22-b3d11d4d77fd>", line 1, in <module>
		str_latin_a = uni_latin_a.encode()
	UnicodeEncodeError: 'ascii' codec can't encode character u'\xe4' in position 0: ordinal not in range(128)

When 'encode()' is called, by default ascii encoding scheme is used. So 'encode()' is equivalent to 'encode('ascii')'. ascii can only encode characters whose codepoint is less than 128. uni_latin_a represents a character whose codepoint is greater than 128. And so we get a UnicodeEncodeError.

utf-8 encoding scheme can encode codepoints greater than 128. Let's use 'utf-8' to encode uni_latin_a.

	>>> str_latin_a = uni_latin_a.encode('utf-8')

This passes. Let's check the type of str_latin_a.

	>>> type(str_latin_a)
	<type 'str'>

"encode()" is meant to be used on a 'unicode' to get a 'str'. Make sure you are call 'encode()' on a 'unicode' and never call it on a 'str'.

	>>> print str_latin_a
	ä

Let's see the hexadecimal representation of str_latin_a.

	>>> str_latin_a
	'\xc3\xa4'

So, utf-8 representation of codepoint 'U+00E4' is '\xc3\xa4'. You can also verify it at the table provided at <a href="http://www.utf8-chartable.de/" target="_blank">http://www.utf8-chartable.de/</a>.

A 'unicode' cannot be written to a file.

	>>> f = open('uni_latin_a.txt', 'w')
	>>> f.write(uni_latin_a)
	Traceback (most recent call last):
		File "<ipython-input-21-38c6475cde9f>", line 1, in <module>
			f.write(uni_latin_a)
	UnicodeEncodeError: 'ascii' codec can't encode character u'\xe4' in
	position 0: ordinal not in range(128)

A 'unicode' object must be encoded to get it's binary representation, and then encoded binary representation gets written to the file.

Python is trying to do implicit encoding here. Python can only write 'str' to a file. Since we are passing a 'unicode' to write, python tries to convert the 'unicode' into 'str'. Internally Python runs f.write(uni_latin_a.encode('ascii')).

ascii encoding scheme can only encode characters whose codepoint is less than 128. uni_latin_a represents a character whose codepoint is greater than 128. And so we get a UnicodeEncodeError.

Encode uni_latin_a using utf-8 so it can be written:

	>>> str_latin_a = uni_latin_a.encode('utf-8')
	>>> f.write(str_latin_a)
	>>> f.close()

Check uni_latin_a.txt in your editor. If your editor understands utf-8 encoded strings, it will show the expected character.

## UnicodeDecodeError

UnicodeDecodeError will usually happen when you try to process something read from a file.

We just wrote a utf-8 encoded character to 'uni_latin_a.txt'. Let's read this file.

	>>> f = open('uni_latin_a.txt')
	>>> read_str_latin_a = f.read()
	>>> f.close()
	>>> print read_str_latin_a
	ä
	>>> type(read_str_latin_a)
	<type 'str'>

'type()' verifies that whenever we read from a file, we get an instance of 'str', and not an instance of 'unicode'.

Let's see the hexadecimal representation of read_str_latin_a

	>>> read_str_latin_a
	'\xc3\xa4'

Several times it makes sense to work with Unicode internally and in such case we will need to convert the read value into unicode.

Decoding is the process of converting an encoded representation into Unicode codepoint.

'.decode()' is meant to convert from 'str' to 'unicode'. Always use '.decode()' on a 'str'. Never use it on 'unicode' object.

Let's try converting read_str_latin_a to a 'unicode' object.

	>>> read_uni_latin_a = read_str_latin_a.decode()
	Traceback (most recent call last):
	  File "<ipython-input-14-adfeb64a792b>", line 1, in <module>
		read_uni_latin_a = read_str_latin_a.decode()
	UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position 0: ordinal not in range(128)

When '.decode()' is called, Python default thinks that the string was encoded using 'ascii'. So it tries to find the Unicode codepoint which corresponds to this encoded representation. In ascii, no Unicode codepoint corrresponds to '\xc3\xa4' and so an error is raised.

We already know that encoding was done using 'utf-8' when writing to the file. So use 'utf-8' with decode().

	>>> read_uni_latin_a = read_str_latin_a.decode('utf-8')
	>>> print read_uni_latin_a
	ä

Suppose we did not know that the file content was encoded with 'utf-8'. In that case, we could have tried decoding it with latin-5 or any other encoding scheme. Suppose we try latin-5.

	>>> read_uni_latin_a = read_str_latin_a.decode('8859')
	>>> print read_uni_latin_a
	Ã¤

### What happened here?

See what read_str_latin_a is:

	>>> read_str_latin_a
	'\xc3\xa4'

In encoding scheme 8859, U+00C3 when encoded gives hexadecimal '\xc3' and U+00A4 when encoded gives hexadecimal '\xa4'. So when '\xc3\xa4' is decoded, it gives back codepoints U+00C3 and U+00A4. Codepoint U+00C3 means 'Ã' and codepoint U+00A4 means '¤'. And that's what we see in output.

That's why it's important to know the encoding of a file otherwise we will read it wrong.

### Takeaway

* Unicode codepoints can be stored in type 'unicode'. When unicode codepoints are defined, they are stored in 'unicode' and haven't been converted to a particular encoding.

* Unicodes are prepended with u''. When we do this, python understands that we want to store the codepoint of the character/string.

* Make sure you .encode() a unicode and not a string.

* Make sure you .decode() a string and not a unicode.

This post is becoming big, so I am putting <a href="http://agiliq.com/blog/2014/12/how-not-knowing-encoding-can-trip-you/" target="_blank">practical example where unicode can trip you</a> in next post.


