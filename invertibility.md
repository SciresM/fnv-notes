# Invertibility

The core problem I care about is taking a known hash `H` solving for unknown `M` such that `FNV(M) == H`.

FNV is, [at its core](https://github.com/SciresM/fnv-notes/blob/master/basics.md), a series of multiples and xors.

Xor is trivially invertible. That said, in my context knowing what values to xor with is equivalent to solving the problem I want to solve, so this is only relevant in a highly pedantic/technical sense.

Multiplies are, less obviously, invertible:

* FNV's construction requires that the `prime` parameter used for multiplication be prime.
* Thus, `prime` is coprime to totient(2^N) (and 2^N).
* Thus, `prime` is a unit in the group multiplication modulo 2^N.

In (only a little) less mathematical terms, there exists some `I` so that `(N * prime * I) % (1 << num_bits) == N` for all N (assuming fixed bit-width).

The relevant values are presented below:

```py

FNV_PRIME_32  = 0x01000193
FNV_PRIME_64  = 0x00000000100000001B3

FNV_PRIME_INV_32  = 0x359C449B
FNV_PRIME_INV_64  = 0xCE965057AFF6957B
```

With these in hand, we can observe that fnv is fully invertible as follows:

```py
# Normal FNV functions
fnv1_32  = lambda data: fnv1_core(data, FNV_BASIS_32, FNV_PRIME_32, 32)
fnv1_64  = lambda data: fnv1_core(data, FNV_BASIS_64, FNV_PRIME_64, 64)

fnv1a_32 = lambda data: fnv1a_core(data, FNV_BASIS_32, FNV_PRIME_32, 32)
fnv1a_64 = lambda data: fnv1a_core(data, FNV_BASIS_64, FNV_PRIME_64, 64)

# Core Inverse functions
# Because these functions differ *only* in the order of mult/xor,
# and "inverting" just requires reverse order, they are inverses of eachother.
fnv1a_inv_core = fnv1_core
fnv1_inv_core = fnv1a_core

# Inverse functions
fnv1_inv_32  = lambda data, basis: fnv1_inv_core(data, basis, FNV_PRIME_INV_32, 32)
fnv1_inv_64  = lambda data, basis: fnv1_inv_core(data, basis, FNV_PRIME_INV_64, 64)

fnv1a_inv_32 = lambda data, basis: fnv1a_inv_core(data, basis, FNV_PRIME_INV_32, 32)
fnv1a_inv_64 = lambda data, basis: fnv1a_inv_core(data, basis, FNV_PRIME_INV_64, 64)
```

With these in hand, I would make the observation that the problem I care about is *equivalent* to the same problem run in reverse.

If we have H = FNV(M) and want to find M, rather than checking if FNV(M) == H for many M, we can instead check if `FNV_INV(M, H) == FNV_BASIS`.

More generally, this means that benefits from common suffixes may be used identically to how one might use common prefixes.

Running FNV in reverse is exactly as expensive as running it forwards, which grants [interesting options when brute forcing](https://github.com/SciresM/fnv-notes/blob/master/meet-in-the-middle.md).
