# Timing Attack

A timing attack is a side-channel attack in which the attacker attempts to compromise a cryptosystem by analyzing the time taken to execute cryptographic algorithms. Every logical operation in a computer takes time to execute, and the time can differ based on the input; with precise measurements of the time for each operation, an attacker can work backwards to the input.

Avoidance of timing attacks involves design of constant-time functions and testing of the final executable code.

## Algorithm

The following C code demonstrates a typical insecure string comparison which stops testing as soon as a character doesn't match.

```c
bool insecureStringCompare(const void *a, const void *b, size_t length) {
  const char *ca = a, *cb = b;
  for (size_t i = 0; i < length; i++)
    if (ca[i] != cb[i])
      return false;
  return true;
}
```

By comparison, the following version runs in constant-time by testing all characters and using a bitwise operation to accumulate the result.

```c
bool constantTimeStringCompare(const void *a, const void *b, size_t length) {
  const char *ca = a, *cb = b;
  bool result = true;
  for (size_t i = 0; i < length; i++)
    result &= ca[i] == cb[i];
  return result;
}
```

## Reference

[Timing attack](https://en.wikipedia.org/wiki/Timing_attack)

