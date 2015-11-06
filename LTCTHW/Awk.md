# awk

> AWK is an interpreted programming language designed for text processing and typically used as a data extraction and reporting tool. It is a standard feature of most Unix-like operating systems.  --Wikipedia

_Awk_ is a text filter, report writer and lightweight script programming language named after its authors Alfred Aho, Peter Weinberger and Brian Kernighan. With its advantage at processing textual data, in most cases, _Awk_ is used as a tool for formatting, transforming, cutting server logs or crash dump. No doubt that, mastering _Awk_ can be your cornerstone of shell programming.

Wait no more, we start it from the standard awk execution structure:

	awk 'pattern{actions}pattern{actions}...' file
	
One pattern and a series of actions consist one rule. For input text, content is separated into records by specified separator(default separator is line break character). Each record is matched against pattern of rules one by one. Pattern match actions perform. For more details, some examples will explain later.
