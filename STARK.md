### Reed-Solomon (RS) Codes

In [SNARK.md](SNARK.md), we discussed R1CS that forms polynomials to represent general problems or computer functions, and some cryptographic tools to prove that you know the polynomial. Now in STARK, we are going to still encode the computer function into a polynomial, with Reed-Solomon codes. RS codes, utilizing polynomials (in a way quite different from those in SNARK), are a set of various encoding methods that can correct errors if there is some in the encoded result. 

The following sections can be very confusing. **You may skip most of them and just read the simple version using only Lagrange interpolation.**

#### Polynomials over Galois field (GF) (Optional)

Remember that in Galois field GF(n), we always play `mod n` after arithmetic operations, in order for the result to be a non-negative integer less than n. Now, to define a polynomial c0+c1x+c2x^2+...+c_d x^d over GF(n), we just require **the coefficients of the polynomial to be less than n**:

- c0, c1, c2, ..., c_d < n

And let's watch some amazing arithmetic phenomena in such polynomials. Consider it: **Is x^4+x+1 a factor of x^15+1, over GF(2)?** Unfortunately, ordinary calculators are unable to help you in the problem. But the answer is **YES**. Given that 1+1==0 in GF(2),

- x^15+1 == (x^4+x+1)(x^11+x^8+x^7+x^5+x^3+x^2+x+1)

In RS codes, we will record the coefficients of a polynomial over GF(a_large_prime_number).

#### Extension fields (Optional)

For GF(P), its extension fields are simply GF(P^m), where P is a prime number, and m = 2,3,4,...

Typically we are interested in P==2 in this material.

#### GF of polynomials instead of numbers; primitive polynomial of degree m over GF(P), as a generator of GF(P^m) (of polynomials) (Optional)

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

#### Isomorphic implementations of GF(P^m) of polynomials (Optional)

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

#### RS codes encoding (Optional)

A systematic RS codeword of length n includes k raw data bits, and n-k==2t **parity-check** bits, in order to recover errors no more than t bits. Given the raw message m(x) (of degree k-1), we generate the codeword c(x) with

- c(x)=g(x)m(x)  (over GF(2); do not mod p(x))

where g(x) is a publicly known generator polynomial. 

- g(x)=(x-a)(x-a^2)...(x-a^(2t)) (of degree 2t, with the coefficient of x^(2t) as 1)
- note that, for polynomials over GF(2), x-a is just x+a

a is a(x), a generator of GF(P^m) discussed before. In this way we require g(x) to have roots {a, a^2, ..., a^(2t)}. Just be aware that do not pick an a(x) making g(x)==0. With the coefficient of x^(2t) as 1 in g(x), we can copy the coefficients of m(x) to x^(n-1), x^(n-2), ..., x^(2t), making the codeword systematic. Now just transmit the (binary) coefficients of c(x) through an unreliable channel.

#### RS codes decoding (Euclidean division algorithm) (Optional)

Now it's time to decode the codeword r(x) after receiving it from an unreliable channel. We introduce a random error polynomial e(x) of degree n over GF(2), with no more than t coefficients being 1:

- r(x)=c(x)+e(x)

Then we compute r(x) mod g(x), in order to find e(x). Because c(x)==g(x)m(x), surely We have

- (c(x)+e(x)) mod g(x) == e(x) mod g(x) (of degree 2t)

Astonishingly, no practical method was given in the original paper of RS codes to recover e(x). We are using **syndrome decoding**, introduced by later papers from others, to get e(x). The word "syndrome" means a group of signs and symptoms that occur together and characterize a particular abnormality.

Assume that in a binary (15,9) RS coding, we have 15 bits in a word with 9 bits of raw information and 6 bits for parity check, which can recover t=3 bits of errors.

- g(x)=(x-a)(x-a^2)...(x-a^6)=x^6+a^10x^5+a^14x^4+a^4x^3+a^6x^2+a^9x+a^6

Additionally, let's use 

- m(x)=a^11 x

