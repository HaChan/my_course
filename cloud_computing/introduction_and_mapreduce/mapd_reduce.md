# MapReduce Paradigm

What is MapReduce?

  - The term "map" and "reduce" are borrowed from Functional Language (Lisp).

Ex: sum of squares:

    (map square(1,2,3,4))

output (1,4,9,16)

    (reduce + (1,4,9,16))

output 30

or it can be written as:

```lisp
(reduce + (map square(1,2,3,4)))
```

```ruby
(1..4).map{|i| i**2}.reduce + #ruby
```

`Map` is a metafunction which processes each record, elements... sequentially and **independently**. In this example, square of 1 is calculated independently from square of 2, 3 and 4.

`Reduce` is a metafunction that processes a set of records or group of records in batches.

Sample application: _Wordcount_:

  -
