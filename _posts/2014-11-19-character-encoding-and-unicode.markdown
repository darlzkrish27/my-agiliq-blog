---
layout: post
title:  "Character encoding and Unicode"
date:   2014-11-19 13:45:15+05:30
tags: encoding
author: akshar
---
### Before you proceed.

This <a href="http://www.joelonsoftware.com/articles/Unicode.html" target="_blank">blog post</a> by Joel Spolsky got me interested in Unicode and character encoding and taught me several things. I suggest you read this.

### Basics

Computers only work with 0 and 1 i.e binary. Any character needs to have a binary representation so computer can store it on disk or in the memory.

Computer cannot store 'a'. Computer can only store a bit pattern, say 01100001. So a character needs to have a binary representation so it can be stored on disk.

When you write 'a' to a file and save it, binary representation 01100001 or whatever is the binary representation of 'a' gets saved to disk. When the text editor reads this file, it finds 01100001 and knows that this is the binary representation of character 'a' and so the text editor shows you 'a'.

There are various ways in which characters can be converted to binary. Those ways are called character encoding schemes. eg: ascii, utf-8

Encoding means the process of converting a string to a binary representation.

A character encoding scheme, say 'encoding1', gives a one-to-one mapping between a character and a bit pattern.  A character say 'a' will have only one binary representation, say 01100001. 'a' cannot map to any other binary representatio apart from 01100001. Vice versa, in a particular character encoding scheme, bit pattern 01100001 can only mean a particular character. This bit pattern cannot map to two different characters.

Different encoding schemes(hereafter called encoding) might have different binary representation for the same character. An encoding named 'encoding1' could represent 'a' as 01100001 while 'encoding2' could represent 'a' as 11111111.

### A tool called **xxd**

Earlier I said, Computers can only store bit pattern. With **xxd**, you can see the binary content of a file. If there is a file called `abc.txt` in your current directory, you can say:

	xxd -b abc.txt

And it will show you the binary representation of what abc.txt stores.

### ASCII:

ASCII is one possible character encoding scheme.

ASCII says that binary representation of 'a' is 01100001. ASCII says 'a' needs to be stored as 01100001 on disk.

Theoritically there could be another character encoding scheme which can say that binary representation of 'a' is 11111111. If this encoding scheme is being used, 'a' would be stored as 11111111 on disk.

Quoting Wikipedia page on ASCII:

	ASCII specifies a correspondence between digital bit patterns and character symbols

It means there is a one to one mapping between characters and their binary representation.

ASCII could only represent English characters. It could not represent Chinese or Tamil characters. Since ASCII could not handle non-english characters, other encoding schemes evolved.

### Unicode

Unicode can represent every character of almost all widely used languages. With unicode it is possible to represent Chinese, Tamil and any other character set you could think of. And if a new language(character set) comes in future, Unicode would be able to represent that too.

But unicode doesn't give a binary representation of characters. The binary representation task is left for the encoding scheme. Unicode is not an encoding scheme. Unicode is a standard and character encoding schems implement this standard in a particular way.

Unicode only gives a **codepoint** for the character.

An example would make it clear. Feminine ordinal indicator, 'ª' has codepoint U+00AA. After giving a codepoint, Unicode doesn't care how the encoding scheme represents it. utf-8 is one encoding scheme which complies with Unicode. utf-8 represents this codepoint as hexadecimal "0xc2 0xaa" whose binary equivalent is "11000010 10101010". iso-latin-5(iso-8859-9), another encoding represents this codepoint as hexadecimal "0xaa" whose binary equivalent is "10101010".

#### Verifying how this character is stored

We will create two files and use different encoding schemes to store this character in the two files.

* First case: Use encoding scheme 'utf-8' to encode the character and store in file char_utf8.txt
* Second case: Use encoding scheme 'iso-latin-5' to encode the character and store in file char_latin.txt

And then we will use `xxd` to find out the binary representation of 'ª' stored in these two files.

