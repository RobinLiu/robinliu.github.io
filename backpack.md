title: DP backpack
date: 2016-08-27 14:35:28
tags:
---
## Backpack 
>Given n items with size Ai, an integer m denotes the size of a backpack. How full you can fill this backpack?
>
You can not divide any item into small pieces.
>
Have you met this question in a real interview? Yes
Example
If we have 4 items with size [2, 3, 5, 7], the backpack size is 11, we can select [2, 3, 5], so that the max size we can fill this backpack is 10. If the backpack size is 12. we can select [2, 3, 7] so that we can fulfill the backpack.
>
You function should return the max size we can fill in the given backpack.

```c++
int backPack(int m, vector<int> A) {
    size_t n = A.size();
    vector<bool> dp(m+1);
    dp[0] = true;
    for (size_t i = 0; i < n; ++i) {
        for (int j = m - A[i]; j >= 0; --j) {
            //dp[j] = dp[j] || (j >= A[i] && dp[j-A[i]]);
            if (dp[j]) dp[j+A[i]] = true;
        }
    }

    for (int i = m; i >= 0; --i) {
        if (dp[i]) return i;
    }

    return 0;
}
```


## Backpack II
0-1 背包问题
>Given n items with size Ai and value Vi, and a backpack with size m. What's the maximum value can you put into the backpack?
>
You cannot divide item into small pieces and the total size of items you choose should smaller or equal to m.
>
Have you met this question in a real interview? Yes
Example
Given 4 items with size [2, 3, 5, 7] and value [1, 5, 2, 4], and a backpack with size 10. The maximum value is 9.

*Solution 1* 
```c++
int backPackII(int m, vector<int> A, vector<int> V) {
    size_t n = A.size();
    vector<vector<int>> f(n+1, vector<int>(m+1));
    
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j >= A[i-1])
                f[i][j] = max(f[i-1][j], f[i-1][j-A[i-1]] + V[i-1]);
            else 
                f[i][j] = f[i-1][j];
        }
    }
    
    return f[n][m];
}
```

*Solution 2*: Rolling array
```c++
int backPackII(int m, vector<int> A, vector<int> V) {
    size_t n = A.size();
    vector<int> pre(m+1), cur(m+1);
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j >= A[i-1]) {
                cur[j] = max(pre[j], pre[j-A[i-1]] + V[i-1]);
            } else {
                cur[j] = pre[j];
            }
        }
        pre = cur;
    }
    return cur[m];
}    
```

*Solution 3*: One array, reverse ordre
```c++
int backPackII(int m, vector<int> A, vector<int> V) {
    size_t n = A.size();
    vector<int> dp(m + 1);
    for (size_t i = 0; i < n; ++i) {
        for (size_t j = m; j != 0; --j) {
            if (A[i] <= j) {
                dp[j] = max(dp[j], dp[j-A[i]] + V[i]);
            }
        }
    }
    return dp[m]; 
}
```


## Backpack III
完全背包问题
>Given n kind of items with size Ai and value Vi( each item has an infinite number available) and a backpack with size m. What's the maximum value can you put into the backpack?
>
You cannot divide item into small pieces and the total size of items you choose should smaller or equal to m.
>
Have you met this question in a real interview? Yes
Example
Given 4 items with size [2, 3, 5, 7] and value [1, 5, 2, 4], and a backpack with size 10. The maximum value is 15.

state:    f[i][j]: max value of i items with total size j;
function: f[i][j] = max(f[i-1][j], f[i][j-A[i]] + V[i]);
和前面不同的地方就是方程中是 f[i][j-A[i]] 而不是 f[i-1]. 表示当前的选(1-N)次.

```c++
int backPackIII(vector<int>& A, vector<int>& V, int m) {
    size_t n = A.size();
    vector<vector<int>> f(n+1, vector<int>(m+1));
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j>=A[i-1])
                f[i][j] = max(f[i-1][j], f[i][j-A[i-1]] + V[i-1]);
            else
                f[i][j] = f[i-1][j];
        }
    }
    
    return f[n][m];
}
```

*Rolling Array*
```c++
int backPackIII(vector<int>& A, vector<int>& V, int m) {
    size_t n = A.size();
    vector<int> pre(m+1), cur(m+1);
    for (int i = 1; i <=n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j >= A[i-1])
                cur[j] = max(pre[j], cur[j-A[i-1]] + V[i-1]);
            else
                cur[j] = pre[j];
        }
        pre = cur;
    }
    
    return cur[m];
}
```

*One Array*
```c++
int backPackIII(vector<int>& A, vector<int>& V, int m) {
    // Write your code here
    size_t n = A.size();
    vector<int> dp(m + 1);

    // for (size_t i = 0; i < n; ++i) {
    //     for (size_t j = m; j != 0; --j) {
    //         if (A[i] <= j) {
    //             dp[j] = max(dp[j], dp[j-A[i]] + V[i]);
    //         }
    //     }
    // }
    
    for (size_t i = 0; i < n; ++i) {
        for (int j = A[i]; j <= m; ++j) {
            dp[j] = max(dp[j], dp[j-A[i]] + V[i]);
        }
    }

    return dp[m];        
}
```

