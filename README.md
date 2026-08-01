# Marching Squares

Contour detection for PPM images, written in C.

Marching Squares is the algorithm behind contour maps, terrain outlines and the
boundaries you see drawn around regions on a heatmap. It is the two-dimensional
version of Marching Cubes. I implemented it from scratch, including the image
parsing, the sample grid, all sixteen cell patterns and the drawing.

## How the algorithm works

The idea is to find the boundary between "inside" and "outside" without ever
tracing it directly. Instead you look at the image four points at a time and
decide, locally, what the boundary must look like there.

**First, upscale.** Each pixel in the source becomes a 4×4 block. Without this
the contour has nowhere to be drawn, since it needs space between the original
pixels.

**Then build a grid.** For every node in the grid, take the average brightness
of the pixels around it and compare it to a threshold. That gives a lattice
where each node is simply on or off.

**Classify each cell.** Take any four neighbouring nodes forming a square. Each
one is on or off, so there are 2⁴ = 16 possible arrangements. Every one of
those sixteen corresponds to a fixed pattern of line segments cutting through
that cell: a corner, a diagonal, a straight line across, or nothing at all.

**March.** Walk over every cell, work out which of the sixteen cases it is, and
draw that pattern. Because neighbouring cells share corners, the segments line
up and join into continuous contours across the whole image. Nothing ever
tracks the shape globally. It emerges from the local decisions.

The sixteen patterns are built once at the start and stored, so the marching
step is just a lookup rather than working them out each time.

## Using it

Commands are read one per line: read an image, resize it, build the grid, march,
write the result, exit. Reading a new image clears whatever was loaded before.
Invalid commands print an error and leave the current image untouched.

## Notes on how it works

The three colour channels are stored as three separate matrices rather than one
interleaved array. It makes the per-channel code simpler to follow, though it
does mean three allocations instead of one.

Grid nodes take the average of a block of pixels rather than a single sample.
Using one pixel makes the contours noisy, turning small variations in
brightness into stray fragments that do not connect to anything.

The sixteen cases are not written out one by one. They are generated at start-up
and grouped by how many corners are on, which cuts the amount of code roughly
in half.

Building the grid and printing it are separate functions, because marching needs
the grid but not the output.

## About

Written for a systems programming course at Politehnica University of
Bucharest, in first year.
