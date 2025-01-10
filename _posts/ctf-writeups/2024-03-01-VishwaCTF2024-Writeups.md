---
layout: post
title: VishwaCTF2024_Writeups
date: 2025-01-05
categories: categories: [CTF-writeups] 
tags: [cryptography, VishwaCTF2024 ]
---
# CODEON

## Author :  Sankalp Chakre

### DESCRIPTION :
* My biochemist friend has doubt that his phone is used by others for calling without knowing him. So, he wants me to find some information related to it, that’s why he sent me some information related to it, also he uses one word short-forms mostly. But I am unable to understand sent data, can you help me?

### Files :
CODEON.txt
```
DATA:
RSC BGT FFH HFT TPD WMA OCZ CCD DCZ ZVA

He gave me some instructions related to it:

1)He told me he saw some names in call logs those are:
	Felix Delastelle, Dmitri Mendeleev

2)He also gave me 2 matrices A and B and told to perform same operation on matrix A and B as of performed on matrix P and Q. Find something related to biochemistry.
  Matrix A =    45 35 93 95 24 65
		25 15 55 64 36 45
		15 65 62 16 65 38
		19 64 35 69 65 63
		47 67 48 60 39 27
		66 48 77 22 10 69
 
Matrix B =	33 25 30 11 68 65
		83 36 19 33 55 51
		20 16 48 63 41 71
		30 42 12 25 31 37
		51 03 44 23 43 85
		20 39 28 41 01 70


Matrix P= 	13 81 60
		40 99 88
		39 87 92

Matrix Q=	50 52 36 
		90 75 28
		80 9  18

Result Matrix of P and Q =	60   82   36 
			   				60   109  104 
							52   75   4

-60112
```

#### Analysis : 
* On reading codeon.txt we get that first we have to find how the matrix R is being calculated from P and Q which is a bit guessy here
* I tried many things here like addition,subtration,dot product,multiplication, But particularly on element-wise-multiplication and since the name of Dmitri Mendeleev was given i thought of the periodic table and after element wise multiplication and then taking mod 118 of each element we get the result matrix 
* so using this script 
```python
import numpy as np

matrix_a = np.array([
    [45, 35, 93, 95, 24, 65],
    [25, 15, 55, 64, 36, 45],
    [15, 65, 62, 16, 65, 38],
    [19, 64, 35, 69, 65, 63],
    [47, 67, 48, 60, 39, 27],
    [66, 48, 77, 22, 10, 69]
])

matrix_b = np.array([
    [33, 25, 30, 11, 68, 65],
    [83, 36, 19, 33, 55, 51],
    [20, 16, 48, 63, 41, 71],
    [30, 42, 12, 25, 31, 37],
    [51, 3, 44, 23, 43, 85],  
    [20, 39, 28, 41, 1, 70]   
])

matrix_elementwise_product = matrix_a * matrix_b
matrix_result_mod = np.mod(matrix_result, 118)
print(matrix_result_mod)
```
* I get 
[69,49,76,101,98,95]
[69,68,101,106,92,53]
[64,96,26,64,69.102]
[98,92,66,73,9,89]
[37,83,106,82,25,53]
[22,102,32,76,10,110]
as the result matrix 

* Now converting the numbers in the matrix to there corresponding elements according to the periodic table 

* [Tm  In  Os  Md  Cf  Am]
[Tm  Er  Md  Sg  U  I]
[Gd  Cm  Fe  Gd  Tm  No]
  [Cf  U  Dy  Ta  F  Ac]
 [Rb  Bi  Sg  Pb  Mn  I]
  [Ti  No  Ge  Os  Ne  Ds] 

* And as we can see that the last column comes out to be amino acids.
* On searching the name of Felix Delastelle on google we get that it redirects us to BIFID Cipher so i guessed that aminoacids is the key for decrypting the DATA that is given to us.
* So i go to CyberChef
 ![alt text](../Screenshot_20240302_150237.png)

* And the output was "UGC GCA CUG CUU GAG AGG GGG AUU UUU ACC"
* Now i again got confused about what to do.
* Then i searched the name of challenge Codeon with the term biochemistry and then i got that the result string is CODON which we have to decode to get the final flag
* So i go to dcode.fr 

 ![alt text](../Screenshot_20240304_161344.png)

 * And here i got CALLERGIFT as the string 

 * So our flag is VishwaCTF{CALLERGIFT}

 

 # LET's SMOTHER THE KING

## Author : Naman Chordia


