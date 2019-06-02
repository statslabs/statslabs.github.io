---
layout: page
title: Statslabs::Matrix API Documentation
permalink: /matrix/guide
---

* Contents
{:toc}

## Basic Matrix Use

+ `Matrix<T, N>` is an `N`-dimensional dense matrix of some value type `T`, with
element stored in row-major ordering (i.e., row by row). It can be used like
this:
```c
  Matrix<double, 0> m0{1};           // zero dimensions: a scalar
  Matrix<double, 1> m1{1, 2, 3, 4};  // one dimension: a vector (4 elements)
  Matrix<double, 2> m2{
      // two dimensions (4*3 elements)
      {00, 01, 02, 03},  // row 0
      {10, 11, 12, 13},  // row 1
      {20, 21, 22, 23}   // row 2
  };

  // three dimensions (4*7*9 elements), all 0-initialized
  Matrix<double, 3> m3(4, 7, 9);
  // 17 dimensions (no elements so far)
  Matrix<complex<double>, 17> m17;
```

+ The element type must be something we can store. Some typical types for T
include `double`, `float`, `std::complex<double>`, `std::complex<float>`, `int`
and `unsigned int`. When using BLAS interface defined in this library, only the
first four types are supported -- namely `double`, `float`,
`std::complex<double>` and `std::complex<float>`.

+ (Advanced) It is also possible to use some other non-typical types.
```c
  // Example:
  Matrix<double, 2> md;  // OK
  Matrix<string, 2> ms;  // OK: just don't try arithmetic operations

  // 3-by-2 matrix of 2-by-2 matrices
  // a matrix is a plausible "number"
  Matrix<Matrix<int, 2>, 2> mm{
      {  // row 0
          { {1, 2}, {3, 4} },  // col0
          { {4, 5}, {6, 7} },  // col1
      },
      {  // row 1
          { {8, 9}, {0, 1} },  // col0
          { {2, 3}, {4, 5} },  // col1
      },
      {  // row 2
          { {1, 2}, {3, 4} },  // col0
          { {4, 5}, {6, 7} },  // col1
      }
  };
```
Matrix arithmetic doesn’t have exactly the same mathematical properties as the
matrix with typical value types, so we must be careful how we use such a matrix.

+ Note: As for `std::vector`, we use `()` to specify sizes and `{}` to specify
element values. The number rows must match the specified number of dimensions
and the number of elements in each dimension (each column) must match.
```c
  // Example:
  Matrix<char, 2> mc1(2, 3, 4);  // error: two many dimension sizes

  Matrix<char, 2> mc2{
      {'1', '2', '3'}  // note: this one should be okay
  };

  Matrix<char, 2> mc3{
      {'1', '2', '3'},
      {'4', '5'}  // error: element missing for third column
  };
```


+ `Matrix<T, N>` has its number of dimensions (its `order()`) specified as a
template argument (here, `N`). Each dimension has a number of elements (its
`extent()`) deduced from the initializer list or specified as a `Matrix`
constructor argument using the `()` notation. The total number of elements is
referred to as its `size()`.
```c
  // Example:
  Matrix<double, 1> m1_new(100);       // one dimension: a vector (100 elements)
  Matrix<double, 2> m2_new(50, 6000);  // two dimensions: 50*6000

  auto d1 = m1_new.order();            // 1
  auto d2 = m2_new.order();            // 2

  auto e1 = m1_new.extent(0);          // 100
//  auto e1a = m1_new.extent(1);         // error: m1 is one-dimensional

  auto e2 = m2_new.extent(0);          // 50
  auto e2a = m2_new.extent(1);         // 6000

  auto s1 = m1_new.size();             // 100
  auto s2 = m2_new.size();             // 50*6000
```

+ `Matrix` elements can be accessed by several forms of subscripting.
```c
  // Example:
  Matrix<double, 2> m{
      // two dimensions (4*3 elements)
      {00, 01, 02, 03},  // row 0
      {10, 11, 12, 13},  // row 1
      {20, 21, 22, 23}   // row 2
  };

  cout << "\nm = " << m << endl;

  Matrix<double, 1> v = m[1];
  cout << "\nv = " << v << endl;

  double dval1 = m(1, 2);  // 12
  double dval2 = m[1][2];  // 12
  double dval3 = v[2];   Guide  // 12
```

+ For convenience the following type aliases have been defined for `Matrix<T, N>`:

