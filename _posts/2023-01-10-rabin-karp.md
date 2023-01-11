---
layout: post
title: Rabin-Karp string-searching!
date: 2023-01-10 10:30
summary: Explorng the theory and application of the Rabin-Karp algorithm with a code example in C++.
categories: algorithms
---

Hello world! The following is my overview of the Rabin-Karp algorithm[^1], which is a string-searching algorithm. Basically, the problem it solves is: given a string $$s$$ and a target string $$t$$, return the first index where $$t$$ is a substring of $$s$$. There are other algorithms such as Knuth-Morris-Pratt that solve the same problem, but this is also a neat algorithm because this is a pattern that can be reused for finding any duplicates in a sequence. On that note, Merkle trees[^2] use that same pattern.

## Theory

The idea of the Rabin-Karp algorithm is to iteratively calculate the hash of every potential substring of $$s$$ of size $$t$$. This way we can check for duplicates which may be a match.
For every possible index $$i < |s| - |t| + 1$$, the substring $$s[i] + s[i + 1] + ... + s[i + |t| - 1]$$ has the following hash:

$$ hash_i =  \sum_{j=0}^{|t| - 1} s[i + j] \cdot P^{|t| - 1 - j} \mod M $$

where $$P$$ is a large prime number (to reduce hash collisions) and $$M$$ is a large mod (in order to contain the size of the hash value).

A key thing to note is that $$ hash_i $$ can be used to calculate $$hash_{i + 1}$$ in $$\mathcal{O}(1)$$.


$$ hash_{i + 1} = hash_i - (s[i] \cdot P^{|t| - 1}) + s[i + |t| - 1] \mod M $$


## Application

I think when translating the above formula to code there are two practical issues that come up:

1. Since the size of $$t$$ can be quite large, computing these primes exponentiated to large numbers can be compute intensive.
2. There is an inelegant case above where calculating the first hash looks to be handled in a special way. The other hashes can be computed from a prior hash.
3. How to deal with hash collisions.

To deal with 1) I incrementally compute hash by multiplying the `hash` by `prime` for each character we check. And to deal with 2) I wrapped the logic for computing characters in a while loop until we've hashed exact number of characters we want. `modpow` [^3] below is a fast modular exponentiation function. In order to deal with hash collisions, we can manually check each potential match with $$t%%.

```c++
template <typename T>
T modpow(T base, T exp, T modulus) {
    base %= modulus;
    T result = 1;
    while (exp > 0) {
        if (exp & 1) result = (result * base) % modulus;
        base = (base * base) % modulus;
        exp >>= 1;
    }
    return result;
}

unordered_map<int, vector<int>> calculate_hashes(string &s, size_t size) {
    unordered_map<int, vector<int>> hash_to_idxs;
    if (s.size() < size) {
        return hash_to_idxs;
    }
    const long prime = 1229827;
    const long mod = 2147483647;
    const long diff = modpow<long>(prime, size - 1, mod);
    int j = 0;
    long hash = 0;
    for (int i = 0; i < s.size() - size + 1; i++) {
        while (j - i < size) {
            hash *= prime;
            hash %= mod;
            hash += s[j];
            hash %= mod;
            j++;
        }
        hash_to_idxs[hash].push_back(i);
        hash += mod;
        hash = hash - ((diff * s[i]) % mod);
        hash %= mod;
    }
    return hash_to_idxs;
}

int find(string &s, string &t) {
    auto hashes = calculate_hashes(s, t.size());
    auto target_hash = *calculate_hashes(t, t.size()).begin();
    for (auto &idx: hashes[target_hash.first]) {
        if (s.substr(idx, t.size()) == t) {
            return idx;
        }
    }
    return -1;
}
```

---
[^1]: [https://en.wikipedia.org/wiki/Rabin-Karp_algorithm](https://en.wikipedia.org/wiki/Rabin-Karp_algorithm)
[^2]: [https://en.wikipedia.org/wiki/Merkle_tree](https://en.wikipedia.org/wiki/Merkle_tree)
[^3]: Schneier, Bruce (1996). Applied Cryptography: Protocols, Algorithms, and Source Code in C, Second Edition (2nd ed.). Wiley. ISBN 978-0-471-11709-4.
