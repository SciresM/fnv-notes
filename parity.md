# Parity

Consider in the one byte case the parity P of an byte B and the corresponding parity P' of its hash FNV(B).

The initial parity of the hash result when beginning to calculate FNV(B) is just the parity of the basis (odd for all FNV_BASIS variants in use).

Then, a multiplication and xor will happen in some order depending on flavor.

The multiplication is by FNV_PRIME, which preserves the parity (FNV_PRIME is odd, odd * odd is odd and odd * even is even).

The parity post xor is, naturally, just the parity of B xor the parity of the intermediate value.

Thus, we have `P' = PARITY(FNV_BASIS) ^ PARITY(B)`, or simplified: `P' = 1 ^ P`.

We can easily extend this to the two byte case where the message M is is comprised of two bytes B1, B2, with parities P1, P2, to find:

`PARITY(FNV(M)) = 1 ^ P1 ^ P2`,

and more generally for a message comprised of bytes B1, B2, ..., BN with parities P1, P2, ..., PN we find:

`PARITY(FNV(M)) = 1 ^ P1 ^ P2 ^ ... ^ PN`.

Although this may *seem* useless, consider the following:

* When looking for M comprised of bytes B1, B2, ..., BN such that FNV(M) = H for fixed H, we know the parity of H.
* Suppose we are performing a brute-force search, and are choosing a value BN to test (after choosing B1, B2, ...).
* We can rewrite our equation as `PN = 1 ^ P1 ^ P2 ^ ... ^ PARITY(FNV(M))`, and evaluate to learn the parity of BN.
* We thus only need to calculate/check values for BN which have the correct parity.

This eliminates half of the options for the last element in a brute force search.

Note that this analysis can be extended (though extension not presented here) to the [meet in the middle attack](https://github.com/SciresM/fnv-notes/blob/master/meet-in-the-middle.md), by calculating + storing parities of suffixes/prefixes and only searching among suffixes which would result in a correct final parity.