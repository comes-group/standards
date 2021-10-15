The COMES Image Format
----------

CIF is a text-based image format designed to be as shitty and verbose as
possible.

All CIF images are either 24bpp or 32bpp, as indicated in the header.
Colors have 8 bits per channel, so RGB8 and RGBA8 formats are supported.
CIF images may have any arbitrary resolution.

Common rules
============

.. code-block:

 ws <- +' '
 nl <- +'\n'
 comma <- ',' * ws
 semicolon <- ';' * ws
 colon <- ':' * ws
 magic <- "CIF"

Language-specific keywords
==========================

Depending on the language chosen, certain keywords change in the format.

Polish
******

.. code-block::

 version <- "WERSJA"
 size <- "ROZMIAR"
 width <- "szerokość"
 height <- "wysokość"
 bpp <- "bitów_na_piksel"
 metadata <- "METADANE"
 node <- "WĘZEŁ"
 branch <- "GAŁĄŹ"
 leaf <- "LIŚĆ"

English
*******

.. code-block::

 version <- "VERSION"
 size <- "SIZE"
 width <- "width"
 height <- "height"
 bpp <- "bits_per_pixel"
 metadata <- "METADATA"
 node <- "NODE"
 branch <- "BRANCH"
 leaf <- "LEAF"

Number encoding
===============

Number encoding changes depending on flags passed to the coder.

compact
*******

Compact should be able to parse any arbitrary 64-bit number.

.. code-block::

 digit <- {'0'..'9'}
 int <- +digit

polish
******

Polish must be able to parse/reproduce numbers at least in the range 0..9999.

*PEG code for the ``int`` rule is not included because it quickly gets really
verbose. Implementation is an exercise for the reader :)*

english
*******

English should be able to parse/reproduce numbers at least in the range
0..9999. `fourty` is preferred over `forty`, but both can be implemented for
completeness (though `forty` is optional).

Image files
===========

Common header
*************

Every image file has a header which signifies metadata about the image.
This header must appear at the very top of the file.

.. code-block::

 separatedList(sep, rule) <- ?(rule * *(sep * rule))

 # Magic + flags must appear at the very first line of the file
 flag <- "polish" | "english" | "compact" | "quadtree"
 magicFlags <- magic * colon * separatedList(comma, flag) * nl

 # Then the version
 versiondef <- ver * ws * int * nl

 # Then the image size
 sizedef <-
   size * ws *
   width * colon * int * comma *
   height * colon * int * comma *
   bpp * colon * int *
   nl

 # Then any number of metadata definitions
 metakey <- +{'a'..'z', 'A'..'Z', '0'..'9', '_'}
 metavalue <- *(1 - nl)
 metadatadef <- metadata * metakey * colon * metavalue * nl
 metadatadefs <- *metadatadef

 # This is the rule for the full header:
 header <- magicFlags * versiondef * sizedef * metadatadefs * nl

Pixel format
************

Depending on whether the image is 24bpp or 32bpp, pixels may be encoded in
one of two ways:

.. code-block::

 if bpp == 24:
   pixel <- int * semicolon * int * semicolon * int
 elif bpp == 32:
   pixel <- int * semicolon * int * semicolon * int * semicolon * int

Image data
**********

Depending on whether quadtree mode is used or not, pixel data may be encoded
in one of the following ways:

Stream mode
~~~~~~~~~~~

Pixels in stream mode are arranged in top to bottom, left to right order.

.. code-block::

 imageData <- *(pixel * nl)

Quadtree mode
~~~~~~~~~~~~~

When quadtree mode is enabled, while coding the image the read/write buffer
must be enlarged to the nearest power of two. Anything out of bounds of the
image size is expected to be solid black (0, 0, 0) in 24bpp mode or
transparent (0, 0, 0, 0) in 32bpp mode.

.. code-block::

 # Quadtrees are built out of branch nodes and leaf nodes.
 # Each branch node has 4 children nodes that may be other branch nodes or
 # leaf nodes.
 # Each leaf node encodes a solid pixel color.

 branchNode <-
   node * ws * branch * nl *
   anyNode * nl *
   anyNode * nl *
   anyNode * nl *
   anyNode * nl

 leafNode <- node * ws * leaf * ws * pixel

 anyNode <- leafNode | branchNode

 imageData <- anyNode * nl

Tying it all together
*********************

Given the previously defined parsing rules, this is how a CIF file should be
read:

.. code-block::

  cif <- header * anyNode
