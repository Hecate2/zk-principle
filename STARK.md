### Reed-Solomon (RS) Codes

In [SNARK.md](SNARK.md), we discussed R1CS that forms polynomials to represent general problems or computer functions. Now in STARK, we are going to alternatively encode the computer function into a polynomial, with Reed-Solomon codes. RS codes, utilizing polynomials (in a way quite different from those in SNARK), are a set of various encoding methods that can correct errors if there is some in the encoded result. 

#### Polynomials over Galois field (GF)

Remember that in Galois field GF(n), we always play `mod n` after arithmetic operations, in order for the result to be a non-negative integer less than n. Now, to define a polynomial c0+c1x+c2x^2+...+c_d x^d over GF(n), we just require **the coefficients of the polynomial to be less than n**:

- c0, c1, c2, ..., c_d < n

And let's watch some amazing arithmetic phenomena in such polynomials. Consider it: **Is x^4+x+1 a factor of x^15+1, over GF(2)?** Unfortunately, ordinary calculators are unable to help you in the problem. But the answer is **YES**. Given that 1+1==0 in GF(2),

- x^15+1 == (x^4+x+1)(x^11+x^8+x^7+x^5+x^3+x^2+x+1)

In RS codes, we will record the coefficients of a polynomial over GF(a_large_prime_number).

#### Extension fields

For GF(P), its extension fields are simply GF(P^m), where P is a prime number, and m = 2,3,4,...

Typically we are interested in P==2 in this material.

#### GF of polynomials instead of numbers; primitive polynomial of degree m over GF(P), as a generator of GF(P^m) (of polynomials)

In [SNARK.md](SNARK.md) we learned to play `mod n` in a GF of integers. Also, remember the generator on an elliptic curve (EC), which can generate many points (sometimes all of the points) on the curve by adding itself, and finally return to itself. This process generates a field of EC points. Similarly, a primitive polynomial can serve as the modulus of a self-multiplying element, helping construct P^m unique elements of polynomials. We can also play `mod a_polynomial` in a GF of polynomials.

A primitive polynomial p(x) is an irreducible polynomial of degree m over GF(P), which divides (x^(P^m -1))+1, but does not divide x^i +1 for i < P^m -1. Here we pick P=2. If you feel confused, you can view the primitive polynomial as a "prime modulus" in the world of integers. Just follow my steps:

First we test whether p(x)=x^4+x+1 is a primitive polynomial of degree m=4 over GF(P=2). We shall confirm that p(x) is irreducible. Given that f(x=a)==0 if f(x) has factor (x-a), we verify the factors of degree 1 in the following way:

- f(0) == 0+0+1 == 1 != 0 . Therefore, (x-0) is not a factor of p(x). 
- f(1) == 1 != 0 . (x-1) is not a factor

We have finished testing all possible factors of degree 1, which are (x-1) and (x-0). Then we continue with all possible factors of degree 2:

- x^2==(x+0)^2 . x^2 is not a factor because (x+0) is not a factor
- x^2+1==(x+1)^2 . x^2+1 is not a factor because (x+1) is not a factor.
- x^2+x==(x+0)(x+1), not a factor because neither (x+0) nor (x+1) is a factor.
- (x^4+x+1)/(x^2+x+1) == x^2+x+1/(x^2+x+1) . Therefore, (x^2+x+1) is not a factor.

For degree-3 factors, we know that we failed to find a factor of degree 1. Therefore, no factor of degree 3 can exist. We conclude that p(x)=x^4+x+1 is irreducible.

Then we need to show that p(x) (of degree m=4) divides x^(2^m-1)+1==x^15+1. This had been shown in a previous subsection. Finally, we omit the steps to verify that p(x) does not divide x^i+1 for i<15, and directly declare that p(x) is a primitive polynomial of degree m=4. 

Now let's generate GF(2^m)==GF(16) with p(x). Notice that the primitive polynomial serve as the modulus, instead of the "generator". For the generator a(x) (usually written as \alpha (x) and called "primitive element" in other materials), we simply pick a(x)=x. We generate the field by inserting 0 and a^0 (which is `1`), and multiplying a(x) with itself: 

- {0, a^0, a^1, a^2, a^3, a^4, ...} == {0, 1, x, x^2, x^3, x^4, ...}

Wait one! Each element should `mod p(x)` before they are inserted. Therefore, we insert `x^4 mod x^4 +x+1 == x+1`, instead of x^4 itself.

- {0, 1, x, x^2, x^3, x+1, x^2+x, x^3+x^2, x^3+x+1, x^2+1, x^3+x, x^2+x+1, x^3+x^2+x, x^3+x^2+x+1, x^3+x^2+1, x^3+1}

Stop here! We got `a^14 mod p(x) ==x^3+1`. If we compute `a^15 mod p(x)`, we get `1`, and the generation process returns to the beginning. And you can check that our set has included all polynomials over GF(2) with degree <= 3.

#### Isomorphic implementations of GF(P^m) of polynomials

In the previous subsection we chose a(x)=x. Yet we could choose another a(x) over GF(2) with degree <=3, since the generation process is ultimately circular. This is similar to choosing another generator point on an EC. For example, if we choose a(x)=x^2, the polynomial set of GF(2^m) becomes:

