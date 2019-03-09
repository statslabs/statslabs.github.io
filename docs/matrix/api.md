---
layout: page
title: matrix
permalink: /matrix/
---

* Contents
{:toc}

This documentation is based on Ch29 "A Matrix Design" of the book "The C++ Programming Language" excluding all the implementation details.

## 1. Basic Matrix Use

`Matrix<T, N>` is an `N`-dimensional matrix of some value type `T`. It can be initialized by

+ a nested std::initializer_list:
```c
  Matrix<double, 0> m0{1};           // zero dimensions: a scalar
  Matrix<double, 1> m1{1, 2, 3, 4};  // one dimension: a vector (4 elements)
  Matrix<double, 2> m2{              // two dimensions (4*3 elements)
      {00, 01, 02, 03},  // row 0
      {10, 11, 12, 13},  // row 1
      {20, 21, 22, 23}   // row 2
  };
```

+ pre-specified dimensions:
```c
  // three dimensions (4*7*9 elements), all 0-initialized
  Matrix<double, 3> m3(4, 7, 9);
```

+ or default constructor:
```c
  // 17 dimensions (no elements so far)
  Matrix<complex<double>, 17> m17;
```

The element type must be something we can store. Some typical types for T include `double`, `float`, 
`std::complex<double>`, `std::complex<float>`, `int` and `unsigned int`. When you use BLAS or LAPACK interface defined 
in this library, only the first four types are supported -- namely `double`, `float`, `std::complex<double>` and 
`std::complex<float>`. For convenience the following typedefs have been defined for `Matrix<T, N>`:

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

It is also possible to use some other non-typical types, for example:
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

As for `std::vector`, we use `()` to specify sizes and `{}` to specify element values. The number rows must match the 
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

We can access `Matrix` elements by several forms of subscripting. For example:
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

The complete source file for this section can be find in file [01_basic_matrix_uses.cc](https://github.com/statslabs/matrix/blob/master/examples/01_basic_matrix_uses.cc).

## 2. Construction and Assignment

+ A `Matrix<T, N>` can be initialized by `Matrix<T, N>`.

  **Note**:
  Internally, a `Matrix<T, N>` has two private data members -- a `std::vector<T>` named `elem_` and a `MatrixSlice<N>` named `desc_`. 
  Using a `std::vector<T>` to hold the elements relieves us from concerns of memory management and exception safety.
  A `MatrixSlice<N>` holds the sizes necessary to access the elements as an `N`-dimensional matrix. 
  The default copy and move operations have just the right semantics: member-wise copy or move of `desc_` (matrix slice 
  descriptor defining subscripting) and the `elem_`.

+ A `Matrix<T, N>` can be initialized by `MatrixInilizer<T, N>`. Here `MatrixInilizer<T, N>` is a suitably nested initializer list.

+ A `Matrix<T, N>` can be initialized by extents. 

+ A `Matrix<T, N>` can be initialized by `MatrixRef<T, N>`. A `MatrixRef<T, N>` behaves just like a `Matrix<T, N>`
except that it refers to a `Matrix<T, N>`, and is typically a sub-Matrix such as a row or a column, rather than owning 
its elements.

The complete source file for this section can be find in file [02_construction_and_assignment.cc](https://github.com/statslabs/matrix/blob/master/examples/02_construction_and_assignment.cc)

## 3. Subscripting and Slicing

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