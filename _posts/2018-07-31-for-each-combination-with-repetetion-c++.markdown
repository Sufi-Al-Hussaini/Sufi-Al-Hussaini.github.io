---
layout: post
title: C++ combinations with repetition
description: A handy template function to generate all possible combinations of a set with repetition
date: '2018-07-31 12:28:23'
tags: c++ algorithms  
---

## A handy template function to generate all possible combinations of a set with repetition.

***

### Combinations

The STL provides some very handy functions for permutations, namely [`next_permutation`](http://www.cplusplus.com/reference/algorithm/next_permutation/) and [`prev_permutation`](http://www.cplusplus.com/reference/algorithm/prev_permutation/). 

A combination is also a selection of items from a collection, but it differs from a permutation in that the order of the selection does not matter.

An SO user @mitchnull has posted a way to generate combinations using `std::prev_permutation` [here](https://stackoverflow.com/a/9430993/3490458). Note that the code there generates combinations without repetition.

***

### Combinations with repetitions

> A k-combination with repetitions, or k-multicombination, or multisubset of size k from a set S is given by a sequence of k not necessarily distinct elements of S, where order is not taken into account: two sequences define the same multiset if one can be obtained from the other by permuting the terms. In other words, the number of ways to sample k elements from a set of n elements allowing for duplicates (i.e., with replacement) but disregarding different orderings. <sup>[1](#combination-wikipedia)</sup>

I found [this C code on SO](https://stackoverflow.com/a/23045070/3490458) and modified it for C++ to add a `for_each` like interface.

```
#include <cmath>

template<typename V, typename Callable>
void for_each_combination(V &v, size_t gp_sz, Callable f) {
    V gp(gp_sz);
    auto total_n = std::pow(v.size(), gp.size());
    for (auto i = 0; i < total_n; ++i) {
        auto n = i;
        for (auto j = 0ul; j < gp.size(); ++j) {
            gp[gp.size() - j - 1] = v[n % v.size()];
            n /= v.size();
        }
        f(gp);
    }
}
```

***

#### Example
```
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {'a', 'b', 'd'};
    for_each_combination(v, v.size(), [&](std::vector<int> &gp) {
        for (auto c: gp)
            std::cout << char(c) << " ";
        std::cout << std::endl;
    });
}
``` 

Output:
```
a a a 
a a b 
a a d 
a b a 
a b b 
a b d 
a d a 
a d b 
a d d 
b a a 
b a b 
b a d 
b b a 
b b b 
b b d 
b d a 
b d b 
b d d 
d a a 
d a b 
d a d 
d b a 
d b b 
d b d 
d d a 
d d b 
d d d 
```


***

##### References:

<a name="combination-wikipedia">1. </a>[ Wikipedia: Combinations](https://en.wikipedia.org/wiki/Combination)<br/>
