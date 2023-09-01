# bc - Command Line Calculator

bc is actually a calculator progamming language. It has variables,
functions, loops, read and print statements, and of course mathematical
and boolean expressions. Most of what you can do with bc you can also do
with awk.

## Basic calculations

Traditional way, using echo:

    $ echo "5*4" | bc
    20

Bash 4 or greater, using "here string":

    $ bc <<< "5*4"
    20

You can use spaces in the expression

    $ bc <<< "5 * 4"

Always put the expression in quotes to protect it from the shell.

### Using "here document"

    $ bc << EOF
    5 * 4
    EOF
    20

In this case, do **not** put the expression in quotes,  
The shell does not expand the star \* within a *here document*, and it
does not remove quotes from a here document.  


### Using a file

    $ cat calcFile 
    5 * 4
    $ bc < calcFile 
    20

As with the here document, do **not** put the expression in quotes.

## Multiple expressions at once

echo:

    $ echo "5 * 4; 5+4; 5 - 4; 5/4" | bc
    20
    9
    1
    1

here string:

    $ bc <<< "5 * 4; 5+4; 5 - 4; 5/4"

In here documents and files, just put each expression on a separate
line. As before, do **not** qoute the expressions.

here document:

    $ bc << EOF
    5 * 4
    5+4
    5 - 4
    5/4
    EOF

file:

    $ cat calcFile 
    5 * 4  
    5+4
    5 - 4
    5/4

    $ bc < calcFile 
     

## Where's the decimal?

By default, bc truncates the decimal output in division, square root,
and some other operations. You can specify how many decimal places with
the "scale" variable:

    $ echo "scale=2; 5/4" | bc
    1.20

However, the scale argument merely *truncates* the result. It does
**not** do *rounding*.

If you want rounding, one way to do this is to let Bash do the rounding
for you via the printf command:

    printf "%.2f\n" $(echo "20 / 3" | bc -l)
    6.67

bc -l brings in the "math Library" and sets the scale to 20

## Store the result of bc operation in a shell variable

    $ x=$(echo "5-4" | bc)
    $ echo $x
    1

or:

    $ x=$(bc <<< "5-4")
    $ echo $x
    1

using backticks:

    $ x=`echo "5-4" | bc`
    $ echo $x
    1

## Converting binary, hex, and decimal

bc provides 2 variables:

- ibase represents the input base, it defaults to decimal. Set it to "2"
  for binary inputs and "16" for hexadecimal inputs.
- obase represents the output base. Set it to "A" for decimal, "2" for
  binary and "16" for Hexadecimal.

Convert decimal 128 to hexadecimal:

    $ echo 'obase=16; 128' | bc
    80

Convert hexadecimal 80 to decimal:

    $ echo 'ibase=16; obase=A; 80' | bc
    128

Convert decimal 128 to binary

    $ echo 'obase=2; 128' | bc
    10000000

Convert binary 10000000 to decimal

    $ echo 'ibase=2;obase=A;10000000' | bc
    128

## Sine, Cosine, Log, exponents, etc

For these additional functions, invoke bc with the -l option. This will
load a math library and set the scale to 20 (which may be more than you
want). There is no tangent function in bc. If you want the tangent of x,
compute s(x)/c(x)

## Decibel calculations

    # db=10*log10(power ratio)
    # db=20*log10(voltage ratio)
    # power ratio=10^(dB/10)
    # voltage ratio=10^(dB/20)

    # bc only has natural log (e) but we can do log10 by e(x)/e(10)
    # Use the fact that number^exponent == e^(exponent*log(number))

    # Power Ratio to dB
    read ratio
    echo 'scale=2; 10*(l('$ratio')/l(10))' | bc -l

    # Voltage Ratio to dB
    read ratio
    echo 'scale=2; 20*(l('$ratio')/l(10))' | bc -l

    # dB to Power Ratio
    read dB
    echo 'scale=4; e(('$dB'/10)*l(10))' | bc -l

    # dB to Volatage Ratio
    read dB
    echo 'scale=4; e(('$dB'/20)*l(10))' | bc -l


