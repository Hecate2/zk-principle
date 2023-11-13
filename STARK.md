### Reed-Solomon (RS) Codes

In [SNARK.md](SNARK.md), we discussed R1CS that forms polynomials to represent general problems or computer functions. Now in STARK, we are going to alternatively encode the computer function into a polynomial, with Reed-Solomon codes. RS codes, utilizing polynomials, are a set of various encoding methods that can correct errors if there is some in the encoded result. 

#### Polynomials over Galois field

Remember that in Galois field GF(n), we always play `mod n` after arithmetic operations, in order for the result to be a non-negative integer less than n. Now, to define a polynomial c0+c1x+c2x^2+...+c_d x^d over GF(n), we just require **the coefficients of the polynomial to be less than n**:

- c0, c1, c2, ..., c_d < n

And let's watch some amazing arithmetic phenomena in such polynomials. Consider it: **Is x^4+x+1 a factor of x^15+1, over GF(2)?** Unfortunately, ordinary calculators are unable to help you in the problem. But the answer is **YES**. Given that 1+1==0 in GF(2),

- x^15+1 == (x^4+x+1)(x^11+x^8+x^7+x^5+x^3+x^2+x+1)

In RS codes, we will record the coefficients of a polynomial over GF(a_large_prime_number).

#### Block codes; coding, error detection and correction

In general communication, we use coding to represent raw information. Typically, we transmit the encoded symbols in a channel to send them to the decoder, and decode the symbols to get the raw information. However, in the transmission channel of the encoded symbols, there can be errors that tamper with the correct codes (due to malicious attacks or natural unreliability). 

Consider a channel that may randomly flip the bits sent through it. If I want to send a message representing a random raw word in the set {"on", "off"}, I cannot just send **binary codes**: 1 for "on", and 0 for "off". They can neither detect not correct errors introduced from the channel. Instead, let's try to (naively) send a block of 3-bit codes [101] for "on", and [010] for "off", which is as different as possible from [101]. The **distance** of the two words is d[101, 010] == 3, because they are different in all the 3 bits. 

Now let's try to decode the naive symbols sent through a noisy or malicious channel. The decoder may receive anything from the set {000, 001, 010, 011, 100, 101, 110, 111}. For [101] received, we can decode it to "on", and for {100, 001, 111}, we may still decode it to "on", because, using the idea of maximum likelihood distance, these codes has a smaller distance from [101] than that of [010]. Our naive block codes made a successful correction. 

But wait a moment! Is it still possible that an input [010] was terribly flipped to [101]? Our naive block code fails on such cases, because It can neither detect, nor correct errors like this. We are going to introduce non-trivial methods to detect and correct all kinds of errors with very large probability:

- Reed-Solomon codes are non-binary, Bose–Chaudhuri–Hocquenghem (BCH), cyclic, linear block, error correction codes.

Do not worry about that many crazy adjunct phrases. They are somehow inclusive, meaning that one phrase may be just a category of another. I am categorizing these phrases in the following tree:

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

#### Linear block codes

- All code words of linear block codes are sum of code words.
- If the code is systematic, then the code words must include the codes of the raw information.

#### Cyclic codes

- Are linear block codes
- Every cyclic shift of a code word is also a code word.

#### q-ary and P-ary codes

When we talk about q-ary codes (replacing q with a specific integer), we mean that each bit can be filled with a non-negative integer less than q. For example, binary codes are 2-ary codes, and we can fill each "bit" with any number from {0,1,2,3} in 4-ary codes. 

But when we talk about P-ary codes (without replacing P), we are emphasizing prime-number-ary codes. Binary codes are the simplest cases of P-ary codes.

#### BCH codes