### DESCRIPTION :
* In my friend circle, Mr. Olmstead and Mr. Ben always communicate with each other through a secret code language that they created, which we never understand. Here is one of the messages Mr. Ben sent to Mr. Olmstead, which I somehow managed to hack and extract it from Ben's PC. However, it's encrypted, and I don't comprehend their programming language. Besides being proficient programmers, they are also professional chess players. It appears that this is a forced mate in a 4-move chess puzzle, but the information needs to be decrypted to solve it. Help me out here to solve the chess puzzle and get the flag.

* Flag format: VishwaCTF{move1ofWhite_move1ofBlack_move2ofWhite_move2ofBlack_move3ofWhite_move3ofBlack_move4ofWhite}.

* Note: Please use proper chess notations while writing any move.
### Files :
code.txt
```
D'`A_9!7};|jE7TBRdQbqM(n&JlGGE3feB@!x=v<)\r8YXtsl2Sonmf,jLKa'edFEa`_X|?UTx;WPUTMqKPONGFjJ,HG@d'=BA@?>7[;4z21U54ts10/.'K+$j(!Efe#z@a}vut:[Zvutsrqj0ngOe+Lbg`edc\"CBXW{[TSRvP8TMq4JONGFj-IBGF?c=a$@9]=6|:3W10543,P*/.'&%$Hi'&%${z@a}vut:xqp6nVrqpoh.fedcb(IHdcba`_X|V[ZYXWPOsMLKJINGkKJI+G@dDCBA#"8=6Z4z21U5ut,P0)(-&J*)"!E}e#"!x>|uzs9qvo5srkpingf,Mchg`_%cbaZBXW{>=YXWPOs65KPImMLEDIBf)(D=a$:?>7<54X210/43,Pqp(',+*#G'gf|B"y?}v^tsr8vuWsrqpi/POkdibgf_%]b[Z_X|?>TYRQu8TMqQPIHGkEJIBA@dD&B;@?8\<5{3W70/.-Q10/on,%Ij('~}|B"!~}_u;y[Zponm3qpoQg-kjihgfeG]#aCBXW{[ZYX:Pt7SRQJONMLEiIHA@?cCB$@9]~<;:921U543,10/.-&J*j('~}${A!~}vuzs9Zponm3qpihmf,jihgfeG]#n
```


#### Analysis : 
* On searching the names of the persons given in description i.e. Ben Olmstead we get that they have a return a coding language called malbolge 
* So I searched for malbolge interpreter online and after running what was given in code.txt there we got
```
White- Ke1,Qe5,Rc1,Rh1,Ne6,a2,b3,d2,f2,h2 Black- Ka8,Qh3,Rg8,Rh8,Bg7,a7,b7,e4,g2,g6,h7
```
* So this is the board setup

![alt text](../1000083909.png)

* And taking the hint form the name of chall that this should be a smother's mate in 4 moves I got the moves

Nc7+
Kb8
Na6+
Ka8
Qb8+
Rxa8
Nc7#
```

So the flag is 
VishwaCTF{Nc7+_Kb8_Na6+_Ka8_Qb8+_Rxb8_Nc7#}

```

# TEYVAT TALES

## DESCRIPTION
All tavern owners in Mondstadt are really worried because of the frequent thefts in the Dawn Winery cellars. The Adventurers’ Guild has decided to secure the cellar door passwords using a special cipher device. But the cipher device itself requires various specifications….which the guild decided to find out by touring the entire Teyvat.

PS: The Guild started from the sands of Deshret then travelled through the forests of Sumeru and finally to the cherry blossoms of Inazuma

## Author: Amruta Patil

### FILES :
 [index.html](../a.html) 
 [script.js](../script.js) 
 [styles.css](../styles.css)

 ### Analysis:
 * On analysing the files given there was a image folder containing various images which we have to decode and they seem to be like some ancient language maybe genshin or something
 * But on reading  [script.js](script.js) we can see that all the answers were given in the js script only which were 
 * enigma m3
 * ukw c
 * rotor1 i p m rotor2 iv a o rotor3 vi i n
 * vi sh wa ct fx

 * on putting these answers on the webpage we got this
 ![alt text](../GenshinNoticeBoard.png)
 
 * So i thought of decoding the text given here by enigma so i go to the website https://cryptii.com/pipes/enigma-machine and put all the conditions of enigma 

 ![alt text](../Screenshot_20240304_170432.png)

 And BOOM! we got this 

 So this is our flag
 ```
 VishwaCTF{beware_of_tone-deaf_bard}
 ```
