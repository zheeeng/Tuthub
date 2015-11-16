## EXAMPLE 2

You may have been tired with writing the awkward _AWK_ command lines, steering eyes toward script writing seems to be a smart choice. Open the manual page of _AWK_, we see such a synopsis:

	awk [ -F fs ] [ -v var=value ] [ 'prog' | -f progfile ] [ file ...  ]

`-f` option provide an _AWK_ running script file access.

Pipe stream also could be input for _AWK_, and dash character `"-"` is used to represent the input stream:

	ls -lh | awk -f 'progfile' filename1 - filename2

In script program, anything following sharp sign character `"#"` and continues to the end of the line is considered as comment by _AWK_. Back slash sign character `"\"` is used to break a line into two lines. If you want to break a long script line into two with a new line character, only it after curly braces or at the end of a command can achieve your goal. Another usage of `"\"` is using it as escape notation in a string constant.
**(Note: "#" -- comment; "\" -- break line between statements, or escape a character in output statements.)**

To talk more about regular expression, arithmetic operation and control statements, here I bring in Example 2 -- a simple script as below.

Content of input file -- `input.txt`:

```
Formatting Test: #65#a#3.1415926#0.6180339#9#100#1.42857#8.57142#There are#365.25#days in a year.#happy!#be#worry#Don't
Conversion Test: @5@543666@17@017@018@0x17
Regex Test: ~[[:alpha:]]{3}[[:blank:]][[:alpha:]]{5}~[[:alnum:]]{3}[[:blank:]][[:digit:]]{5}~[[:digit:]]{3}[[:blank:]][[:digit:]]{5}
```

Content of test script -- `test.awk`:

```
#!/bin/awk -f
BEGIN { FS = ""; }
/Formatting Test:/ {
	fs = FS; FS = "#"; ofmt = OFMT; OFMT = "%.3g"; $0 = $0;
	printf "\n[Formatting Test]\n";	    
	printf "1. ASCII character %%c converting from: 65 -> %c, \"a\" -> %c;\n", $2, $3;
	printf "2. Decimal integer: 65 -> %d(%%d) or 96 -> %i(%%i)\n", $2, $2 + 32;
	printf "3. Unsigned integer: -65 -> %u(%%u)\n", -$2;
	printf "4. Floating-point number: 3.1415926 -> %07.3f(%%07.3f)\n", $4;
	printf "5. Scientific number:\n\t0.6180339 -> %10.3e(%%10.3e),\n\t0.6180339 -> %-10.3E(%%-10.3E)\n", $5, $5;
	printf "6. Shorter number form:\n\t900000 -> %g(%%g),\n\t9000000 -> %#G(%%#G)\n", $6 * 10 ^ 5, $6 * 10 ^ 6;
	printf "7. Octal integer: 100 -> %#o(%%#o)\n", $7;
	printf "8. Hexadecimal integer: 100 -> %x(%%x), 100 -> %#X(%%#X)\n", $7, $7;
	print  "9. OFMT: 1.42857 -> ", $8 ", 8.57142 -> ", $9;
	printf "10. String output with dynamic format specifier: %*s %*.*f %*s(%%*s %%*.*f %%*s)\n", 15, $10, 10, 0, $11, 10, $12;
	printf "11. Concatenation format specifier: " "%#s " "%6.2f " "%s" "(\"%%#s \" \"%%6.2f \" \"%%s\")\n", $10, $11, $12;
	    printf "12. Positional specifier:\n"
    printf "\tOriginal order: %s %s %s %s\n", $13, $14, $15, $16;
    printf "\tReversed order: %4$s %3$s %2$s %1$s\n", $13, $14, $15, $16;
	FS = fs; OFMT = ofmt;
}
/Conversion Test:/ {
    fs = FS; FS = "@"; $0 = $0;
    printf "\n[Conversion Test]\n";
    printf "Plus sign modifier: %+d %+d\n", $2, -$2;
    printf "Space modifier: % d % d\n", $2, -$2;
    printf "123e3hello + 543666 equal to 123e3 + 543666 = " ("123e3hello"+ $3) "\n";
    printf "hello123e3 + 543666 equal to 0 + 543666 = " ("hello123e3" + $3) "\n";
    printf "17 -> %d, 017 -> %d, 018-> %d, 0x17 -> %d\n", $4, $5, $6, $7;
    FS = fs;
} 
/String Test:/ {
    fs = FS; FS = "*"; $0 = $0;
    printf "\n[String Test]\n";
    printf "Original order: %s %s %s %s\n", $2, $3, $4, $5;
    printf "Reversed order: %4$s %3$s %2$s %1$s\n", $2, $3, $4, $5;
    FS = fs;
}
/Regex Test:/ {
	fs = FS; FS = "~"; $0 = $0;
	printf "\n[Regex Test]\n";
	print "Regex1:" $2;
	print "Regex2:" $3;
	print "Regex3:" $4;
	print "\t\tRegex1\tRegex2\tRegex3";
	print "One Piece\t" ("One Piece" ~ $2 ? "Yes ": "No" ) "\t" ("One Piece" ~ $3 ? "Yes ": "No" ) "\t" ("One Piece" ~ $4 ? "Yes ": "No" );
	print "0n3 p13c3\t" ("0n3 p13c3" ~ $2 ? "Yes ": "No" ) "\t" ("0n3 p13c3" ~ $3 ? "Yes ": "No" ) "\t" ("One piece" ~ $4 ? "Yes ": "No" );
	FS = fs;
}
```

For different patterns, example apples different processing flag --  by changing flags on-the-fly, by storing `FS`, `OFMT` temporarily, after the actions done, restore flags. `$0 = $0`is using to hack records re-evaluation.

### GAWK

**Important:** In this example, install the GNU Project's implementation of _AWK_ -- _GAWK_ is a requirement. For instance, though _AWK_ works with most common regular expression, but not the Interval Expression and Character Lists. For compatibility with POSIX, _GAWK_ is a solution for applying meta characters `{n}`, `{n,}` and `{n,m}` and Character Lists. In older version of _GAWK_, method is adding `-W posix / --posix` or `-W re-interval / --re-interval` to _GAWK_ options specifying. Difference from _AWK_ to _GAWK_ is talking in Example 3, more things need to be extending. 

> POSIX (/ˈpɒzɪks/ poz-iks), an acronym for Portable Operating System Interface, is a family of standards specified by the IEEE Computer Society for maintaining compatibility between operating systems. POSIX defines the application programming interface (API), along with command line shells and utility interfaces, for software compatibility with variants of Unix and other operating systems.    - Wikipedia

Other variations of _AWK_ include _NAWK_, _MAWK_, _TAWK_ and _JAWK_. Check your version of _AWK_ through the command line: `awk --version`, or `ls -lh $(which awk)`. A possible result is:

	> lrwxrwxrwx 1 root root 21 May 29 07:54 /usr/bin/awk -> /etc/alternatives/awk
	
Check the target file of symbolic link:

	$ ls -lh /etc/alternatives/awk
	> lrwxrwxrwx 1 root root 13 May 29 07:54 /etc/alternatives/awk -> /usr/bin/mawk
	
As above, the default _AWK_ is linked to _MWAK_. Use your software package manager to install _GAWK_ like following:

	$ sudo apt-get install gawk; ls -lh /etc/alternatives/awk
	> ...
	> lrwxrwxrwx 1 root root 13 Nov 10 16:54 /etc/alternatives/awk -> /usr/bin/gawk

Exclude symbolic _AWK_ to _GAWK_, make an alias is also recommended:

	$ alias awk="gawk"

In following part, _AWK_ refers to _GAWK_.


### AWKPATH

Same as environment variable `$PATH`	tell system where to search executable programs, `$AWKPATH` tell _GAWK_ where to search for executable _AWK_ scripts with or without suffix `.awk`. If environment variable `$AWKPATH` doesn't be defined, _GAWK_ built-in variable `AWKPATH` take effect to provide the _AWK_ script lib directory by default. For just extending this function, do concatenation _AWK_'s `ENVIRON["AWKPATH"]` with system variable `AWKPATH`.

	$ export AWKPATH=$(awk 'BEGIN{print ENVIRON["AWKPATH"]}'):$AWKPATH:/custom/awk-scripts/stored-directory

### STRING CONSTANT

Know about format control letters and format modifiers are necessary for understating output statements. There are two tables of references:

Format control letters	| Description
:--------------------:	| :---------------------------------:
$s						| String output
%c						| ASCII character output converting from number
%d, %i					| Decimal integer output
%u						| Unsigned decimal integer output
%f						| Decimal floating-point number
%e, %E					| Decimal scientific number output
%g, %G					| Decimal scientific or floating-point number output depending on which the length produces is shorter
%o						| Unsigned octal integer output
%x, %X					| Unsigned hexadecimal integer output

Format modifiers	| Description
:----------------:	| :------------:
`N$`				| Positional specifier
Leading `0`			| Fill output padding with number 0
`width`				| Minimum width specifier
`.prec`				| Precision of number, maximum width of string
-					| Left-justify modifier, if output padding is necessary.
+					| For numeric conversion, comparing with `space` modifier, prefix positive sign before positive value instead of `space`
`space`				| For numeric conversion, using it before `width` modifier, prefix positive value with space and negative value with minus sign
\#					| Add notation for indicating output format

To tell details about `Formatting Test`, `Conversion Test` and `String Test` part, embedding computed fields of `input.txt` into `test.awk` contributes to show a refined snippet: 

```
/Formatting Test:/ {
	fs = FS; FS = "#"; ofmt = OFMT; OFMT = "%.3g"; $0 = $0;
	printf "\n[Formatting Test]\n";	    
	printf "1. ASCII character %%c converting from: 65 -> %c, \"a\" -> %c;\n", 65, "a";
	printf "2. Decimal integer: 65 -> %d(%%d) or 96 -> %i(%%i)\n", 65, 97;
	printf "3. Unsigned integer: -65 -> %u(%%u)\n", -65;
	printf "4. Floating-point number: 3.1415926 -> %07.3f(%%07.3f)\n", 3.1415926;
	printf "5. Scientific number:\n\t0.6180339 -> %10.3e(%%10.3e),\n\t0.6180339 -> %-10.3E(%%-10.3E)\n", 0.6180339, 0.6180339;
	printf "6. Shorter number form:\n\t900000 -> %g(%%g),\n\t9000000 -> %#G(%%#G)\n", 900000, 90000000;
	printf "7. Octal integer: 100 -> %#o(%%#o)\n", 100;
	printf "8. Hexadecimal integer: 100 -> %x(%%x), 100 -> %#X(%%#X)\n", 100, 100;
	print  "9. OFMT: 1.42857 -> ", 1.42857 ", 8.57142 -> ", 8.57142;
	printf "10. String output with dynamic format specifier: %*s %*.*f %*s(%%\*s %%\*.\*f %%\*s)\n", 15, "There are", 10, 0, "365.25", 10, "days in a year.";
	printf "11. Concatenation format specifier: " "%#s " "%6.2f " "%s" "(\"%%#s \" \"%%6.2f \" \"%%s\")\n", "There are", "365.25", "days in a year.";
    printf "12. Positional specifier:\n"
    printf "\tOriginal order: %s %s %s %s\n", $2, $3, $4, $5;
    printf "\tReversed order: %4$s %3$s %2$s %1$s\n", $2, $3, $4, $5;
	FS = fs; OFMT = ofmt;
}
/Conversion Test:/ {
    fs = FS; FS = "@"; $0 = $0;
    printf "\n[Conversion Test]\n";
    printf "Plus sign modifier: %+d %+d\n", 5, -5;
    printf "Space modifier: % d % d\n", 5, -5;
    printf "123e3hello + 543666 equal to 123e3 + 543666 = " ("123e3hello"+ 543666) "\n";
    printf "hello123e3 + 543666 equal to 0 + 543666 = " ("hello123e3" + 543666) "\n";
    printf "17 -> %d, 017 -> %d, 018-> %d, 0x17 -> %d\n", 17, 017, 018, 0x17;
    FS = fs;
} 
```

_AWK_ provides methods to convert number to string or reversely, by concatenating a number with empty string `""` or adding plus sign to string. The procedures of converting input fields to string or numerical arguments which are preparing for `printf` are omitted.

#### Formatting Test

From `Formatting Test` some notable points are concluded and sorted by number mark of output statement in following:

1. The sequence `"%%"` outputs one `"%"`. It is used to escape `"%"`. `%c` convert single character string or a number(regard as ASCII code) to one character.**(Note: suggestion for escape "%" you'd better use "%%", not "\%", otherwise warning rise.)**
2. `%d` is exactly equivalent to  `%i`
3. `%u` treats an integer as an unsigned integer between 0 ~  INT_MAX(2^64 for 64bit system).
4. Leading `0` only has an effect when the field width is wider than the value to print.
5. `%e` prints a number with scientific notation, such as `6.180e-01`. `%E` use uppercase `"E"` instead of `"e"`. `"-"` make output be left-justify.
6. A shorter format on output will be picked automatically from `%f` and `%e` by control letter `%g`. `%G` use `"E"` instead of `"e"`.
7. `"#"` makes octal number start with leading `"0"`, hexadecimal number start with leading `"0x"` or '"0X"`, scientific number and floating-point number always contain a decimal point.
8. `%#x` applies lowercase notation `"0x"`, compare with `%#x` apply uppercase notation `"0X"`.**(Note: suggestion add modifier "#" before control letters for floating-point number, scientific number and octal and hexadecimal number.)** 
9. _AWK_ built-in variable `OFMT` specifies how `print` statement formats the string which is converted from number. The approximate value that precision satisfied is produced in the result.
10. `*` is employed to applying dynamic width or precision.
11. The concatenation of format arguments also works fine.
12. This sample reverses the words sequence through applying format modifiers `"N$"`.

A standard output is like:

```
[Formatting Test]
1. ASCII character %c converting from: 65 -> A, "a" -> a;
2. Decimal integer: 65 -> 65(%d) or 96 -> 97(%i)
3. Unsigned integer: -65 -> 18446744073709551551(%u)
4. Floating-point number: 3.1415926 -> 003.142(%07.3f)
5. Scientific number:
	0.6180339 ->  6.180e-01(%10.3e),
	0.6180339 -> 6.180E-01 (%-10.3E)
6. Shorter number form:
	900000 -> 900000(%g),
	9000000 -> 9.00000E+06(%#G)
7. Octal integer: 100 -> 0144(%#o)
8. Hexadecimal integer: 100 -> 64(%x), 100 -> 0X64(%#X)
9. OFMT: 1.42857 ->  1.42857, 8.57142 ->  8.57142
10. String output with dynamic format specifier:       There are        365 days in a year.(%*s %*.*f %*s)
11. Concatenation format specifier: There are 365.25 days in a year.("%#s " "%6.2f " "%s")
12. Positional specifier:
	Original order: happy! be worry Don't
	Reversed order: Don't worry be happy!
```

**Pay attention:**

* In command mode of awk, you have to escape single quote by `"'\''"`: `awk 'BEGIN{printf "%s\n", "don'\''t worry be happy";}'`. Another replacement is `"\047"`. Octal ASCII code is supported by printf default. Hexadecimal ASCII code is extensive. For information about ASCII, use command:

		man ascii

#### Conversion Test

In `Conversion Test`, `"+"` makes number convertible constant to a positive number with plus sign or a negative number with minus sign. Compare with `"+"`, `"space"` use space instead of plus sign before a positive number. These two modifiers not only work with integer control letters, but also other signed `"%s"`, `"%f"`, `"%e/%E"`, `"%g/%G"`. When they apply on unsigned control letters, negative number will be converted from two's complement value to an unsigned value.

When string concatenate with a number or string by `"+"`, the begin substring which is the maximum and number convertible will be converted to number automatically.

When control letter is specified to octal number or a hexadecimal number, if an input field is invalid, such as `018`, it will be treated as a decimal number.**(Note: don't use invalid octal and hexadecimal number, otherwise it is possible to get an unexpected result.)**

The ideal result:

```
[Conversion Test]
Plus sign modifier: +5 -5
Space modifier:  5 -5
123e3hello + 543666 equal to 123e3 + 543666 = 666666
hello123e3 + 543666 equal to 0 + 543666 = 543666
17 -> 17, 017 -> 17, 018-> 18, 0x17 -> 0
```

### REGULAR EXPRESSIONS

Regular expressions exist in rules of _AWK_ everywhere. Like many other program languages, it is used as constant, but there two operators -- `"~"` and `"!~"` are designed to get a match or mismatch result through dynamic regex, such as string, number, normal regex.

	$ awk 'BEGIN{print "Saturday is the 7th day of a week" ~ /Saturday/}'
	> 1
	$ awk 'BEGIN{print "Saturday is the 7th day of a week" ~ "week";}'
	> 1
	$ awk 'BEGIN{print "Saturday is the 7th day of a week" ~ 7;}'
	> 1
	$ awk 'BEGIN{DAY = "day"; print "Saturday is the 7th day of a week" ~ DAY;}'
	> 1

In _AWK_ patterns, a regular expression represents matching `$0` with itself. This is the base we use `"/String Test/"` instead of `"$0 ~ /String Test/"` to match patterns.

**Pay attention:**

* There is a dark corner: `/foo/ ~ $1` equal to `($0 ~ /foo/) ~ $1`.
* Escape meta-characters of regex in string is essential: `"one+two" ~"one\\+two"`.

Character Lists Reference:

Character List	| Description
:-------------:	| :------------:
[:alnum:]		| Alphanumeric characters
[:alpha:]		| Alphabetic characters
[:blank:]		| Space and tab characters
[:cntrl:]		| Control characters
[:digit:]		| Digits
[:graph:]		| Graphical characters: `[:alnum:]` and `[:punct:]`
[:lower:]		| Lower-case letters
[:print:]		| Printable characters: `[:alnum:]`, `[:punct:]`, and space
[:punct:]		| Punctuation characters
[:space:]		| Space characters: in the ‘C’ locale, this is tab, newline, vertical tab, form feed, carriage return, and space
[:upper:]		| Upper-case letters
[:xdigit:]		| Hexadecimal digits

**(Note: Interval Expression and Character Lists are associated with system locale which refers to environment variable $LC_ALL and its association)**

With mentioned above, it is not hard to infer the result:

```
[Regex Test]
Regex1:[[:alpha:]]{3}[[:blank:]][[:alpha:]]{5}
Regex2:[[:alnum:]]{3}[[:blank:]][[:digit:]]{5}
Regex3:[[:digit:]]{3}[[:blank:]][[:digit:]]{5}
		Regex1	Regex2	Regex3
One Piece	Yes 	No	No
0n3 p13c3	No	No	No
```

Ok, if you mastering this part, read Example 3 part and it is going to finish the tutorial about _AWK_.