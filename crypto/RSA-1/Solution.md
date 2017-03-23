# 50 - RSA 1
#### Writeup by Sanguinius (pwang00)
## Overview
RSA 1 was the first RSA challenge on EasyCTF.  All parameters were provided (Public key, exponent, 
primes).
## Objective
Perform normal RSA decryption using the provided paramters.
## Solution
We are given an text file with the following parameters:
    
    p: 33132682577565126969140002129633649554751216806378793848788711477877065876428791
    q: 30499295022924348240291300676281238378975669046700846004703307670180486189350193
    e: 65537
    c: 660474724815540720613531868092397691552741064511546824568120281740415543775213104869225380836741678627218552010355976744867444511891060921755575057758398772339

The RSA decryption process is as follows:

1. Compute N = p*q
2. Compute phi(N). phi = (*p*-1) * (*q*-1)
3. Calculate the modular multiplicative inverse of *e* and phi(N), which gives us the value for *d*
4. Perform modular exponentiation with the parameters c, d, N:  m = c<sup>d</sup> mod N
5. Convert the resulting number to hex and then ASCII.

Here is a script to calculate *m* from the given parameters and decrypt the flag:
    
    import binascii
    p = 33132682577565126969140002129633649554751216806378793848788711477877065876428791
    q = 30499295022924348240291300676281238378975669046700846004703307670180486189350193
    e = 65537
    c = 660474724815540720613531868092397691552741064511546824568120281740415543775213104869225380836741678627218552010355976744867444511891060921755575057758398772339
    N = p * q
    phi = (p - 1) * (q - !)
    def egcd(a, b): 
        if a == 0: 
            return (b, 0, 1) 
        else: 
            g, y, x = egcd(b % a, a) 
            return (g, x - (b // a) * y, y) 

    def modinv(a, m): 
        g, x, y = egcd(a, m) 
        if g != 1: 
            raise 'Modular inverse does not exist.' 
        else: 
            return x % m
    d = modinv(e,phi)
    print(binascii.unhexlify(hex(pow(c,d,N))[2:]))
    
    >>> easyctf{wh3n_y0u_h4ve_p&q_RSA_iz_ez_8b1f4c9a}

# Flag

easyctf{wh3n_y0u_h4ve_p&q_RSA_iz_ez_8b1f4c9a}
