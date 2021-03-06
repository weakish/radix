# Radix Integer

Radix provides an Integer class for working with integers in various bases.

    require 'radix'

## Initialization

Radix::Integer's initializer can accept either an Integer, String or
Array as a value and an integer base.

Give an integer value, it will automatically be converted to the base
specified.

    check do |integer, base, digits|
      r = Radix::Integer.new(integer, base)
      r.digits.assert == digits
    end

    ok  8,  2, [1,0,0,0]
    ok  4,  2, [1,0,0]
    ok  8, 10, [8]
    ok 10, 10, [1, 0]
    ok  8, 16, [8]
    ok 16, 16, [1, 0]

Where as a String value is taken to already be in the base given.

    ok "1000", 2, [1,0,0,0]
    ok  "100", 2, [1,0,0]

    ok  "8", 10, [8]
    ok "10", 10, [1, 0]
    ok  "8", 16, [8]
    ok "10", 16, [1, 0]

And an Array is also taken to be in the base given.

    ok %w[1 0 0 0], 2, [1,0,0,0]
    ok %w[  1 0 0], 2, [1,0,0]

    ok %w[  8], 10, [8]
    ok %w[1 0], 10, [1, 0]
    ok %w[  8], 16, [8]
    ok %w[1 0], 16, [1, 0]

Integers can also be negative, rather than positive. In each case
just prepend the value with a minus sign.

    check do |integer, base, digits|
      r = Radix::Integer.new(integer, base)
      r.digits.assert == digits
      r.assert.negative?
    end

    ok             -8, 2, ['-',1,0,0,0]
    ok        "-1000", 2, ['-',1,0,0,0]
    ok  %w[- 1 0 0 0], 2, ['-',1,0,0,0]

If a value has a digit outside of the range of the base an ArgumentError
will be raised.

    expect ArgumentError do
      Radix::Integer.new('9', 2)
    end

Radix provides a convenience extension method to Integer, String and Array
called #b, to more easily initialize a Radix numeric object. The method simply
passes the receiver on to `Radix::Integer#new`.

    check do |integer, base, digits|
      r = integer.b(base)
      r.assert.is_a?(Radix::Integer)
      r.digits.assert == digits
    end

    ok 8, 2, [1,0,0,0]
    ok 4, 2, [1,0,0]

    ok "1000", 2, [1,0,0,0]
    ok  "100", 2, [1,0,0]

    ok %w"1 0 0 0", 2, [1,0,0,0]
    ok   %w"1 0 0", 2, [1,0,0]

## Conversion

Radix integers can ve converted to other bases with the #convert method.

    b = "1000".b(2)
    d = b.convert(10)
    d.digits.assert == [8]

We can convert a Radix::Integer to a regular base-10 Integer with the #to_i
method.

    b = "1000".b(2)
    d = b.to_i
    d.assert == 8

## Equality

Radix extend the Integer, String and Array classes with the #b method
which simplifies the creation of Radix::Integer instances. The following
return the equivalent instance of Radix::Integer.

    a = 8.b(2)
    b = "1000".b(2)
    c = [1, 0, 0, 0].b(2)

    a.assert == b
    b.assert == c
    c.assert == a

    a.assert == 8
    b.assert == 8
    c.assert == 8

More stringent equality can be had from #eql?, in which the other integer
must be a Radix::Integer too.

    a.assert.eql?(b)
    a.refute.eql?(8)

## Operations

Radix::Integer supports all the usual mathematical operators.

### Addition

    check do |a, b, x|
      (a + b).assert == x
    end

    ok "1000".b(2), "0010".b(2), "1010".b(2)
    ok "1000".b(2),    "2".b(8), "1010".b(2)
    ok "1000".b(2),    "2".b(8),   "10".b(10)

A more complex example.

    x = "AZ42".b(62) + "54".b(10)
    x.assert == "2518124".b(10)
    x.assert == 2518124

Adding negative integers will, of course, be akin to subtraction.

    ok "1000".b(2), "-0010".b(2), "110".b(2)
    ok "1000".b(2),    "-2".b(8), "110".b(2)
    ok "1000".b(2),    "-2".b(8),   "6".b(10)

    ok "-1000".b(2), "0010".b(2), "-110".b(2)
    ok "-1000".b(2),    "2".b(8), "-110".b(2)
    ok "-1000".b(2),    "2".b(8),   "-6".b(10)

    ok "-1000".b(2), "-0010".b(2), "-1010".b(2)
    ok "-1000".b(2),    "-2".b(8), "-1010".b(2)
    ok "-1000".b(2),    "-2".b(8),   "-10".b(10)

### Subtraction

    check do |a, b, x|
      (a - b).assert == x
    end

    ok "1000".b(2), "10".b(2), "0110".b(2)
    ok "1000".b(2),  "2".b(8), "0110".b(2)
    ok "1000".b(2),  "2".b(8), "6".b(8)
    ok "1000".b(2),  "2".b(8), "6".b(10)

A more complex example.

    x = "AZ42".b(62) - "54".b(10)
    x.assert == "2518016".b(10)
    x.assert == 2518016

### Multiplication

    check do |a, b, x|
      (a * b).assert == x
    end

    ok "1000".b(2), "10".b(2), "10000".b(2)
    ok "1000".b(2),  "2".b(8), "10000".b(2)
    ok "1000".b(2),  "2".b(8), "20".b(8)
    ok "1000".b(2),  "2".b(8), "16".b(10)

A more complex example.

    x = "Z42".b(62) * "4".b(10)
    x.assert == "539160".b(10)
    x.assert == 539160

### Division

    check do |a, b, x|
      (a / b).assert == x
    end

    ok "1000".b(2), "10".b(2), "100".b(2)
    ok "1000".b(2),  "2".b(8), "100".b(2)
    ok "1000".b(2),  "2".b(8), "4".b(8)
    ok "1000".b(2),  "2".b(8), "4".b(10)

A more complex example.

    x = "AZ42".b(62) / "54".b(10)
    x.assert == "46630".b(10)
    x.assert == 46630

### Power

    check do |a, b, x|
      (a ** b).assert == x
    end

    ok "1000".b(2), "10".b(2), 64

### Modulo

    check do |a, b, x|
      (a % b).assert == x
    end

    ok "1000".b(2), "10".b(2), 0
    ok "1000".b(2), "11".b(2), 2

### Bitwise Shift

    check do |a, b, x|
      (a << b).assert == x
    end

    ok "10".b(2), "10".b(2), "1000".b(2)
    ok "10".b(2),         2, "1000".b(2)
    ok "10".b(2),         2, 8

### Bitwise AND

    check do |a, b, x|
      (a & b).assert == x
    end

    ok "1010".b(2), "10".b(2), "10".b(2)
    ok "1010".b(2),  "2".b(8), "10".b(2)

## Coerce

When a Radix::Integer is the operand in an operation against a regular
Ruby Integer, the calculation should still work via #coerce.

    check do |a, b, x|
      (a + b).assert == x
    end

    ok 10, "10".b(2), "12".b(10)

