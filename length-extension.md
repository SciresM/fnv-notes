# Length extension

In my threat model's context, FNV is vulnerable to length-extension attacks.

This might not make very much sense to you, as "length-extension attacks" are weird to talk about in non-authentication cases. What I mean by this is kind of technical, so please bear with me.

FNV is a linear algorithm which performs a multiply and xor for each byte of data. In other words, it's O(N) -- it takes ~N time to compute FNV on an input of length N.

However, suppose that you have two inputs `X`, `Y` of length N, and you want to compute `FNV(X)`, `FNV(Y)`, `FNV(X+Y)`.

Naively, you would expect this would take `len(X) + len(Y) + len(X + Y) == 4N` time.

Considering [FNV's construction](https://github.com/SciresM/fnv-notes/blob/master/basics.md), we can observe that there is no special operation done at the end of the hash; the intermediate state is returned directly.

Thus, we could calculate `hash_x = fnv(X)` (and similar for Y) using the default basis, but then calculate `hash_xy = fnv(Y, basis=hash_x)`.

This would perform FNV on strings of length N three times, calculating our desired hashes in `len(X) + len(Y) + len(Y) == 3N` time, instead of 4N.

From a perspective of searching for input that gives a desired hash, what this means is that *intermediate work can be re-used to save time*.

This is extremely valuable particularly in the context of brute-force, as it allows skipping the multiplications/xors for any prefix you have previously calculated them for, and thus saves search time.