# Understanding fixed point Arithmetic in solidity with Solady 

#### Overview
Numbers, something as simple as 1, 2, or 3, can become quite complex as their use cases grow. In the world of programming, integers, whole numbers, fractions, decimals, and even `floats` and `doubles` are used to represent these numbers. However, in Solidity and the Ethereum Virtual Machine (EVM), there is a fundamental difference—there's essentially one set of numbers: integers. These integers come in two flavors: signed and unsigned, with different bit lengths such as `uint8`,`int64`, and `uint256`. Despite the variations, they all boil down to integers.

But what about those numbers like 1.5 liters of soda or a $9.99 for a pair of shoes on sale? Are they the same, and how are they represented in EVM? How do we work with them, especially when Solidity(& EVM) doesn't natively support them? 
Let's explore :

#### Prerequisites
To fully grasp the concepts discussed in this article, it's essential to have some basic knowledge in the following areas:
1. Solidity basics and syntax.
2. Yul (a brief understanding).
  <!--@todo Brief info in a sentence or two  -->

## Floating-Points to Fixed-Points
The decimal numbers we encounter in our daily lives are most prominiently repsesented as floating-point numbers. They are called "`floating`" because the position of the decimal point can float around the number, indicating the value and its representation. In the world of computers, they are represented in a specific way.
Consider the number "π" (Pi)  `3.1415`, where the position of the decimal point indicates that 3 is the integer part, and 1415 are the fractional parts. The rest of decimals are truncated.

