Title: R - Reading binary files
Published: 3/11/2019
Tags: [R, Binary] 
---

I had to get some data out of files written by Fortran. The format was a large array of doubles. 

- numeric() for doubles
- integer() for integers 

readBin takes the input file and reads the binary data. If you want to read to the end of the file, don't give a dimension as 3rd parameter.

```R
#### Reading value field from file
z <- file("/BoundValueRemain.bin", "rb")
y <- readBin (z, numeric(), 51*50*50*15*13)
close(z)
dim(y) <- c(51, 50, 50, 15, 13)
```