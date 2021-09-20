+++
title = "Project Euler: Solving problem 9 - Pythagorean triplet"
date = "2021-09-20"
author = "Vitali Plagov"
tags = ["project-euler"]
draft = false
+++

The [problem 9](https://projecteuler.net/problem=9) of the Project Euler is about a 
[Pythagorean triplet](https://en.wikipedia.org/wiki/Pythagorean_triple). It's a good problem as it gives a chance to 
practice implementing mathematical formulae with the Java code.
<!--more-->

# Task

So, the problem states that there's only one Pythagorean triple for which a sum of all its elements gives 1000.
We need to find the product of this triplet.

# Solution

A Pythagorean triplet is a triplet of natural numbers that satisfies the following condition:
<p align="center">x<sup>2</sup> + y<sup>2</sup> = z<sup>2</sup></p>

In our case, we know neither `x`, nor the `y`, nor the `z`. The only condition that is known is that the sum of 
all three members is equal to 1000. The brute-force approach to find all three numbers is to start from `1`, 
check if numbers satisfy the condition and if not, increment and repeat. This approach is not optimal at all. It 
will be doing too many unnecessary operations.

The Pythagorean triple has many features and characteristics. One among them helps to find all three elements. If we 
take two mutually prime numbers that are `u > v > 0`, then finding `x`, `y` and `z` is pretty simple:
<p align="center">x = u<sup>2</sup> - v<sup>2</sup></p>
<p align="center">y = 2 * u * v</p>
<p align="center">z = u<sup>2</sup> + v<sup>2</sup></p>

With help of these formulas, it is now easier to find the triplet. We just need to find the correct pair of `u` and `v`.

In my implementation, I decided to define the initial values of both `u` and `v` equal to `1` and `2` respectively.
Then, calculate `x`, `y`, `z` and their sum. Next, add a condition - if the sum is higher than the sum of the 
triplet from the task (1000), then increase both `u` and `v` by one and repeat. If the sum is lower than 1000, then 
increase just the `u` and repeat.

This whole calculation is repeated while the sum won't be equal to 1000. So, the whole method will look as follows:
`x`, `y` and `z` are defined as instance variables.

```java
public long productOfPythagoreanTripletWhichSumIsEqualTo(int sumOfPythagoreanTriplet) {
    long v = 1;
    long u = 2;

    while (sum != sumOfPythagoreanTriplet) {
        x = (long) (Math.pow(u, 2) - Math.pow(v, 2));
        y = 2 * u * v;
        z = (long) (Math.pow(u, 2) + Math.pow(v, 2));
        sum = x + y + z;
        if (sum > sumOfPythagoreanTriplet) {
            v++;
            u = v + 1;
        }
        u++;
    }

    return x * y * z;
}
```

When running a simple unit test, the execution time varies between 40 - 70 ms on an average Intel laptop. That looks 
pretty fast and efficient.

# Links

Visit my [Project Euler repository](https://github.com/plagov/project-euler) on GitHub to see the
[full solution](https://github.com/plagov/project-euler/blob/master/src/main/java/net/projecteuler/problems/Problem009.java)
of this problem. Also, check the [test](https://github.com/plagov/project-euler/blob/master/src/test/java/net/projecteuler/problems/Problem009Test.java) that finds the 
product of the Pythagorean triplet as requested by the task.
