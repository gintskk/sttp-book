---
chapter-number: 5
title: Boundary Testing
layout: chapter
toc: true
---

## Introduction

A high number of bugs happen in the **boundaries** of your program.
In this chapter we are going to derive tests for these boundaries.

The boundaries can reside between partitions or specific conditions.
Off-by-one errors, i.e., when your program outcome is "off by one", 
often occur because of the lack of boundary testing. 

## Boundaries in between classes/partitions

Whenever we devised partitions/classes, these classes have "close boundaries"
with the other classes.

We can find such boundaries by finding a pair of consecutive input values $$[p_1,p_2]$$, where $$p_1$$ belongs to partition A, and $$p_2$$ belongs to partition B.
In other words,
the boundary itself is where our program changes from our class to the other. 
As testers, we should make sure that everything works smoothly (i.e., the program
still behaves correctly) near these values.

{% include example-begin.html %}
**Requirement: Calculating the amount of points of the player**

Our program has two inputs: the score of the player and the remaining life of the player.
If the player's score is below 50, then it always adds 50 points.
If the remaining life is above or equal to 3 lives and the score is greater than or equals to 50, the player's score is doubled.

When devising the partitions to test this method, we come up with the following partitions:

1. Score < 50
2. Score >= 50 and remaining life < 3
3. Score >= 50 and remaining life >= 3

When the score is strictly smaller than 50 it is part of the first partition.
From a score of 50 the test case will be part of partitions 2 or 3.
Now we have found a boundary between partitions 1 and 2, 3.
This boundary is between the score of 49 and 50.

We also find a boundary between partitions 2 and 3.
This boundary is between the remaining life.
When the remaining life is smaller than 3, it belongs to partition 2; otherwise it belongs to partition 3.

We can visualize these partitions with their boundaries in a diagram.
![Partitions with their boundaries](/assets/img/chapter3/examples/partition_boundaries.svg)

{% include example-end.html %}

To sum up: you should devise tests for the inputs at the 
boundaries of your classes.

{% assign todo = "Bring the chocolate example, from the video here, and discuss its partitions." %}
{% include todo.html %}

## Analysing conditions

We briefly discussed the off-by-one errors earlier.
Errors where the system behaves incorrectly for values on and close to the boundary are very easily made.
In practice, think of how many times the bug was in the boolean condition of your `if` or `for` condition, and the fix was basically
replacing a `>=` by a `>`.

When we have a properly specific condition, e.g., `x > 100`, we can analyse the boundaries of this condition.
First, we need to go over some terminology:

- On-point: the value that is on the boundary. This is the value we see in the condition.
- Off-point: the value closest to the boundary that flips the conditions. So if the on-point makes the condition true, the off point makes it false and vice versa. Note: when dealing with equalities or non equalities (e.g. $$x == 6$$ or $$x != 6$$), there are two off-points; one in each direction.
- In-points are all the values that make the condition true.
- Out-points are all the values that make the condition false.

Note that, depending on the condition, an on-point can be either an in- or an out-point.

{% include example-begin.html %}
Suppose we have a program that adds shipping costs when the total price is below 100.

The condition used in the program is $$x < 100$$.

* The on-point is $$100$$, as that is the value in the condition.
* The on-point makes the condition false, so the off-point should be the closest number that makes the condition true.
This will be $$99$$,  $$99 < 100$$ is true.
* The in-points are all the values smaller than or equal to $$99$$.
* The out-points are all values larger than or equal to $$100$$.

We show all these points in the diagram below.

![On- and off-points, in- and out-points](/assets/img/chapter3/examples/on_off_points.svg)
{% include example-end.html %}

As a tester, you devise test cases for these different points.

Now that we know the meaning of these different points we can start the boundary analysis in more complicated cases.
In the previous example, we looked at one condition and its boundary.
However, in most programs you will find statements that consist of multiple conditions.

To test the boundaries in these more complicated decisions, we use the **simplified domain testing strategy**.
The idea of this strategy is to test each boundary separately, i.e. independent of the other conditions.
To do so, for each boundary:

