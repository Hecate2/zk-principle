Zero-knowledge proof (ZKP) is art of polynomials and exponents with modular arithmetic.

### Requirements

English; maths at junior high school level

### Basic knowledge

#### exponent arithmetic

a^b == a to the (power of) b ==

```python
# a**b
result = 1
for i in range(b):
    result = result * a
return result
```

(a^b)(a^c) == a^(b+c)

a^(bc) == (a^b)^c == (a^c)^b. We may often skip between a^(bc) and (a^c)^b

(a^b)^c != a^(b^c)

#### polynomial

f(x) == c0 + c1x + c2x^2 + c3x^3 + ... + c_d x^degree(f)

c0, c1, ... c_d are constants

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

**Till now we are just providing basic mathematical tools to verify that some prover knows some *s(x)*, not trying to make the computation be ZK at all. We will make the real ZK algorithm in a very later section.** 

### Homomophic encryption: presenting the random value *r* to the prover, without letting the prover know what *r* is

Homomorphic encryption allows us to perform arithmetic operations on encrypted numbers, instead of the original raw numbers. For example, we can pick a publicly known base number `5`, and let all the raw numbers be the exponent of the raw number. Then, we compute a raw operation `3+2` as multiplications of encrypted numbers:

- 5^3 * 5^2 == 125 * 25 == 3125

Then we decrypt `3125` with `log5` operation to recover `log5(3125) == 5 == 3+2`. Also we can make multiplications for raw numbers by multiple additions, which means multiple multiplications, or just power operations, on the encrypted numbers. Addition (and subtraction) and multiplication are enough for a raw number in a raw polynomial, because there is no division in a polynomial. 

#### Modular arithmetic

Here we still face the problem of the prover taking `log5` operation on the encrypted number `5^3` which recovers `3`. In practice, we use **modular arithmetic** to prevent it. For simplicity, assume that we have a computer with its **unsigned integer ranging from only 0 to 6**. In other words, we compute everything with "modulo 7" if the result is greater than or equals 7. Now check how difficult it is to reveal the raw number:

- 5^5 mod 7 == 3
- 5^11 mod 7 == 3
- 5^17 mod 7 == 3

**It is difficult to execute log operations in modular arithmetic** (the property will be used very often in this tutorial). This means the prover cannot know whether the original raw number is 5 or 11 or 17, etc. In practice, we pick very large prime numbers. To be pedant-like, we define the homomophic encryption function for an original raw value *v* as:

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

#### Computation without revealing the raw random number *r*

Now we can happily run an algorithm like this:

- Publicly known things:
  - a public polynomial *t(x)* (for the computation *p(x) = t(x)s(x)*)
  - the homomorphic encryption function *E(v)=g^v* (mod *n*)

- Verifier
  - Pick a random number *r*
  - (For computational simplicity,) calculate encryptions of *r* for all powers, i.e. *E(r), E(r^2), E(r^3), ..., E(r*^degree*(t))*
  - Provide the array [*E(r), E(r^2), E(r^3), ..., E(r^*degree*(t))*] to the prover
  - Compute *t(r)* (will be used later)
- Prover
  - Compute the polynomial *E(s(r))*. That is to calculate *s(r)* in the homomorphically encrypted field, using multiplication instead of addition.
  - Evaluate *E(p(r))*.
  - Return *E(p(r))* and *E(s(r))* to the verifier
- Verifier
  - Check whether ***E(p(r)) == E(t(r)s(r))***. Here we do not have to decrypt things. Just notice that we have *E(p(r)) == g^p(r) and E(s(r)) == g^s(r)* returned from the prover, and that we just need to compute *E(t(r)s(r)) == g^(t(r)s(r)) == **(E(s(r)))^(t(r))***
  - There is still possibility that the prover randomly guesses *E(p(r))* and *E(s(r))* that do form a "correct" proof. But when the prime modulo divisor *n* is large, it becomes difficult to make such a lucky guess. We can repeat the whole algorithm *d* times with different *r* to reduce the possibility of the prover's random guessing.

### Knowledge-of-Exponent Assumption (KEA): forbidding prover's arbitrary operations taken on the proved polynomial *s(x)*

Unfortunately, we see that the verifier simply check whether ***E(p(r)) == (E(s(r)))^(t(r))***, where *E(p(r))* and *E(s(r))* are both returned by the prover. For now, the prover can still pick some arbitrary values *z_p* and *z_h* such that *z_p == (z_h)^t(r)*. We can repeat the proving algorithm for *d* times for *d==*degree*(s(r))* to make the proof reasonable, but who knows whether the prover had picked a polynomial *s(x)* with a higher degree than *d*? **We need to forbid arbitrary operations on *s(x)*, and make sure the *s(x)* provided by the prover is identical.**

The solution is to ask the prover to keep performing the same arithmetic operations on an original value and another exponentially shifted value (at a random exponent *e* picked by the verifier and kept unknown to the prover). The verifier can keep checking whether the original and the shifted value differs only in the random exponent *e*. 

This is what KEA will do. We are still going to use the property that **it is difficult to execute log operations in modular arithmetic**. 