Then we have c(x) and an assumed e(x) along with r(x)

- c(x)=a^11x^7+a^8x^5+a^10x^4+a^4x^3+a^14x^2+a^8x+a^12
- e(x)=x^8+x^2

- r(x)=**x^8**+a^11x^7+a^8x^5+a^10x^4+a^4x^3+**a^3x^2**+a^8x+a^12

Be aware that for the coefficient of x^2, we do have a^14+1==a^3 (watch the construction of GF(P^m) to check it). Now let's compute the **syndrome components, defined as S_i=r(x=a^i), for i in {1, 2, ..., 2t==6}**. Remember that r(x) should have roots {a, a^2, ..., a^2t}. If there were no error in r(x), we should have S_i==0 for all i. But actually,

- S1=r(a)==1
- S2=r(a^2)==1
- S3=r(a^3)==a^5
- S4=r(a^4)==1
- S5=r(a^5)==0
- S6=r(a^6)==a^10

Now we define the **syndrome polynomial**:

- S(x)=S1+S2x+...+S_{2t}x^(2t-1)=a^10x5+x^3+a^5x^2+x+1

Then we are going to find an **error locator polynomial s(x)** (often \sigma (x) or \Lambda (x) in other materials). Divide x^(2t) by S(x), then S(x) by the 1st remainder r1(x), then r1(x) by r2(x), ..., until the degree of r_i <= t.

Divide x^(2t) by S(x) to get a quotient q(x) and the remainder r1(x)

- x^6 = (a^5 x)s(x) + (a^5x^4+a^10x^3+a^5x^2+a^5x); q1(x)=a^5x, r1(x)=a^5x^4+a^10x^3+a^5x^2+a^5x

The degree of r1(x) is 4, greater than t=3. In this case we need to keep dividing S(x) by r1(x)

- S(x)=(a^5x+a^10)r1 + 1; q2(x)=a^5x+a^10, r2(x)=1

The degree of r2(x) is 0, less than t=3. Stop here. Now put the previous results into the following form:

- S(x)s(x)=A(x)+x^(2t)B(x)

Given that

- r1(x)=x^6+q1(x)S(x)
- r2(x)=S(x)+q2(x)r1(x)

We can combine the equations about remainders to form a single equation

- r2(x)=x^6q2(x)+S(x)(1+q1(x)q2(x))

Equivalently

- 1=x^6(a^5x+a^10)+S(x)(a^10x^2+x+1)

Further, rewrite the equation in the form of S(x)s(x)=A(x)+x^(2t)B(x):

- S(x)(a^10x^2+x+1)=1+x^6(a^5x+a^10)

This means that the error locator polynomial is

-  s(x)=a^10x^2+x+1

Now we try to find the roots of s(x). The answer is simply:

- s(x)=(a^2x-1)(a^8x-1)

We actually employ **Chien search** to find the roots. This is exactly a brute-force algorithm that I am not going to explain in detail. You can just try all of a^0, a^1, ..., a^(2t-2) if you do not want to learn about it. The roots of s(x) indicates that error occurs in x^8 and x^2 items of r(x).

Finally we solve a set of degree-1 equations to find the error values.

- e1a^2+e2a^8=S1
- e1(a^2)^2+e2(a^8)^2=S2

The solution is e1=1 and e2=1, and e(x) assumed by us is x^8+x^2. 

#### RS code using symbol erasure

Let's assume an intelligent signal receiver that can identify whether the symbols in a received codeword are reliable enough. When a symbol is probably unreliable, the receiver erases the symbol (leaving it blank) and leave the problem to the decoder. This is actually marking the errors in a codeword. With symbol erasure (assuming no error in non-erased symbols), we can let RS code work even when 2*t* error+erasure occur. I am not going to explain the details any more, because it is quite away from our main topic STARK.

#### Domain extension: very simple version of RS erasure code using only Lagrange interpolation

Now you are going to transmit an array [4, 5, 3], but the transmission channel may introduce random erasure (but no error). What if we need our code to correct 1 erasure?

