---
title: Flipping Bits
date: 2024-01-01T14:02:48-07:00
tags:
---
You will be given a list of 32 bit unsigned integers. Flip all the bits (1 + 0 and 0 —+ 1) and return the result as
an unsigned integer.

Example

n=%

910 = 10012. We're working with 32 bits, so:

000000000000000000000000000010012 = 919
11111111111111111111111111110110) = 429496728619

Return 4294967286
Function Description
Complete the flippingBits function in the editor below.

flippingBits has the following parameter(s):

* intn:an integer
Returns

* int: the unsigned decimal integer result
Input Format

The first line of the input contains g, the number of queries

Each of the next q lines contain an integer, n, to process.
Constraints

1<q <100
O<n<2”

3 
2147483647 
1 
0
Sample Output

2147483648 
4294967294 
4294967295
Explanation

Take 1 for example, as unsigned 32-bits is 00000000000000000000000000000001 and doing the flipping we get 11111111111111111111111111111110 which in turn is 4294967294.