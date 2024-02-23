---
title: 'Leetcode 374 weekly competition'
date: 2023-12-03
permalink: /posts/2023/12/Leetcode374week/
tags:
  - Leetcode
  - competition
---



[找出峰值](https://leetcode.cn/problems/find-the-peaks/)

[需要添加的硬币的最小数量](https://leetcode.cn/problems/minimum-number-of-coins-to-be-added/)

[统计完全子字符串](https://leetcode.cn/problems/count-complete-substrings/)

[统计感冒序列的数目](https://leetcode.cn/problems/count-the-number-of-infection-sequences/)



#### 1. Find the peaks

```python
class Solution:
    def findPeaks(self, a: List[int]) -> List[int]:
        return [i for i in range(1, len(a) - 1) if a[i - 1] < a[i] > a[i + 1]]
```

The first problem is too easy!!!

#### 2. Minimum Number of Coins to be Added

```python
class Solution:
    def minimumAddedCoins(self, coins: List[int], target: int) -> int:
        # sorted all the coin in coins so that we can get all vlaue from small to large
        coins.sort()
        # firstly set the s=1, which means that all values lower than s can be got now
        ans, s, i = 0, 1, 0
        # recycly until the largest value we can get achieve the target
        while s <= target:
            # when index is smaller than the length pf coins and coins[i] is smaller than s
            # Current section is [0, s-1], when a new value appeared the new section is [0, s-1]+coins[i]=[coins[i], s-1+coins[i]] and [0, s-1], which is feasible when coins[i]<=s and the two sections can be intersected as [0, s-1+coins[i]]
            if i < len(coins) and coins[i] <= s:
                s += coins[i]
                i += 1
            # if coins[i]>s, the new section is [0, s-1] and [coins[i], s-1+coins[i]], the values between s-1 and coins[i] can not be got, so we have to add new value into the coins.
            # firstly we should add the smallest value can not be got s into coins, or else larger s will not be got. The new section is updated as [0, s-1], [s, 2s-1]
            else:
                s *= 2  # 必须添加 s
                ans += 1
        return ans
```

1. _sorted all the coin in coins so that we can get all vlaue from small to large_
2. _firstly set the s=1, which means that all values lower than s can be got now_
3. _recycly until the largest value we can get achieve the target_
4. _when index is smaller than the length pf coins and `coins[i]` is smaller than s. Current section is `[0, s-1]`, when a new value appeared the new section is `[0, s-1]+coins[i]=[coins[i], s-1+coins[i]]` and [0, s-1], which is feasible when coins[i]<=s and the two sections can be intersected as `[0, s-1+coins[i]]`_
5. _`if coins[i]>s`, the new section is `[0, s-1]` and `[coins[i], s-1+coins[i]]`, the values between `s-1` and `coins[i]` can not be got, so we have to add new value into the coins. firstly we should __add the smallest value can not be got s__ into coins, or else larger s will not be got. The new section is updated as `[0, s-1], [s, 2s-1]`_

#### 3. Count Complete Substrings

```python
class Solution:
    def countCompleteSubstrings(self, word: str, k: int) -> int:
        def f(s):
            res = 0
            # judge how many distinct alphas appear
            for m in range(1, 27):
                if k*m>len(s):
                    break
                # Now the max distinct alphas is m-1
                counter = {}
                for i in range(len(s)):
                    counter[s[i]] = counter.get(s[i], 0) + 1
                    # calculate current left doundary
                    left = i+1-k*m
                    if left>=0:
                        res += all(c==0 or c==k for c in counter.values())
                        counter[s[left]] -= 1
            return res
        ans = i = 0
        while i<len(word):
            st =  i
            i += 1
            while i<len(word) and abs(ord(word[i])-ord(word[i-1]))<=2:
                i += 1
            ans += f(word[st:i])
        return ans
```

___Optimizer___:

```python
class Solution:
    def countCompleteSubstrings(self, word: str, k: int) -> int:
        def f(s):
            res = 0
            # judge how many distinct alphas appear
            for m in range(1, 27):
                if k*m>len(s):
                    break
                # Now the max distinct alphas is m-1
                cnt = Counter(s[:k*m])
                # count each time how many alphas, now largest m
                cc = Counter(cnt.values())
                if cc[k] == m:
                    res += 1
                for in_, out in aip(s[size:], s):
                    # due to the slide of window, the count of the times corresponding with in_ should
                    cc[cnt[in_]] -= 1
                    cnt[in_] += 1
                    cc[cnt[in_]] += 1
                    cc[cnt[out]] -= 1
                    cnt[out] += 1
                    cc[cnt[out]] += 1
                    if cc[k] == m:
                        res += 1
            return res

        ans = i = 0
        while i<len(word):
            st =  i
            i += 1
            while i<len(word) and abs(ord(word[i])-ord(word[i-1]))<=2:
                i += 1
            ans += f(word[st:i])
        return ans
```

#### 4. Count the Number of Infection Sequences

```python
MOD = 1_000_000_007
MX = 100_000

# 组合数模板
fac = [0] * MX
fac[0] = 1
for i in range(1, MX):
    fac[i] = fac[i - 1] * i % MOD

inv_fac = [0] * MX
inv_fac[MX - 1] = pow(fac[MX - 1], -1, MOD)
for i in range(MX - 1, 0, -1):
    inv_fac[i - 1] = inv_fac[i] * i % MOD

def comb(n: int, k: int) -> int:
    return fac[n] * inv_fac[k] % MOD * inv_fac[n - k] % MOD

class Solution:
    def numberOfSequence(self, n: int, a: List[int]) -> int:
        m = len(a)
        total = n - m
        ans = comb(total, a[0]) * comb(total - a[0], n - a[-1] - 1) % MOD
        total -= a[0] + n - a[-1] - 1
        e = 0
        for p, q in pairwise(a):
            k = q - p - 1
            if k:
                e += k - 1
                ans = ans * comb(total, k) % MOD
                total -= k
        return ans * pow(2, e, MOD) % MOD
```

__We figure out this problem from special to normal:__

1. _firstly, we think the condition that all people are not sick in in the middle, i.e.`x o o o x`, so we only need to think how many probable sequence there are the middle be sicked,_ _the first people can be the left or the right(if left then `x x o o x`), the second also right or left(if left then `x x x o x`,_ _so the kind of probable sequence is_ $2^{m-1}$, _m is the positions in the middle._
2. _then, we think the condition that there are several interspace, i.e. `x x o o o x o o o o x`, the time is seven, so we need to choose three times for the first interspace and four for the second, $C_7^3$, and then total probable sequence is $C_7^3 \ C_4^4 \ 2^{3-1}*2^{4-1}$._
3. _more normal condition can be considered_