Remember that with Lagrange interpolation, we can generate a polynomial

- p(x)=-1.5x^2+2.5x+4

with

- p(0)==4
- p(1)==5
- p(2)==3

Now that

- p(3)==-2

We can actually send the array [4,5,3,-2], and ask the receiver to run Lagrange interpolation again. Evidently, any single erasure does not prevent us to recover p(x) with only 3 points. 

Generally, when you want to send *k* symbols that are resistant to 2*t* erasures, you can always use Lagrange interpolation on the k symbols to get a polynomial p(x) of degree k-1, then evaluate additional 2t values of the polynomial, and send all the k symbols along with the 2t additional values. As long as there are no more than 2t erasures, the receiver can recover p(x) and then the whole message.

### Arithmetization

Let's play with a simple example: Alice wants to prove it to Bob that she has 10^6 integers within range 0, 1, 2, ..., 9. It's easy to form the constraints of the problem:

- for 1<=i<=10^6, A_i(A_i-1)(A_i-2)...(A_i-9)==0

Then, Alice finds a Lagrange interpolation P(x) for her 10^**6** integers, where x ranges from 1 to 10^**9**. Yes! Note that x in P(x) ranges from 1 to 10^**9**, instead of 10^**6**! Now surely,

- for 1<=x<=10^**6**, P(x)(P(x)-1)...(P(x)-9)==0

Let's name Q(P(x))=P(x)(P(x)-1)...(P(x)-9). Then surely,

- degree(Q(P(x)))==10^7  (We can always compute the degree of Q(x), based on the constraints)
- Q(P(1))==Q(P(2))==...==Q(P(10^6))==0

Now let's name a public polynomial T(x)=(x-1)(x-2)...(x-10^6). Because P(x) is a Lagrange interpolation on [1, 10^6], Q(P(x)) has roots 1, 2, ..., 10^6. Therefore, Alice can convert the problem to another form: she wants to prove that she knows a secret polynomial S(x), such that

- for 1<=x<=10^**9**, Q(P(x))=S(x)T(x)
- degree(S(x))==10^7-10^6

Then it's simple for Bob to verify the claim.

**In practice, we often use FFT instead of Lagrange interpolation, because FFT takes O(nlogn) time, while Lagrange interpolation takes O(n^2).**

#### Degree adjustment

For the sake of computational convenience, we want the degrees of each constraint to be (a power of 2) - 1 (e.g. 511, 1023, 2^32 - 1, ...). we multiply each original constraint polynomial by (ax^n+b), where a and b are random constants chosen by the verifier for each constraint, and n is a proper number to make the degree be (a power of 2) - 1. In this tutorial we can forget degree adjustment. 

### Interactive zk-STARK

Prover preparation:

- for x in range [1, 10^**9**], evaluate P(x) and S(x)
- Build a merkle tree
  - The leaf nodes are P(x) and S(x). The initial non-leaf nodes are hash(P(x), S(x)) for x in [1, 10^**9**]
  - We, in practice, actually pick a number of power of 2, instead of 10^9 in this example

Verifier:

- pick a random value *r* (in range (10^**6**, 10^**9**])
  - Do not pick any *r* in [1, 10^6]. This reveals the original value of P(x), breaking the ZK requirement

- send *r* to the prover

Prover:

- return P(r), S(r), root of the merkle tree, and the path (all the nodes of hashes along the way) to reach the root of the merkle tree, starting from hash(P(r), S(r))

Verifier:

- Check whether we can get the correct root through the merkle tree
- Check whether Q(P(r))==S(r)T(r)

#### Soundness?

What if Alice is not honest, where she uses a Q(P(x))!=S(x)T(x)? Given that degree(Q(P(x)))==10^7, there are at most 10^7 intersections between Q(P(x)) and S(x)T(x). For a randomly picked x==r in range (10^6, 10^9], the probability for Q(P(r))==S(r)T(r) is at most (10^7)/(10^9)==0.01. The verifier can repeat with a few different *r* to quickly reduce the probability of prover's cheating.

