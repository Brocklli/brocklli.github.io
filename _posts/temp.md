**Work done in python through VSCode**
{% highlight py %}

    import math
    from functools import reduce
    # Run with python3 crypto.py

    # # Problem 1
    # p = 31; a = 7
    # ai = pow(a, 0)
    # mod = ai % p
    # for i in range(p - 1):
    #     i=i+1
    #     ai = pow(a, i)
    #     mod = ai % p
    #     if not i % 2 == 0:
    #         if  not i % 3 == 0:
    #             if  not i % 5 == 0:
    #                 print(i, mod)

    # # Problem 4
    # mod = 0
    # for i in range(991):
    #     mod = pow(6, i) % 991
    #     if mod == 687:
    #         print("k =",i, "| 6^k % 991 =",mod)

    # Computes the Greatest Common Divisor between two numbers
    def gcd(a, b):
        # Everything divides 0
        if a == 0:
            return b
        if b == 0:
            return a
        # Base case
        if a == b:
            return a
        # a is greater
        if a > b:
            return gcd(a - b, b)
        return gcd(a, b - a)

    # Computes Least Common Multiple between two numbers
    def lcm(a, b):
        return (a * b) / gcd(a, b)

    # a = base, n = exponent, b = constant, m = mod
    # Task is to compute a^n % m for a large number n
    def PowMod(a, n, b, m):
        if n == 0:
            return b % m
        elif n == 1:
            return (a * b) % m
        elif n % 2 == 0:
            return PowMod((a * a) % m, (n / 2), b, m)
        else:
            return PowMod(a, n - 1, (a * b) % m, m)

    # Test if element a is congruent to 1 % p
    def congOne(a, p):
        return PowMod(a, p, 1, p + 1) == 1

    # Computes all primitive elements mod p
    def peMod(p, a):
        ai = 1; i = 1; mod = ai % p
        for i in range(p):
            ai = pow(a, i)
            mod = ai % p
            print(i, "->", mod)

    # returns true if p is most likely prime
    def fermatWitnessTest(p):
        a = 5
        for a in range(2, p):
            if congOne(a, p - 1):
                print("p is definitively not prime.")
                return False
            print("Test failed with int ->", a)
        print("Test failed for all a; p is PROBABLY prime.")
        return True

    def testStop(b, n):
        if b == n - 1:
            print("STOP: Test inconclusive. -1 mod.")
            return False
        elif b == 1:
            print("STOP: Number is not prime.")
            return False
        return True

    # Prime test
    def millerRabin(n, a):
        s = 0; m = n - 1
        if gcd(n, a) != 1:
            print("Failure, n is not prime!")
            return False
        while m % 2 == 0:
            s += 1; m /= 2
        b = pow(a, m)   
        bmod = b % n
        # print("s:", s)
        # print("m:", m)
        # print("a:", a)
        # print("b:", b)
        # print("b0:", bmod)
        if not testStop(bmod,n):
            return False
        for i in range(s):
            b = pow(bmod, 2)
            bmod = b % n
            # print("b", i + 1, ": ", bmod, sep = "")
            if not testStop(bmod, n):
                return False
        print("Test is inconclusive.")
        return True

    # Works the same as ORing the bits
    def polyAdd(a, b):
        return a | b

    # constants used in the multGF2 function
    mask1 = mask2 = polyred = None
    
    def setGF2(degree, irPoly):
        """Define parameters of binary finite field GF(2^m)/g(x)
        - degree: extension degree of binary field
        - irPoly: coefficients of irreducible polynomial g(x)
        """
        global mask1, mask2, polyred
        mask1 = mask2 = 1 << degree
        mask2 -= 1
        if sum(irPoly) <= len(irPoly):
            polyred = reduce(lambda x, y: (x << 1) + y, irPoly[1:])    
        else:
            polyred = poly2Int(irPoly[1:]) 
            
    def multGF2(p1, p2):
        """Multiply two polynomials in GF(2^m)/g(x)"""
        p = 0
        while p2:
            if p2 & 1:
                p ^= p1
            p1 <<= 1
            if p1 & mask1:
                p1 ^= polyred
            p2 >>= 1
        return p & mask2

    def polyExp(p1, p2):
        p = p1
        for i in range(p2):
            p = multGF2(p, p1)
        return p

    def poly2Int(hdPoly):
        """Convert a "high-degree" polynomial into a "big" integer"""
        bigInt = 0
        for exp in hdPoly:
            bigInt += 1 << exp
        return bigInt

    def i2P(sInt):
        """Convert a "small" integer into a "low-degree" polynomial"""
        return [(sInt >> i) & 1 for i in reversed(range(sInt.bit_length()))]

    # Class fermat
    def fermatFactor(n):
        l = []
        for y in range(1,21):
            x = (n + y**2)**0.5
            if x - int(x) == 0:
                item = (y,(n + y**2)**0.5)
                l.append(item)
        return l

    # Test if element x^2 is congruent to y^2 % n
    def cong(x, y, n):
        print(y, n, x)
        print(pow(y, 2) % n)
        return (pow(y, 2) % n) == pow(x, 2)

    # mod squares
    def squareFactorEqCall(n):
        x = (n - 1) / 2
        l = []
        for y in range(int(x) + 1):
            l.append(y**2 % n)
        return l

    # mod squares
    def squareFactor(n):
        x = (n - 1) #/ 2
        l = []
        for y in range(int(x) + 1):
            z = (y, y**2 % n)
            l.append(z)
        return l

    # HW2 Problem 3
    def squareEq(n):
        l = []
        l2 = squareFactorEqCall(n)
        for x in range(n):
            z = ((x**3) + (5*x) + 2) % n
            if z in l2:
                w = (x, z)
                l.append(w)
        return l

    def legendre(a, n):
        assert(n > a > 0 and n%2 == 1)
        t = 1
        while a != 0:
            while a % 2 == 0:
                a /= 2
                r = n % 8
                if r == 3 or r == 5:
                    t = -t
            a, n = n, a
            if a % 4 == n % 4 == 3:
                t = -t
            a %= n
        if n == 1:
            return t
        else:
            return 0

    def evalZero(a):
        return a == 0

    def evalOne(a):
        return a == 1

    def modulo(a, p):
        return a >= p

    # Alice and Bob's Zero Knowledge Proof
    # n and x are stationary, y is given to Bob from Alice,
    # Bob responds with either 0 or 1 and alice will respond with
    # a z for bob to compare and either accept or reject.
    #
    # If b = 0, check if z^2 is congruent to y % n
    # If b = 1, check if z^2 = xy % n
    def zeroProof(n, x, y, b, z):
        if(b == 0):
            return "Bob Accepts." if ((pow(z,2)) % n) == (y % n) else "Bob Rejects."
        elif(b == 1):
            return "Bob Accepts." if ((pow(z,2)) % n) == ((x * y) % n) else "Bob Rejects."

    def SolovayStrassen():
        print("Todo...")

    def sqrt():
        print("Todo...")

    i = 11
    if i == -1:
        print("Waiting for directions...")
    if i == 0:
        print("PowMod:", PowMod(40513610, 32559247, 1, 62251979))
    elif i == 1:
        print("gcd:", gcd(66,81))
    elif i == 2:
        print("lcm:", lcm(108,84))
    elif i == 3:
        print("peMod:", peMod(31, 7))
    elif i == 4:
        print("fermatWitnessTest:", fermatWitnessTest(35))
    elif i == 5:
        print("millerRabin:", millerRabin(561, 2))
    elif i == 6:
        print("Polynomial addition:", polyAdd(0b1010, 0b1100))
    elif i == 7:
        # Set binary field to 2^4 -> x^4 + x^3 + 1 == 0b11001
        setGF2(4, i2P(0b11001))
        # Evaluate (number * number) to get the remainder after division
        print("9 + 13 = ", polyAdd(0b1001, 0b1101))
        print("9 * 13 = ", multGF2(0b1001, 0b1101))
    elif i == 8:
        print("Fermat Factor", fermatFactor(295927))
    elif i == 9:
        # print("Square Mod Factor:", squareFactor(13))
        # print("Problem 4C:\n", squareFactor(41))
        print("Problem 4C:\n", squareFactor(21))
    elif i == 10:
        print("HW3 problem 2:", squareEq(13))
    elif i == 11:
        symbol = legendre(41, 79)
        print("Legendre symbol = ", symbol, ", so then", sep = '')
        if symbol ==  0: print("p divides a")
        if symbol ==  1: print("a is a nonzero square mod p")
        if symbol == -1: print("a is not a square mod p")
    elif i == 12:
        # zeroProof(n, x, y, b, z)
        print("Alice and Bob:")
        n = 58014043
        x = 18059241
        print(zeroProof(n, x, 3965911, 0, 18763))
        print(zeroProof(n, x, 48963613, 1, 32270779))
        print(zeroProof(n, x, 8302702, 1, 767595))
        print(zeroProof(n, x, 50014686, 0, 976657))
        # print("\nProblem 3A:")
        # # If b = 0, check if z^2 is congruent to y % n
        # # If b = 1, check if z^2 = xy % n
        # print(zeroProof(58014043, 18059241, 100000000, 0, 10000))
        # print(zeroProof(58014043, 18059241, 12, 1, 34))
        # print(zeroProof(58014043, 18059241, 12, 1, 34))
        # print(zeroProof(58014043, 18059241, 12, 0, 34))
    elif i == 13:
        n = 6509
        a = 17
        x = gcd(a, n)
        print("gcd of", a, "and", n, "is x:", x)
        w = PowMod(a, (n - 1) / 2, 1, n)
        print("w from PowMod:", w)
        y = legendre(x, n)
        print("legendre of", x, "and", n, "y:", y)
        print("x % n:", x % n, "\ny % n:", y % n)
        print("SolovayStrassen:", SolovayStrassen())