* We pick the on- and off-point and we create one test case each.
* If we use multiple variables, we need values for those as well.
As we want to test each boundary independently, we choose in-points for the other variables. Note: We always choose in points, regardless
of the two boolean expressions being connected by ANDs or ORs. In practice, we want all the other conditions to return true, so that
we can evaluate the outcome of the condition under test independently.
* It is important to vary these in-points and to not choose the on- or off-point.
This gives us the ability to partially check that the program gives the correct results for some in-points.
If we would set the in-point to the on- or off-point, we would be testing two boundaries at once.

To find these values and display the test cases in a structured manner, we use a **domain matrix**.
In general the table looks like the following:

![Template for domain matrix](/assets/img/chapter3/boundary_template.png)

In this template, we have two conditions with two parameters (see the $$x > a \land y > b$$ condition).
We list the variables, with all their conditions.
Then each condition has two rows: one for the on-point and one for the off-point.
Each variable has an additional row for the typical (or in-) values.
These are used when testing the other boundary.

Each column that corresponds to a test case has two colored cells.
In the colored cells you have to fill in the correct values.
Each of these pairs of values will then give a test case.
If we implement all the test cases that the domain matrix gives us, we exerise each boundary both for the on- and off-point independent of the other parameters.

{% include example-begin.html %}
We have the following condition that we want to test: `x >= 5 && x < 20 && y <= 89`

We start by making the domain matrix, having space for each of the conditions and both parameters.

![Empty boundary table example](/assets/img/chapter3/examples/boundary_table_empty.png)

Here you see that we need 6 test cases, because there are 3 conditions.
Now it is time to fill in the table.
We get the on- and off-points like in the previous example.

![Boundary tables example filled in](/assets/img/chapter3/examples/boundary_table.png)

Now we have derived the six test cases that we can use to test the boundaries.
{% include example-end.html %}

## Automating boundary testing with JUnit (via Parameterized Tests)

We just analysed the boundaries and derived test cases using a domain matrix.
It is time to automated these tests using JUnit.

You might have noticed that in the domain matrix we always have a certain amount of input values and, implicitly, an expected output value.
We could just implement the boundary tests by making a separate method for each test.
However, the amount of tests can quickly become large and then this approach is not maintainable.
Also, the code in these test methods will be largely the same.
After all, we do the same thing with just different input and output values.

Luckily, JUnit offers a solution where we can implement the tests with just one method: Parameterized Tests.
As the naming suggests, with a parameterized test we can make a test method with parameters.
So we can create a test method and make it act based on the arguments we give the method.
To define a parameterized test you can use the `@ParameterizedTest` annotation instead of the usual `@Test` annotation.

Now that we have a test method that has some parameters, we have to give it the values that it should execute the tests with.
In general these values are provided by a `Source`.
Here we focus and a certain kind of `Source`, namely the `CsvSource`.
With the `CsvSource`, each test case is given as a comma separated list of input values.
We give this list in a string.
To execute multiple test with the same test method, the `CsvSource` expects list of strings, where each string represents the values for one test case.
The `CsvSource` is an annotation itself, so in an implementation it would like like the following: `@CsvSource({"value11, value12", "value21, value22", "value31, value32", ...})`

{% include example-begin.html %}
We are going to implement the test cases that we found in the previous example using the parameterized test.
Suppose we are testing a method that returns the result of the decision we analyzed in the example.
To automate the tests we create a test method with three parameters: `x`, `y`, `expectedResult`.
`x` and `y` are integers.
The `expectedResult` is a boolean, as the result of the method is also a boolean.

```java
@ParameterizedTest
@CsvSource({
  "5, 24, true",
  "4, 13, false",
  "20, -75, false",
  "19, 48, true",
  "15, 89, true",
  "8, 90, false"
})
public void exampleTest(int x, int y, boolean expectedResult) {
  Example example = new Example();

  assertEquals(expectedResult, example.run(x, y))
}
```

The assertion in the test checks if the result of the method, with the `x` and `y` values, is the expected result.

From the values you can see that each of the six test cases corresponds to one of the test cases in the domain matrix.
{% include example-end.html %}

