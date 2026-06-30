# SPJINCS-SM3
An attempt for ISC

## Experimental parameter set: sphincs-sm3-224f

This repository contains an experimental SM3 backend and a truncated-output
parameter set named `sphincs-sm3-224f`.

The experiment follows the waterline suggested by the SHA-256 category-five
analysis: if the practical concrete bottleneck is about 2^217.4 work, using
256-bit `n` everywhere gives signature bytes that do not buy meaningful
end-to-end security in that model.  The experimental set therefore truncates
SM3 to `n = 28` bytes, i.e. 224 bits, while keeping the `256f` tree/FORS shape:

```text
n = 28
h = 68
d = 17
a = 9
k = 35
w = 16
```

The SPHINCS+ signature length formula used by the reference code is:

```text
sig_bytes = n + (a + 1) * k * n + d * len * n + h * n
len       = len1 + len2
len1      = 8n / log2(w)
len2      = floor(log_w(len1 * (w - 1))) + 1
```

For `sphincs-sha2-256f`, this is:

```text
n=32, len=67, sig_bytes=49856
```

For `sphincs-sm3-224f`, this is:

```text
n=28, len=59, sig_bytes=39816
```

The reduction is 10040 bytes, or about 20.14%.

Build and test, on a machine with `make` and `gcc` available:

```sh
cd ref
make clean
make test/spx PARAMS=sphincs-sm3-224f THASH=robust
./test/spx
```

## d*n-preserving set: sphincs-sm3-224f-dn

`sphincs-sm3-224f-dn` follows the d*n preservation rule for the WOTS-layer
term.  The SHA2-256f baseline has:

```text
d * n = 17 * 32 = 544
```

After truncating to `n = 28`, choose `d = 20` so that:

```text
d * n = 20 * 28 = 560
```

This keeps `w = 16`; changing to `w = 256` is a separate size/speed tradeoff,
not part of the hash-truncation argument.  The implementation needs
`h` divisible by `d`.  To avoid reducing the hypertree height below the original
`h = 68`, choose the smallest multiple of 20 that is at least 68: `h = 80`.

```text
n = 28
h = 80
d = 20
a = 9
k = 35
w = 16
sig_bytes = 45108
```

This is 4748 bytes shorter than `sphincs-sha2-256f`, or about 9.52%.

```sh
cd ref
make clean
make test/spx PARAMS=sphincs-sm3-224f-dn THASH=robust
./test/spx
```