It is easy with Python to convert a codepoint to the encoding scheme's binary representation. So I will use Python. It's only few lines of Python, you should be able to follow even if you don't use Python. You only need Python installed on your machine, so you can execute the snippets.

##### Using encoding utf-8

Start Python shell. (I am using Python 2.7)

	# Store the codepoint of `ordinal` character in a variable
	ordinal_unicode = u'\u00aa'

	# Open a file where you will store this character, after encoding it to utf8
	f = open('ordinal_utf8.txt', 'wb')

	# Encode the character to utf-8 and write to the file.
	# Encoding means converting character to a bit representation, remember?
	f.write(ordinal_unicode.encode('utf-8'))

	f.close()

##### Using encoding iso-latin-5

	ordinal_unicode = u'\u00aa'

	f = open('ordinal_latin.txt', 'wb')

	# Latin 5 is also called iso-8859-9
	f.write(ordinal_unicode.encode('iso-8859-9'))

	f.close()

Check the content of both file using xxd.

	~ $ xxd -b ordinal_utf8.txt
	0000000: 11000010 10101010

	~ $ xxd -b ordinal_latin.txt
	0000000: 10101010

So, the same character when encoded with different encoding schemes, give different binary representation.

In `xxd` output, you only need to see the bits after ":". Initial "0000000" do not concern us.
	
#### Magic with **cat**

If you use Windows, you'll have to use **type** instead of **cat**.

I am on Mac, and my terminal encoding is set to utf-8. So when I try `cat <filename>`, my terminal will see the binary representation of file and see what characters they map to in utf-8 and will show those characters. If your terminal encoding is not utf-8, change it to utf-8 for now.

	$ cat ordinal_utf8.txt
	ª

Feminine ordinal indicator gets printed in this case. Now try cat on file where Latin-5 encoded character is written.

	$ cat ordinal_latin.txt
	�

Some gibberish gets printed. Terminal reads the binary content of file. But in utf-8(my terminal's encoding), *10101010*, doesn't map to any character. So terminal doesn't know what character to print and so prints question mark.

**Change the terminal encoding to Latin-5**

On Mac, it can be changed in following way. **Profiles** -> **Open Profiles** -> **Edit Profiles** -> **Terminal** -> **Character Encoding** -> **Choose "Turkish (ISO Latin 5)"**

Again cat ordinal_latin.txt

	$ cat ordinal_latin.txt
	ª

Terminal reads the binary content of file. In Latin-5, *10101010* is mapped to feminine ordinal character. And hence that gets printed.

#### Surprise

Cat on ordinal_utf8.txt

	$ cat ordinal_utf8.txt
	Âª	

Why did output of ordinal_utf8.txt change? As we know, binary content of ordinal_utf8.txt is *11000010 10101010*. It is 2 bytes. Our terminal encoding is set to Latin-5. In Latin-5, *11000010*(first byte) is mapped to character *Â* and *10101010*(second byte) is mapped to character *ª*. And that's what our output shows.

Change your terminal encoding back to whatever was your default encoding to avert any surprising behavior.


### More takeaway

* Unicode gives **codepoints** for different characters. It can give codepoint for any character in any language. Suppose a new character gets added in future, Unicode would give a codepoint for that new character too.

* Each encoding, which confirms to Unicode, has a one-to-one mapping between a codepoint and the binary representation of code point.

* In file I write 'a' in encoding A, so it's written as 00000001. But my terminal is set to encoding B. And if encoding B thinks 00000001 as character 'b'. Then my terminal will show as 'b'.

* You need to know the encoding in which a file is written to meaningfully read it. Else, you will have a file written in Latin-5 and you will assume it is in utf-8 and will see gibberish in that case.

### Updates

Added a <a href="http://agiliq.com/blog/2014/12/understanding-python-unicode-str-unicodeencodeerro/" target="_blank">blog post</a> about Python unicode handling, UnicodeEncodeError and UnicodeDecodeError. Wrote a post to describe a <a href="http://agiliq.com/blog/2014/12/how-not-knowing-encoding-can-trip-you/">practical scenario</a> where not knowing encoding of a file can hurt.