## Backpack IV
>Given n items with size nums[i] which an integer array and all positive numbers, no duplicates. An integer target denotes the size of a backpack. Find the number of possible fill the backpack.
Each item may be chosen unlimited number of times
>
Have you met this question in a real interview? Yes
Example
Given candidate items [2,3,6,7] and target 7,
>
A solution set is: 
[7]
[2, 2, 3]
return 2

state: f[i][j] : number of ways of first i items with target j
function: `f[i][j] += f[i-1][j] + f[i][j-nums[i]];`

```c++
int backPackIV(vector<int>& nums, int target) {
    // Write your code here
    size_t n = nums.size();
    int m = target;
    vector<vector<int>> f(n+1, vector<int>(m+1));
    // f[0][0] = 1;
    for (int i = 0; i <=n; ++i) f[i][0] = 1;
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j >= nums[i-1])
                f[i][j] += (f[i-1][j] + f[i][j-nums[i-1]]);
            else
                f[i][j] += f[i-1][j];
        }
    }
    return f[n][m];
}    
```


```c++
int backPackIV(vector<int>& nums, int target) {
    // Write your code here
    vector<int> f(target + 1);
    f[0] = 1;
    for (auto num : nums) {
        for (int i = 1; i <= target; ++i) {
            if (num <= i) f[i] += f[i-num];
        }
    }
    return f[target];

    // not working
    // for (int i = 1; i <= target; ++i) {
    //     for (auto num : nums) {
    //         if (num <= i) f[i] += f[i-num];
    //     }
    // }
    // return f[target];
}
```


## Backpack V
>Given n items with size nums[i] which an integer array and all positive numbers. An integer target denotes the size of a backpack. Find the number of possible fill the backpack.
>
Each item may only be used once

state: f[i][j]: number of ways of putting first i items into size j; 
function: `f[i][j] = f[i-1][j] + f[i-1][j-nums[i-1]]`; 
init: f[i][0] = 1 
result: f[n][m] 

```c++
int backPackV(vector<int>& nums, int target) {
    // Write your code here
    size_t n = nums.size();
    int m = target;
    vector<vector<int>> f(n+1, vector<int>(m+1));
    for (int i = 0; i <= n; ++i) {
        f[i][0] = 1;
    }
   
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j >= nums[i-1])
                f[i][j] = f[i-1][j] + f[i-1][j-nums[i-1]];
            else 
                f[i][j] = f[i-1][j];
        }
    }
    
    return f[n][m];
}
```

*Optimize 1: rolling array;*

```c++
int backPackV(vector<int>& nums, int target) {
    size_t n = nums.size();
    int m = target;
    vector<int> pre(target+1), cur(target+1);
    pre[0] = cur[0] = 1;
    
    for (int i = 0; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (j >= nums[i-1])
                cur[j] = pre[j] + pre[j-nums[i-1]];
            else
                cur[j] = pre[j];
        }
        pre = cur;
    }
    
    return cur[target];
}
```

*Optimize 2: one array;*
The inner loop is starting from *largest value to small(reversed order)*. The reason is that large vaule depends on smaller previous row's value. If update smaller value first, then the larger value's dependency is updated before used. But update from larger value will keep previous row's small value for the entire loop.

```c++
int backPackV(vector<int>& nums, int target) {
        vector<int> f(target+1);
        f[0] = 1;
        for (auto num : nums) {
            for (int i = target; i >= num; --i) {
                f[i] += f[i-num];
            }
        }
        
        return f[target];
}       
```

## Backpack VI

>Given an integer array nums with all positive numbers and no duplicates, find the number of possible combinations that add up to a positive integer target.
>
The different sequences are counted as different combinations.
>
Have you met this question in a real interview? Yes
Example
Given nums = [1, 2, 4], target = 4
>
The possible combination ways are:
    [1, 1, 1, 1]
    [1, 1, 2]
    [1, 2, 1]
    [2, 1, 1]
    [2, 2]
    [4]
return 6

解法与第四题的区别就是循环换了个位置
```c++
int backPackVI(vector<int>& nums, int target) {
    // Write your code here
    vector<int> f(target + 1);
    f[0] = 1;
    
    // not working
    // for (auto num : nums) {
    //     for (int i = 1; i <= target; ++i) {
    //         if (num <= i) f[i] += f[i-num];
    //     }
    // }
    // return f[target];

    for (int i = 1; i <= target; ++i) {
        for (auto num : nums) {
            if (num <= i) f[i] += f[i-num];
        }
    }
    return f[target];
}
```
