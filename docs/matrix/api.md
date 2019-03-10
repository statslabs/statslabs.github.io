---
layout: page
title: matrix
permalink: /matrix/
---

* Contents
{:toc}

## Overview

Statslabs::matrix is the fundamental package of Statslabs for statistical computing in C++. The Statslabs::matrix library code is based on the matrix design chapter in 'The C++ Programming Language (4th Edition)' and provides:
  + A Matrix Template: Construction and Assignment; Subscripting and Slicing
  + Matrix arithmetic operations: Scalar Operations; Additions; Multiplication
  + Matrix Implementation: slice; MatrixSlice; MatrixRef; Matrix List Initialization; Matrix Access; Zero-Dimensional Matrix
  + An interface to Intel(R) MKL BLAS operations which apply to the Matrix template

## Prerequisites

    CMake >= 3.6.0
    Intel Math Kernel Library (Intel MKL) or OpenBLAS
   
## Installation on Ubuntu / macOS
1. Clone the repository.
   ```sh
   git clone git@github.com:statslabs/matrix.git
   ```
2. Configure the project.
   ```sh
   cd matrix
   mkdir build && cd build
   cmake ..
   ```
3. Compile and install the library.
   ```sh
   make
   make install
   ```

## Example program
`examples/demo.cc`:
```c
#include <iostream>
#include "slab/matrix.h"

using namespace std;
using namespace slab;

int main() {
  mat A = {
      {1, 2, 3},
      {4, 5, 6},
      {7, 8, 9}
  };

  mat B = {
      {1, 2, 3},
      {4, 5, 6},
      {7, 8, 9}
  };

  // Element-wise addition
  cout << "A + B = " << A + B << endl;

  // Element-wise subtraction
  cout << "A - B = " << A - B << endl;

  // Element-wise multiplication
  cout << "A * B = " << A * B << endl;

  // Element-wise division
  cout << "A / B = " << A / B << endl;

  // Matrix multiplication
  cout << "matmul(A, B) = " << matmul(A, B) << endl;

  return 0;
}
```

## Integration of Matrix in your own project
To make the project simple enough, we will create a CMake project for `demo.cc`.

1. Make a project folder.
   ```sh
   mkdir example && cd example
   ```

2. Create `demo.cc` and `CMakeLists.txt` in the project folder where file `CMakeLists.txt` should look like:
   ```cmake
   cmake_minimum_required(VERSION 3.0)
   project(example)
   set(CMAKE_CXX_STANDARD 11)
   add_executable(example demo.cc)

   find_package(Matrix REQUIRED)
   target_link_libraries(example Statslabs::Matrix)
   ```

## API Documentation

### Matrix<T, N> / MatrixRef<T, N> Classes

+ `Matrix<T, N>` is an `N`-dimensional dense matrix of some value type
`T`, with element stored in row-major ordering (i.e., row by row).

+ `MatrixRef<T, N>` is basically a clone of `Matrix<T, N>` which
simply points to the elements of "its" `Matrix` and is normally used
to represent sub-`Matrix`es.

+ The element type must be something we can store. Some typical types
for T include `double`, `float`, `std::complex<double>`,
`std::complex<float>`, `int` and `unsigned int`. When you use BLAS
interface defined in this library, only the first four types are
supported -- namely `double`, `float`, `std::complex<double>` and
`std::complex<float>`.

+ For convenience the following typedefs have been defined for `Matrix<T, N>`:

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

+ In this documentation the `mat` type is used for convenience and `sub_mat` is used to denote a `MatrixRef<double, 2>`; it is possible to use other types and dimensions instead, e.g., `fcube`.

+ (Optional) It is also possible to use some other non-typical types, for example:
```c
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
Matrix arithmetic doesn’t have exactly the same mathematical properties as the matrix with typical value types, 
so we must be careful how we use such a matrix.

#### Construction and Assignment:
A `Matrix<T, N>` can be intialized by 

+ `mat()` -- default constructor:
```c
  // 17 dimensions (no elements so far)
  Matrix<complex<double>, 17> m17;
