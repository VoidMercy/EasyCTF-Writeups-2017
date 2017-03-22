# 130 - RSA 3

## Overview

RSA 3 was a 130 point RSA challenge on EasyCTF 2017.  The only parameters that are provided are {*N,e,c*}.

## Objective

Factor an RSA modulus that is a product of twin primes.  

## Solution

We are given the following information:

    N = 0x27335d21ca51432fa000ddf9e81f630314a0ef2e35d81a839584c5a7356b94934630ebfc2ef9c55b111e8c373f2db66ca3be0c0818b1d4eda7d53c1bd0067f66a12897099b5e322d85a8da45b72b828813af23L
    e = 0x10001
    c = 0x9b9c138e0d473b6e6cf44acfa3becb358b91d0ba9bfb37bf11effcebf9e0fe4a86439e8217819c273ea5c1c5acfd70147533aa550aa70f2e07cc98be1a1b0ea36c0738d1c994c50b1bd633e3873fc0cb377e7L

The size of the modulus *N* renders classical factorization algorithms, including General Number Field Sieve (GNFS), impractical.  There must be another way to factorize *N*, and thankfully, there is.  We convert *N* into decimal, and find something suspicious: 

    >>> int("0x27335d21ca51432fa000ddf9e81f630314a0ef2e35d81a839584c5a7356b94934630ebfc2ef9c55b111e8c373f2db66ca3be0c0818b1d4eda7d53c1bd0067f66a12897099b5e322d85a8da45b72b828813af23",0)

    >>> 11721152358236061520231546547637479326600733856476942783391651261161073932522251528868410691769508664585409225554242334270429894938143275720784205110462278896087432260439082955593874915727686637956899
    
*N* ends in *99*, meaning that we may actually be able to solve for *p* and *q* via difference of squares: for some value *x*, (*x* - 1) * (*x* + 1) = *N*

We can solve for x algebraically:

*x*<sup>2</sup> - 1 = *N*

*x*<sup>2</sup> = *N* + 1

*x* = sqrt(*N* + 1)

Since Python floating point values lose precision after around 17 digits, I used the bigfloat module instead.

    from bigfloat import sqrt, precision
    import binascii
    N = 0x27335d21ca51432fa000ddf9e81f630314a0ef2e35d81a839584c5a7356b94934630ebfc2ef9c55b111e8c373f2db66ca3be0c0818b1d4eda7d53c1bd0067f66a12897099b5e322d85a8da45b72b828813af23L
    c = 0x9b9c138e0d473b6e6cf44acfa3becb358b91d0ba9bfb37bf11effcebf9e0fe4a86439e8217819c273ea5c1c5acfd70147533aa550aa70f2e07cc98be1a1b0ea36c0738d1c994c50b1bd633e3873fc0cb377e7L
    e = 65537
    x = int(sqrt(N+1,precision(1000))) #This gives us the value for *x*, and we can derive *p* and *q* by (*x* - 1) * (*x* + 1).
    with precision(1000):
        assert int(x**2) - 1 == N  #Assertion successful

    p, q = (x - 1), (x + 1)
    phi = (p - 1) * (q - 1)
    e = 65537
    
    def egcd(a, b): 
        if a == 0: 
            return (b, 0, 1) 
        else: 
            g, y, x = egcd(b % a, a) 
            return (g, x - (b // a) * y, y) 

    def modinv(a, m): 
        g, x, y = egcd(a, m) 
        if g != 1: 
            raise BaseException('Modular inverse does not exist.') 
        else: 
            return x % m
    d = modinv(e, phi)
    print(binascii.unhexlify(hex(pow(c,d,n))[2:-1]))
    
    >>> easyctf{tw0_v3ry_merrry_tw1n_pr1m35!!_417c0d}
    
And there we have it!

# Flag

easyctf{tw0_v3ry_merrry_tw1n_pr1m35!!_417c0d}
 