- {0, 1, x^2, x+1, x^3+x^2, x^2+1, x^2+x+1, x^3+x^2+x+1, x^3+1, x, x^3, x^2+x, x^3+x+1, x^3+x, x^3+x^2+x, x^3+x^2+1}

The new isomorphic GF(P^m) includes still the same elements, but in a different order.

#### Block codes; coding, error detection and correction

In general communication, we use coding to represent raw information. Typically, we transmit the encoded symbols in a channel to send them to the decoder, and decode the symbols to get the raw information. However, in the transmission channel of the encoded symbols, there can be errors that tamper with the correct codes (due to malicious attacks or natural unreliability). 

Consider a channel that may randomly flip the bits sent through it. If I want to send a message representing a random raw word in the set {"on", "off"}, I cannot just send **binary codes**: 1 for "on", and 0 for "off". They can **neither detect not correct errors** introduced from the channel. Instead, let's try to (naively) send a block of 3-bit codes [101] for "on", and [010] for "off", which is as different as possible from [101]. The **distance** of the two words is d[101, 010] == 3, because they are different in all the 3 bits. This is a perfect **block (3,1) code**, where 3 bits are used for a single codeword, while only 1 bit is actually needed.

Now let's try to decode the naive symbols sent through a noisy or malicious channel. The decoder may receive anything from the set {000, 001, 010, 011, 100, 101, 110, 111}. For [101] received, we can decode it to "on", and for {100, 001, 111}, we may still decode it to "on", because, using the idea of maximum likelihood distance, these codes has a smaller distance from [101] than that of [010]. Our naive block codes made a successful correction. 

But wait a moment! Is it still possible that an input [010] was terribly flipped to [101]? Our naive block (3,1) code fails on such cases, because It can neither detect, nor correct errors where more than 1 bit had been flipped. We are going to introduce non-trivial methods to detect and correct all kinds of errors with very large probability:

- Reed-Solomon codes are non-binary, Bose–Chaudhuri–Hocquenghem (BCH), cyclic, linear block, error correction codes.

**(OPTIONAL)** Do not worry about that many crazy adjunct phrases. They are somehow inclusive, meaning that one phrase may be just a category of another. I am categorizing these phrases in the following tree:

- block error correction codes
  - our naive version of block codes
  - linear block codes
    - cyclic codes
      - primitive, q-ary BCH codes
        - non-binary BCH codes
          - RS codes
            - RS codes over GF(2^m)
      - non-primitive, q-ary BCH codes
        - non-binary BCH codes
          - RS codes
            - RS codes over GF(2^m)
- tree error correction codes **(not our focus in this material)**
  - convolutional codes

#### Linear block codes (OPTIONAL)

- Sum of code words are still valid code words.
  - Consider that we can use the coefficients of polynomials in GF(P^m) to encode things.
  - Sum of polynomials in GF(P^m) (mod p(x)) are still in GF(P^m)
- If the code is defined "**systematic**", the code words must include the codes of the raw information.

#### Cyclic codes (OPTIONAL)

- Are linear block codes
- Every cyclic shift of a code word is also a valid code word.
  - Consider that we can shift the coefficients of polynomials in GF(P^m)

#### q-ary and P-ary codes (OPTIONAL)

When we talk about q-ary codes (**replacing q with a specific integer**), we mean that each bit can be filled with a non-negative integer less than q. For example, **binary codes are 2-ary codes (q==2)**, and we can fill each "bit" with any number from {0,1,2,3} in 4-ary codes. 

But when we talk about P-ary codes (without replacing P), we are emphasizing prime-number-ary codes. Binary codes are the simplest cases of P-ary codes.

#### RS codes encoding

A systematic RS codeword of length n includes k raw data bits, and n-k==2t parity bits, in order to recover errors no more than t bits. Given the raw message m(x) (of degree k-1), we generate the codeword c(x) with

- c(x)=g(x)m(x)  (over GF(2); do not mod p(x))

where g(x) is a publicly known generator polynomial

- g(x)=(x-a)(x-a^2)...(x-a^(2t)) (of degree 2t, with the coefficient of x^(2t) as 1)

a(x) is a generator of GF(P^m) discussed before. Just be aware that do not pick an a(x) making g(x)==0. We create such g(x) in order to let g(x) have roots a, a^2, ..., a^(2t). With the coefficient of x^(2t) as 1 in g(x), we can copy the coefficients of m(x) to x^(n-1), x^(n-2), ..., x^(2t), making the codeword systematic. Now just transmit the (binary) coefficients of c(x) through an unreliable channel.

#### RS codes decoding

Now it's time to decode the codeword r(x) after receiving it from an unreliable channel. We introduce a random error polynomial e(x) of degree n over GF(2), with no more than t coefficients being 1:

- r(x)=c(x)+e(x)

Then we compute r(x) mod g(x), in order to find e(x). Because c(x)==g(x)m(x), surely We have

- (c(x)+e(x)) mod g(x) == e(x) mod g(x) (of degree 2t)

Astonishingly, no practical method was given in the original paper of RS codes to recover e(x). We are using syndrome decoding, introduced by later papers from others, to get e(x).

