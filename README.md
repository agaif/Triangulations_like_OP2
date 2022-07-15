# Triangulations of manifolds like the octonionic projective plane

This repository contains C++ programs implementing algorithms for the paper 

Alexander A. Gaifullin, "634 vertex-transitive and more than $10^{103}$ non-vertex-transitive 27-vertex triangulations of manifolds like the octonionic projective plane", arXiv???

which is further referred to as [G].

This README file contains only technical information on the realizations of the algorithms. For the aims and descriptions of the algorithms, and also for mathematical terminology used below, see the quoted paper [G].

## General information

Each directory, except for the directory **common**, contains a separate program. Below is the list of all these programs with their short descriptions. The directory **common** contains two **.cpp** files (together with the corresponding **.hpp** files) which are used by all the programs. Namely, **myiofunctions.cpp / myiofunctions.hpp** contain input/output functions and **permutation.cpp / permutation.hpp** contain the class **Permutation**, which allows working with elements of permutation groups.

**List of programs:**

1. **find**:<br/> 
   Finds all weak pseudo-manifolds $K$ with the prescribed vertex set $V$, dimension $d$, and symmetry group $G$ that have at least $N$ maximal simplices and do not contain a pair of simplices $\sigma$ and $\tau$ such that $\sigma\cup\tau$ is the whole vertex set $V$. Implements the algorithm described in Section 4 of [G].
2. **check**: <br/> 
Checks that the given $G$-invariant simplicial complex $K$ is a combinatorial manifold. (A positive answer guarantees that $K$ is a combinatorial manifold but a negative answer does not guarantee that $K$ is not.) Implements the algorithm described in Section 5 of [G].
4. **permute**: <br/>
 This program, for a given $G$-invariant simplicial complex $K$ and an element $h$ in the normalizer of $G$, produces the $G$-invariant simplicial complex $h(K)$. This simple script was conveniently used to decompose the list of 24 combinatorial manifolds from Theorem 2.2 of [G] (which is produced by **find**) into 4 orbits with respect to the group $C_2\times C_3$ generated by the sign reversal and the Frobenius automorphism.
6. **num_neighbors**: <br/>
This program, for a given $d$-dimensional simplicial complex $K$, computes the distribution of the numbers $s(\rho)$ for $(d-2)$-simplices $\rho$, where $s(\rho)$ is the number of $d$-simplices $\sigma\in K$ containing $\rho$. It was used to produce Table 4 in [G]. 
8. **num_neighbors_pairs**: <br/>
This program, for a given $d$-dimensional simplicial complex $K$ and two given vertices $u$ and $v$ of it, computes the matrix $(N_{pq})$, where $N_{pq}$ is the number of $(d-1)$-simplices $\tau\in K$ such that $u,v\in\tau$, $s(\tau\setminus \\{u\\})=p$, and $s(\tau\setminus \\{v\\})=q$. It was used to produce Tables 6-10 in [G]. 
9. **triples**: <br/>
 This program computes the $\nu$-parameters of simplices of a pure $d$-dimensional simplicial complex. Besides, when $K$ is a combinatorial manifold, it finds all distinguished subcomplexes, that is, subcomplexes of the form $(\Delta_1*\partial\Delta_2)\cup(\Delta_2*\partial\Delta_3)\cup(\Delta_3*\partial\Delta_1)$, where $\Delta_1$, $\Delta_2$, and $\Delta_3$ are $(d/2)$-simplices.
10. **allsimp**: given a symmetry group $G$ and a list of representatives of $G$-orbits of simplices produces the list of all simplices in these orbits.
11. **operations**: performs basic operations (compairing, union, intersection, difference) for simplicial complexes.  

Among these programs, the only two non-trivial are the programs **find** and **check**. For them, we will produce detailed descriptions below. The other programs are very simple. They do not require any special explanation.

## Input/output format

The files with the input data are to be put in the directory of the program (for instance, **find**). The output files also are written to the same directory. Below, we first describe the standard formats that are used for main types of data (symmetry group, simplex, simplicial complex), and then the input/output data for all the 8 programs listed above. 

The main object we are working with is a pure $d$-dimensional simplicial complex with $n$ vertices numbered from $1$ to $n$ with the given symmetry group $G$. In the most important for us case, $d=16$, $n=27$, and $|G|=351$. 

### Standard format for a group $G$

The following description of the symmetry $G\subset S_n$ should be provided in the file **symmetry_group.dat**. The first line of the file contains the degree $n$ of the permutation group. Further lines contain generators of $G$, one generator per line. Each generator is given in cycle notation, cycles are surrounded by round brackets, vertices in each cycle are separated by spaces. Additional spaces are irrelevant. Empty lines are not allowed.  For instance, the $351$-element group $G_{351}\subset S_{27}$ is given by the following file: 
```
27
(1 2 3 4 5 6 7 8 9 10 11 12 13)(14 15 16 17 18 19 20 21 22 23 24 25 26)
(1 14 27)(2 4 10)(3 22 13)(5 6 21)(7 25 11)(8 19 18)(9 16 26)(12 20 24)(15 23 17)
```

