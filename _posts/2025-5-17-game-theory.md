---
layout: post
title: A Brief Introduction of Game Theory
date: 2024-06-17 19:42:56+0800
last_updated: 2025-05-17 16:19:12+0800
description:
    This post introduce some simple examples of game theory.
tags:
  - Mathematics
  - Game Theory
categories: Potpourri
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---
## Nim Game

### The Simplest One

#### Description

You are playing the following Nim Game with your friend:

There is a heap of stones on the table, each time one of you take turns to remove 1 to 3 stones.
The one who removes the last stone will be the winner.
You will take the first turn to remove the stones.
Both of you are very clever and have optimal strategies for the game.
How to know if you can win the game?

For example, if there are 4 stones in the heap and now it's your turn,
you will never win the game:
no matter 1, 2, or 3 stones you remove,
the last stone will always be removed by your friend.
Another example: if there are 5 stones in the heap and now it's your turn, you will win the game.
You can remove 1 stone, then no matter how many stones your friend remove,
you can remove the last stone and win the game.

#### Solution

Let's start with the simplest one.
If there are 1, 2, or 3 stones, you can win the game.
And from the examples, we know that when there is 4 stones, you will lose.
But when there are 5 stones, you will win.

What's the next? Actually:

* `Theorem 1`: when the number of the stones is multiple of 4, you will lose the game.
Otherwise, you will win the game.

But why? In order to prove this, we must introduce some important definitions:

* `Winning State`: the state you will win the game.
* `Losing State`: the state you will lose the game.

We can find that: if a state is a winning state,
we can make a move to change the state to a losing state (make your opposite lose).
And if a state is a losing state, no mater which move we choose,
the state will be changed to a winning state (make your opposite win).

Now, we can prove the `theorem 1` by proving the following two claims:

* `Claim 1`: If the number of the stones is multiple of 4,
any move will make the number of the rest stones not multiple of 4
(from `losing state` to `winning state`).
* `Claim 2`: If the number of the stones is not multiple of 4,
we can make a move to make the number of the rest stones multiple of 4
(from `winning state` to `losing state`).

The proof of the `claim 1` is obvious.
Because you can only remove 1, 2, or 3 stones at a time,
if the number of the stones is multiple of 4,
no matter how many stones you remove,
the number of the rest stones will not be multiple of 4.

For `claim 2`, suppose the number of the stones is $$ n $$, and $$ n $$ is not multiple of 4.
We just need remove $$ n \mod 4 $$ stones,
then the number of the rest stones will be $$ n - n \mod 4 $$, which is multiple of 4.

Then we know that 0 is the simplest losing state.
Then we have proven the `theorem 1` by induction
(compare the claims and the definitions of the winning state and the losing state).

### Multiple Piles

#### Description

There are several piles of stones.
You and your friend will take turns to remove stones from the piles.
On each turn, a player can remove any number of stones from a single pile.
The one who removes the last stone will be the winner.
You will take the first turn to remove the stones.

For example, if there are two piles of stones,
the first one with 2 stones and the second one with 3 stones,
you will win the game.
You can remove 1 stone from the second pile,
then no matter how many stones your friend remove,
you just to make a move to make the two piles have the same number of stones,
then you will win the game.

#### Solution

We can solve this problem by using the `Nim Sum`.
The `Nim Sum` of a set of numbers is the bitwise `XOR` of the numbers.

`XOR`: if two bits of binary representations are different, the result is 1,
otherwise, the result is 0.

For the problem, the solution is below:

* `Theorem 2`: the `Nim Sum` of a set of numbers is 0 if and only if
the set of numbers is in the losing state.

We can prove the `theorem 2` similarly as the `theorem 1`. We can prove the following two claims:

* `Claim 3`: If the `Nim Sum` of a set of numbers is 0,
any move will make the `Nim Sum` of the rest numbers not 0.
* `Claim 4`: If the `Nim Sum` of a set of numbers is not 0,
we can make a move to make the `Nim Sum` of the rest numbers 0.

The proof of the `claim 3` is easy.
Suppose we remove $$ x $$ stones from a pile,
and the number of the rest stones of the pile is $$ r $$.
Then the `Nim Sum` of the rest numbers is $$ (x + r) \oplus r $$.
When this number is $$ 0 $$, $$ x $$ must be $$ 0 $$, which is not a legal move.

