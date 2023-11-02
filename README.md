Zero-knowledge proof (ZKP) is art of polynomials and exponents with modular arithmetic.

### Requirements

English; maths at junior high school level; basic computer programming (arrays and `for` loops)

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

a^0 == 1 for any a

#### polynomial

f(x) == c0 + c1x + c2x^2 + c3x^3 + ... + c_d x^degree(f)

c0, c1, ... c_d are constants

#### roots of polynomial

f(x) has root *a* means that f(a)==0. Also, f(x) has divisor (x-a).

If a polynomial has 3 (and only 3) roots a, b and c, then f(x) has divisor (x-a)(x-b)(x-c).

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

Therefore:

- Verifier chooses a random number *r*, locally calculates *t(r)*, and tells *r* to the prover
  - A calculation of polynomial at degree(*t*)
- Prover calculates *p(r)* and *s(r)*, and gives *p(r)* and *s(r)* to the verifier.
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

- E(v) = g^v mod n  (g is a public constant base)

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
- degree(*s*) (we will omit that we publicly know degree(*s*) in later discussions)
    - Evidently degree(*p*)==degree(*t*)degree(*s*), so degree(*p*) is also publicly known
  
- Verifier
  - Pick a random number *r*
  - (For computational simplicity,) calculate encryptions of *r* for all powers, i.e. *E(r), E(r^2), E(r^3), ..., E(r*^degree*(p))*
  - Provide the array [*E(r), E(r^2), E(r^3), ..., E(r^*degree*(p))*] to the prover
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

The solution is to ask the prover to keep performing the same arithmetic operations on an original value and another exponentially shifted value (at a random exponent *e* picked by the verifier and kept unknown to the prover; *e* is also known as \alpha in other materials). The verifier can keep checking whether the original and the shifted value differs only in the random exponent *e*. The pair of two values acts like "**checksum**", with which you ensure that the prover does not cheat.

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
  - Provide the array [*E(r), E(r^2), E(r^3), ..., E(r^*degree*(p))*] (mod *n*) to the prover.
    - This is just [g^r^1, g^(r^2), ... , g^(r^degree(p))] (mod *n*)
  - Provide exponentially shifted values [*E(1)^e, E(r)^e, E(r^2)^e, ..., E(r^*degree*(p))^e*] to the prover
    - This is [g^e, g^r^e, g^(r^2)^e, ... , g^(r^degree(p))^e] (mod *n*)
  - The two arrays provided by the verifier is called **common reference string (CRS)**
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