```

+ `mat(mat)` -- a `Matrix<T, N>`.
(Advanced) Internally, a Matrix<T, N> has two private data members – a std::vector<T> named elem_ and a MatrixSlice<N> named desc_. Using a std::vector<T> to hold the elements relieves us from concerns of memory management and exception safety. A MatrixSlice<N> holds the sizes necessary to access the elements as an N-dimensional matrix. The default copy and move operations have just the right semantics: member-wise copy or move of desc_ (matrix slice descriptor defining subscripting) and the elem_.

+ `mat(sub_mat)` -- a `MatrixRef<T, N>`. Again, a `MatrixRef<T, N>` behaves just like a `Matrix<T, N>`
except that it refers to a `Matrix<T, N>`, and is typically a sub-Matrix such as a row or a column, rather than owning 
its elements.

+ `mat(MatrixInitializer)` -- a nested `std::initializer_list`:
```c
  Matrix<double, 0> m0{1};           // zero dimensions: a scalar
  Matrix<double, 1> m1{1, 2, 3, 4};  // one dimension: a vector (4 elements)
  Matrix<double, 2> m2{              // two dimensions (4*3 elements)
      {00, 01, 02, 03},  // row 0
      {10, 11, 12, 13},  // row 1
      {20, 21, 22, 23}   // row 2
  };
```

+ `mat(n_rows, n_cols, n_...)` -- pre-specified dimensions:
```c
  // three dimensions (4*7*9 elements), all 0-initialized
  Matrix<double, 3> m3(4, 7, 9);
```

Note: As for `std::vector`, we use `()` to specify sizes and `{}` to specify element values. The number rows must match the 
specified number of dimensions and the number of elements in each dimension (each column) must match. For example:
```c
  Matrix<char, 2> mc1(2, 3, 4);  // error: two many dimension sizes

  Matrix<char, 2> mc2{
      {'1', '2', '3'}  // note: this one should be okay
  };

  Matrix<char, 2> mc3{
      {'1', '2', '3'},
      {'4', '5'}  // error: element missing for third column
  };
```

#### Subscripting and Slicing

A `Matrix<T, N>` can be accessed through subscripting (to elements or rows), through rows and columns,
or through slices (parts of rows or columns).

| `Matrix<T, N>` Access    | Description                                                                                |
|--------------------------|--------------------------------------------------------------------------------------------|
| `m[i]`                   | C-style subscripting: `m.row(i)`                                                           |
| `m(i,j)`                 | Fortran-style element access: `m[i][j]`; a `T&`; <br> the number of subscripts must be `N` |
| `m.row(i)`               | Row i of m; a `MatrixRef<T, N-1>`                                                          |
| `m.col(i)`               | Column i of m; a `MatrixRef<T, N-1>`                                                       |
| `m.rows(i, j)`           | Row i to row j of m; a `MatrixRef<T, N>`                                                   |
| `m.cols(i, j)`           | Column i to column j of m; a `MatrixRef<T, N>`                                             |
| `m(slice(i,n),slice(j))` | Submatrix access with slicing: a `MatrixRef<T, N>`; <br> `slice(i,n)` is elements `[i:i+n)` of the subscript's dimension; <br> `slice(j)` is elements `[i:max)` of subscript's dimension; `max`is the dimension's extent; the number of subscripts must be `N`|
| `m.subvec(first_index, last_index)`                  | Subvector of m (available only when N == 1, i.e., m is a vector); a `MatrixRef<T, 1>` |
| `m.submat(first_row, first_col, last_row, last_col)` | Submatrix of m (available only when N == 2, i.e., m is a matrix); a `MatrixRef<T, 2>` |

Example:
```c
  Matrix<double, 2> m{                 // two dimensions (4*3 elements)
      {00, 01, 02, 03},  // row 0
      {10, 11, 12, 13},  // row 1
      {20, 21, 22, 23}   // row 2
  };
  cout << "\nm = " << m << endl;       // print a Matrix

  Matrix<double, 1> v = m[1];

  double dval1 = m(1, 2);              // 12
  double dval2 = m[1][2];              // 12
  double dval3 = v[2];                 // 12
```

#### Matrix Arithmetic Operations

| (Member) function / Operator | Description |
| + , += | Addition of two objects |
| - , -= | Subtraction of one object from another or negation of an object |
| * , *= | Element-wise multiplication of an object by another object or a scalar |
| / , /= | Element-wise division of an object by another object or a scalar |
|matmul(m1, m2)| Matrix multiplication of two objects |
|matmul_n(m1, m2, m3)| Matrix multiplication of more than two objects |

### Data Member and Member Functions

`Matrix<T, N>` has its number of dimensions (its `order()`) specified as a template argument (here, `N`).
Each dimension has a number of elements (its `extent()`) deduced from the initializer list or specified
as a Matrix constructor argument using the `()` notation. The total number of elements is referred to
as its `size()`. For example:
```c
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

The complete source file for this section can be find in file [01_basic_matrix_uses.cc](https://github.com/statslabs/matrix/blob/master/examples/01_basic_matrix_uses.cc).