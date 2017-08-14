---
title: 'Leetcode 648. Replace Words'
layout: post
tags:
  - leetcode
  - contest
  - C++
category: Algorithms 
---

Useful code: Judge whether string A is a prefix of string B.


<!--more-->

## Description
In English, we have a concept called root, which can be followed by some other words to form another longer word - let's call this word successor. For example, the root an, followed by other, which can form another word another.

Now, given a dictionary consisting of many roots and a sentence. You need to replace all the successor in the sentence with the root forming it. If a successor has many roots can form it, replace it with the root with the shortest length.

You need to output the sentence after the replacement.

Example 1:

```
Input: dict = ["cat", "bat", "rat"]
sentence = "the cattle was rattled by the battery"
Output: "the cat was rat by the bat"
```

Note:
The input will only have lower-case letters.

1. 1 <= dict words number <= 1000
2. 1 <= sentence words number <= 1000
3. 1 <= root length <= 100
4. 1 <= sentence words length <= 1000

## Complexity Analysis

Time: O(N * M * W)
Space: O(N * w + M * W)

- N: sentence words number.
- M: diction words number.
- W: root length.
- w: sentence words length.

## Code

**My Code**:

```cpp
class Solution {
public:
    string replaceWords(vector<string>& dict, string sentence) {
        if (sentence == "") return "";
        sort(dict.begin(), dict.end());
        int l = 0, r = 0;
        vector<string> v;
        while (r < sentence.size()) {
            if (sentence[r] == ' ') {
                v.push_back(sentence.substr(l, r - l));
                r++;
                l = r;
            }
            r++;
        }
        if (sentence[r - 1] != ' ') v.push_back(sentence.substr(l, r - l));
        
        for (int i = 0; i < v.size(); i++) {
            for (int j = 0; j < dict.size(); j++) {
                auto res = std::mismatch(dict[j].begin(), dict[j].end(), v[i].begin());
                if (res.first == dict[j].end()) {
                    v[i] = dict[j];
                    break;
                }
            }
        }
        string ret = v[0];
        for (int i = 1; i < v.size(); i++) {
            ret += " " + v[i];
        }
        return ret;
    }
};
```

## Useful Code

Judge whether string A is a prefix of string B.

```cpp
std::string foo("foo");
std::string foobar("foobar");

auto res = std::mismatch(foo.begin(), foo.end(), foobar.begin());

if (res.first == foo.end())
{
  // foo is a prefix of foobar.
}
```
mismatch (first1, last1, first1)

Compares the elements in the range [first1,last1) with those in the range beginning at first2, and returns the first element of both sequences that does not match.