- E(p(r))==g^p(r)
- E(s(r))==g^s(r)
- E(s'(r))==(g^s(r))^e==g^s(r^e)==g^s'(r)

And the verifier checks 2 conditions:

- E(p(r)) == E(t(r)s(r))
- E(s(r))^e == E(s(r^e)) (or E(s'(r)))

Now the remaining problem is that the verifier may still extract some information from the proof. In the next section about real ZK, we will prevent the verifier from learning anything with arbitrary methods.

### ZK

We have used exponents a lot in previous sections, because it is difficult to make log operations in modular arithmetic. Now, still, exponent is all you need. 

For the original proof (E(p(r)), E(s(r)), E(s'(r))) returned by the prover, we ask the prover to pick another random exponent *d* (sorry but we are running out of English letters), and instead return

- E(p(r))^d
- E(s(r))^d
- E(s'(r))^d
- d **(The prover should now additionally return *d* itself!)**

It is easy to show that the 2 verifying operations can still hold firm:

- E(p(r))^d == g^(p(r)d) == g^(t(r)s(r)d) == E(t(r)s(r))^d
- E(s(r))^de == g^(s(r^e))^d == E(s'(r))^d

Now, though easy steps, the verifier know nothing about E(p(r)) or E(s(r)).

### Non-interactive ZKP: multiplying 2 encrypted values

Till now we have got an **interactive** ZKP. This requires the verifier and the prover to stay online, and pick their own secret parameters, making the proof valid for this time only. Third-parties cannot trust the result of untrusted verifiers. Additionally, the verifier has to remember the picked *r*, *e* and *t(r)*, making ZKP a stateful operation, dirtier to handle in computer systems. In practice, we still want a non-interactive ZKP system, and meanwhile make trustworthy proofs for everyone. 

Remember that the verifier should pick secret values *r* and *e*, and **have to remember 2 stateful values *e* and *t(r)* until the proof is verified, without revealing *r, t(r)* and *e* to the prover**. Now, to convice the public, *e* and *t(r)* needs to be encrypted and stored in the public. On the first touch, we may think we can use exponents again, and homomorphically encrypt the two values. 

But unfortunately, we have to check E(p(r)) == E(t(r)s(r)) with *E(t(r)s(r)) == g^(t(r)s(r)) == (E(s(r)))^(t(r))*, where t(r) was kept as unencrypted secret by the verifier in interactive ZKP. However, now we need to store an encrypted t(r) in the public, and **we cannot multiply two different homomorphically encrypted values** (we just used multiplication of a secret value with a public one). Also **we cannot use a homomorphically encrypted value as an exponent**. Consider that, E(p) and E(q) are usually in an extremely large range (remember that the modulus *n* is large; otherwise we can use brute force to find p and q). If we want to compute pq, we try p+p+p+... for q times in the raw number field, and actually multiply E(p) for E(q) times in the encrypted field, which is too slow to be computed.

We will use another tool called **cryptographic pairing** (or **bilinear map**) for the purpose. In the following sections we are going to introduce the tools we need for it.

#### Set (optional)

A set is a quite primitive concept, meaning just a set of things. When a specific set is defined, we can tell whether a specific thing belongs to the set or not. A set can include an infinite count of things. For example, you can define set **Z** that includes all integers, and tell that the number 0.1 does not belong to the set. Sets can include tuples of "things". For example, you can define **R^3** as all tuples of **three** real numbers. Using the set **R^3** you can describe coordinates in a 3-D space.

#### Map and function (optional)

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

In practice, computers perform accurate calculation only for integers. For a practical integer-only curve, we simply **operate everything under modulo *p* (where *p* is a very large prime number)**. Here, given an integer *x* on a general curve y² = x³ + ax + b (mod p), we find *y* with **square root under modulo *p*.** It can be proved that there is only 1 square root result for each integer under modulo *p*. 

- sqrt(x)==x^((p+1)/2) (mod p)  (The exponent is an integer, because *p* is a large prime which is odd.)

Now let's calculate the result of P(xP, yP)+Q(xQ, yQ) == R(xR, yR) . We first calculate the slope *m* of line PQ:

- m=(yP-yQ)/(xP-xQ) (mod p)  (Watch out: a mod p division returns integer)

If P==Q, we need a tangent line, whose slope is

- m=(3xP²+a)/(2yP) (mod p)

Then (do not forget that R is the symmetric point about X axis of the 3rd intersection point)

- xR=(m²-xP-xQ) (mod p)  (Use Vieta theorem for cubic equations)
- yR=yP+m(xR-xP)=yQ+m(xR-xQ) (mod p)

Play with visual calculations in the following page:

https://andrea.corbellini.name/ecc/interactive/modk-add.html

By the way, we can count all the integer points in the curve defined under modulo *p*, and define the count of points as the **order** of the curve. In practice where *p* is very large, we define a point *G* called **generator** on a curve. By multiplying *G* by a fixed integer (mod *p*) for many times, we can cyclically return to G itself. During the many times of multiplication, we pass by many points, which build up a **subgroup** of (or maybe all of) all the possible points on the curve. We can also count the order of the subgroup. 

Some EC has more than 1 generators, with which you can find different subgroups that do not intersect.

And finally, for an elliptic curve E (mod *p*) with order *r*, let *k* be the minimum integer for which *r* is a divisor of *p*^*k* - 1. Such *k* is defined as the **embedding degree** of E (over *p*)

If you cannot really understand the 2 paragraphs above, **just remember that order and embedding degree are numbers, and generator is an EC point**. You can view the generator as "1" in the sense of multiplication (in that subgroup) in an EC.

#### Bilinear map / pairing

Let's first compare our current demand with the previous case of homomorphic encryption, where we need to find functions f and E so that we can verify f(E(*p*), E(*q*)) == E(*p+q*). We used a public constant g, to encrypt the raw value *v* to E(*v*)=*g*^*v* (mod *n*), and execute multiplication E(*v*)*g*^*a* when we need addition to the raw *v+a*. That is, we used f==multiplication, and E(*v*)=*g*^*v*, to verify that *f(E(v), E(a)) == E(v+a)*. Also, modular arithmetic prevents *v* being extracted, when E(v)==*g*^*v* is known.

And now, remember that we are going to multiply two encrypted values. We are going to use EC for multiplying two encrypted values. For an EC with 2 generator points G1 and G2, what we do is to:

- **Find a pairing function f such that f(pG1, qG2) == f(G2, pqG1) == f(pqG2, G1)**

The pairing function is the core technology that I am unable to describe within a few tons of pages. In theory, we use Weil pairing, Tate pairing, or optimal Ate pairing in the details for function f. Visit some pedantic papers if you are interested in them. And fortunately, **it is difficult to find p when you know pG**. 

In the actual world, we can find such an f in many cryptography programming libraries:

```python
# pip install py_ecc
from py_ecc.bn128 import G1, G2, pairing, multiply
P = multiply(G2, 2)
Q = multiply(G1, 3)
R = multiply(G2, 2 * 3)
assert pairing(P, Q) == pairing(R, G1)
```

In Ethereum there is a precompiled contract `ecPairing` which implements the pairing function on curve `alt_bn128`.

#### Non-interactive ZKP with EC

Recall that the verifier **has to remember 2 stateful values *e* (also known as \alpha) and the public polynomial *t(r)* until the proof is verified, without revealing *r* and *e* to the public.** *r* is a random value to evaluate the polynomial p(x)==s(x)t(x) with x==r, while *e* is used to exponentially shift the values, to form a tuple of "checksum" to prevent prover's cheating with arbitrary operations. Also the verifier should provide CRS to the public. In practice, we divide all the information above into 2 groups:

- Verification key: E(t(r)) and E(1)^e (remembered by the public for verification)
- Proving key (also known as evaluation key) (given to the prover to generate proof):
  - [*E(r), E(r^2), E(r^3), ..., E(r^*degree*(p))*]
  - and its exponentially shifted version [*E(r)^e, E(r^2)^e, ..., E(r^*degree*^e*] (excluding E(1)^e; it is stored publicly, so the prover still has access to it)

In the interactive version we used E(v) = g^v (mod n). But for multiplication of two encrypted values, we encrypt everything with EC, and modify the encryption function to E(v)=vG (mod p), where G is a generator of the EC. For [E(r), E(r^2), ...] (the 1st item of proving key) we use G=G1, and for the exponentially shifted [E(r)^e, E(r^2)^e, ...] (the 2nd item of proving key) we use G=G2. For verification key we always use G=G2. **G1 and G2 can be the same point in some cases.**

So the EC-version CRS keys are:

- Verification key: t(r)G2, eG2 (remembered by the public for verification)
- Proving key (also known as evaluation key) (given to the prover to generate proof):
  - [*rG1, (r^2)G1, (r^3)G1, ..., (r^*degree)G1]
  - [*erG2, e(r^2)G2, ..., e(r^*degree)G2] (excluding eG2; it is stored publicly, so the prover still has access to it)

Additionally, we use s'(r)=s(r)e. So the prover should return:

- p(r)G1 == s(r)t(r)G1
- s(r)G1
- s'(r)G2 == s(r)eG2
- No need for the prover to use his own exponent *d* (not an exponent but a multiplier here) for ZK, because we cannot find s(r) from s(r)G.
- If you do want to use your own *d*, just return p(r)dG1, s(r)dG1 and s'(r)dG2. You will see the verification step below that the verifier do not have to care if you have multiplied a *d*.

Instead of checking p(r)==t(r)s(r) and E(s'(r))==E(s(r))^e, now we need to check:

- f(p(r)G1, G2) == f(s(r)G1, t(r)G2)
- f(G1, s'(r)G2) == f(s(r)G1, eG2)

It does not matter whether the prover has used a private multiplier *d*. When *d* is used, we check:

- f(p(r)dG1, G2) == f(s(r)dG1, t(r)G2)
- f(G1, s'(r)dG2) == f(s(r)dG1, eG2)

The verifier does not have to know the value of *d* in order to verify the proof.

### Trusted party setup

We still need to solve the problem of trust, because the verifier and the prover can plot together for a cheat. **The critical problem is that the verifier should not tell *r* and *e* to the prover.** As a verifier delegated by the public, at least you need a computing machine that is honest and trusted by the public. Otherwise all the proofs are not trusted. In practice, we use multi-party computation (MPC), involving many computers that you may choose to trust or not. We try to build a system that, when at least one verifier is honest (trusted by you), the final result is valid (for you). So here is how multiple parties can generate a single number that is used for computation, and then forgotten, immediately and honestly, **if at least one of the parties forget it**.

The process is very simple. Each participant of the computation just provide his/her own random CRS ( [(r^i)G1], [e(r^i)G2] for *i* in [1, 2, ... degree]), and multiply it with the product of all CRS-es provided by others. If a single participant does not provide his/her *r* and *e* to the prover, the prover cannot find the product of all *r*-s and *e*-s, based on the EC points.

We now reaches the goal of **succinct non-interactive argument of knowledge (SNARK)**. Congratulations on being a pro of **zk-SNARK**.

### Representing a general practical problem with a polynomial

What ZKP does is to prove that a secret polynomial s(x) does have value s(r) when x==r. But in practice, we need to prove that we know a secret solution to a public problem, without revealing the secret solution. We are now going to discuss how to translate the problem and the solution to polynomials. 

#### Identifying the constraint of a public problem

Let's start with an old mathematical joke. What is the number after the array 1, 3, 5, 7, (?)

You may say the answer 9, but I think the answer is **114514**. This is because, if

- *s*(*x*)=(114505/24)*x*^4−(572525/12)*x*^3+(4007675/24)*x*^2−(2862601/12)*x*+114504

then s(1)==1, s(2)==3, s(3)==5, s(4)==7, s(5)==114514.

How did I came up with such a crazy s(x)? Actually I used Lagrange's interpolating polynomial, with

- *s*(*x*)=(*x*−2)(*x*−3)(*x*−4)(*x*−5)/((1−2)(1−3)(1−4)(1−5))+3(*x*−1)(*x*−3)(*x*−4)(*x*−5)/((2−1)(2−3)(2−4)(2−5))+5(*x*−1)(*x*−2)(*x*−4)(*x*−5)/((3−1)(3−2)(3−4)(3−5))+7(*x*−1)(*x*−2)(*x*−3)(*x*−5)/((4−1)(4−2)(4−3)(4−5))+114514(*x*−1)(*x*−2)(*x*−3)(*x*−4)/((5−1)(5−2)(5−3)(5−4))

Just unbracket everything, and you'll get the succinct yet crazy s(x). You can think about the original s(x) above to know how I **constrained** it with the array [1, 3, 5, 7, (?)]

For ZKP, the core of the problem here is not that s(5)==9 or s(5)==114514, but that:

- I know a secret polynomial s(x) such that s(1)==1, s(2)==3, s(3)==5, s(4)==7

In other words, the **constraint** of the problem is the 4 beginning items of the array. And the constraint is always public (otherwise nobody knows about the problem being solved). Then, using the ZKP process, I can prove that I know a secret number that follow the array 1, 3, 5, 7.

#### In programming: "proof of operation"

Ideally we want polynomials to express computer programming codes, so that we can prove that with certain (secret or public) inputs and (secret or public) operations, we reach a result that meets some constraints.

In a real computer program, we have operators and operands. They make up a computer function with basic operations like this:

- output = left_operand **operator** right_operand

In order to prove that the whole function is executed correctly, we just need to prove that **each operation is correct**. For example, consider a computer function that intakes `a`, `b` and `c` and returns `a*b*c`. This function involves 2 operations (where r1 and r2 are intermediate results):

- a*b=r1
- r1*c=r2

We view these two operations **non-interfering**, and our target is to prove with polynomials that **each of the 2 operations are computed correctly**. 

Generally, for a single basic computer operation, we try to map it into a polynomial equation like this:

- o(x) = l(x) **operator** r(x) (for **operator** in (+,-,*,/))

Note that modulo and comparison (>, <) are not directly supported, because we represent problems with polynomials, which cannot execute these operators. And we will find a polynomial

- p(x) = l(x) **operator** r(x) - o(x)

We'll directly use this p(x) for the ZKP p(x) = t(x)s(x) and prove that l(x) **operator** r(x) - o(x) = t(x)s(x) in later sections. In our example `a*b*c`, the operator is `*` (multiplication), which means p(x)=l(x)r(x)-o(x). 

Also we may use non-arithmetic operators like `if else` in computer programs. For example, 

```js
    function calculate(uint256 a, uint256 b, bool w) public pure returns (uint256) {
        if (w) {
            return a * b;
        } else {
            return a + b;
        }
    }
```

We translate the Solidity code above into `w(ab)+(1-w)(a+b)`, where w==1 for true, and w==0 for false. Then you can try to respectively prove the correct execution of ab, or a+b, etc. Additionally, we can require **w*w == w for w to be either 0 or 1**. **Polynomials are (almost) Turing complete to represent any computer operation.** 

#### R1CS: representing a computer function with a virtual circuit of gates, and then vectors

Here we are preparing another mathematical tool to finally express a computer function with polynomials. We'll use an example problem described by Vitalik Buterin in his blog: proving that you know the (secret) solution to the equation -*x*³+*x*+5==35. We break the operations into basic ones:

- x*x=var1
- var1*x=var2
- var2+x=var3
- var3+5=out

This is constructing a virtual circuit of 4 gates. Each gate has 2 inputs and 1 output. Now we represent all the variables related to the whole problem, in a solution vector S:

- S=[1, x, out, var1, var2, var3]

R1CS is **rank-1 constraint system**. It is a triplet of vectors a, b and c, as well as a solution vector S, representing a single operation in a single gate as **<a,S> * <b, S>=<c, S>**, where <\*, \*> is the dot product of two vectors (returning a number). Now let's represent a function with a polynomial using R1CS.

1 is always included in the vector S in order to represent all kinds of constants. Now we are going to construct R1CS vectors a1, b1 and c1 for first gate var1=x*x. a1 and b1 are responsible for the inputs (which are both x), and c1 is responsible for the output of the gate.

- a1=[0, 1, 0, 0, 0, 0]  (<a1,S> representing x)
- b1=[0, 1, 0, 0, 0, 0]  (<b1,S> representing x)
- c1=[0, 0, 0, 1, 0, 0]  (<c1,S> representing var1)

In this way we express the 1st gate x\*x=var1 with **<a1,S><b1, S>=<c1, S>**. Similarly, for the 2nd gate var1*x=var2:

- a2=[0, 0, 0, 1, 0, 0]
- b2=[0, 1, 0, 0, 0, 0]
- c2=[0, 0, 0, 0, 1, 0]

In the 3rd gate var2+x=var3, things are a bit different. We were forced to use multiplication of numbers in the expression <a,S> ***** <b, S>=<c, S>, so we are going to apply addition through the dot product of vectors:

- a3=[0, 1, 0, 0, 1, 0]  (<a3, S> representing var2+x)
- b3=[1, 0, 0, 0, 0, 0]  (<b3, S> representing 1)
- c3=[0, 0, 0, 0, 0, 1]

And similarly for the 4th gate:

- a4=[5, 0, 0, 0, 0, 1]
- b4=[1, 0, 0, 0, 0, 0]
- c4=[0, 0, 1, 0, 0, 0]

Great. But by the way, we still have more concise methods to express the 4 multiplications of vector dot products. We are using a **matrix** (plural form: matrices), which is a 2-D array of numbers, made up of vectors. 

```
A=[          B=[          C=[
	a1,          b1,          c1,
	a2,          b2,          c2,          
	a3,          b3,          c3,
	a4,          b4,          c4,
]            ]             ]
```

That is,

```
A=[                          B=[                          C=[
	[0, 1, 0, 0, 0, 0],          [0, 1, 0, 0, 0, 0],          [0, 0, 0, 1, 0, 0],
	[0, 0, 0, 1, 0, 0],          [0, 1, 0, 0, 0, 0],          [0, 0, 0, 0, 1, 0],
	[0, 1, 0, 0, 1, 0],          [1, 0, 0, 0, 0, 0],          [0, 0, 0, 0, 0, 1],
	[5, 0, 0, 0, 0, 1],          [1, 0, 0, 0, 0, 0],          [0, 0, 1, 0, 0, 0],
]                            ]                            ]
```

Now **consider these 2-D arrays vertically in columns**. For example, the 2nd column of A represents the role that x plays in the whole gate. In other words, x has coefficient 1 in the 1st gate, and has coefficient 1 in the 4th gate. 

#### Quadratic Arithmetic Program (QAP) form of R1CS: transforming vectors / matrices into polynomials



#### Interpolation: finding a polynomial that goes across given points

Additionally, we need systematic methods to form a polynomial from points that it intersects, instead of using some crazy come-ups. Pedantically, we have the following systematic methods to go (just choose **one of** them):

- Set of equations with unknowns (solve `d` equations for a polynomial across `d` points, with all coefficients of the polynomial unknown)
- Lagrange polynomial (described in the example of [1, 3, 5, 7, (?)])
- Newton polynomial
- Neville's algorithm
- FFT

FFT is crucial for practical systems to be fast enough. We are not going to explain any of them further. Help yourself if you are interested.