NOTE: You may ask why the `Nim Sum` of the rest numbers is $(x + r) \oplus r$.
This is because the `XOR` operation is associative and commutative.
After removing $x$ stones, we change the number from $x + r$ to $r$.
We can use the origin `Nim Sum` 0 to calculate the new `Nim Sum`: $0 \oplus (x + r) \oplus r$.
The $\oplus (x + r)$ means we remove the whole pile,
and the $\oplus r$ means we add new pile of $r$ stones.
By the two steps, we can make a pile from $x + r$ stones to $r$ stones.
And $$ 0 $$ is the identity element of the `XOR` operation.
Therefore the `Nim Sum` of the rest numbers is $(x + r) \oplus r$.

The proof of the `claim 4` is a little bit complex.
Suppose the $$ i $$-th bit of the `Nim Sum` is $$ b_i $$ (the right most bit's index is $$ 1 $$),
and the left most bit is $$ b_m $$.
Note that $$ b_m = 1 $$.
Then we can find at least one pile of stones whose $$ m $$-th bit is $$ 1 $$.
Now we try to remove some stones from this pile to make the `Nim Sum` of the rest numbers $$ 0 $$.
We just need to consider the rightmost $$ m $$ bits of the pile.
We consider each bit of the rightmost $$ m $$ bits of the pile,
if the corresponding `Nim Sum` bit is $$ 1 $$,
we must make the bit of the rest number changed to change the `Nim Sum`'s bit to $$ 0 $$;
if the corresponding `Nim Sum` bit is $$ 0 $$,
we must make the bit of the rest number unchanged to keep the `Nim Sum` bit to be $$ 0 $$.
We could record each bit of the result.
Subtracting the result number from the origin number's rightmost $$ m $$ bits
could get the number of stones we should remove from the pile.
Because the $$ m $$-th bit of the rest is not $$ 0 $$,
we can find that the number of removed stones is positive.

Then we have proven the `theorem 2` by induction.

This proof may be complex,
there is a example of how to make a move to make the `Nim Sum` $$ 0 $$
when the original `Nim Sum` is not 0.

There are three piles of stones: 1, 10, 13.

The binary representations: 1, 1010, 1101.

The `Nim Sum` is $$ 110 $$, we can find $$ 1101 $$ is the pile
whose 3rd bit is $$ 1 $$: $$ 1(1)01 $$.
Now we consider each bit of the right $$ 3 $$ bits of the pile $$ 1101 $$,
the first one is $$ 1 $$: $$ 110(1) $$,
and the corresponding `Nim Sum` bit is $$ 0 $$: $$ 11(0) $$, we keep this;
the second one is $$ 0 $$: $$ 11(0)1 $$,
and the corresponding `Nim Sum` bit is $$ 1 $$: $$ 1(1)0 $$, we change this;
the third one is $$ 1 $$: $$ 1(1)01 $$,
and the corresponding `Nim Sum` bit is $$ 1 $$: $$ 1(1)0 $$, we change this.
After the three steps, we get a number $$ 011 $$,
then we use the rightmost $$ 3 $$ bits of pile $$ 1(101) $$ to subtract the number $$ 011 $$,
then we get $$ 10 $$, which is $$ 2 $$ in decimal.
Therefore we remove $$ 2 $$ stones from the pile.
After this move, the rest three piles will be: $$ 1 $$, $$ 1010 $$, $$ 1011 $$,
whose `Nim Sum` is 0.

### Staircase Nim

#### Description

There are several piles of stones.
You and your friend will take turns to move one or more stones from a pile
that is not the left most one to the one adjacent to its left side.
The one who moves the last stone will be the winner.
You will take the first turn to move the stones.

#### Solution

We index the left most one as $$ 0 $$,
and the right most one as $$ n $$.
Let me show you the solution first:

* `Theorem 3`: when the `Nim Sum` of the odd index piles is $$ 0 $$,
you will lose the game. Otherwise, you will win the game.

Let me explain this. We still can prove by proving the following two claims:

* `Claim 5`: if the `Nim Sum` of the odd index piles is $$ 0 $$,
any move will make the `Nim Sum` of the rest odd index numbers not $$ 0 $$.
* `Claim 6`: if the `Nim Sum` of the odd index piles is not $$ 0 $$,
we can make a move to make the `Nim Sum` of the rest odd index numbers $$ 0 $$.

Actually, we have proven this two already.

For `claim 5`, consider one move from a odd position to its left,
this is same with removing some stones from it, which we have proved.
For one move from a even position to its left, after the move,
one number participating calculation of the `Nim Sum` will be increased a positive number,
we record this number as $$ r $$, and the original one as $$ x $$,
then the new number is $$ x + r $$,
and the `Nim Sum` of the rest numbers is $$ (x + r) \oplus x = r $$, which is not 0.

For `claim 6`, we just do what we have done in proof of the `claim 4`.
We can find the pile and move some stones to its left.

Then we have proven the `theorem 3` by induction.

## Variations of Nim Game

### Variation 1

#### Description

There are several buckets from $$ 0 $$ to $$ n $$ (from left to right).
Some buckets are empty,
and some buckets have some cookies.
You and your friend will take turns to move one cookie from a bucket
that is not the bucket $$ 0 $$ to any one which is at the left side of the bucket.
The one who moves the last cookie will be the winner
. You will take the first turn to move the cookie.

#### Solution

This one is the same as the [multiple piles](###multiple-piles) problem.
For each cookie, we can consider it as a pile of stones,
and the number of the stones is the index of the bucket.
Then we can use the `Nim Sum` to solve this problem.

### Variation 2

#### Description

There are several piles of stones.
Initially, the numbers are non-descending.
You and your friend will take turns to remove stones from a pile
and make sure that the rest numbers are non-descending.
The one who moves the last stone will be the winner.
You will take the first turn to move the stones.

#### Solution

We can calculate the difference between the adjacent piles.
For example, the differences of $$ 1, 2, 2, 3 $$ are $$ 1, 1, 0, 1 $$.
We can find that the differences are non-negative.
For one move, supposed we remove $$ x $$ stones from the $$ i $$-th pile,
then the difference between the $$ i $$-th pile and the $$ i+1 $$-th pile will increase by $$ x $$,
and the difference between the $$ i $$-th pile and the $$ i-1 $$-th pile will decrease by $$ x $$.
This is similar with the move in the [Staircase Nim](###staircase-nim) problem.
The only difference in this case is that we move the stones to the one adjacent to its right side,
rather than the left side.

### Variation 3

#### Description

There are several buckets from $$ 0 $$ to $$ n $$.
Some buckets are empty, and some buckets have one cookie.
You and your friend will take turns to move one cookie from a bucket
that is not the bucket $$ 0 $$ to any one which is at the left side of the bucket,
but you can not jump over any buckets that contains cookies.
The one who moves the last cookie will be the winner.
You will take the first turn to move the cookie.

This one is similar with [variation 2](###variation-2).
For each cookie, we just need to consider it as a pile of stones,
and the number of the stones is the index of the bucket.
Then this one becomes exactly the same as `variation 2`.

### Variation 4

#### Description

There are several coins on a line.
Some coins are heads up, and some coins are tails up.
You and your friend will take turns to choose one head-up coin and any other coin
that is at the left side of the chosen coin, and flip them.
The one who flips the last coin will be the winner.
You will take the first turn to flip the coins.

#### Solution

This one is similar with the [multiple piles](###multiple-piles) problem.
We consider the head-up coins as the stones,
and the number of the stones are the index of the coins.
For the chosen two head-up coins, it is same with removing the whole two piles of stones.
If the left chosen coin is tail-up, the result is we remove some stones from a pile.

With this conversion,
we can find that we can not make any two piles of stones with the same number.
This looks different from [multiple piles](###multiple-piles),
but actually they are the same, we have two claims below:

* `Claim 7`: If the `Nim Sum` of a set of numbers is $$ 0 $$,
any move will make the `Nim Sum` of the rest numbers not $$ 0 $$.
* `Claim 8`: If the `Nim Sum` of a set of numbers is not $$ 0 $$,
we can make a move to make the `Nim Sum` of the rest numbers $$ 0 $$.

For the `claim 7`, we only need consider if we can make the `Nim Sum` $$ 0 $$
by removing two piles at the times.
For any two piles $$ x $$ and $$ y $$, they must have different numbers of stones.
So after removing, the `Nim Sum` is not $$ 0 $$.

For the `Claim 8`, suppose we can make two piles same.
After one move, if the `Nim Sum` $$ 0 $$ and there are two same piles,
actually we can remove the two same piles when take the move and the `Nim Sum` is still $$ 0 $$.