Let us begin with a simple example f(x)=cx of degree 1, where *c* is a constant coefficients. Given that, for a random input x=r, we have f(r)==cr and the homomorphically encrypted f(r) as E(f(r))==g^(cr) (mod n) == (g^cr) (mod n) .

- Verifier
  - Choose a random *r* and exponent *e*
  - provide the tuple (E(r), E(r)^e) to the prover. In this case the tuple is (g^r, g^(re))
- Prover
  - Apply coefficient *c* in the secret polynomial *E(s(x))*. This is to return ((g^r)^c, (g^(re))^c) to the verifier. Note that everything is run under modulo *n*, and the prover cannot find the value of *e*.
- Verifier
  - Check whether ((g^r)^c)^e == (g^(re))^c. This is to raise the first item returned from the prover to the power of *e*, and check whether the power equals the second item. Also note that the verifier cannot find the value of *c*.

And the algorithm remains the same for other degrees x^0, x^2, x^3, ... . So for a secret polynomial s(x) = c0+c1x+c2x^2+c3x^3+...+c_d x^(degree(s)), we can modify the algorithm in the previous section, adding some steps to prevent prover's arbitrary operations:

- Publicly known things
  - a public polynomial *t(x)* (for the computation *p(x) = t(x)s(x)*)
  - the homomorphic encryption function *E(v)=g^v* (mod *n*)

- Verifier
  - Choose a random *r* and exponent *e*
  - Provide the array [*E(r), E(r^2), E(r^3), ..., E(r^*degree*(t))*] (mod *n*) to the prover.
    - This is just [g^r^1, g^(r^2), ... , g^(r^degree(t))] (mod *n*)
  - Provide exponentially shifted values [*E(r)^e, E(r^2)^e, E(r^3)^e, ..., E(r^*degree*(t))^e*] to the prover
    - This is [g^r^e, g^(r^2)^e, ... , g^(r^degree(t))^e] (mod *n*)
