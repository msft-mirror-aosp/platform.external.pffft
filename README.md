# PFFFT: a pretty fast FFT.

## TL;DR

PFFFT does 1D Fast Fourier Transforms, of single precision real and
complex vectors. It tries do it fast, it tries to be correct, and it
tries to be small. Computations do take advantage of SSE1 instructions
on x86 cpus, Altivec on powerpc cpus, and NEON on ARM cpus. The
license is BSD-like.


## Why does it exist:

I was in search of a good performing FFT library , preferably very
small and with a very liberal license.

When one says "fft library", FFTW ("Fastest Fourier Transform in the
West") is probably the first name that comes to mind -- I guess that
99% of open-source projects that need a FFT do use FFTW, and are happy
with it. However, it is quite a large library , which does everything
fft related (2d transforms, 3d transforms, other transformations such
as discrete cosine , or fast hartley). And it is licensed under the
GNU GPL , which means that it cannot be used in non open-source
products.

An alternative to FFTW that is really small, is the venerable FFTPACK
v4, which is available on NETLIB. A more recent version (v5) exists,
but it is larger as it deals with multi-dimensional transforms. This
is a library that is written in FORTRAN 77, a language that is now
considered as a bit antiquated by many. FFTPACKv4 was written in 1985,
by Dr Paul Swarztrauber of NCAR, more than 25 years ago ! And despite
its age, benchmarks show it that it still a very good performing FFT
library, see for example the 1d single precision benchmarks
[here](http://www.fftw.org/speed/opteron-2.2GHz-32bit/). It is however not
competitive with the fastest ones, such as FFTW, Intel MKL, AMD ACML,
Apple vDSP. The reason for that is that those libraries do take
advantage of the SSE SIMD instructions available on Intel CPUs,
available since the days of the Pentium III. These instructions deal
with small vectors of 4 floats at a time, instead of a single float
for a traditionnal FPU, so when using these instructions one may expect
a 4-fold performance improvement.

The idea was to take this fortran fftpack v4 code, translate to C,
modify it to deal with those SSE instructions, and check that the
final performance is not completely ridiculous when compared to other
SIMD FFT libraries. Translation to C was performed with [f2c](
http://www.netlib.org/f2c/). The resulting file was a bit edited in
order to remove the thousands of gotos that were introduced by
f2c. You will find the fftpack.h and fftpack.c sources in the
repository, this a complete translation of [fftpack](
http://www.netlib.org/fftpack/), with the discrete cosine transform
and the test program. There is no license information in the netlib
repository, but it was confirmed to me by the fftpack v5 curators that
the [same terms do apply to fftpack v4]
(http://www.cisl.ucar.edu/css/software/fftpack5/ftpk.html). This is a
"BSD-like" license, it is compatible with proprietary projects.

Adapting fftpack to deal with the SIMD 4-element vectors instead of
scalar single precision numbers was more complex than I originally
thought, especially with the real transforms, and I ended up writing
more code than I planned..


## The code:

Only two files, in good old C, `pffft.c` and `pffft.h`. The API is very
very simple, just make sure that you read the comments in `pffft.h`.

This archive's source can be downloaded with git including the submodules:

```
git clone --recursive https://github.com/hayguen/pffft.git`
```

With `--recursive` the submodules for Green and Kiss-FFT are also fetched,
to use them in the benchmark. You can omit the `--recursive`-option.


## CMake:
The core are still the 2 files; that's everything you need.
There's now CMake support to build the static library `libPFFFT.a` from
these files, plus the additional `libFFTPACK.a` library. Later one's
sources are there anyway for the benchmark.


## Comparison with other FFTs:

The idea was not to break speed records, but to get a decently fast
fft that is at least 50% as fast as the fastest FFT -- especially on
slowest computers . I'm more focused on getting the best performance
on slow cpus (Atom, Intel Core 1, old Athlons, ARM Cortex-A9...), than
on getting top performance on today fastest cpus.

It can be used in a real-time context as the fft functions do not
perform any memory allocation -- that is why they accept a 'work'
array in their arguments.

It is also a bit focused on performing 1D convolutions, that is why it
provides "unordered" FFTs , and a fourier domain convolution
operation.


## Benchmark results

The benchmark results are stored in a separate git-repository:
See [https://github.com/hayguen/pffft_benchmarks](https://github.com/hayguen/pffft_benchmarks).

This is to keep the sources small.