### Standard format for a simplex

Every simplex is encoded by a row of $n$ binary digits. The $i^{th}$ from the left digit is 1 whenever $i$ is a vertex of the simplex, and is 0 otherwise. For instance, the row 
```
111010100000000
``` 
encodes the simplex $\\{1,2,3,5,7\\}$ in a simplicial complex with 15 vertices. We conveniently interpret every such 
row as a reversed binary notation for a number. For instance, the above row corresponds to the binary number
$1010111_2=87$. We order the simplices according to the corresponding numbers from smallest to largest in all lists we use.

### Standard format for a simplicial complex

We typically work with rather large $G$-invariant simplicial complexes.

The file describing a simplicial complex is usually called **triang.dat**.

The first line of the description of a simplicial complex contains the number $k$ of $G$-orbits of maximal simplices. The consecutive $k$ lines contain rows of 0's and 1's encoding representatives of $G$-orbits of maximal simplices as described above. The smallest representative must be chosen in each orbit and these representatives must be ordered from smallest to largest (w.r.t. the ordering described above). For instance, a file describing a simplicial complex can look like that:
```
286
111111111111110111000000000
111111111111011111000000000
111111101111111111000000000
. . . . .
110111110010111001111110000
```

The file **triang.dat** does not contain a description of the group $G$. So, in fact, a simplicial complex is described only by the two files **symmetry_group.dat** and **triang.dat** together. 

### Program "find"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **dimnum.dat** containing two numbers $d$ and $N$ separated by a space (or several spaces); the first number $d$ is the dimension of weak pseudomanifolds $K$ we are looking for, and the second number $N$ is the smallest number of $d$-simplices in $K$, for instance,  
`16 100386` <br/> 
     

**Output data:**

+ A file **triangulations.dat** containing the list of all weak pseudomanifolds found. The information about each pseudomanifold starts with the line of the form <br/> <br/> 
` *** <number of the pseudomanifold> ` <br/><br/>
after which follows the description of the pseudomanifold in the standard format. Different pseudomanifolds are separated by empty lines. The numbering of pseudomanifolds starts from 1. So the file  **triangulations.dat** looks like that 
    ```
    *** 1
    286
    111111111111110111000000000
    . . . . . 
    110111110010111001111110000

    *** 2
    286
    111111111111111110000000000
    . . . . . 
    001111110110111010111110000

    . . . . . 
    ```

+ A file **triangulations_with_sizes.dat** containing the same list with the additionally information on the sizes of the $G$-orbits of maximal simplices. The size of each $G$-orbit is written directly after the representative of this orbit (separated by space). For instance,
    ```
    *** 1
    286
    111111111111110111000000000 351
    . . . . . 
    110111110010111001111110000 351

    *** 2
    286
    111111111111111110000000000 351
    . . . . . 
    001111110110111010111110000 351

    . . . . . 
    ```
### Program "check"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **triang.dat** describing a pure simplicial complex $K$ in the standard format.

**Output data:**

The program **check** says whether it has succeeded to check that $K$ is a combinatorial manifold. So there are two possibilities:

+ If the program succeeds to check that $K$ is a combinatorial manifold, then the output is just the line
    ``` 
    This is a combinatorial manifold
    ```

+ If the program does not succeed to check that $K$ is a combinatorial manifold, then it writes the smallest representatives of $G$-orbits of simplices for which it has failed to check that their links are combinatorial spheres to the file **bad_simplices.dat** in the standard format. Besides, the program produces the line 
     ```
     There are <number of orbits of bad simplices> bad links
     ```
     

**Remark.** If a simplex is put to the list of *bad simplices*, it does not follow that its link is not a combinatorial sphere. A positive result guarantees that $K$ is a combinatorial manifold but a negative result does not guarantee that $K$ is not.

### Program "permute"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **triang.dat** describing a pure simplicial complex $K$ in the standard format.
+ A file **permutation.dat** consisting of a single line that contains a permutation $h$ in cycle notation.

**Output data:**

First, the program finds out whether the permutation $h$ normalizes the group $G$. If $h$ does not normalize $G$, then the only output is the line

```
The permutation does not belong to the normalizer of the group
```

Second, if $h$ belongs to the normalizer of $G$, then $h(K)$ is again a $G$-invariant simplicial complex. Then the output is a file **triang_new.dat** containing $h(K)$ in the standard format.


### Program "num_neighbors"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **triang.dat** describing a $d$-dimensional pure simplicial complex $K$ in the standard format.

**Output data:**

