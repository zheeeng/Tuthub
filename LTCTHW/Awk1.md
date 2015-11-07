## EXAMPLE 1

Consider the _Awk_ structure, let me say some shallow features extraction from an example:

```
# awk -F ':' ' \
	BEGIN { \
		RS = "\n"; OFS = "-> "; ORS = " !;\n"; mode = "normal"; line_count = 0; \
		printf "\nTerminal: " ENVIRON["SHELL"] "\n"; \
		printf "Count from files " ARGV[1] " and " ARGV[2] "(total " ARGC-1 " files)\n"; \
		printf "[Count Test Start]\n"; \
	} \
	/^_?[t-z]+/ &&  $3 % 2 == 0 && mode == "normal" || mode == "debug" { \
		line_count++; field_count += NF; \
		print NR, FNR, "\t" $3 "\t" $1 "\t\t" $6 "\t[" FILENAME "]" \
	} \
	END { \
		printf "[Count Test Completed]\n"; \
		printf "The number of matched records: " line_count "\n"; \
		printf "Total fields: " field_count " \n"; \
	} \
' /etc/passwd /etc/shadow
```

In the end of the command, input could be a text file or pipeline textual stream. Here input is specified as file `/etc/passwd` and `/etc/shadow`, some gists about this file here:

1. File `/etc/passwd` is text file storing user account information. Each record of it looks like `root:*:0:0:System Administrator:/var/root:/bin/sh`
2. For `/etc/passwd`, each field is separated with `":"`, and are as follows:
	1. Username
	2. Password, is stored with encryption in file `/etc/shadow`.
	3. User ID
	4. Group ID
	5. User ID Info
	6. Home directory
	7. Shell access
3. Each record of file `/etc/shadow` contains 8 fields look like `proxy:*:16583:0:99999:7:::`.
4. For `/etc/shadow`, the meaning of the fields are:
	1. Username
	2. Password, blank value indicates no requirements for password, "*" or "!" means account disable.
	3. Days since latest password changed(count since 1/1/1970)
	4. Days the password may be changed
	5. Days after which password must be changed
	6. Days before warn user to change password
	7. Days for account disable delay after password expires
	8. Days after which account be disabled(count since 1/1/1970)
	9. Reserved field for possible future use

Print specified fields filtered out by some patterns and a simple statistics are our goals in this example.

### PATTERNS:

Each record of `/etc/passwd` or `/etc/shadow` match against the specified patterns, if the value of pattern matching expression is nonzero or non-null string, namely the equivalent of boolean true in _Awk_, the followed actions will execute on this record **(Note: Processing -- Actions execute on pattern matched records )**. In _Awk_, either of pattern and actions (but not both) may be omitted. Omitted pattern match all records and omitted actions execute printing the original records **(Note: Omitted patterns and omitted actions all is ok)**. In the example above, there are two special patterns `BEGIN{ mode = ...}` supply startup actions and `END{print ...}` supply cleanup actions **(Note: Two special patterns -- BEGIN pattern and END pattern)**. Both execute without any conditions. Apart from the omitted pattern and the special patterns, how about the common patterns? _Awk_ use patterns as following:

1. /Regular expression/
2. Common expression
3. Pattern paris specifies input and output record match

Comprehensive pattern `/^_?[l-z]+/ &&  $3 % 2 == 0 && mode == "normal" || mode == "debug"` is an always pass through without pattern verify if variable `mode` is assigned "debug". While "normal" mode, valid records are regular expression `/^_?[l-z]+/` matched, and built-in variable `$3` is even number. Not only logical operators, arithmetic operators and comparison operators work in _Awk_, most C-style operators are supported (sorted by priority from higher to lower, bold items are not C-style operators):

Operators		|Meaning			|Priority	| Direction
:-------------:	|:----------------:	|:--------:	|:------------:
()				| Group				| 1			| left →
**$ (unary)**	| Field reference	| 2			| ← right
++x (unary)	| Pre-increment		| 3			| ← right
--x (unary)		| Pre-decrement		| 3			| ← right
x++ (unary)		| Post-increment	| 3			| ← right
--x	(unary)		| Post-decrement	| 3			| ← right
** ^,\*\* **	| Exponentiation	| 4			| left →
- (unary)		| Negation			| 5			| ← right
**+ (unary)**	| Convert expression to number	| 5 | ← right
! (unary)		| Logical "not"		| 5			| ← right
\*				| Multiplication	| 6			| left →
/				| Division			| 6			| left →
%				| Modulus			| 6			| left →
\+				| Addition			| 7			| left →
\-				| Subtraction		| 7			| left →
**blank**		| String concatenation	| 8		| left →
==				| Equal sign		| 9			| left →
!=				| Not-equal sign	| 9			| left →
\>				| Greater-than sign	| 9			| left →
<				| Less-than sign	| 9			| left →
\>=				| Not-less-than sign		| 9 | left →
<=				| Not-greater-than sign		| 9 | left →
**> (following a command)**	| File write redirection  | 9 | left →
**>> (following a command)**	| File append redirection | 9 | left →
**\| (following a command)**	| Program redirection     | 9 | left →
**~**			| Regular-match sign	| 10	| left →
**!~**			| Regular-unmatch sign	| 10	| left →
**in**			| Array membership	| 11		| iteration
&& 				| Logical "and"		| 12		| left →
\|\|			| Logical "or"		| 13		| left →
? : (ternary)	| Conditional expression | 14	| left →
=				| Assignment		| 15		| ← right
+=,-=,\*=,/=,%=,**^=,\*\*=** | Self compute & Assignment | 15 | ← right

