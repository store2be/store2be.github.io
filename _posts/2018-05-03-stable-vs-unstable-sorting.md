---
date: 2018-05-03
categories:
- Rails
- Ruby
- Algorithms
- Testing
- Environment
title: 🔁 Stable vs unstable sorting
author_staff_member: 04_alex
---

At [store2be](http://store2be.com) we noticed a problem when running our Rails test suites in our two main development environments, on Mac OS X we had a test that would consistently fail, while on Linux it would pass fine every time, which in a way proves the old software development adage about the importance of having development machines as similar as possible to the the ones your production code will be running on.

Investigating we found that the issue was with the way the particular test was expecting elements to be sorted.

## What was happening

The basic difference between [stable sorting algorithms and unstable ones](https://en.wikipedia.org/wiki/Category:Stable_sorts) is that the stable ones make sure to preserve the original order of elements when sorting by a non-unique value that groups more than one element together. This is essentially what we were testing, that elements were sorted in groups, and that within each group they remained in their original order (the order they were added in).

The easiest way to see this difference is to sort by a non-unique value that defines a group of elements, for example an "urgency" field on a task. If a sorting function only considers this urgency field, the final ordering of two elements with the same urgency is left to the mercy of the algorithm used and it's not guaranteed that their original order will be maintained. Another example is sorting by the first letter of the word instead of the entire word. Any sorting algorithm will give you a correctly sorted array, but only a stable algorithm will maintain the original order of the elements within each group.

We were creating an array of objects (representing documents) that would then be sorted by their "type" (an alphanumeric string) which didn't have to be unique within the array (e.g. two documents could both have a type of "contract"). In our spec however, we expected documents of the same type to remain in the same order that we added them to the array in. On our Linux machines and on CI this was working as intended, on our Macs it was not.

## Why the behaviour was different

The issue was occurring within our Rails application, so we determined the root cause by running an awesome tool called [Pry](https://github.com/pry/pry) to inspect the source code behind the Ruby `sort` method. Sorting happens thanks to a `qsort` C function call. On Mac OS X this does in fact utilize a quick sort algorithm as the name would suggest, which is not inherently stable. In the specific case of the Mac OS X implementation (which in turn comes from FreeBSD) it is unstable.

We can see this unstableness in action if we consider the following array `[21, 12, 47, 41, 33, 11, 13, 31, 43]`. Sorting by the first digit on Mac OS X, either in C (using qsort) or in Ruby (sort or sort_by), we get `[12, 13, 11, 21, 33, 31, 47, 43, 41]` back, and although this is in fact sorted by the first digit, we see that the `11` follows `13` in the final array, even though in the original array it actually appears before it. `41` and `43` also get switched around in the final array. Here is an illustration of the [Mac OS X quick sort implementation](https://opensource.apple.com/source/xnu/xnu-1456.1.26/bsd/kern/qsort.c) in action that demonstrates precisely this behaviour:

![Animation showing the Mac OS X implementation of qsort() sorting the [21, 12, 47, 41, 33, 11, 13, 31, 43] array by the first digit, obtaining [12, 13, 11, 21, 33, 31, 47, 43, 41]]({{ site.url }}/images/stable_vs_unstable_sorting/qsort_animation.gif)

We can see that when the `13` swaps with the `47` they swap over the `11`, which ends up after the `13` in the final array.

Without going into rigorous proofs of stability/instability, we see that it's quick sort's use of multiple, non-adjacent pointers to swap elements that allows them to fall out of their original order and get mixed up.

Unlike the name suggests, `qsort` is not necessarily a quick sort on all operating systems. Mats Linander [has a very nice article](http://calmerthanyouare.org/2013/05/31/qsort-shootout.html) about all the differences in qsort implementations across a wide range of different libraries.

Regarding GLIBC's (the one used by most distributions of Linux) implementation, he mentions:

> This qsort() is interesting in that it shuns quicksort in favour of merge sort.

Ay, there's the rub!

It is indeed interesting, because merge sort is actually stable! If we run `qsort()` (or `sort_by` in ruby for that matter) on our example array from before on a Linux machine, again sorting by just the first digit and not the entire number, we see that for our example the algorithm is in fact stable (it returns `[12, 11, 13, 21, 33, 31, 47, 41, 43]`). Our tests were passing because our array was being sorted as expected, grouping our documents, but keeping them in their original order amongst similar documents. Merge sort compares elements with elements directly adjacent to them, therefore no "jumping" that can potentially alter the original ordering occurs. Here is an illustrated example:

![Animation showing a merge sort algorithm sorting the [21, 12, 47, 41, 33, 11, 13, 31, 43] array by the first digit, obtaining [12, 11, 13, 21, 33, 31, 47, 41, 43]]({{ site.url }}/images/stable_vs_unstable_sorting/merge_sort_animation.gif)

## What we changed

Code behaviour and tests should be as reproducible and deterministic as possible, instead of allowing failing tests on our Macs, we fixed the issue. Ruby itself however doesn't provide any simple alternatives to guarantee sort stability, so we came up with our own solution, (arguably) the simplest.

We continued to use Ruby's sort, but instead of sorting exclusively by our non-unique field, we would also consider the index of each element in the original array. That way if we had two elements that "tied" during sorting, they would always end up in the order they were originally in thanks to their different indices.

[This gist shows the solution](https://gist.github.com/indifferentalex/08a2434192399bcd13042deda73935c5) that was used. The reason an array of the element and its index was used for sorting instead of some sort of concatenation of the element and the index is that we don’t need to worry about any sort of padding in case different elements are of different size/length.

Stable sorting has given us deterministic behaviour and passing tests in all of our environments, just the way it should be. ✅