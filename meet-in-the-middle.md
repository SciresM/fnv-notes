# Meet in the middle

FNV is vulnerable to [length extension](https://github.com/SciresM/fnv-notes/blob/master/length-extension.md) and is [invertible](https://github.com/SciresM/fnv-notes/blob/master/invertibility.md). This leads directly to its being vulnerable to [meet in the middle attacks](https://en.wikipedia.org/wiki/Meet-in-the-middle_attack).

One way to think about the hash recovery problem is to observe that there are N unknown bytes of input that we want to guess, each of which may take 256 values.

If we wanted to search all possible inputs, naively we would have to calculate FNV(X) for all X of length N; this would be 256^N computations of FNV which would be (256^N) * N "operations" (one xor and one multiply).

By length extension, we could reuse our work, and calculate only 256^N computations of length 1 (256^N * 1 operations).

However, because FNV is invertible, we can make the following time-space trade-off:
* Calculate the result of FNV_INV(b, H) for all possible last bytes B. This will give a table T of 256 intermediate states.
* Instead of calculating FNV(M), calculate FNV(M[:-1]) and check if the result intermediate state is contained in T.

This saves the final add/multiply at the cost of multiplying our storage costs by 256.

By sorting T, we can quickly check if a value is contained in it, and thus quickly check if a prefix is valid for any suffix whose hash is in T.

The meet in the middle attack would make this trade-off repeatedly
* For some prefix length P, pre-compute FNV(X) for all X of length P, giving a table of size 256^P.
* For some suffix length S, pre-compute FNV_INV(X, H) for all X of length S, giving a table of size 256^S.
* Check if any entries in the prefix table are also contained in the suffix table.

This takes space `O(256^P + 256^S)` and time `O(256^P + 256^S)` to generate the tables.

Without sorting, time `O(256^(P+S))` to search the suffix table linearly for each prefix.
With sorting, time `O((256^P + 256^S) * S)` total (`O((256^P) * S)` to binary search the suffix table for each prefix, time `O((256^S) * S)` to sort the table).

This can lead to big speedup when attempting to solve for the unknown input that generates a desired hash.

Note, too, that rather than checking each prefix P's hash, one can further brute force using each P as "basis" with a "middle" of length L to search all inputs of length (P+L+S). This multiplies the time cost by 256^L.

Also, of course, rather than treating the fundamental unit as a byte, if the input is generically comprised of (delimited) tokens selected from a fixed set, all the same logic applies with 256 replaced by the size of the set.
* You can think of "bytes" as being the fixed set [0...255] with empty delimiter.
* An easy example would be that one could use the above attack on the set "english words" delimited by an underscore.

Similarly, if one has *multiple* target hashes H1, H2 which are expected to share a common prefix and differ only in suffix, one can:
* Generate suffix tables T1, T2 for some desired suffix length from H1/H2.
* Check for any table overlap; if a value is contained in both tables, the suffixes are recovered and one can solve the easier problem of finding M such that FNV(M) == [shared intermediate state] (and you can do a meet in the middle attack on that intermediate state).

One problem I care about is generically wants to find a message M such that FNV(M) is equal to any H contained in a set of target hashes { H1, H2, ..., HN }, and all hashes are known to be generated using delimited tokens from a set.

I would note that in this particular case, there is an inequality in terms of trade-off.
* Prefix table size grows at order `(token set size)^P` for prefix length P.
* Suffix table size grows at order `(token set size)^S * (hash set size)` for suffix length S.

In this variant of the problem, prefixes are cheaper in terms of space than suffixes, so depending on the magnitudes of the sets it may make sense to choose P > S.