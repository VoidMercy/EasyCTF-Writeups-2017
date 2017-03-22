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

The size of the modulus *N* renders classical factorization algorithms, including General Number Field Sieve (GNFS) impractical.  Realistically, however,  We convert *N* into decimal, and find something suspicious: 

    >>> int("0x27335d21ca51432fa000ddf9e81f630314a0ef2e35d81a839584c5a7356b94934630ebfc2ef9c55b111e8c373f2db66ca3be0c0818b1d4eda7d53c1bd0067f66a12897099b5e322d85a8da45b72b828813af23",0)

    >>> 11721152358236061520231546547637479326600733856476942783391651261161073932522251528868410691769508664585409225554242334270429894938143275720784205110462278896087432260439082955593874915727686637956899
    
*N* actually ends in *99*, meaning that we may actually be able to solve for *p* and *q* via difference of squares: (*p8*-1) * (*p*+1) = *N*

We can solve for p algebraically:

*p*<sup>2</sup> - 1 = *N*

*p*<sup>2</sup> = *N* + 1

*p* = sqrt(*N* + 1)

Since Python floating point values lose precision after around 17 digits, I used the bigfloat module instead.

    >>> from bigfloat import *
    >>> from math import *
    >>> p = sqrt(N+1,precision(340)) 
    >>> print(p)
    >>> BigFloat.exact('3423616853305296708261404925903697485956036650315221001507285374258954087994492532947084586412780870.0000', precision=340)   


This gives us the value for 
