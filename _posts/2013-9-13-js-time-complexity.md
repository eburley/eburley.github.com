---
layout: post
title: JS time complexity - Back-story and Arrays
excerpt: container algorithmic performance?
---

### Back-story ###
In recalling the heady academic days of yore, I remember being taught all about algorithmic complexity, and data structures.  Particularly of interest was always the performance in space and time as data sets got really large.  There are a ton of treatise on this, and [wikipedia's article on big O notation](http://en.wikipedia.org/wiki/Big_O_notation) is a good place to start.

Some time ago I was a newly minted C++ programmer. I used to use the STL library ( this was before it became incorporated into runtime as the [Standard Library](http://en.wikipedia.org/wiki/C%2B%2B_Standard_Library)) for storing and processing data.  I was a big fan of the STL, finding it's iterator and predicate patterns a nerdy pleasure to use.  One of the great things about the library was that it provided guarantees around the algorithmic complexity (both average and worst case behavior) of the containers it implemented. 

### Arrays ###
I wanted to see if JavaScript containers behave similarly to their equivalents in other languages.  There are really only two included containers in ecma 1.2: objects and arrays. I wanted to focus on arrays (typed arrays are [just becoming available](http://caniuse.com/typedarrays)).  The closest c++ equivalent is [vector](http://www.cplusplus.com/reference/vector/vector/).  Vector stipulates that it should be implemented in such a way that:
- adding or removing elements from the end should be an amortized constant time operation
- accessing any element of the array should be a pointer operation (in other words constant time)

### Results ###
Using the excellent [jsperf.com](http://jsperf.com) I tested with 1000, 5000, and 100000 element arrays.  If they behave as expected ( amortized constant time for insert and random lookup) I would expect to see identical performance regardless of array size.  The results for [appending](http://jsperf.com/array-append-time-metrics-simple/2) and [random](http://jsperf.com/simple-array-access) access confirmed my expectations.

### Other Stuff ###
Other takeaways are that push and pop are definitely the way to go instead of using 

    array_var\[array_var.length\];
    delete array_var\[array_var.length-1\];


Also, it really behooves you to use an Array when you mean it (using objects instead of arrays is about 99 times [slower](http://jsperf.com/object-literal-treated-as-array) on append)