| T (value_type)         | vector <br> (N == 1) | matrix <br> (N == 2) | cube <br> (N == 3) |
|------------------------|----------------------|----------------------|--------------------|
| `double` (default type)| `vec`                | `mat`                | `cube`             |
| `double`               | `dvec`               | `dmat`               | `dcube`            |
| `float`                | `fvec`               | `fmat`               | `fcube`            |
| `std::complex<double>` | `cx_vec`             | `cx_mat`             | `cx_cube`          |
| `std::complex<double>` | `cx_dvec`            | `cx_dmat`            | `cx_dcube`         |
| `std::complex<float>`  | `cx_fvec`            | `cx_fmat`            | `cx_fcube`         |
| `unsigned int`         | `uvec`               | `umat`               | `ucube`            |
| `int`                  | `ivec`               | `imat`               | `icube`            |

The complete source file for this section can be find in file
[01_basic_matrix_uses.cc](https://github.com/statslabs/matrix/blob/master/examples/01_basic_matrix_uses.cc).

### Construction and Assignment
A `Matrix<T, N>` can be intialized by:

+ `Matrix<T, N>()` -- using default constructor:

+ `Matrix<T, N>(Matrix<T, N>)` -- using another `Matrix`. 

+ `Matrix<T, N>(MatrixRef<T, N>)` -- using a `MatrixRef`. `MatrixRef<T, N>` is
basically a clone of `Matrix<T, N>` which does not have its own elements. It
behaves just like a `Matrix` and is normally used to represent sub-`Matrix`es.
```c
  // Example:
  Matrix<double, 1> m1{1, 2, 3};
  auto mr1 = m1(slice(1));
  Matrix<double, 1> m1sub(mr1);

  Matrix<double, 2> m2{ {1, 2, 3}, {4, 5, 6}, {7, 8, 9} };
  auto mr2 = m2(slice(1), slice(1));
  Matrix<double, 2> m2sub(mr2);

  Matrix<double, 3> m3{ { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} },
                        { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} },
                        { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} } };
  auto mr3 = m3(slice(1), slice(1), slice(1));
  Matrix<double, 3> m3sub(mr3);
```

+ `Matrix<T, N>(n_1, n_1, ..., n_N)` -- using pre-specified dimensions:
```c
  // Example:
  Matrix<double, 0> m0;
  Matrix<double, 1> m1(3);
  Matrix<double, 2> m2(3, 4);
  Matrix<double, 3> m3(3, 4, 5);
```

+ `Matrix<T, N>(MatrixInitializer)` -- using `MatrixInitializer` which is a
  nested `std::initializer_list`:
```c
  // Example
  Matrix<double, 0> m0{1};
  Matrix<double, 1> m1{1, 2};
  Matrix<double, 2> m2{ {1, 2}, {3, 4} };
  Matrix<double, 3> m3{ { {1, 2}, {3, 4} }, { {5, 6}, {7, 8} } };
```

Similarly, we can assign a `Matrix` object using:
+ Another `Matrix`
+ `MatrixRef`. Again, a `MatrixRef` behaves just like a `Matrix` except that it
refers to a `Matrix`, and is typically a sub-Matrix such as a row or a column,
rather than owning its elements.
```c
  // Example:
  Matrix<double, 1> m1{1, 2, 3};
  auto mr1 = m1(slice(1));
  Matrix<double, 1> m1sub = mr1;

  Matrix<double, 2> m2{ {1, 2, 3}, {4, 5, 6}, {7, 8, 9} };
  auto mr2 = m2(slice(1), slice(1));
  Matrix<double, 2> m2sub = mr2;

  Matrix<double, 3> m3{ { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} },
                        { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} },
                        { {1, 2, 3}, {4, 5, 6}, {7, 8, 9} } };
  auto mr3 = m3(slice(1), slice(1), slice(1));
  Matrix<double, 3> m3sub = mr3;
```
+ `MatrixInitializer`, which is a nested `std::initializer_list`.
```c
  // Example
  Matrix<double, 0> m0 = {1};
  Matrix<double, 1> m1 = {1, 2};
  Matrix<double, 2> m2 = { {1, 2}, {3, 4} };
  Matrix<double, 3> m3 = { { {1, 2}, {3, 4} }, { {5, 6}, {7, 8} } };
```

The complete source file for this section can be find in file
[02_construction_and_assignment.cc](https://github.com/statslabs/matrix/blob/master/examples/02_construction_and_assignment.cc).

### Subscripting and Slicing

A `Matrix<T, N>` can be accessed through subscripting (to elements or rows),
through rows and columns, or through slices (parts of rows or columns).

| `Matrix<T, N>` Access    | Description                                                                                |
|--------------------------|--------------------------------------------------------------------------------------------|
| `m.row(i)`               | Row i of m; a `MatrixRef<T, N-1>`                                                          |
| `m.col(i)`               | Column i of m; a `MatrixRef<T, N-1>`                                                       |
| `m.rows(i, j)`           | Row i to row j of m; a `MatrixRef<T, N>`                                                   |
| `m.cols(i, j)`           | Column i to column j of m; a `MatrixRef<T, N>`                                             |
| `m[i]`                   | C-style subscripting: `m.row(i)`                                                           |
| `m(i,j)`                 | Fortran-style element access: `m[i][j]`; a `T&`; <br> the number of subscripts must be `N` |
| `m(slice(i,n),slice(j))` | Submatrix access with slicing: a `MatrixRef<T, N>`; <br> `slice(i,n)` is elements `[i:i+n)` of the subscript's dimension; <br> `slice(j)` is elements `[i:max)` of subscript's dimension; `max`is the dimension's extent; the number of subscripts must be `N`|
| `m.subvec(first_index, last_index)`                  | Subvector of m (available only when N == 1, i.e., m is a vector); a `MatrixRef<T, 1>` |
| `m.submat(first_row, first_col, last_row, last_col)` | Submatrix of m (available only when N == 2, i.e., m is a matrix); a `MatrixRef<T, 2>` |

```c
  // Example
  Matrix<int, 2> m{ {01, 02, 03}, {11, 12, 13} };

  cout << "m = " << m << endl;

  m(1, 2) = 99;  // overwrite the element in row 1 column 2; that is 13

  //  auto d1 = m(1);      // error: too few subscripts
  //  auto d2 = m(1,2,3);  // error: too many subscripts

  cout << "m = " << m << endl;

  Matrix<int, 2> m2{ {01, 02, 03}, {11, 12, 13}, {21, 22, 23} };

  auto m22 = m2(slice{1, 2}, slice{0, 3});

  cout << "m2 = " << m2 << endl;
  cout << "m22 = " << m22 << endl;

  m2(slice{1, 2}, slice{0, 3}) = { {111, 112, 113}, {121, 122, 123} };

  cout << "m2 = " << m2 << endl;

  Matrix<int, 2> m3 = { {01, 02, 03}, {11, 12, 13}, {21, 22, 23} };

  auto m31 = m3(slice{1, 2}, 1);  // m31 becomes { {12}, {22} }
  auto m32 = m3(slice{1, 2}, 0);  // m32 becomes { {11}, {21} }
  auto x = m3(1, 2);              // x == 13

  cout << "m31 = " << m31 << endl;
  cout << "m32 = " << m32 << endl;
  cout << "x = " << x << endl;
```

The complete source file for this section can be find in file
[03_subscripting_and_slicing.cc](https://github.com/statslabs/matrix/blob/master/examples/03_subscripting_and_slicing.cc).

### Matrix Arithmetic Operations

| (Member) function / Operator | Description |
| + , += | Addition of two objects |
| - , -= | Subtraction of one object from another or negation of an object |
| * , *= | Element-wise multiplication of an object by another object or a scalar |
| / , /= | Element-wise division of an object by another object or a scalar |
|matmul(m1, m2)| Matrix multiplication of two objects |
|matmul_n(m1, m2, m3)| Matrix multiplication of more than two objects |

```c
// Example
Matrix<int, 2> m1{ {1, 2, 3}, {4, 5, 6} };  // 2-by-3
Matrix<int, 2> m2{m1};                      // copy
m1 *= 2;                                    // scale: { {2,4,6}, {8,10,12} }
Matrix<int, 2> m3 = m1 + m2;                // add: { {3,6,9}, {12,15,18} }
Matrix<int, 2> m4{ {1, 2}, {3, 4}, {5, 6} };// 3-by-2
Matrix<int, 2> m5 = matmul(m1, m4);         // multiply: { {44,56}, {98,128} }
```

The complete source file for this section can be find in file
[04_matrix_arithmetric_operations.cc](https://github.com/statslabs/matrix/blob/master/examples/04_matrix_arithmetric_operations.cc).

## More (Member) Functions for Matrix

### Generated Vectors/Matrices/Cubes
### Functions of Vectors/Matrices/Cubes
### Decompositions, Factorisations, Inverses and Equation Solvers

## BLAS interface for Matrix

### BLAS Level 1 Routines and Functions
### BLAS Level 2 Routines and Functions
### BLAS Level 3 Routines and Functions