+ A file **result.dat**. Each line of this file contains two numbers $n$ and $r(n)$ separated by a space, where $r(n)$ is the number of $(d-2)$-simplices $\rho\in K$ with $n(\rho)=n$. Recall that $n(\rho)$ is the number of $d$-simplices $\sigma\in K$ containing $\rho$. For instance, the file **result.dat** can look like that: 

    ```
    3 849771
    4 1509651
    5 697788
    6 201942
    7 40716
    8 9477
    9 351
    ```
      
### Program "num_neighbors_pairs"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **triang.dat** describing a $d$-dimensional pure simplicial complex $K$ in the standard format.
+ A file **edge.dat** containing two (different) numbers $u$ and $v$ separated by a space. These are numbers of vertices, so they must be in the range  from 1 to the total number of vertices $n$. 

**Output data:**

+ A file **result.dat**. The first line of this file contains a single number $k$ that is equal to the maximal among all numbers $n(\tau\setminus\\{u\\})$ and $n(\tau\setminus\\{v\\})$, where $\tau$ runs over all $(d-1)$-dimensional simplices $\tau\in K$ containing both $u$ and $v$. The next $k$ lines contain the $k\times k$ matrix $N_{pq}$, $1\le p,q\le k$, where $N_{pq}$ is the number of $(d-1)$-simplices $\tau\in K$ such that $u,v\in\tau$, $n(\tau\setminus \\{u\\})=p$, and $n(\tau\setminus \\{v\\})=q$. (The entries of each row of the matrix are separated by spaces.)


### Program "triples"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **triang.dat** describing a $d$-dimensional pure simplicial complex $K$ in the standard format.
+ The program asks the user whether it should look for distinguished triples or not. (The reason is that the procedure for finding the distinguished triples works correctly only when $K$ is a combinatorial manifold.)
 
**Output data:** 

+ A file **nu_param.dat**. This file lists the (smallest) representatives of all $G$-orbits of $d$-simplices $\sigma\in K$ with at least one $\nu$-parameter greater than or equal to $d/2$. The first line of the file contains the number $q$ of such $G$-orbits. Each of the next $q$ orbits contains a simplex $\sigma$ (in standard format), and the $\nu$-parameters $\nu_K(\sigma,v_i)$ for $i=1,\ldots,n-d-1$, where $v_1 <\cdots < v_{n-d-1}$ are all the vertices not belonging to $\sigma$. These numbers are separated with spaces, for instance, 

    ```
    110110011101100 1 1 2 0 4 1
    ```

+ If the user has chosen to look for distinguished triples, then the program also produces a file
**distinguished.dat** containing information on representatives of $G$-orbits of distinguished triples. The description of each distinguished triple $(\Delta_1,\Delta_2,\Delta_3)$ consists of 4 lines. The first 3 lines constain the simplices $\Delta_1$, $\Delta_2$, and $\Delta_3$ (in the standard format), and the last line is for the *type* of the triple. (A distinguished triple has *type 1* if the three simplices $\Delta_1$, $\Delta_2$, and $\Delta_3$ lie in the same $G$-orbit, and has *type 2* otherwise.) Descriptions of different distinguished triples are separated by empty lines. An example of the description of a triple is as follows: 
 
    ```
    001100110001111010000000000
    110011001110000100000000100
    000000000000000001111111011
    type 2
    ```

### Program "allsimp"

**Input data:**

+ A file **symmetry.dat** describing the symmetry group $G$ in the standard format.
+ A file **triang.dat** describing a $d$-dimensional pure simplicial complex $K$ in the standard format.

**Output data:**

+ A file **triang_all.dat**. The first line contains the total number of $d$-simplices of $K$; the following lines contain the list of all the $d$-simplices in the standard format. 

### Program "operations"

**Input data:**

+ Files **triang1.dat** and  **triang2.dat** describing two $d$-dimensional pure simplicial complexes $K_1$ and $K_2$, respectively (in the standard format). The program works correctly only if the complexes $K_1$ and $K_2$ have equal dimensions and their descriptions correspond to the same symmetry group $G$. (Nevertheless, the description of the group $G$ is not needed.) 
+ The user is asked to choose the type of the operation:

    ```
    Input the type of the operation:
      1 compare
      2 union
      3 intersection
      4 difference
    ```
    
**Output data:**

+ For the operation **compare** the program gives one of the four answers: <br/>
`The complexes coincide` <br/>
`The first complex is contained in the second one` <br/>
`The second complex is contained in the first one` <br/>
`Neither of the complexes is contained in the other one` <br/>

+ For any of the other three operations, the reslting simplicial complex is written to the file **triang_res.dat**. Note that the operations **intersection** and **difference** are understood in the sense of pure $d$-dimensional complexes. This means that they yield the simplicial complexes consisting of all $d$-simplices $\sigma\in K_1$ that belong (respectively, do not belong) to $K_2$, and all their faces.
