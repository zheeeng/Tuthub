# AWK

> AWK is an interpreted programming language designed for text processing and typically used as a data extraction and reporting utility. It is a standard feature of most Unix-like operating systems.    - Wikipedia

_AWK_ is a text filter, report writer and lightweight script programming language named after its authors Alfred Aho, Peter Weinberger and Brian Kernighan. With its advantage at processing textual data, in most cases, _AWK_ is used as a tool for formatting, transforming, cutting server logs or crash dump. No doubt that, mastering _AWK_ can be your cornerstone of shell programming.

Wait no more, we start it from the standard awk execution structure:

	awk 'pattern{actions}pattern{actions}...' [input-files]
	
Elemental unit of _AWK_ is a rule which consist of one pattern and a series of actions. For input text, content is separated into records by specified separator(default separator is line break character). Each record is matched against pattern of rules one by one. Pattern matched associated actions perform. For more details, some examples will explain them later.
