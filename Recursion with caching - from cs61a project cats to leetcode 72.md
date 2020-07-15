
The second project of cs 61a, CATS, is a program to measure the typing speed. During the process, I need to design a button for autocorrect; before autocorrect button, I need to set a limit to identify if a typing word is wrong; to identify if the limit is touched, I need to calculate how many steps to translate one word into another. Here I use a **recursive function** to break up the whole problem into smaller ones.

It is very similar with the problem from Leetcode 72. The difference between them is about the "limit". There is no limit in the leetcode problem. I tried the same code from cs61a to the leetcode problem and get an error saying "Time Limit Exceeded". Then I learned the lesson, **during dynamic programming, caching is essential for recursion**.

## Start from `meowstake_matches` in cs61a project ...

### Problem Description

`meowstake_matches`, is a diff function that returns the minimum number of edit operations needed to transform the `start` word into the `goal` word.

There are three kinds of edit operations:

- Add a letter to `start`,
- Remove a letter from `start`,
- Substitute a letter in `start` for another.

Each edit operation contributes 1 to the difference between two words.

```
>>> big_limit = 10
>>> meowstake_matches("cats", "scat", big_limit)       # cats -> scats -> scat
2
>>> meowstake_matches("purng", "purring", big_limit)   # purng -> purrng -> purring
2
>>> meowstake_matches("ckiteus", "kittens", big_limit) # ckiteus -> kiteus -> kitteus -> kittens
3
```

If the number of edits required is greater than `limit`, then `meowstake_matches` should return any number larger than `limit` and should minimize the amount of computation needed to do so.

### Here is the thought process:

Given two words, consider the base case: 

- if they are the same, `return` 0;
- if one of them is empty, while another one is not, `return` the length of the other one;
- if limit is touched, `return` a **LARGE** number, here I choose a positive infinity;

Then, if base cases are not satisfied, check from the first location of `start`. 

- if the first location of `start` equals the first location of `goal`, compare the rest, and step unchanged;
- else, there are three operations to the first location of `start`
    - after **add** one letter, we need to compare `start` and `goal[1:]`, and step += 1;
    - after **remove** one letter, we need to compare `start[1:]` and `goal`, and step += 1;
    - after **substitute** one letter, we need to compare `start[1:]` and `goal[1:]`, and step += 1;
    
Aboves are three recursive calls. Since the problem is trying to find the minimum steps, use `min` in each recursive step to find the optimized option.

### Applied in Python:


```python
def meowstake_matches(start, goal, limit):
    """A diff function that computes the edit distance from START to GOAL."""
    
    if start == goal: 
        return 0
    
    elif start == "" or goal == "": 
        return max(len(start), len(goal))
    
    elif limit == 0:
        return float("inf")

    else:
        if start[0] == goal[0]:
            return meowstake_matches(start[1:], goal[1:], limit)
        else:
            add_diff = meowstake_matches(start, goal[1:], limit-1)    
            remove_diff = meowstake_matches(start[1:], goal, limit-1)
            substitute_diff = meowstake_matches(start[1:], goal[1:], limit-1)
            return (min(add_diff, remove_diff, substitute_diff))+1
```

## ... to Leetcode 72. Edit Distance

### Problem Description
Given two words word1 and word2, find the minimum number of operations required to convert word1 to word2.

You have the following 3 operations permitted on a word:

- Insert a character
- Delete a character
- Replace a character

Example 1:
```
Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation: 
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e')
```

Example 2:
```
Input: word1 = "intention", word2 = "execution"
Output: 5
Explanation: 
intention -> inention (remove 't')
inention -> enention (replace 'i' with 'e')
enention -> exention (replace 'n' with 'x')
exention -> exection (replace 'n' with 'c')
exection -> execution (insert 'u')
```

### Here is the thought process:

At the first beginning, I simply modified my codes above, deleted `limit` cases, substituted `start` into `word1` and `goal` into `word2`, and submitted my solution on Leetcode, but get a **Time Limit Exceeded**. Did I do something wrong? Let's look at a simple example:

`word1 = "horse"`

`word2 = "hello"`

The recursive tree is like:

```
f("horse","hello")
    f("orse", "ello")
        f("orse","llo") # add
            f("orse", "lo") # add
                f("orse","o") # add
                    f("rse","")
                f("rse","lo") # delete
                    ...
                    ...
                    ...
                f("rse","o") # substitute
                    ...
                    ...
                    ...
            f("rse","llo") # delete
                ...
                ...
                ...
            f("rse","lo") # substiute
                ...
                ...
                ...
        f("rse", "ello") # delete
            ...
            ...
            ...
        f("rse","llo") # substitute
            ...
            ...
            ...
```

It is really a **HUGE** tree with only five letter words. The runtime is exponential. Actually there are many repeating work. The way to fix it is by **caching**, to save intermediate computations in a dictionary. As a result, when we met the same subproblem, we return the saved value, instead of doing the repeated work.

Here I tried to use `key: (i,j); value: steps` to record the steps for comparing from location `i` of `word1` and `j` of `word2`. The `cache` makes sure that I won't repeat any steps involving the same location.

### First try in Python:


```python
def minDistance(word1, word2, i=0, j=0, cache=None):
    
    # caching, i and j record the location of words
    if not cache:
        cache = {}
    if i == len(word1) and j == len(word2):
        return 0
    elif i == len(word1):
        return len(word2) - j
    elif j == len(word2):
        return len(word1) - i
    
    if (i,j) not in cache:
        if word1[i] == word2[j]:
            solution = minDistance(word1, word2, i+1, j+1, cache)
        else:
            add_diff = minDistance(word1, word2, i, j+1, cache)    
            remove_diff = minDistance(word1, word2, i+1, j, cache)
            substitute_diff = minDistance(word1, word2, i+1, j+1, cache)
    
            solution = (min(add_diff, remove_diff, substitute_diff))+1
        cache[(i,j)] = solution
    return cache[(i,j)]
        
```

We can also use a 2D array to do essentially the same thing as the dictionary of cached values. When we do this, we build up solutions from smaller subproblems to bigger subproblems (bottom-up). 

It is not a recursive way. We initialized a m by n table, where `m` is the length of `word1` and `n` is the length of `word2`.

The base case is the first row, involving an empty `word1` and the first column, involving an empty `word2`

In this way, both of the runtime and space complexity are `O(mn)`.

### Second try in Python


```python
def minDistance(word1, word2):
    
    # initialize the table
    m = len(word1)
    n = len(word2)
    table = [[0]*(n+1) for _ in range(m + 1)]
    
    for i in range(m+1):
        table[i][0] = i
    for j in range(n+1):
        table[0][j] = j
        
    for i in range(1,m+1):
        for j in range(1, n+1):
            if word1[i-1] == word2[j-1]:
                table[i][j] = table[i-1][j-1]
            else:
                table[i][j] = (min(table[i-1][j], table[i][j-1], table[i-1][j-1]))+1
    
    # return the bottom right corner
    return table[-1][-1]
```


```python

```