{% endhighlight %}

**Work done in python through VSCode (contributed by mits online)**
{% highlight py %}

    # Python3 program to implement Solovay-Strassen
    # Primality Test
    import random
    
    # modulo function to perform binary
    # exponentiation
    def modulo(base, exponent, mod):
        x = 1;
        y = base;
        while (exponent > 0):
            if (exponent % 2 == 1):
                x = (x * y) % mod;
    
            y = (y * y) % mod;
            exponent = exponent // 2;
    
        return x % mod;
    
    # To calculate Jacobian symbol of a
    # given number
    def calculateJacobian(a, n):
    
        if (a == 0):
            return 0;# (0/n) = 0
    
        ans = 1;
        if (a < 0):
            
            # (a/n) = (-a/n)*(-1/n)
            a = -a;
            if (n % 4 == 3):
            
                # (-1/n) = -1 if n = 3 (mod 4)
                ans = -ans;
    
        if (a == 1):
            return ans; # (1/n) = 1
    
        while (a):
            if (a < 0):
                
                # (a/n) = (-a/n)*(-1/n)
                a = -a;
                if (n % 4 == 3):
                    
                    # (-1/n) = -1 if n = 3 (mod 4)
                    ans = -ans;
    
            while (a % 2 == 0):
                a = a // 2;
                if (n % 8 == 3 or n % 8 == 5):
                    ans = -ans;
    
            # swap
            a, n = n, a;
    
            if (a % 4 == 3 and n % 4 == 3):
                ans = -ans;
            a = a % n;
    
            if (a > n // 2):
                a = a - n;
    
        if (n == 1):
            return ans;
    
        return 0;
    
    # To perform the Solovay- Strassen
    # Primality Test
    def solovoyStrassen(p, iterations):
    
        if (p < 2):
            return False;
        if (p != 2 and p % 2 == 0):
            return False;
    
        for i in range(iterations):
            
            # Generate a random number a
            a = random.randrange(p - 1) + 1;
            jacobian = (p + calculateJacobian(a, p)) % p;
            mod = modulo(a, (p - 1) / 2, p);
    
            if (jacobian == 0 or mod != jacobian):
                return False;
    
        return True;
    
    # Driver Code
    iterations = 50;
    num1 = 15;
    num2 = 13;
    
    if (solovoyStrassen(num1, iterations)):
        print(num1, "is prime ");
    else:
        print(num1, "is composite");
    
    if (solovoyStrassen(num2, iterations)):
        print(num2, "is prime");
    else:
        print(num2, "is composite");
    
    # This code is contributed by mits

{% endhighlight %}

**Work in Sage through CoCalc**
{% highlight py %}

    #RSA - Example
    p = 71; q = 59; n = p*q; e = 723; d = inverse_mod(e,(p-1)*(q-1));
    p1 = 809; p2 = 2008; p3 = 518; p4 = 5;
    c1 = p1^e % n; c2 = p2^e % n; c3 = p3^e % n; c4 = p4^e % n;
    q1 = c1^d % n; q2 = c2^d % n; q3 = c3^d % n; q4 = c4^d % n;
    q1,q2,q3,q4;

        (809, 2008, 518, 5)

    #RSA - Challenge 1
    n = 25330309; d = 1061;
    c1 = 19019931; c2 = 1619805; c3 = 740498; c4 = 2671344;
    q1 = c1^d % n; q2 = c2^d % n; q3 = c3^d % n; q4 = c4^d % n;
    q1,q2,q3,q4;

        (31805, 11303, 80505, 1905)

    #RSA - Challenge 2
    n = 59264263; e = 2909; d = n * prod([1-1/p for p in prime_divisors(n)]);
    c1 = 35270141; c2 = 9642524; c3 = 49091707;
    q1 = c1^d % n; q2 = c2^d % n; q3 = c3^d % n;
    q1,q2,q3;

        (1, 1, 1)

    # Miller-Rabin
    n = 14491; a = 2;


    s = 0; m = n - 1; b = -1; bmod = -1;

    while(m % 2 == 0):
        s = s + 1;
        m = m / 2;
    b = pow(a, m);
    bmod = b % n;
    print("s: ", s);
    print("m: ", m);
    print("a: ", a);
    # print("b: ", b);
    print();
    print("b 0: ", bmod);
    if(bmod == -1 or bmod == 1):
        print("Fail 1: STOP test is inconclusive.")
        exit(1);
    for i in range(s):
        b = pow(bmod, 2);
        bmod = b % n;
        print("b", i + 1, ": ", bmod);
        if(bmod == n - 1):
            print("Fail 2: STOP test is inconclusive. -1 mod");
            exit(2);
        if(bmod == 1):
            print("Fail 3: STOP n is not prime.");
            exit(3);

    print();
    print("Fail 4: STOP test is inconclusive.");

        s:  1
        m:  7245
        a:  2

        b 0:  10448
        b 1 :  1
        Fail 3: STOP n is not prime.
        Error in lines 16-25
        Traceback (most recent call last):
        File "/cocalc/lib/python3.8/site-packages/smc_sagews/sage_server.py", line 1230, in execute
            exec(
        File "", line 10, in <module>
        File "/usr/lib/python3.8/_sitebuiltins.py", line 26, in __call__
            raise SystemExit(code)
        SystemExit: 3

{% endhighlight %}