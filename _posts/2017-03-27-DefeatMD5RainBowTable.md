---
title: 'Defeat MD5: Rainbow Table'
layout: post
tags:
  - Web Security
  - Algorithm
category: Internet
---

The reality is, given the MD5 of a password with alphabets and digits that with length less than 9, to output the initial password only need 1 second. 
<!--more-->

## Realiaty

Despite you may hear about how powerful the MD5 is, but the MD5 of below range of password can be cracked in 1 second...

Length | Combinations
--- | ---
1-6 | alphabets + digits + symbols
7 | lower case alphabets + digits
8 | lower case alphabets
8 | lower case alphabets + digits
8 - 11 | digits

## Naive Method

To defeat the MD5, a naive method is to record, for example, all the 3 digits number's MD5 value and the initial number. Then obviously we can crack any 3 digits numbers' MD5.

But this method will cost too much space to store the MD5 table. For example, to crack all length of 8 lower case alphabets MD5, you will need to store (26^8) * 16 bytes = 3328 GB space. (Here each MD5 value is 16 bytes).

Apparently, this is not the solution, but it inspire us to have the below solution.

## Rainbow Table

**Reduction Function**

First let `H` be the hash function, let `p` be a set of all 3 lengths lower case alphabets and let `h` be a set of all hash values of the 3 lengths lower case alphabets. In mathematical representation `H: p -> h`

Now image a function `R` that `R: h -> p`, Note that R is not the reverse function of `H`. It is impossible to get an inverse function of `H`. 

Rather, `R` is just a function that maps set `h` to set `p`. For example, if `H(nus)=0e23df0`, then maybe `R(0e23df0)=wef`. The output of `R` does not need to be same as `nus`, but it must output a lower case string with length of 3. We call `R` the reduction function.

**Hash Chains**

So far so good, we just need one more step to get to the key idea of rainbow table. See below chain:

```
      H              R          H              R
nus -----> 0e23df0 -----> wef -----> a3bc8aa -----> ytr
```
Obviously, given any element in the chain, we can easily find the elements after it by just store function `H` and `R`. But that's not what we want. What we want is given an element in the chain, we can find the element before it. Is that possible? Well, how about we store not only `H` and `R`, but also store the head and tail of the chain which is, in this case, `nus` and `ytr`.

If we store the head, tail, `H` and `R`(Note we didn't store the whole chain), it means given a hash value in the chain, we can find the head of the chain. Further more, since we have the head, we can find all elements before the given element.

Congratulations! If you understand what I am talking about, you should already know how to get the initial value of a hash value. 

Image that if we have many of this kind of chains that contains all the hash values in set `h` within the chains' body, then given any hash value in `h`, we can start from `H(this hash value)` and then use `H` and `R` alternatively to go along with a chain until the tail. And since we have stored the head and tail of all the chains, we can start from the head to find out the plain text within the chain.

**Rainbow Table**

But, we have a problem here, consider below situation:

```
       H              R          H              R           H              R
[nus -----> 0e23df0 -----> wef -----> a3bc8aa -----> ytr] -----> becd2a4 -----> mng

      H              R           H              R          H              R
trf -----> b983ffe -----> [nus -----> 0e23df0 -----> wef -----> a3bc8aa -----> ytr]

```

Have you found the redundant part of the two chains? These two chains contain nearly the same hash key-value pair but we can not tell it they are. Why? Because we just store the head and the tail of chains (if you forget why we do this, it is because we want to save space), and the two chains have different head and tail.

The redundant happens because the sparsity of function `R` is not good since it maps a 'long' value to a 'short' value. There have to be many collisions. We don't want redundant because we don't want to use too much space.

To resolve this problem, we just use different `R` at each step at the hash chain. As show below.

```
       H               R1         H              R2         H              R3
[nus -----> 0e23df0] -----> wef -----> a3bc8aa -----> jgh -----> 4a3bdca -----> opn

      H              R1          H               R2         H              R3
trf -----> b983ffe -----> [nus -----> 0e23df0] -----> ndu -----> bac334f -----> che
```

In the above chains, collision happened again at `nus` element. But because we used different `R` at each step, the next element will not collision again. So the whole chain won't be no use.

## Complexity

Rainbow table is a typical example of time-space tradeoff. It will cost more time on searching along the chains but cost less space to store the map relation between hash value and plain text.

> Author question: But the real situation is the rainbow table is faster - -.

Brute-force:  
Time Complexity: `O(|p|)`  
Space Complexity: `O(|p|*n)`, where n is the length of elements in `h`.

Rainbow Table:  
Time Complexity: `O(|p|/k)`  
Space Complexity: `O(|p|*n/k)`

Where `k` is the number of different `R` used in the chains. (`R1, R2, ..., Rk`)

If all `R` functions are perfect, then no collision will occur, so we only need to store `O(1/k)` of the elements in `p`. And since when we search along the chain, we use `H` and `R` instead of looking for the hard disk. So search each chain will cost only `O(1)`.

## Protect Against Rainbow Table

- Add salt to your password before hashing it.
   `saltedhash(password) = hash(password + salt)`  
    or `saltedhash(password) = hash(hash(password) + salt)`
- Don't use the default salt value of some open source framework.
- Make sure others don't know your salt.