<iframe width="560" height="315" src="https://www.youtube.com/embed/rPcMJg62wM4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## The CORRECT way

{% assign todo = "Write the accompanying text." %}
{% include todo.html %}


<iframe width="560" height="315" src="https://www.youtube.com/embed/oxNEUYqEvzM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## References

* Jeng, B., & Weyuker, E. J. (1994). A simplified domain-testing strategy. ACM Transactions on Software Engineering and Methodology (TOSEM), 3(3), 254-270.

* Chapter 7 of Pragmatic Unit Testing in Java 8 with Junit. Langr, Hunt, and Thomas. Pragmatic Programmers, 2015.

## Exercises

Below you will find some exercises to practise with the material discussed in this chapter.
After each exercise you can view the answer by clicking the button.

{% include exercise-begin.html %}
We have the following method.

```java
public String sameEnds(String string) {
  int length = string.length();
  int half = length / 2;

  String left = "";
  String right = "";

  int size = 0;
  for(int i = 0; i < half; i++) {
    left += string.charAt(i);
    right = string.charAt(length - 1 - i) + right;

    if (left.equals(right))
      size = left.length();
  }

  return string.substring(0, size);
}
```

Perform boundary analysis on the condition in the for-loop: `i < half`, i.e. what are the on- and off-point and the in- and out-points?
You can give the points in terms of the variables used in the method.
{% include answer-begin.html %}
The on-point is the value in the conditions: `half`.

When `i` equals `half` the condition is false.
Then the off-point makes the condition true and is as close to `half` as possible.
This makes the off-point `half` - 1.

The in-points are all the points that are smaller than half.
Practically they will be from 0, as that is what `i` starts with.

The out-points are the values that make the condition false: all values equal to or larger than `half`.
{% include exercise-answer-end.html %}

{% include exercise-begin.html %}
Perform boundary analysis on the following decision: `n % 3 == 0 && n % 5 == 0`.
What are the on- and off-points?
{% include answer-begin.html %}
The decision consists of two conditions, so we can analyse these separately.

For `n % 3 == 0` we have an on point of 3.
Here we are dealing with an equality; the value can both go up and down to make the condition false.
As such, we have two off-points: 2 and 4.

Similarly to the first condition for `n % 5 == 0` we have an on-point of 5.
Now the off-points are 4 and 6.
{% include exercise-answer-end.html %}

{% include exercise-begin.html %}
A game has the following condition: `numberOfPoints <= 570`.
Perform boundary analysis on the condition.
What are the on- and off-point of the condition?
Also give an example for both an in-point and an out-point.
{% include answer-begin.html %}
The on-point can be read from the condition: 570.

The off-point should make the condition false (the on-point makes it true): 571.

An example of an in-point is 483.
Then the condition evaluates to true.

An example of an out-point, where the condition evaluates to false, is 893.
{% include exercise-answer-end.html %}

{% include exercise-begin.html %}
We extend the game with a more complicated condition: `(numberOfPoints <= 570 && numberOfLives > 10) || energyLevel == 5`.

Perform boundary analysis on this condition.
What is the resulting domain matrix?
{% include answer-begin.html %}
<!-- !!Not sure if this domain matrix is correct!!  -->
![Answer domain matrix](/assets/img/chapter3/exercises/domain_exercise.png)

Note that we require 7 test cases in total: `numberOfPoints <= 570` and `numberOfLives > 10` each have one on- and one off-point.
`energyLevel == 5` is an equality, so we have two off-points and one on-point.
This gives a total of 7 test cases.

For one of the first two conditions we need two typical rows. \\
Let's rewrite the whole condition to: `(c1 && c2) || c3`.

To test `c1` we have to make `c2` true, otherwise the result will always be false. \\
The same goes for testing `c2` and then making `c1` true.

However, when testing `c3`, we need to make `(c1 && c2)` false, otherwise the result will always be true.
That is why, when testing `c3`, `c1` or `c2` has to be false, i.e. and out-point instead of an in-point.
Therefore we use two different typical rows for the `numberOfLives` variable.
The same could have been done with two typical rows for the `numberOfPoints` variable.
{% include exercise-answer-end.html %}