---
title: 'Fast Sum of Two Squares Algorithm'
date: 2021-01-23
tags: ["Gaussian integers", "number theory", "algorithms"]
categories: ["number theory", "algorithms"]
katex: true 
draft: false
---

In this post I show how writing primes as the sum of two squares is related to factoring Gaussian integers. I then describe an algorithm to compute the sum of two squares representation.

## Gaussian Integers

I have a special penchant for Gaussian integers because they are the first ring I learned about (besides the regular integers of course).
Mathematically, they are the set of complex numbers $a+bi$ with $a,b$ integers, and they are denoted as $\mathbb{Z}[i]$.

We can add, subtract, and multiply Gaussian integers making it a structure called a **ring**.
However, $\mathbb{Z}[i]$ is a very special type of ring because it has a norm


If we have two Gaussian integers $z_1$ and $z_2 \ne 0$, we can divide $z_1$ by $z_2$

$$z_1 = q z_2 + r$$

where $q,r \in \mathbb{Z}[i]$ and $N(r) < N(z_2)$. That means the Euclidean algorithm works the same as in the
regular integers! This type of ring is called a [Euclidean domain](https://en.wikipedia.org/wiki/Euclidean_domain).

If the remainder $r = 0$, then we say $z_1$ is divisible by $z_2$. A **Gaussian prime** is a Gaussian integer that
has no divisors except for itself, and the units $\pm 1, \pm i$.
It is a theorem in algebra that every Euclidean domain has unique factorization, that is, 
we can factor every Gaussian integer into Gaussian primes in a unique way (up to multiplication by $\pm 1, \pm i$).

Plotting the Guassian primes on the complex plane makes quite a pretty pattern.

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/8/85/Gaussian_primes.png#center" >}}

One thing you might wonder is how the familiar integer primes $2,3,5,7,\dots$ are related to
the Gaussian primes. For example $2 = (1+i)(1-i)$ so it is not a Guassian prime!
It turns out we can describe the relationship very simply:

**Theorem**: 
Let $p>2$ be an integer prime. 
Then $p$ is a Gaussian prime if and only if $p \equiv 3 \pmod{4}$.

This is closely related to the problem of representing an integer as the sum of two squares,
since if we can factor $p = (a+bi)(a-bi)$ then $p = a^2 +b^2$. Moreover, if $q = (c+di)(c-di)$
is also the sum of two squares, then so is $pq$ since 

$$pq = ((ac-bd)+i(ad+bc))((ac-bd)-i(ac+bd))$$

From this we can determine all integers that can be represented as a sum of two squares
by looking at the primes in its prime factorization (in regular integers).
For example, $15 = 3\cdot 5$ is not the sum of squares because we can't factor the $3$ in Gaussian integers.

### Finding Sums of Squares

How can we efficiently factor primes in the Gaussian integers? Here is one 
very fast way due to Serret and Hermite. Let $p = 4k+1$ be prime.

1. Find a quadratic non-residue c mod p
2. Let x = c<sup>(p-1)/4</sup>  mod p so that x<sup>2</sup> = -1 mod p (this follows from [Euler's criterion](https://en.wikipedia.org/wiki/Euler%27s_criterion) )
3. Use the Euclidean algorithm on p and x until we get two remainders s,r < √p

Then, magically $p = r^2+s^2$.

The idea of the algorithm is to use a symmetry in the Euclidean algorithm.
In the Euclidean algorithm, we have a sequence of remainders $r_0 = p, r_1 = x, r_2, \dots, r_n$ that end with the 
greatest common divisor $r_n = \gcd(p, x) = 1$. We compute these recursively:

$$r_{i-1} = q_{i}r_{i}+ r_{i+1}$$

where $q_i = \lfloor r_{i-1}/r_{i} \rfloor$ is the quotient. We can define another sequence $(t_i)$ by the 
same recurrence, but with initial values $t_0 = 0$, $t_1 = 1$. It turns out that this sequence is just the 
reverse of the sequence $(r_i)$, up to signs. Moreover, one can see using the recurrence that

$$t_i x \equiv r_i \pmod{p}$$

for all i. This is the key equation, because we can square this equation and use $x^2 \equiv -1 \pmod{p}$ to get 

$$t_i^2 + r_i^2 \equiv 0 \pmod{p}$$

From here we just need to find the $t_i$ and $r_i$ that are the right size. I encourage you to work out the details.
If you get stuck, most of the proof is in [this article](https://www.jstor.org/stable/2323912).

Assuming the [Generalized Riemann Hypothesis](https://en.wikipedia.org/wiki/Generalized_Riemann_hypothesis#Extended_Riemann_hypothesis_(ERH)) is true,
we only have to check $O( \log^2(p))$ numbers to find the non-residue $x$. Empirically, we rarely have to search more than $\log(p)$.
The Euclidean algorithm is the faster part and is around $O(\log(p))$.

## Implementation

As an example, let's sieve a bunch of primes then find their sum of two squares representation.
To find c we loop through the primes q that we found.  
Since $p \equiv 1 \pmod{4}$, [quadratic reciprocity](https://en.wikipedia.org/wiki/Quadratic_reciprocity) implies that
$q$ is a quadratic residue mod $p$ if and only if $p$ is a quadratic residue mod $q$, which is much easier to check. 
We can do this using Euler's criterion.

```cpp
int non_res( int p, const vector<int> &primes){
    //find a quadratic non-residue mod p
    int q;
    for (auto it = primes.begin(); it != primes.end(); ++it){
        q = *it;
        if (q == 2){ 
            //2 is a quadratic residue iff p = 1 or -1 (mod 8)
            if (p%8 != 7 && p%8 != 1)  
                return q;
        }else if (q == 3){ 
            //3 is a quadratic residue iff p = 1 (mod 3)
            if (p%3 == 2)
                return q;
        }else if (p != q){ 
            //use quadratic recprocity and Euler's criterion
            if (mod_pow( p%q, (q-1)/2, q) != 1)
                return q%p; 
        }   
    }   
}
```

I checked 2 and 3 separately but it's not necessary.

After that, we just need a function to do the Euclidean algorithm part. 
(mod_pow is just fast mod p exponentiation)

```cpp
pair<int, int> gauss_factor(int p, vector<int> &primes){
    //factor prime p = 4k + 1 in Gaussian integers
    int b = p;
    int c = non_res(p, primes);
    int x = mod_pow(c, (p-1)/4, p);

    while ( x*x >= p || b*b >= p){
        if ( x > b){
            x %= b;
        }else{
            b %= x;
        }
    }
    pair <int, int> out(x,b);
    return out;
}
```

Testing on my junk laptop, it only took 2.07 seconds to factor all the primes below $10^8$!
The best part is now we can enjoy long lists of primes as sums of squares. Soothing...
```
	18413 = 118^2 + 67^2 
	18433 = 127^2 + 48^2 
	18457 = 36^2 + 131^2 
	18461 = 106^2 + 85^2 
	18481 = 16^2 + 135^2 
	18493 = 123^2 + 58^2 
	18517 = 119^2 + 66^2 
	18521 = 136^2 + 5^2 
	18541 = 125^2 + 54^2 
	18553 = 108^2 + 83^2 
	18593 = 47^2 + 128^2 
	18617 = 136^2 + 11^2 
	18637 = 94^2 + 99^2 
	18661 = 81^2 + 110^2 
	18701 = 115^2 + 74^2 
	18713 = 32^2 + 133^2 
	18749 = 43^2 + 130^2 
```

## Exercises and Further Reading
* Try to prove the Theorem (hint: the Norm is the key)! [Here](https://kconrad.math.uconn.edu/blurbs/ugradnumthy/Zinotes.pdf) are some excellent notes if you get stuck.
* Write a program that outputs the prime factorization of any Gaussian integer
* Gaussian integers are also related to Pythagorean Triples. Can you characterize which integers $a$ satisfy $a^2 =b^2 +c^2$ for some integers b and c?
* Another cool ring is the [Eisenstein Integers](https://en.wikipedia.org/wiki/Eisenstein_integer). What are the Eisenstein integer primes? Can you factor Eisenstein integers efficiently? 