To represent such numbers in binary, we follow these steps:
- Divide the integer part of the number by the base of the new number system:
<br>
  ![Integer Part](https://github.com/Poudel0/Security-Research/blob/main/image.png)
<br>

- Multiply the fractional part of the number by the base of the new number system:
<br>
  ![Fractional Part](https://github.com/Poudel0/Security-Research/blob/main/image-1.png)
<br>
Now, combine both parts, as the result, the conversion becomes:

3.1415<sub>10</sub>= 11.0010010000<sub>2</sub> 

There is a standard method of representing such numbers in the form of  `sign`,`mantissa`,`base` and `exponents` as:
`(s) M*b^E`. Which is out of scope.

However, due to the lack of native support for floating-point numbers in the EVM, different techniques are applied by third-party libraries to mitigate this problem, utilizing Fixed-Point Numbers.
                                                           

#### FixedPoints

Let's consider an `uint8` for the sake of simplicity, but larger sizes use the same concept with larger value representation. An `uint8` is represented as:


| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |  
|---|---|---|---|---|---|---|---|
|---|---|---|to|---|---|---|---|
| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |  

i.e 0 to 2<sup>8</sup> -1 = 255

This is how an integer is represented. To represent fractional numbers, we assign a precision value. We divide the bits  into 2 categories. One for `integer` and other for `fractional` part.  
Consider a precision of `4` bits. i.e 4 bits after decimal point.
                                                           

| 0 | 0 | 0 | 0 | . | 0 | 0 | 0 | 0 |                                   
|---|---|---|---|---|---|---|---|---|
|2<sup>3</sup>|2<sup>2</sup>|2<sup>1</sup>|2<sup>0</sup>|    |2<sup>-1</sup>|2<sup>-2</sup>|2<sup>-3</sup>|2<sup>-4</sup>|


It can represent from 0 as shown above to 

| 1 | 1 | 1 | 1 | . | 1 | 1 | 1 | 1 |  
|---|---|---|---|---|---|---|---|---|   

equivalent to `15.9375`.

Here, the range of numbers it can now represent is equivalent to 2<sup>4</sup> bits representation for either part. As the precision basically divides the `uint8` to `uint4` for integer and `uint4` for fractional part. This is one __limitation__ of using fixed point numbers. Even if the bits are not utilized, they are __initialized__ thus rendering them wasted while not in use.

However, we can define the precision required per application.
This concept of representation is used to overcome the inherent lack of floating points in the EVM. We assign the presicion factor for certain values that needs to be represented in fractional terms. Example: `1e18` is the precision taken for native currency of Ethereum. Here the smallest vaalue is designated as 1 Wei. 
i.e `1e18 Wei = 1 eth`



Fixed-point arithmetic is useful when dealing with fractional numbers in Solidity. It is used to represent decimal numbers with a fixed number of digits after the decimal point. This is particularly useful in financial applications(DeFi), where precision is important.

For example, if you are building a decentralized application that involves currency exchange, you might want to use fixed-point arithmetic to ensure that the exchange rates are precise upto certain decimals.
Another use case for fixed-point arithmetic is when you need to perform calculations that involve fractional numbers, such as interest rates, percentages, or ratios.



####  Library Overview: Solady
We will be looking at the following library :[FIxedPointMathLib](https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L46-L107Make)
The functions in question are the following four: 
- Multiply
- Multiply rounding Up
- Divide
- Divide rounding Up



We can vaugely divide the portion of the library into the following parts:
1. __Precision__ / _Scale Factor_
```solidity
    uint256 internal constant WAD = 1e18;
```
Here, constant value named WAD is declared with the decimal value of `1e18`. This constant represents the precision or scaling factor for the fixed-point arithmetic. In this context, WAD is equivalent to 1e18 (10^18). Meaning: 18 out of 256 bits are to be used for decimal points.

The number of bits that will represent the fractional portion.

2. __Function definition__
```solidity
    function mulWad(uint256 x, uint256 y) internal pure returns (uint256 z) 
    //
    function mulWadUp(uint256 x, uint256 y) internal pure returns (uint256 z) 
    //
    function divWad(uint256 x, uint256 y) internal pure returns (uint256 z) 
    //
    function divWadUp(uint256 x, uint256 y) internal pure returns (uint256 z) 

```
These are the four functions we will be tackling.

3. __Checks (Overflow)__

There are 2 types of checks in these functions. 

One for overflow, as used in `mulWad()` and `mulWadUP()` functions.
```solidity
 if iszero(mul(y, iszero(mul(WAD, gt(x, div(not(0), WAD)))))) {
                mstore(0x00, 0x7c5f487d) // `DivWadFailed()`.
                revert(0x1c, 0x04)
            }
```
`div(not(0), y)`  is equivalent to `type(uint256).max / y`. It is achieved with complementing all the 0s in `uint256`(convert 0 to 1 and vice versa).  This part calculates the maximum value that a uint256 can hold (which is type(uint256).max) divided by y. `gt(x, ...)` compares the value of x with the result from the previous step.
The entire expression inside if is multiplied by y. If the result of the multiplication is `>= y`, it means that the calculation would result in an overflow, and the require statement would be triggered. This is where the check for overflow occurs.


Another for Overflow and non-Zero check for `y`. The additional non-Zero check is required to bound values to be representable. i.e non-`infinity` or non-`indeterminate`.

```solidity 
if iszero(mul(y, iszero(mul(WAD, gt(x, div(not(0), WAD))))))
```

This line performs a check to prevent division by zero and overflow during the multiplication of WAD and x. It ensures that both y is not zero and that the multiplication does not exceed the maximum value that a `uint256` can hold. If the check fails (i.e., division by zero or overflow would occur), the function reverts, indicating a failed division .


4.  __Function Logic__
```solidity
  function mulWad(uint256 x, uint256 y) internal pure returns (uint256 z) {
        assembly {
            if mul(y, gt(x, div(not(0), y))) {
                mstore(0x00, 0xbac65e5b) 
                revert(0x1c, 0x04)
            }
            z := div(mul(x, y), WAD)       // Like this line
        }
    }
```



#### Functions Workings

The functions in question are:

1. 
```solidity
  function mulWad(uint256 x, uint256 y) internal pure returns (uint256 z) {
    <!--  -->
            z := div(mul(x, y), WAD)
        
    }
```

 

The function is meant to perform a fixed-point multiplication `x*y` and then divide the result by `WAD`, effectively scaling down the result. The "rounded down" means that the division result is truncated, and any fractional part is discarded.

##### It is to be noted that multiplication is done before division to preserve as much bits as possible and mitigate the precision loss.

<br>
<br>

2. 
```solidity
function mulWadUp(uint256 x, uint256 y) internal pure returns (uint256 z) {
       <!--  -->
            z := add(iszero(iszero(mod(mul(x, y), WAD))), div(mul(x, y), WAD))
        <!--  -->
    }
```

It computes the result of `x * y`, then takes the modulo (mod) of that result with `WAD`. The modulo operation calculates the remainder when dividing by `WAD`, effectively capturing the fractional part of the result.

`div(mul(x, y), WAD)` calculates the integer part of the result, effectively implementing fixed-point arithmetic.

`iszero(iszero(...))` ensures that any remainder (fractional part) is not equal to zero. If there's a fractional part, the result is adjusted upward to the nearest integer using the add function.


3. 
```solidity
function divWad(uint256 x, uint256 y) internal pure returns (uint256 z) {
       <!--  -->
            z := div(mul(x, WAD), y)
        <!--  -->
    }

```

It is meant to  computes the result of `x * WAD` and then divide it by `y`.`x` and `WAD` are multiplied first as to preserve the maximum bits that might be truncated after division. The division operation rounds the result down to the nearest integer.


4. 
```solidity
 function divWadUp(uint256 x, uint256 y) internal pure returns (uint256 z) {
   <!--  -->
            z := add(iszero(iszero(mod(mul(x, WAD), y))), div(mul(x, WAD), y))
        <!--  -->
    }
```

Here, it computes the result of `x * WAD`, then takes the modulo (mod) of that result with `y`. The modulo operation calculates the remainder when dividing by y,  capturing the fractional part of the result.

`iszero(iszero(...))` ensures that any remainder (fractional part) is not equal to zero. If there's a fractional part, the result is adjusted upward to the nearest integer using the add function.

`div(mul(x, WAD), y)` calculates the integer part of the result, implementing fixed-point arithmetic.







<!-- @todo -->

