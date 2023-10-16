### How to make sure that two polynomials are the same?

We know that:

- A polynomial equation of degree *d* has *d* real-number solutions at most.

And thus:

- Two polynomials of maximum degree *d* can intersect at most *d* points.

Therefore, for any polynomial *f(x)*:

- Verifier chooses a random value *x* and calculate *f(x)* locally.
- Verifier asks the prover to calculate *f(x)* and check whether it is correct.

**For a (secret) polynomal *s(x)* of degree *d*, it is impossible for one to find another polynomial of the same or lower degree that has *d+1* or more intersection points.** 

### How to prove a secret polynomial: Factorization of polynomials

Now the prover want to prove that **he knows a secret polynomial *s(x)*, without telling *s(x)* itself to anyone**. Assume that we additionally have a public polynomial *t(x)* that everyone knows. 

Evidently, the prover can generate a product *p(x)* like this:

- *p(x) = t(x)s(x)*

where

- *s(x) = p(x)/t(x)*

Therefore:

- Verifier chooses a random number *r*, tells *r* to the prover, and locally calculates *t(r)*
  - A calculation of polynomial at degree(*t*)
- Prover calculates *p(r)* and *s(r) = p(r)/t(r)*, and gives *p(r)* and *s(r)* to the verifier.
  - A calculation of 2 polynomials at degree(*p*) and degree(*t*), respectively
- Verifier checks whether *p(r) == t(r)s(r)*
  - A simple multiplication of two numbers

We can repeat the algorithm above by picking different *r* values, in order to let the verifier believe that the prover knows *s(x)*. **Notice that the calculation difficulty for the verifier is typically much less than that of the prover, if degree(*t*) is relatively much smaller than degree(*p*).** 

But the proof is not firm enough for now. Notice some **methods for the prover to fake an *s(x)***:

1. Calculate *t(r)*, pick a random number *h* and set a fake *p(r) := t(r)h*, without knowing *s(x)* at all.
2. Alternatively, use another polynomial that has a degree much higher than the actual *s(x)*, which still satisfies the verifier's *r* requirements.

### Homomophic encryption: presenting the random value *r* to the prover, without letting the prover know what *r* is

Homomorphic encryption allows us to perform arithmetic operations on encrypted numbers, instead of the original raw numbers. For example, we can pick a publicly known base number `5`, and let all the raw numbers be the exponent of the raw number. Then, we compute a raw operation `3+2` as multiplications of encrypted numbers:

- 5^3 * 5^2 == 125 * 25 == 3125

Then we decrypt `3125` with `log5` operation to recover `log5(3125) == 5 == 3+2`. Also we can make multiplications for raw numbers by multiple additions, which means multiple multiplications, or just power operations, on the encrypted numbers. Addition (and subtraction) and multiplication are enough for a raw number in a raw polynomial, because there is no division in a polynomial. 

#### Modular arithmetic

Here we still face the problem of the prover taking `log5` operation on the encrypted number `5^3` which recovers `3`. In practice, we use **modular arithmetic** to prevent it. For simplicity, assume that we have a computer with its **unsigned integer ranging from only 0 to 6**. In other words, we compute everything with "modulo 7" if the result is greater than or equals 7. Now check how difficult it is to reveal the raw number:

- 5^5 mod 7 == 3
- 5^11 mod 7 == 3
- 5^17 mod 7 == 3

This means the prover cannot know whether the original raw number is 5 or 11 or 17, etc. In practice, we pick very large prime numbers. To be pedant-like, we define the homomophic encryption function as:

- E(v) = g^v mod n

Note that we always apply `mod n` for the encrypted number E(v), but not for the original raw number `v`. In fact, we are not trying to apply any arithmetic operation on original numbers at all. We just encrypt `v` and then computed everything in the encrypted field. 

#### Modular division

We are still leaving a small problem here: if the raw polynomial includes subtraction (minus, `-`) operations, how can I try a division in modular arithmetic? The idea is simple: for a/b under modulo n, we just check if `inverse of b` under modulo n exists. When `inverse of b` does not exist, we are unable to apply modular division in the case. 

- **The inverse of an integer *x* (under modulo *n*) is *y* such that (xy) mod n == 1**

In other words, the greatest common divisor of *x* and *y* is 1. *x* and *y* are co-prime.

Additionally, it can be proved (in textbooks of abstract algebra) that

- An integer cannot have an inverse unless the integer is relatively prime to the modulus *n*.
  - That is why we use large prime numbers for the modulus *n*
- The inverse (modulo *n*) is unique if it exists.

#### ZK without revealing the random number *r*

Now we can happily run an algorithm like this:

- Publicly known things:
  - a public polynomial *t(x)* (for the computation *p(x) = t(x)s(x)*)
  - the homomorphic encryption function *E(v)=g^v* (mod *n*)

- Verifier
  - Pick a random number *r*
  - (For computational simplicity,) calculate encryptions of *r* for all powers, i.e. *E(r), E(r^2), E(r^3), ..., E(r^*degree*(t))*
  - Compute *t(r)*
  - Provide the array [*E(r), E(r^2), E(r^3), ..., E(r^*degree*(t))*] to the prover
- Prover
  - Compute the polynomial *E(s(r))*. That is to calculate *s(r)* in the homomorphically encrypted field, using multiplication instead of addition.
  - Evaluate *E(p(r))*.
  - Return *E(p(r))* and *E(s(r))* to the verifier
- Verifier
  - Check whether ***E(p(r)) == E(t(r)s(r))***. Here we do not have to decrypt things. Just notice that we have *E(p(r)) == g^p(r) and E(s(r)) == g^s(r)* returned from the prover, and that we just need to compute *E(t(r)s(r)) == g^(t(r)s(r)) == **(E(s(r)))^(t(r))***
  - There is still possibility that the prover randomly guesses *E(p(r))* and *E(s(r))* that do form a "correct" proof. But when the prime modulo divisor *n* is large, it becomes difficult to make such a lucky guess. We can repeat the whole algorithm *d* times with different *r* to reduce the possibility of the prover's random guessing.

### Restricting the proved polynomial *s(x)*

Unfortunately, we see that the verifier simply check whether ***E(p(r)) == (E(s(r)))^(t(r))***, where *E(p(r))* and *E(s(r))* are both returned by the prover. For now, the prover can still pick some arbitrary values *z_p* and *z_h* such that *z_p == (z_h)^t(r)*. We can repeat the proving algorithm for *d* times for *d==*degree*(s(r))* to make the proof reasonable, but who knows whether the prover had picked a polynomial *s(x)* with a higher degree than *d*?

To be continued...