But there is still a method for the prover to cheat. Because we did not actually limit the degree of P(x), the prover can choose a secret P(x) with degree > 10^6. Then the prover can interpolate anything that satisfies Q(P(x))==S(x)T(x). We are going to use low degree testing to limit degree(P(x))<=10^6.

### Low degree testing (LDT)

#### Naive version: direct test

In this version we ensure P(x) to have degree <=d with d+1 queries

Verifier:

- Pick random values a1, a2, ..., a_10^6 and send them to the prover

Prover:

- return P(a1), P(a2), ..., P(a_10^6)

Verifier:

- Find the Lagrange interpolation of P(a1), P(a2), ..., P(a_10^6)
- Pick a random value b and send it to the prover

Prover:

- return P(b)

Verifier:

- Check whether P(b) is correct

Well, this is simply asking the prover to help recover the whole information about P(x), which is neither zk nor efficient. Now we are going to introduce the most magnificent tool in zk-STARK to run LDT with O(log(d)) queries.

#### Divide and conquer

We split a polynomial into even-degree and odd-degree terms. For example, with an f(x) to be run LDT on,

- f(x)=a0+a1x+a2x^2+a3x^3+a4x^4+a5x^5

we can derive two polynomials

- g(x)=a0+a2x+a4x^2
- h(x)=a1+a3x+a5x^2

Then we can write

- f(x)=g(x^2)+xh(x^2)

where degree(g(x)), degree(h(x)) <= degree(f(x))/2. Within log(d) steps, we can reach results of constant polynomials. Also, we need some methods to hide the original f(x) and achieve zk. Therefore, we finally form a protocol called FRI.

#### Fast RS Interactive Oracle Proofs (IOP) of Proximity (IOPP)  (FRI)

Commit phase: (the word "commit" originates from polynomial commitment)

- Prover
  - Split the initial f0(x) into even- and odd-degree terms g0(x) and h0(x), such that
    - f0(x)=g0(x^2)+xh0(x^2)
- Verifier
  - pick a random value a0, send it to the prover
- Prover
  - compute f1(x)=g0(x)+a0h0(x), of degree <= degree(f0(x))/2
- Repeat the steps above, until a constant polynomial is obtained. Do not forget any f(x), g(x) and h(x).
- Prover
  - send the constant value to the verifier

Query phase (we want to ensure that the prover did not cheat):

- Verifier:
  - pick a random value z and send it to the prover
- Prover:
  - return f0(z), f0(-z) and f1(z^2)
- Verifier
  - Compute g0(z^2) and h0(z^2), by solving 2 equations below:
    - f0(z)=g0(z^2)+zh0(z^2)
    - f0(-z)=g0(z^2)-zh0(z^2)
  - Compute f1(z^2)=g0(z^2)+a0h0(z^2). Check whether it equals the f1(z^2) returned by the prover.
  - Repeat all the steps above for all f(z^2). Check whether the final constant is correct.

FRI seems to be solid, but in its original paper, there is a rigorous proof that the verifier can identify a malicious prover at a probability of only 10%. This is because a practical polynomial usually has too many zero coefficients, and the prover often reach a final zero polynomial with the random values picked. We have to repeat FRI for many (hundreds of) times to reduce the probability to practically zero. 

### Non-interactive zk-STARK

Armed with the experience of achieving non-interactive zk-SNARK, we should now understand that we need to eliminate all subjective parameters offered by the verifier. We use Fiat-Shamir transformation for a non-interactive zk-STARK. I will not explain the details, but the idea is simple: hash all the public parameters about the problem to get the random value. Typically we use Rescue or MiMC as the hash function in practice.

- In interactive zk-STARK, we hash the root of the merkle tree to replace the random number chosen by the verifier.
- In FRI, we can hash(previous_hash_result, values_returned_by_prover_if_there_is_any)

