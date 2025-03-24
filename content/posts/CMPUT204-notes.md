+++
date = '2025-03-22T12:35:58-06:00'
title = 'CMPUT204 Notes'
description = 'An overview of key concepts algorithms and their analysis.'
tags = ["computer-science", "algorithms"]
toc = true
+++

## Analysis and Proof
To do analysis we must first abstract away as many particulars of the machine we are working with as possible in order to get the problem to a manageably simple level. Typically in the course we are asked to write psudo-code but as far as I'm concerned the languages around today are the psudo-code of the 50s, so... I'll just write some mix of Rust, Zig, C and Python.

### Random Access Machine
The random access machine is a simplified view of a computer. It has the following properties:
- Direct CPU access to any memory location
- Move data between memory locations
- Can branch based on data
- Able to read from input to memory
- Able to write from memory to output
- Variables are local by default
- Parameters are passed by value

Key Properties: 
- Primitive operations take 1 time step
- Loops count by number of times executed
- Memory access is instant

Primitive Operations
- Assign a value
- Arithmetic
- Comparison
- Calling a function
- Returning a value

Components:
- Input
- Output
- Memory: `M[0]`, `M[1]`, ...
- Program

### Asymptotic Notation
These are basically just a strange way of phrasing the key relations in computer algorithm analysis. We want to have an idea of when functions are "better", "worse", or "as good as". Of course with the large variety in hardware, different instruction sets, etc that can be very hard to do if you just want to measure instruction counts. (Though that's partly what the RAM is for.) Instead of talking about the relations as one typically would (saying on function is related to another) in computer science the approach has been to talk about the different equivalence classes so that we can also talk about how the classes relate to eachother more naturally.

Here our different partitions end up being defined by $\theta(f), o(f)$ and $\omega(f)$ where $f : \R \rightarrow \R $. So each function class defines its own relation (which is part of the reason it makes more sense to think about the classes, we can compare different relations).

### Proving Correctness
**Induction** seems to do alot of heavy lifting in algorithms proofs. 

### Solving Recurrences

#### The Master Theorem
#### Recurrence Tree


## Sorting Algorithms
### Insertion Sort
**Key Idea:** An array of length one is sorted. Ignore everything else in the array given to us and think of it as an array of one `A[0..=0]`. Then increase its size and slide the new element to where they belongs. Repeat until sorted.

```rust
fn insertion_sort<T: Ord>(arr: &mut [T]) {
    for i in 2..arr.len() {
        let key = arr[i];

        let mut j = i - 1;
        while j > 0 && arr[j] > key {
            arr[j + 1] = arr[j];
            j -= 1;
        }

        arr[j + 1] = key;
    }
}
```

## Heaps

## Priority Queues

## Greedy Methodology
Greedy Algorithms are a class of algorithms that are used to solve optimization problems (at least I can't think of any other problem type I've used them for). A greedy algorithm locally considers what the best choice is at makes that without regard for how that will impact the values of future choices. 

- Very easy to come up with, but still hard to prove correctness
- Choices are final, there is no going back

That choices are final property is what allows greedy to be so efficient.

### Classic Greedy Problems
#### Fractional Knapsack
Let $\vec{x}, \vec{w} \in \R^n$ where $0 \le \vec{x}_i \le \vec{w}_i \space \forall i$ and $W \in \R$. Then maximize
$$\vec{x} \cdot \vec{w}$$
subject to 

$$\sum_{i = 1}^{n} \vec{x}_i \le W$$ 


#### Job Scheduling
#### Activity Selection





## Divide and Conquer
### Karatsuba's Algorithm (BigInt Mul)
Naive multiplication is $O(n^2)$ it would be nice if we could do better. 

### Strassen's Algorithm (Matrix Mul)


## Dynamic Programming

