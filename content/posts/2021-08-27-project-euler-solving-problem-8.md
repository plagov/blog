+++
title = "Project Euler: Solving problem 8"
date = "2021-08-27"
author = "Vitali Plagov"
tags = ["project-euler"]
+++

Project Euler is a website with hundreds of mathematical and algorithmic problems. Some time ago I started solving those
issues using Java. My goal is to practice general programing and algorithmic skills using the latest version of Java.
So, in this blog post, I will shortly describe the solution to problem 8.
<!--more-->

# Task

There's a series of 1000 random digits split into 20 lines.

```text
73167176531330624919225119674426574742355349194934
96983520312774506326239578318016984801869478851843
85861560789112949495459501737958331952853208805511
12540698747158523863050715693290963295227443043557
66896648950445244523161731856403098711121722383113
62229893423380308135336276614282806444486645238749
30358907296290491560440772390713810515859307960866
70172427121883998797908792274921901699720888093776
65727333001053367881220235421809751254540594752243
52584907711670556013604839586446706324415722155397
53697817977846174064955149290862569321978468622482
83972241375657056057490261407972968652414535100474
82166370484403199890008895243450658541227588666881
16427171479924442928230863465674813919123162824586
17866458359124566529476545682848912883142607690042
24219022671055626321111109370544217506941658960408
07198403850962455444362981230987879927244284909188
84580156166097919133875499200524063689912560717606
05886116467109405077541002256983155200055935729725
71636269561882670428252483600823257530420752963450
```

The task is to find a series of adjacent 13 digits that produce the highest product and return the value of this 
product.

# Solution

First, I put this sequence of digits into a txt file under `src/main/java/resources`. Next, to read it into a 
`String`, I wrote the following util function to read a file from resources:

```java
public static String readInputFile(String resourceFile) {
  try {
    Path path = Paths.get(
      Objects.requireNonNull(
        FileUtil.class.getClassLoader().getResource(resourceFile),
        String.format("Resource %s could not be found", resourceFile)
      ).toURI()
    );
    return Files.readString(path, StandardCharsets.UTF_8);
  } catch (URISyntaxException | IOException exception) {
    throw new IllegalArgumentException("Invalid location: " + resourceFile, exception);
  }
}
```

Next, I need to prepare the content of the file into a format that will be useful to work with. I 
need to have a list of integers. So this function will read the content of the file, join all the lines by replacing 
a new-line character with an empty string, so it will produce one string line with 1000 digits. Then, split this 
string into a list with each character as a separate element and finally transform it into a list of integers.

```java
private List<Integer> getListOfDigits() {
  String string = FileUtil.readInputFile("problem008.txt").replace("\n", "");
  return Arrays.stream(string.split("")).map(Integer::valueOf).collect(Collectors.toList());
}
```

Now, when I have a list of integers, I can start with problem-solving. I come up with the following algorithm. 
There will be two variables: `startIndex` and `endIndex`. The start will be at point zero and the end index will 
point to the index that is equal to the number of adjacent digits. For this task, this number is 13. This way, I will 
take the first 13 digits, will calculate the product of these digits and then will increase both start and end 
indices by one until the `endIndex` is equal to the size of the whole list. Overall, this method looks as follows:

```java
public long highestProductOfAdjacentDigits(int numberOfAdjacentDigits) {
  var allDigits = getListOfDigits();
  var startIndex = 0;
  var endIndex = numberOfAdjacentDigits;
  List<Long> products = new ArrayList<>();
  while (endIndex != allDigits.size()) {
    var adjacentDigits = allDigits.subList(startIndex, endIndex);
    if (!adjacentDigits.contains(0)) {
      var product = adjacentDigits.stream().mapToLong(a -> a).reduce(1, (a, b) -> a * b);
      products.add(product);
    }
    startIndex++;
    endIndex++;
  }
  return Collections.max(products);
}
```

To optimize the algorithm, I added a check. If the sublist of 13 digits contains zero, then it can skip the 
product calculation, since multiplying by zero will always produce zero as a result.
Every time a product is calculated, it is stored into an array list of products and in the end, the method returns the 
max product from the list using a built-in `Collections#max` function.

Another important thing to consider is to use a proper data type for the list of products. Multiplication of 13 
numbers may produce a massive number that won't fit into the bounds of `int` type. Thus, better to use a `long` type.

# Links

Visit my [Project Euler repository](https://github.com/plagov/project-euler) on GitHub to see the 
[full solution](https://github.com/plagov/project-euler/blob/master/src/main/java/net/projecteuler/problems/Problem008.java) 
of this problem. Also, check [tests](https://github.com/plagov/project-euler/blob/master/src/test/java/net/projecteuler/problems/Problem008Test.java) that assert the 
algorithm against both sample and task inputs.