- Prover
  - Compute the polynomial *E(s(r))* and return it to the verifier.
    - E(s(r)) == g^s(r) == (g^c0)(g^c1^r)(g^c2^r^2)(g^c3^r^3)...(g^c_d^r^degree(s))
    - Also E(s(r)) == g^(c0+c1x+c2x^2+c3x^3+...+c_d x^(degree(s))) == g^(s(r))
  - Compute *E(s(r^e))* using the exponentially shifted input, and return it to the verifier.
    - E(s(r^e)) == g^s(r^e) == (g^c0)(g^c1^(r^e)(g^c2^r^2^e)(g^c3^r^3^e)...(g^c_d^r^degree(s)^e)
    - Also E(s(r^e)) == g^(e(c0+c1x+c2x^2+c3x^3+...+c_d x^(degree(s)))) == g^(s(r))^e
    - meaning that there should be **E(s(r^e)) == g^(s(r))^e == E(s(r))^e**
- Verifier
  - Check whether E(p(r)) == E(t(r)s(r)).
  - Check whether E(s(r))^e == E(s(r^e)). This means that the prover is always providing an identical *s(x)*.

For simplicity, we now name the exponentially shifted polynomials with an apostrophe ('). For example,

- s'(x) == s(x^e)

The prover now is going to provide the whole proof of 3 values:

- E(p(r))==g^p(r), E(s(r))==g^s(r), E(s'(r))==g^s(r)^e==g^s(r^e)==g^s'(r)

And the verifier checks 2 conditions:

- E(p(r)) == E(t(r)s(r))
- E(s(r))^e == E(s(r^e)) (or E(s'(r)))

The problem is that the verifier may still extract some information from the proof. In the next section about real ZK, we will prevent the verifier from learning anything with arbitrary methods.

### ZK

We have used exponents a lot in previous sections, because it is difficult to make log operations in modular arithmetic. Now, still, exponent is all you need. 

For the original proof (E(p(r)), E(s(r)), E(s'(r))) returned by the prover, we ask the prover to pick another random exponent *d* (sorry but we are running out of English letters), and instead return (E(p(r))^d, E(s(r))^d, E(s'(r))^d, d) **(The prover should now additionally return *d* itself!)**. It is easy to show that the 2 verifying operations can still hold firm:

- E(p(r))^d == g^(p(r)d) == g^(t(r)s(r)d) == E(t(r)s(r))^d
- E(s(r))^de == g^(s(r^e))^d == E(s'(r))^d

Now, though easy steps, the verifier know nothing about E(p(r)) or E(s(r)).

### Non-interactive ZKP

Till now we have got an **interactive** ZKP. This requires the verifier and the prover to stay online, and pick their own secret parameters, making the proof valid for this time only. Third-parties cannot trust the result of untrusted verifiers. Additionally, the verifier has to store the picked *r*, *e* and *t(r)*, making ZKP a stateful operation, dirtier to handle in computer systems. In practice, we still want a non-interactive ZKP system, and meanwhile make trustworthy proofs for everyone. 

Remember that the verifier should pick secret values *r* and *e*, and **have to remember 2 stateful values *e* and *t(r)* until the proof is verified**. The two values now needs to be encrypted to be stored in the public. On the first touch, we may think we can use exponents again, and homomorphically encrypt the two values. But unfortunately, **we cannot multiply two different homomorphically encrypted values** (we just used multiplication of a secret value with a public one). (Also **we cannot use a homomorphically encrypted value as an exponent**.) We will use another tool called **cryptographic pairing** (or **bilinear map**) for the purpose. In the following sections we are going to introduce the tools we need for it.

#### Set

A set is a quite primitive concept, meaning just a set of things. When a specific set is defined, we can tell whether a specific thing belongs to the set or not. A set can include an infinite count of things. For example, you can define set **Z** that includes all integers, and tell that the number 0.1 does not belong to the set. Sets can include tuples of "things". For example, you can define **R^3** as all tuples of **three** real numbers. Using the set **R^3** you can describe coordinates in a 3-D space.

#### Map and function

A **map** is also a serious primitive mathematical concept, which, when you query something from a source set (called "domain set"), picks **one or more things** from a target set (called "range set"). Just think of a real map that maps the coordinates on the surface of the earth, to things drawn on paper. When you pick a point on the surface of the earth, a real map tells you there is a parking lot, and also nice restaurant to try. 

A **function** is just a map that intakes a tuple (including one or multiple "things") inputs picked from a set, and maps the tuple of inputs to **exactly one thing** in another set. For any given input, the output should be unique.

In fact, arithmetic operations (multiplication or addition) are just functions like f(a,b) that map a tuple of two (or maybe more or less) values to the set of all numbers.

But this time we are going to map not numbers, but points on a defined curve, to another point on the same curve, in the next subsection. We call this function "addition" on the curve. 

#### Arithmetic Operations on elliptic curves (EC)

Elliptic curves are those defined by equation y² = x³ + ax + b (just try to find all the points (x,y) that satisfy the equation), where a and b are constant parameters, and 4a³+27b² != 0. We are going to define a specific curve named BLS12–381, using the following equation:

- **y²=x³+4** (mod p)

with p= 0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaab (a very large 381-bit hexadecimal prime number)

Now we are going to define an addition function that maps a tuple of 2 arbitrary points on the curve, to a third point on the curve. On elliptic curves like this, we can **pick points P, Q on the curve, and compute addition of points P+Q by drawing a straight line over P and Q, finding third point that intersects the curve, and return the third point's symmetry point about X axis. If P and Q are the same point on the curve, we draw a tangent line of the curve over P, and find the symmetry of the other intersection point.** When there is no extra intersection, the result is infinity. Finally, infinity adding any point returns infinity. You can find visual calculations in the following page:

https://andrea.corbellini.name/ecc/interactive/reals-add.html

Use a=0 and b=4 for our BLS12-381 curve. If you are wondering if the addition operation is truly well-defined, please visit math textbooks about group theory.

Then we simply define multiplication of a number *n* and a point P as P+P+P+... for *n* counts of P.  

#### EC with integer-only computation

In practice, computers do not perform accurate calculation for real numbers. Therefore, we need integers, **simply by operating everything under modulo *p*. Here, given an integer *x* on a general curve y² = x³ + ax + b (mod p), we find *y* with square root under modulo p.** It can be proved that there is only 1 square root for every integer under modulo *p*. 

- sqrt(x)==x^((p+1)/2) (mod p)  (The exponent is an integer, because *p* is a large prime which is odd.)

Now let's calculate the result of P(xP, yP)+Q(xQ, yQ) == R(xR, yR) . We first calculate the slope *m* of line PQ:

- m=(yP-yQ)/(xP-xQ) (mod p)  (Watch out: a mod p division returns integer)

If P==Q, we need a tangent line, whose slope is

- m=(3xP²+a)/(2yP) (mod p)

Then

- xR=(m²-xP-xQ) (mod p)  (Use Vieta theorem for cubic equations)
- yR=yP+m(xR-xP)=yQ+m(xR-xQ) (mod p)

Play with visual calculations in the following page:

https://andrea.corbellini.name/ecc/interactive/modk-add.html

By the way, we can count all the integer points in the curve defined under modulo *p*, and define the count of points as the **order** of the curve. In practice where *p* is very large, we define a point *G* called **generator** on a curve. By multiplying *G* by a fixed integer (mod *p*) for many times, we can cyclically return to G itself. During the many times of multiplication, we pass by many points, which build up a subgroup of (or maybe all of) all the possible points on the curve. We can also count the order of the subgroup. 

#### Weil pairing

Remember that we are going to multiply two encrypted values. To be pedantic, what we need is a 2-input function e(g^x, g^y) that intakes two encrypted values g^x and g^y (with the common base *g*), and output their **encrypted product** e(g,g)^(xy). With the notation, we mean that in the process of multiplication, we do not have to care for the meaning of e(g,g), and preferably the x and y can be extracted outside (the "**bilinear**" property). Then later (for example) we can verify for some a, b whether e(g^x, g^y)e(g^a, g^b) == e(g,g)^(xy+ab). Just keep reading if you get confused.