**(Note: Most operators are C-style operators)** 

### FIELDS:

You may be confused by `$3`. It is a built-variable refers to a field of record. In _Awk_, `$0` refers to the entire record, `$1 ~ $n` refer to fields from the 1st to the nth field which is separated by space or `tab` character by default **(Note: Field reference -- $0 and $1 ~ $n)**. In our case, separate symbol is specified as `":"` through `awk -F ':'`, alternative specifying method is:

	awk 'BEGIN{FS=":"}...`
	
Should be pointed out that, when executing the `BEGIN pattern`, there are no available field references.

### BUILT-IN VARIABLES:

We have known built-in variables `$0` and `$1 ~ $n`. In our case above some special characters symbol `FS`, `RS`, `OFS`, `ORS`, `ENVIRON`, `ARGV`, `ARGC`, `NR`, `FNR` and `FILENAME` are also built-in variables. Where are the others? A Summary of _Awk_ built-in variables is as the following table, bold items mean checkable/useable values while program running:

Built-in Variables	|Meaning 
:-----------------:	|:-----------------:
**$0**				| Entire record
**$n**				| The nth record filed
FS 					| Field separator, default: `space` or `tab` 
RS					| Record separator, default: `line break -- \n`
OFS					| Output field separator, default: `":"`
ORS					| Output record separator, default: `line break`
CONVFMT				| A string that controls the conversion of numbers to strings, the default value is %.6g
OFMT				| Early version of `CONVFMT`
SUBSEP				| Subscript separator, default: `"\034"`
**NF**				| Number of fields in the current reading record
**NR**				| Number of input records has processed
**ARGC**			| Number of arguments
**ARGV**			| Arguments
**FILENAME**		| Filename
**FNR**				| Current record number
**RSTART**			| Start-index of the substring that is matched by the `match` function
**RLENGTH**			| Length of the substring that is matched by the `match` function
**ENVIRON**			| Associative array of system environment variables

`FS` is illustrated above, and `OFS` `ARGV` and `FILENAME` will be explain in the following ACTIONS section, here we mark others as points which are talking about later.

### ACTIONS:

Actions set is required to be wrapped by curly braces, and consisted of series statements. Seeing into our example, I'm going to explain it with all gists mentioned.
	
	// Startup actions
	SET record separator `RS` to "\n"
	SET output field separator `OFS` to "-> "
	SET output record separator `ORS` to " !;\n"
	SET variable `mode` to "normal"
	SET variable `line_count` to 0
	PRINT process starting prompt with filename --`ARGV[1]`, `ARGV[2]` and number files `ARGC`
	// Pattern matching
	IF `mode` equal "normal":
		IF record match regular expression `/^_?[t-z]+/` and the numeric user id(`$3`) is even:
			PRINT information include total number of records have been scanned(`NR`), number of records have been scanned for current scanning file(`FNR`), specified fields(`$3`, `$1`, `$6`) of matched record and current scanning file's name(`FILENAME`)
	ELSE IF `mode` equal "debug":
		PRINT information about all records
	ELSE
		DO nothing
	// Cleanup actions
	PRINT process completion prompt with total number of records and fields
			
In the `printf statement`, (1) Program employ ARGV[1] which refers to the first input file's name -- "/etc/passwd", and ARGV[2] which refers to "/etc/shadow"( Meanwhile, ARGV[0] refers to program's name -- "awk", easy to know, ARGV[n] refers to the nth input file) cos `FILENAME` is unavailable in `BEGIN patten`; (2) Add "\n" manually at the end of `printf statement` for line break is necessary(`print` insert line break character at the end of statement automatically) **(Note: printf produce output strictly follow formatting structure & print produce output with line break)**. Actually `print` and `printf` have many difference with each other, we simply conclude `print` is a fancy, handy and no strictly formatting requirement replacement of `printf`.

Another thing should be noted: Variable `field_count` is called without any declaration, with program working fine, it is easy to speculate `field_count` is assigned with initial value 0 while its first calling **(Note: non-declaration value is auto initialized as 0 while first calling)**.

We have seen many statements in above example, as a supplement, here list all types of statements supported by _Awk_ --

1. Expressions, which can call functions or assign values to variables.
2. Control statements:
	* `if`
	* `for`
	* `while`
	* `do`
3. Compound statements
4. Input statements: 
	* `getline`
	* `next`
	* `nextfile`
5. Output statements:
	* `print`
	* `printf`
6. Deletion statements 

Oh, I see more expansion for _Awk_, before that, let me end this section --

Finally, what an ideal program output should look like? Compare your results with below.

```
Terminal: /bin/bash
Count from files /etc/passwd and /etc/shadow(total 2 files)
[Count Test Start]
11-> 11-> 	10	uucp		/var/spool/uucp	[/etc/passwd] !;
27-> 27-> 	108	tomcat7		/usr/share/tomcat7	[/etc/passwd] !;
55-> 27-> 	16626	tomcat7		7	[/etc/shadow] !;
[Count Test Completed]
The number of matched records: 3
Total fields: 23
```