---
layout: page
title: API Documentation for Matrix
permalink: matrix
---

* Contents
{:toc}

## 1. Basic Matrix Use

`Matrix<T, N>` is `N`-dimensional matrix of some value type `T`. It can be initialized by

+ nested std::initializer_list:
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

+ or nothing:
```c
  // 17 dimensions (no elements so far)
  Matrix<complex<double>, 17> m17;
```

`Matrix<T, N>` has its number of dimensions (its `order()`) specified as a template argument (here, N).
Each dimension has a number of elements (its `extent()`) deduced from the initializer list or specified
as a Matrix constructor argument using the () notation. The total number of elements is referred to
as its `size()`.

For convenience the following typedefs have been defined for `Matrix<T, N>`:

| T (value_type)         | Vector <br> (N == 1) | Matrix <br> (N == 2) | Cube <br> (N == 3) |
|------------------------|----------------------|----------------------|--------------------|
| `double`               | `vec`                | `mat`                | `cube`             |
| `double`               | `dvec`               | `dmat`               | `dcube`            |
| `float`                | `fvec`               | `fmat`               | `fcube`            |
| `std::complex<double>` | `cx_vec`             | `cx_mat`             | `cx_cube`          |
| `std::complex<double>` | `cx_dvec`            | `cx_dmat`            | `cx_dcube`         |
| `std::complex<float>`  | `cx_fvec`            | `cx_fmat`            | `cx_fcube`         |
| `unsigned int`         | `uvec`               | `umat`               | `ucube`            |
| `int`                  | `ivec`               | `imat`               | `icube`            |

## 2. Construction and Assignment

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