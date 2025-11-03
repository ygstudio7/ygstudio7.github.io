# Converting logic to real or real to logic


**SystemVerilog** provides several ways to convert between different data types, including converting **logic** (which represents digital signals) to **real** (which represents floating-point numbers). Let’s explore some options:

1. **`$realtobits` and `$bitstoreal`**:
   - `$realtobits` converts a real number to a 64-bit representation, allowing it to be passed through a module port.
   - `$bitstoreal` converts a 64-bit bit-vector representation back to a real number.
2. **`$rtoi` and `$itor`**:
   - `$rtoi` truncates a real number to form an integer value.
   - `$itor` converts an integer to a real value.

Here’s an example of how you might use these functions:

```systemverilog
reg [63:0] A;
real B;
B = $bitstoreal(A); // Convert bit-vector A to real B

integer C = 14;
B = $itor(C); // Convert integer C to real B (result: B = 14.0)
```



Remember that these conversions are essential when interfacing between digital logic and analog signals or when performing mathematical operations involving real numbers in your SystemVerilog designs.





# Real Number Conversion Functions

In Verilog and SystemVerilog, several system functions can handle real number values (the `real` and `shortreal` types). These functions allow you to convert values to and from real number values and make signed or unsigned values conversions.

The following real number conversion functions are commonly used:

1. `$rtoi`: Converts a real value to an integer by truncating it.
2. `$itor`: Converts an integer value to a real value.
3. `$realtobits`: Converts a real value to a 64-bit vector representation.
4. `$bitstoreal`: Converts a 64-bit vector representation back to a real value.
5. `$shortrealtobits`: Converts a shortreal value to a 32-bit vector representation.
6. `$bitstoshortreal`: Converts a 32-bit vector representation back to a shortreal value.

 