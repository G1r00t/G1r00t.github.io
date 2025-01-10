---
layout: post
title: Point_Addition
date: 2025-01-05
categories: categories: [Cryptohack-writeups, Elliptic-curves, Starter] 
tags: [cryptography, elliptic-curve , sage , Cryptohack]
---
```
a = 497
b = 1768
p = 9739
F = GF(p)
E = EllipticCurve(F, [a, b])
p_x = 493
p_y = 5564
P = E(p_x,p_y)
x = 1539
y = 4742
Q = E(x,y)
x = 4403
y = 5202
R = E(x,y)
print(2*P + Q + R)
```
#crypto{4215,2162}