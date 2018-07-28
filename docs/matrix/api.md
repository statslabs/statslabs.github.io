---
layout: page
title: API Documentation for Matrix
permalink: matrix
---

`Matrix<T, N>` is `N`-dimensional matrix of some value type `T`.
For convenience the following typedefs have been defined:


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


A `Matrix` can be accessed through subscripting (to elements or rows), through rows and columns,
or through slices (parts of rows or columns).

| `Matrix<T, N>` Access    | Description                                                                                |
|--------------------------|--------------------------------------------------------------------------------------------|
| `m.row(i)`               | Row i of m; a `MatrixRef<T, N-1>`                                                          |
| `m.col(i)`               | Column i of m; a `MatrixRef<T, N-1>`                                                       |
| `m.rows(i, j)`           | Row i to Row j of m; a `MatrixRef<T, N-1>`                                                          |
| `m.cols(i, j)`           | Column i to Column j of m; a `MatrixRef<T, N-1>`                                                       |
| `m[i]`                   | C-style subscripting: `m.row(i)`                                                           |
| `m(i,j)`                 | Fortran-style element access: `m[i][j]`; a `T&`; <br> the number of subscripts must be `N` |
| `m(slice(i,n),slice(j))` | Submatrix access with slicing: a `MatrixRef<T, N>`; <br> `slice(i,n)` is elements `[i:i+n)` of the subscript's dimension; <br> `slice(j)` is elements `[i:max)` of subscript's dimension; `max`is the dimension's extent; the number of subscripts must be `N`|