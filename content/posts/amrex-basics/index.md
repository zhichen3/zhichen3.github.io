+++
title = "AMReX Basic Syntax"
author = ["zhi"]
date = 2026-06-16T00:00:00-04:00
tags = ["amrex"]
categories = ["tutorial"]
type = "posts"
draft = false
weight = 1002
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Dimensionality Macros (`AMReX_SPACE.H`)](#dimensionality-macros--amrex-space-dot-h)
- [ParallelDescriptor (`AMReX_ParallelDescriptor.H`)](#paralleldescriptor--amrex-paralleldescriptor-dot-h)
- [AMReX Array Types](#amrex-array-types)
- [IntVect (`AMReX_IntVect.H`)](#intvect--amrex-intvect-dot-h)
    - [Arithmetic Operations](#arithmetic-operations)
    - [Refinement and Coarsening](#refinement-and-coarsening)
- [IndexType (`AMReX_IndexType.H`)](#indextype--amrex-indextype-dot-h)
- [Box (`AMReX_Box.H`)](#box--amrex-box-dot-h)
- [Dim3 and XDim3](#dim3-and-xdim3)
- [RealBox (`AMReX_RealBox.H`)](#realbox--amrex-realbox-dot-h)
- [Geometry (`AMReX_Geometry.H`)](#geometry--amrex-geometry-dot-h)
- [BoxArray (`AMReX_BoxArray.H`)](#boxarray--amrex-boxarray-dot-h)
- [DistributionMapping (`AMReX_DistributionMapping.H`)](#distributionmapping--amrex-distributionmapping-dot-h)
- [BaseFab](#basefab)
- [FabArray, MultiFab and iMultiFab](#fabarray-multifab-and-imultifab)
- [MFIter without Tiling](#mfiter-without-tiling)
- [Loop Tiling](#loop-tiling)
    - [What is Loop Tiling?](#what-is-loop-tiling)
    - [MFIter with Tiling](#mfiter-with-tiling)
- [ParallelFor](#parallelfor)

</div>
<!--endtoc-->

[AMReX](https://github.com/amrex-codes/amrex) is a public software framework designed for building
massively parallel block-structured adaptive mesh refinement applications.
Here I keep track of the basic syntax or classes used in AMReX
for my personal notes.

**Most are copied and paste from [AMReX documentaion](https://amrex-codes.github.io/amrex/docs_html/Introduction.html).**

Typing them down helps me go through the documentation
and filter out unnecessary information for my personal preference.


## Dimensionality Macros (`AMReX_SPACE.H`) {#dimensionality-macros--amrex-space-dot-h}

For a given problem, AMReX stores the dimensionality
of the problem via macro: `AMReX_SPACEDIM`.

-   `AMReX_D_EXPR(a,b,c)`: evaluates the expression a,b,c based on `AMReX_SPACEDIM`, i.e. evaluates expression a only for DIM=1, a and b for DIM=2. Used to evaluate different expressions
    ```c++
    AMREX_D_EXPR(vect[0] *= s, vect[1] *= s, vect[2] *= s);
    ```
-   `AMREX_D_DECL(a,b,c)`: expands to a comma-separated list of, 1, 2,
    or 3 arguments based on `AMReX_SPACEDIM`. Mainly used to write portable
    function calls and declarations.
    ```c++
    return IntVect(AMREX_D_DECL(p[0] + s, p[1] + s, p[2] + s));
    ```
    Then it evaluates to `IntVect(p[0]+s)` or `IntVect(p[0]+s, p[1]+s)`, etc...
-   `AMREX_D_TERM(a,b,c)`: expands to a whitespace-separated list of, 1, 2,
    or 3 arguments based on `AMReX_SPACEDIM`.
    ```c++
    vol = AMREX_D_TERM(dx, *dy, *dz);
    ```
    Then it evaluates to `vol=dx` or `vol=dx*dy` or `vol=dx*dy*dz`.
-   `AMReX_D_PICK(a,b,c)`: expands to a single result equal to the
    1st, 2nd, or 3rd arguments of the call based on `AMReX_SPACEDIM`.
    ```c++
    maxsize = AMREX_D_PICK(1024, 128, 32);
    ```
    This evaluates to 1024, 128, **OR** 32.

---


## ParallelDescriptor (`AMReX_ParallelDescriptor.H`) {#paralleldescriptor--amrex-paralleldescriptor-dot-h}

Mainly used functions are

```c++
// Return the rank
int myproc = ParallelDescriptor::MyProc();

// Return the number of processes
int nprocs = ParallelDescriptor::NProcs();

if (ParallelDescriptor::IOProcessor()) {
    // Only the I/O process executes this
    // Mainly used to do printing
    std::cout << "Some info..." << std::endl;
}

int ioproc = ParallelDescriptor::IOProcessorNumber();  // I/O rank

// Wait until all MPI ranks have reached here.
// Ensure all ranks have finished some operation before continuing.
ParallelDescriptor::Barrier();

// Broadcast 100 ints from the I/O Processor
Vector<int> a(100);
ParallelDescriptor::Bcast(a.data(), a.size(),
                    ParallelDescriptor::IOProcessorNumber())

// See AMReX_ParallelDescriptor.H for many other Reduce functions
ParallelDescriptor::ReduceRealSum(x);
```

---


## AMReX Array Types {#amrex-array-types}

-   Vector (`AMReX_Vector.H`): basically `std::vector` but provides bound-checking
    when compiled with `DEBUG=TRUE`.
    ```c++
    amrex::Vector<amrex::Real> myvector {1.0_rt, 2.0_rt};
    ```
-   Array (`AMReX_Array.H`): basically `std::array`. 0-based index 1D array.
    ```c++
    amrex::Array<amrex::Real, 3> arr {1.0_rt, 2.0_rt, 3.0_rt};
    ```
-   GpuArray: Array marked with `AMREX_GPU_HOST_DEVICE`, so that it works on both HOST and DEVICE.
    But modern C++ after C++11 should make `std::array` compatible on both HOST and DEVICE.
    But it is still safer to use this. Access via `arr[2]`.
    ```c++
    amrex::GpuArray<amrex::Real, 3> arr {1.0_rt, 2.0_rt, 3.0_rt};
    ```
-   Array1D, Array2D, Array3D: GPU safe multidimensional views array in contiguous memory that
    can have non-zero based index. Access via `arr(1,2)`.
    ```c++
    amrex::Array2D<amrex::Real, 1, 2, 1, 3, amrex::Order::C> arr
                                        {{1.0_rt, 2.0_rt, 3.0_rt,
                                          4.0_rt, 5.0_rt, 6.0_rt}};
    ```

---


## IntVect (`AMReX_IntVect.H`) {#intvect--amrex-intvect-dot-h}

`IntVect`: integer vector in `AMREX_SPACEDIM` dimensional space.
It essentially records the index representing a given cell.

```c++
amrex::IntVect iv(AMREX_D_DECL(i, j, k));
iv[0] == i;
iv[1] == j;
iv[2] == k;
```

It is essentially

```c++
int vec[AMREX_SPACEDIM];
```


### Arithmetic Operations {#arithmetic-operations}

```c++
IntVect iv(AMREX_D_DECL(19, 0, 5));
IntVect iv2(AMREX_D_DECL(4, 8, 0));
iv += iv2;  // iv is now (23,8,5)
iv *= 2;    // iv is now (46,16,10);
```


### Refinement and Coarsening {#refinement-and-coarsening}

AMR codes often need to deal with refinement and coarsening.
The refinement operation can be done simply using the
multiplication operation or , i.e.

```c++
IntVect iv(AMREX_D_DECL(127,127,127));
IntVect rr(AMREX_D_DECL(2,2,2,));

// Refine all component by 2
iv *= 2;

// Or use refine function?
iv.refine(rr);
```

For coarsening, one should ideally use builtin `coarsen` function.
For example, `i=-1/2` would give `i=0`, but what we usually want is `i=-1`.

```c++
IntVect iv(AMREX_D_DECL(127,127,127));
IntVect coarsening_ratio(AMREX_D_DECL(2,2,2));
iv.coarsen(2);                 // Coarsen each component by 2
iv.coarsen(coarsening_ratio);  // Component-wise coarsening

// Return an IntVect w/o modifying iv
const auto& iv2 = amrex::coarsen(iv, 2);

// iv not modified
IntVect iv3 = amrex::coarsen(iv, coarsening_ratio);
```

---


## IndexType (`AMReX_IndexType.H`) {#indextype--amrex-indextype-dot-h}

This class defines whether an index is cell-based or node-based along
each dimesion. Default is to use cell-based.

One can construct `IndexType` with an `IntVect` with:

-   _0_: Cell-centered
-   _1_: Node-centered

<!--listend-->

```c++
// Node centered in x-dir and cell-centered in y and z-dir
IndexType xface(IntVect{AMREX_D_DECL(1,0,0)});
```

This class provides other useful functions to tell whether its node or cell centered.

```c++
// True if the IndexType is cell-based in all directions.
bool cellCentered () const;

// True if the IndexType is cell-based in dir-direction.
bool cellCentered (int dir) const;

// True if the IndexType is node-based in all directions.
bool nodeCentered () const;

// True if the IndexType is node-based in dir-direction.
bool nodeCentered (int dir) const;
```

---


## Box (`AMReX_Box.H`) {#box--amrex-box-dot-h}

A `Box` defines a rectangular indexing region of `AMREX_SPACEDIM` dimension.
It has 2 components:

1.  An `IndexType` defining whether it is cell-centered or node-centered
2.  Two `IntVect` defining the lower and upper corners of the rectangular region.

A typical way of defining the `Box` is the following

```c++
IntVect lo(AMREX_D_DECL(64,64,64)); // Lower corner
IntVect hi(AMREX_D_DECL(127,127,127)); // Upper corner
IndexType typ({AMREX_D_DECL(1,1,1)}); // Node center in all dir

// Construct a cell-centered Box by default
Box cc(lo,hi);

// Construct a nodal Box, where we explicitly pass in IndexType
Box nd(lo,hi+1,typ);
```

We can convert `Box` from cell-centered to node-centered via

```c++
// Construct all cell-centered box by default
Box b0 ({64,64,64}, {127,127,127});

// Convert cell-centered box to node-centered.
Box b1 = surroundingNodes(b0);  // A new Box with type (node, node, node)
Print() << b1;                  // ((64,64,64) (128,128,128) (1,1,1))
Print() << b0;                  // Still ((64,64,64) (127,127,127) (0,0,0))

// Convert the node-centered box to cell-centered again
Box b2 = enclosedCells(b1);     // A new Box with type (cell, cell, cell)
if (b2 == b0) {                 // Yes, they are identical.
    Print() << "b0 and b2 are identical!\n";
 }

// Use convert() to not have uniform cell- or node-centered box
Box b3 = convert(b0, {0,1,0});  // A new Box with type (cell, node, cell)
Print() << b3;                  // ((64,64,64) (127,128,127) (0,1,0))

b3.convert({0,0,1});            // Convert b0 to type (cell, cell, node)
Print() << b3;                  // ((64,64,64) (127,127,128) (0,0,1))

b3.surroundingNodes();          //  ((64,64,64) (128,128,128) (1,1,1))
b3.enclosedCells();             //  ((64,64,64) (127,127,127) (0,0,0))
```

Sometimes given a `Box`, we want to know its corners. There are multiple ways
to do this

```c++
Box bx ({64,64,64}, {127,127,127});

// Get the small end (lower corner) of the Box
IntVect lo = bx.smallEnd();

 // Get the big end (upper corner) along y-dir
bx.bigEnd(1);

// Get const pointer the lower end and upper end (Legacy interface)
const int* lo = bx.loVect();
const int* hi = bx.hiVect();

// Use lbound and ubound to get lower and upper corner
// they return Dim3 instead of IntVect -- prefered for GPU access
const Dim3 lo = lbound(bx);
const Dim3 hi = ubound(bx);
```

Lastly, we want to do refinement and coarsen of a given `Box`.
Note that the behavior depends on whether the `Box` is cell-centered or node-centered.
A refined `Box` always covers the same physical domain as the original `Box`.
A coarsened `Box` does **NOT** always cover the same physical domain -- see example below.

```c++
// Cell-centered Box
Box ccbx ({16,16,16}, {31,31,31});
ccbx.refine(2);
Print() << ccbx;              // ((32,32,32) (63,63,63) (0,0,0))
Print() << ccbx.coarsen(2);   // ((16,16,16) (31,31,31) (0,0,0))

// Node-centered Box
Box ndbx ({16,16,16}, {32,32,32}, {1,1,1});
ndbx.refine(2);
Print() << ndbx;              // ((32,32,32) (64,64,64) (1,1,1))
Print() << ndbx.coarsen(2);   // ((16,16,16) (32,32,32) (1,1,1))

// Node-centered in x-dir
Box facebx ({16,16,16}, {32,31,31}, {1,0,0});
facebx.refine(2);
Print() << faceb x;           // ((32,32,32) (64,63,63) (1,0,0))
Print() << facebx.coarsen(2); // ((16,16,16) (32,31,31) (1,0,0))

// Uncoarsenable Box
Box uncbx ({16,16,16}, {30,30,30});
Print() << uncbx.coarsen(2);  // ((8,8,8), (15,15,15));
Print() << uncbx.refine(2);   // ((16,16,16), (31,31,31));
                              // Different from the original!
```

One can also expand a given box -- this is almost always used to
get the ghost cells.

```c++
// Increase a box in all directions by 2 cells,
// basically the lower corners are subtracted by 2 in all dir
// and upper corners are added by 2 in all dir
const Box& obx = amrex::grow(bx, 2);

// Only expand in y-dir by 4 cells.
const Box& qbx = amrex::grow(bx, 1, 4);

// Only grow the lower and upper corner
// One can also use negative values to do 'shrink'
Box gbx = amrex::growLo(bx, 1, 3);
Box gbx = amrex::growHi(bx, 1, 3);
```

---


## Dim3 and XDim3 {#dim3-and-xdim3}

These are plain structs are holds 3 fields: `x,y,z`.

```c++
struct Dim3 { int x; int y; int z; };
struct XDim3 { Real x; Real y; Real z; };
```

Note that they also have 3 fields regardless of dimensions.

And we see that `IntVect` can be easilly used to convert to `Dim3`.

```c++
IntVect iv(...);
Dim3 d3 = iv.dim3();
```

Given a `Box`, it is often used to get upper and lower bound,
and use them to write a loop

```c++
Box bx(...);
Dim3 lo = lbound(bx);
Dim3 hi = ubound(bx);
for (int k = lo.z; k <= hi.z; ++k) {
    for (int j = lo.y; j <= hi.y; ++j) {
        for (int i = lo.x; i <= hi.x; ++i) {
        }
    }
 }
```

---


## RealBox (`AMReX_RealBox.H`) {#realbox--amrex-realbox-dot-h}

A `RealBox` stores the _physical_ location in floating-point numbers of the
lower and upper corners of a rectangular domain, rather than storing
indices compared to `Box`.

```c++
// A rectangular box whose length is [-1.0, 1.0] for each dir.
RealBox rb({AMREX_D_DECL(-1.0,-1.0,-1.0)},
           {AMREX_D_DECL( 1.0, 1.0, 1.0)});

// Lower and upper corner in physical location.
// One can also specify dir.
auto lo = rb.lo();
auto hi = rb.hi();
```

---


## Geometry (`AMReX_Geometry.H`) {#geometry--amrex-geometry-dot-h}

With the `Box` and `RealBox` specified, we can now create the `Geometry`.

```c++
int n_cell = 64;

// This defines a Box with n_cell cells in each direction.
Box domain(IntVect{AMREX_D_DECL(       0,        0,        0)},
           IntVect{AMREX_D_DECL(n_cell-1, n_cell-1, n_cell-1)});

// This defines the physical box, [-1,1] in each direction.
RealBox real_box({AMREX_D_DECL(-1.0,-1.0,-1.0)},
                 {AMREX_D_DECL( 1.0, 1.0, 1.0)});

// This says we are using Cartesian coordinates
int coord = 0;

// This sets the boundary conditions to be doubly or triply periodic
Array<int,AMREX_SPACEDIM> is_periodic {AMREX_D_DECL(1,1,1)};

// This defines a Geometry object
Geometry geom(domain, real_box, coord, is_periodic);
```

Now we normally want a `Geometry` object for every AMR-level.
Hence, the `Box` and `RealBox` here represents the full-domain of that level.
Every level needs a `Geometry` object since stuff like `CellSize`, `volume` differs.
Here are some information you can get from `Geometry`

```c++
// Lower corner of the physical domain
// Returns GpuArray<Real,AMREX_SPACEDIM>
const auto problo = geom.ProbLoArray();

// y-direction upper corner
Real yhi = geom.ProbHi(1);

// Cell size for each direction.
const auto dx = geom.CellSizeArray();

// Index domain -- the Box containing upper lower indices
const Box& domain = geom.Domain();

// Is periodic in x-direction?
bool is_per = geom.isPeriodic(0);
```

---


## BoxArray (`AMReX_BoxArray.H`) {#boxarray--amrex-boxarray-dot-h}

`BoxArray` is a collection of `Boxes`.
You can create a `BoxArray` from a single `Box`, i.e. a single `Box`
representing the full domain. And then chop up the `BoxArray`
into smallers `Box`.

```c++
// Create a Box representing the full domain
Box domain(IntVect{0,0,0}, IntVect{127,127,127});

// Make a new BoxArray out of a single Box
BoxArray ba(domain);

// Get the number of boxes in the BoxArray, which is 1 for now.
Print() << "BoxArray size is " << ba.size() << "\n";

// Forces each Box in BoxArray to have sides <= maxSize
// so chop up the single Box into smaller boxes 64^3 cells
ba.maxSize(64);
Print() << ba; // Now shows 8 Boxes
```

One thing to note is that `BoxArray` is a _global data structure_.
This means that all processes knows about the whole `BoxArray`.
It holds all Boxes in a collection, but a single process in parallel
can only own _some_ of its Boxes via domain decomposition -- this
refers to the actual data, i.e. `FArrayBox` or FAB, see next few sections.

Similar to `Box`, `BoxArray` has an `IndexType`, and _all_ Boxes in the `BoxArray`
will have same type as the `BoxArray` itself. One can convert the
`IndexType` of a `BoxArray` via

```c++
// Create a cell-centered BoxArray from a single Box
BoxArray cellba(Box(IntVect{0,0,0}, IntVect{63,127,127}));

// Chop up BoxArray so that the length of each Box is < 64
cellba.maxSize(64);

// Make a copy
BoxArray faceba = cellba;

// Convert BoxArray to index type (cell, cell, node)
faceba.convert(IntVect{0,0,1});

// Return an all node BoxArray
const BoxArray& nodeba = amrex::convert(faceba, IntVect{1,1,1});

// We can access individual Box in BoxArray via []
// Note that it returns by VALUE instead of REFERENCE
Print() << cellba[0] << "\n";  // ((0,0,0) (63,63,63) (0,0,0))
Print() << faceba[0] << "\n";  // ((0,0,0) (63,63,64) (0,0,1))
Print() << nodeba[0] << "\n";  // ((0,0,0) (64,64,64) (1,1,1))
```

`BoxArray` also allows all Boxes to be refined or coarsened via

```c++
BoxArray ba(Box(IntVect{0,0,0}, IntVect{63,127,127}));
IntVect rr {2,2,4};

// Refine each Box in BoxArray in all dir by 2
ba.refine(2);

// Different refinement ratio for each dir
ba.refine(rr);

// Coarsen each Box in BoxArray by 2
ba.coarsen(2);
ba.coarsen(rr);
```

---


## DistributionMapping (`AMReX_DistributionMapping.H`) {#distributionmapping--amrex-distributionmapping-dot-h}

`DistributionMapping` uses an algorithm to describe which process
owns the data living on the domains specified by the Boxes in `BoxArray`.
One can construct a `DistributionMapping` given a `BoxArray`.

```c++
DistributionMapping dm {ba};
```

---


## BaseFab {#basefab}

`BaseFab` is a class containing multi-dimensional array data for a single `Box`.
This is like a step up of a `Box` where not only does it know about
lower and upper corner indices, it knows about the data specified in that region.
The dimensionality of the array is `AMREX_SPACEDIM+1`. The additional
dimension is to account for the number of components, say we have
density, momentum, and energy. Then we have 5 components.

```c++
// First create a single Box
Box bx(IntVect{-4,8,32}, IntVect{32,64,48});

// Specify the number of components
int numcomps = 4;

// Create the array
BaseFab<Real> fab(bx,numcomps);
```

Now usually we don't work `BaseFab`, but classes derived from it.

-   `FArrayBox` = `BaseFab<Real>`
-   `IArrayBox` = `BaseFab<int>`

One can easily get the associated `Box` and the number of components via

```c++
// Get the Box
Box bx = fab.box();

// Get the number of components
int ncomp = fab.nComp();
```

One can also change the `Box` and number of components a `FArrayBox` is defined on
via the `resize` method or `define` method. Both are accomplishes the same thing,
but use `define` when first constructing the object and `resize` when changing
an existing one.

```c++
// using define
fab.define(box, ncomp);

// using resize
fab.resize(box, ncomp);
```

One can do simple arithmetic operations using builtin functions

```c++
// Create the Box and specify number of components
Box box(IntVect{0,0,0}, IntVect{63,63,63});
int ncomp = 2;

// Create the ArrayBox holding the data
FArrayBox fab1(box, ncomp);
FArrayBox fab2(box, ncomp);

// Fill fab1 with 1.0
fab1.setVal(1.0);

// Multiply component 0 by 10.0
fab1.mult(10.0, 0);

// Fill fab2 with 2.0
fab2.setVal(2.0);

// For all components, fab2 <- a * fab1 + fab2
Real a = 3.0;
fab2.saxpy(a, fab1);
```

For more complicated operations, one needs to work the array directly.
The actual 4D array object in a `BaseFab` class is contained using `Array4`.
Here is an example

```c++
// Given FArrayBox and IArrayBox
FArrayBox afab(...), bfab(...);
IArrayBox ifab(...);

// Get the appropriate array containing data using fab.array()
Array4<Real> const& a = afab.array();
Array4<Real const> const b = bfab.const_array();
Array4<int const> m = ifab.array();

// Get the lower and upper bound indices of the Box
// and the number of componenets
Dim3 lo = lbound(a);
Dim3 hi = ubound(a);
int nc = a.nComp();

// Do a 4D loop loop
for (int n = 0; n < nc; ++n) {
    for (int k = lo.z; k <= hi.z; ++k) {
        for (int j = lo.y; j <= hi.y; ++j) {
            for (int i = lo.x; i <= hi.x; ++i) {
                if (m(i,j,k) > 0) {
                    a(i,j,k,n) *= 2.0;
                } else {
                    a(i,j,k,n) = 2.0*a(i,j,k,n) + 0.5*(b(i-1,j,k,n)+b(i+1,j,k,n));
                }
            }
        }
    }
 }

// Using ParallelFor instead of manual loop (GPU friendly)
amrex::ParallelFor(bx, NUM_STATE,
[=] AMREX_GPU_DEVICE (int i, int j, int k, int n) noexcept
{
     if (m(i,j,k) > 0) {
         a(i,j,k,n) *= 2.0;
     } else {
         a(i,j,k,n) = 2.0*a(i,j,k,n) + 0.5*(b(i-1,j,k,n)+b(i+1,j,k,n));
     }
}
```

---


## FabArray, MultiFab and iMultiFab {#fabarray-multifab-and-imultifab}

`FabArray` is a collection of `BaseFab`, similar to `BoxArray` is a collection of `Box`.
Similar to `BoxArray`. Similar to `BaseFab`, we usually don't work with `FabArray`
directly, but classes derived from it. This is usually

1.  `MultiFab` = `FabArray<FArrayBox>>` = `FabArray<BaseFab<Real>>`
2.  `iMultiFab` = `FabArray<IArrayBox>>` = `FabArray<BaseFab<int>>`

Now because `FabArray` is a parallel data structure where the data, i.e. `FArrayBox` or FAB,
are distributed among parallel processes. For each process, a `FabArray` contains only
the FAB objects owned by that process and only operates on that data.
For operations that require data owned by other processes, remote communications are needed,
so we `FabArray` needs `DistributionMapping` that specifies which process owns which Box.
So for each process, `FabArray` knows about the full `BoxArray`, but only the `FAB` that is assigned
to that process.

To create a `FabArray` or `MultiFab`,

```c++
// ba is BoxArray
// dm is DistributionMapping
// 4 components
int ncomp = 4;

// 1 ghost cell, so all Boxes have grown by 1 cell.
// If BoxArray has Box{(7,7,7) (15,15,15)}
// then the one used for FArrayBox in MultiFab
// is then Box{(6,6,6) (16,16,16)}.
int ngrow = 1;
MultiFab mf(ba, dm, ncomp, ngrow);
```

When creating the `MultiFab`, we use the individual `Box` in  `BoxArray`
to create the individual `FArrayBox`. So a `MultiFab` contains both
`BoxArray` and a collection of `FArrayBox` which also knows about its
corresponding `Box`. Now usually these two `Box` are the same, except
when we create `MultiFab` with ghost cells.
Internally the `FArrayBox` is created via

```c++
// Create the FAB but with grown Box
FArrayBox fab(amrex::grow(ba[0], ngrow),ncomp);
```

So the `Box` used to create the `FArrayBox` is enlarged, but the
`Box` stored in `MultiFab` stays unchanged and this is contains
the _valid zones_. And notice that `FArrayBox` itself doesn't have
the concept of ghost cells, it is simply allocated using a grown
`Box`.

For a given `MultiFab`, we can iterate through all the `FArrayBox`
via `MFIter`

```c++
for (MFIter mfi(mf); mfi.isValid(); ++mfi)
{
    // Valid Box without ghost cells
    const Box& valid = mfi.validbox();

    // With ghost cells
    const Box& fabbx = mfi.fabbox();

    // Get the Array
    Array4<Real const> const& a = mf.array(mfi);

    Print() << valid << "\n";
    Print() << fabbx << "\n";
}
```

We can also create a `MultiFab` from another `MultiFab`.

```c++
// mf0 is an already defined MultiFab
// Get properties like the BoxArray and DistributionMap
const BoxArray& ba = mf0.boxArray();
const DistributionMapping& dm = mf0.DistributionMap();
int ncomp = mf0.nComp();
int ngrow = mf0.nGrow();

// Create new MultiFabs with existing properties
MultiFab mf1(ba,dm,ncomp,ngrow);

// change the IndexType to face centered at x-dir for x-flux
MultiFab xflux(amrex::convert(ba, IntVect{1,0,0}), dm, ncomp, 0);
```

We can also do some basic operations on a `MultiFab` or between
`MultiFab` built with the _same_ `BoxArray` and `DistributionMapping`.

```c++
// Minimum value in component 3 of MultiFab mf
// no ghost cells included
Real dmin = mf.min(3);

// Maximum value in component 3 of MultiFab mf
// including 1 ghost cell
Real dmax = mf.max(3,1);

// Set all values to zero including ghost cells
mf.setVal(0.0);

// Add mfsrc to mfdst
MultiFab::Add(mfdst, mfsrc, sc, dc, nc, ng);

// Copy from mfsrc to mfdst
MultiFab::Copy(mfdst, mfsrc, sc, dc, nc, ng);

// MultiFab mfdst: destination
// MultiFab mfsrc: source
// int  sc   : starting component index in mfsrc for this operation
// int  dc   : starting component index in mfdst for this operation
// int  nc   : number of components for this operation
// int  ng   : number of ghost cells involved in this operation
//             mfdst and mfsrc may have more ghost cells
```

Now the `BoxArray` used to define the `MultiFab` is usuallly non-intersecting,
except when due to a nodal index type. But if `MultiFab` has ghost cells,
then `FArrayBox` which was constructed via a grown `BoxArray` can have overlaps,
even though the `BoxArray` is still the original one, which means that
the `Box` from `FArrayBox` is larger than the one in `BoxArray`.
This means that parallel communication is needed to fill ghost cells
from other `FArrayBox` which can be possibly owned by a different process.
To do this, use `FillBoundary`

```c++
MultiFab mf(...parameters omitted...);
Geometry geom(...parameters omitted...);

// Fill ghost cells for all components
// Periodic boundaries are not filled.
mf.FillBoundary();

// Fill ghost cells for all components
// Periodic boundaries are filled.
// This matters when grids are touching physical periodic boundaries.
// Note that if physical boundary is not periodic, like outflow,
// it just leaves them as garbage data since no grid overlap with ghost cells.
mf.FillBoundary(geom.periodicity());

// Fill 3 components starting from component 2
mf.FillBoundary(2, 3);
mf.FillBoundary(geom.periodicity(), 2, 3);
```

Another type of parallel communication is copying data from One
`MultiFab` to another `MultiFab` with a different or same  `BoxArray`
with a different `DistributionMapping`.
The copy will only be performed on the _regions of intersection_
as one would expect.

```c++
mfdst.ParallelCopy(mfsrc, compsrc, compdst, ncomp, ngsrc, ngdst, period, op);
```

---


## MFIter without Tiling {#mfiter-without-tiling}

We previously discussed manipulating individual `FArrayBox` in `MultiFab`
using `MFIter`. Let's discuss this in more detail. Here is an example

```c++
for (MFIter mfi(mf); mfi.isValid(); ++mfi) // Loop over grids
{
    // This is the valid Box of the current FArrayBox.
    // By "valid", we mean the original ungrown Box in BoxArray.
    const Box& box = mfi.validbox();

    // A reference to the current FArrayBox in this loop iteration.
    FArrayBox& fab = mf[mfi];

    // Obtain Array4 from FArrayBox.  We can also do
    //     Array4<Real> const& a = mf.array(mfi);
    Array4<Real> const& a = fab.array();

    // Call function f1 to work on the region specified by box.
    // Note that the whole region of the Fab includes ghost
    // cells (if there are any), and is thus larger than or
    // equal to "box".
    // So we don't have to worry about accessing data outside
    // of box when doing loops.
    f1(box, a);
}

// f1 operates on Box and can be something like
void f1 (Box const& bx, Array4<Real> const& a) {
   // Or use ParallelFor...
   const auto lo = lbound(bx);
   const auto hi = ubound(bx);
   const int ncomp = a.nComp();
   for (int n = 0; n < ncomp; ++n) {
     for (int k = lo.z; k <= hi.z; ++k) {
       for (int j = lo.y; j <= hi.y; ++j) {
         for (int i = lo.x; i <= hi.x; ++i) {
           a(i,j,k,n) = ...;
         }
       }
     }
   }
}
```

Here note that `MFIter` only loops over grid owned by this process.

We can also work with multiple `MultiFab` for a given loop. Note that
they need to have the same `DistributionMapping`, but not necessary
the same `BoxArray`, i.e. they can differ due to index types.

```c++
// U and F are MultiFabs
for (MFIter mfi(F); mfi.isValid(); ++mfi) // Loop over grids
{
    const Box& box = mfi.validbox();

    Array4<Real const> const& u = U.const_array(mfi);
    Array4<Real      > const& f = F.array(mfi);

    f2(box, u, f);
}
```

---


## Loop Tiling {#loop-tiling}

Let's first dicuss what is Loop Tiling


### What is Loop Tiling? {#what-is-loop-tiling}

Tiling, also known as cache blocking -- a loop transformation technique
for improving data locality. It transforms loops into tiles or smaller blocks
first and then do the looping within tiles.
The idea is that it minimizes reading data in memory.
Here is an online [webpage](https://open-catalog.codee.com/Glossary/Loop-tiling/) explaining loop tiling.

Before talking about loop tiling, let's first discuss
how CPU gets its data. Upon a memory request for CPU, it fetches the data
with following flow:

RAM (GBs) &rarr; L3 (8-32MB) &rarr; L2 (256KB-1MB) &rarr; L1 (32-64KB) &rarr; registers &rarr; CPU

Also note that RAM is basically a 2D block of bytes where it has _N_ number
of rows and _M_ number of columns. Here _M_ is the _cache line size_,
which is typically 64 bytes, i.e. each row has 64 memory addresses.
For example, a single precision number uses 4 bytes and double precision
uses 8 bytes.
Now when a CPU fetches the desired data, it gets all data of that
specific row contains the data requested, i.e. it gets 64 bytes
of data every single time. Now this is why it is faster to access data
adjacent in memory address, since the neighboring data are already loaded in cache.

The order at which CPU checks whether data exists is the reverse order
of the above. Each level is smaller, faster and closer to the CPU.
Now once the data gets filled out in those levels, it gets removed
or _data eviction_ following some policies. The most common one is
_Least Recently Used_ or LRU, which removes data that is not used in the
longest time.

Now let's consider this example

```c++
for (int i = 0; i < n; i++) {
  for (int j = 0; j < m; j++) {
    c[i][j] = a[i] * b[j];
  }
 }
```

When doing this loop, CPU first gets a[0] and then subsequently loads
in b[0], b[1], b[2], ..., b[m]. Now if array _b_ is large, then the earlier
elements of array b gets removed since it exceeds the cache size.
But if array _b_ is small enough so that all elements can live within cache
without it being evicted, then we maximize speed.

Now consider array _a_ is small and array _b_ is large so that we just
need to do tiling on _b_. This means we can do the following:

```c++
for (int jj = 0; jj < m; jj += TILE_SIZE) {
  for (int i = 0; i < n; i++) {
    for (int j = jj; j < MIN(jj + TILE_SIZE, m); j++) {
      c[i][j] = a[i] * b[j];
    }
  }
 }
```

Now the access pattern goes like:

-   (0,0) &rarr; (0,1) &rarr; ... &rarr; (0, TILE_SIZE-1) &rarr;
-   (1,0) &rarr; (1,1) &rarr; ... &rarr; (1, TILE_SIZE-1) &rarr;
-   ...
-   (n-1,0) &rarr; (n-1,1) &rarr; ... &rarr; (n-1, TILE_SIZE-1) &rarr;
-   (0,TILE_SIZE) &rarr; (0,TILE_SIZE+1) &rarr; ... &rarr; (0, 2\*TILE_SIZE-1) &rarr;
-   (1,TILE_SIZE) &rarr; (1,TILE_SIZE+1) &rarr; ... &rarr; (1, 2\*TILE_SIZE-1) &rarr;
-   ...

Now notice that we're accessing the same subset of array _b_,
from b[0] to b[TILZE_SIZE-1] over and over again. Assume
this is small enough, and so these elements never got evicted from cache.
Hence we eliminated the stall of waiting to fetch these data from RAM.
Now if array _a_ is also large, then we should do the same tiling on _a_,
which then looks like

```c++
for (int ii = 0; ii < n; ii += TILE_SIZE_I) {
  for (int jj = 0; jj < m; jj += TILE_SIZE_J) {
    for (int i = ii; i < MIN(n, ii + TILE_SIZE_I); i++) {
      for (int j = jj; j < MIN(m, jj + TILE_SIZE_J); j++) {
        c[i][j] = a[i] * b[j];
      }
    }
  }
}
```

In this case, we would want the total tile size, _TILE_SIZE_I \* TILE_SIZE_J_
to be small enough so that all these subset of array _a_ and _b_ can live in cache.
Now ideally we want the total size of the data for each tile to be smaller than
the L1 cache. However, we don't want the TILE_SIZE to be too small for two reasons:

1.  Loop iterations have a fixed cost -- loop overhead.
2.  Cache lines are underutilized -- fetch more data than actually used.

When should we use it?

1.  Iterating over the same dataset multiple times
2.  Cases with "wrong" memory access patterns that cannot be fixed with interchange
    of the loop index. An example is matrix tranposition
    ```c++
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < n; j++) {
        // access to array a is column-major
        // use tiling to reduce time between
        // consecutive access to neighboring of array a
        b[i][j] = a[j][i];
      }
    }
    ```


### MFIter with Tiling {#mfiter-with-tiling}

From previous example we see that the major downside
is that the syntax becomes ugly. But we can easily enable
tiling with the following synteax when looping over boxes.

```c++
//               * true *  turns on tiling
for (MFIter mfi(mf,true); mfi.isValid(); ++mfi) // Loop over tiles
{
    //                   tilebox() instead of validbox()
    const Box& box = mfi.tilebox();

    ...
}
```

We can also specify the tile size when defining `MFIter`,

```c++
// No tiling in x-direction. Tile size is 16 for y and 32 for z.
// The default tile size is IntVect(1024000,8,8)
for (MFIter mfi(mf,IntVect(1024000,16,32)); mfi.isValid(); ++mfi) {...}
```

Note if the tile size is larger than the grid size, then it means
that tiling is disabled in that direction. So the example above
has no tiling in x-dir.

---


## ParallelFor {#parallelfor}

ParallelFor allows you to not manually write out all the loops when
iterating over a `Box`. This works on either GPU or CPU so its convenient to use.
Here is an example

```c++
#ifdef AMREX_USE_OMP
#pragma omp parallel if (Gpu::notInLaunchRegion())
#endif
  for (MFIter mfi(mfa,TilingIfNotGPU()); mfi.isValid(); ++mfi)
  {
    const Box& bx = mfi.tilebox();
    Array4<Real> const& a = mfa[mfi].array();
    Array4<Real const> const& b = mfb[mfi].const_array();
    Array4<Real const> const& c = mfc[mfi].const_array();

    // Assume a single component.
    ParallelFor(bx, [=] AMREX_GPU_DEVICE (int i, int j, int k)
    {
      a(i,j,k) += b(i,j,k) * c(i,j,k);
    });
  }
```

Here are some other versions

```c++
// 1D for loop
ParallelFor(N, [=] AMREX_GPU_DEVICE (int i) { ... });

// 4D for loop
ParallelFor(box, numcomps,
            [=] AMREX_GPU_DEVICE (int i, int j, int k, int n) { ... });
```

---
