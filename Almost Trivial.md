I asked CoPilot as AIs still do not draw this perfectly: I have known this projection for some 20 years, but lately made it really consistent, and now want to use it in automates, from now on; the following task I tried to make *maximally trivial*:

Sorry, two triangles is rhomb - four is square.

Please follow a simpler algorithm:
- Triangulate the sphere so that there are four times more triangles than squares are needed - 0D 1*1 (1 square); 1D 2*2; 2D 4*4; 3D 16*16 - dimensionality marks precision as power of 2, not dimension of the object.
- Triangulation depth can be selected.
- Draw just one black-white square field.
- To create squares: choose random point which connects corners of triangles. Count all triangles which touch it. You got one perfect square and it's *exactly four triangles and definitely so*. Now: at edges of this square, and corners, find it's neighgbours: you can travel edges and for each edge of a square, if the square at that way is not registered yet, take the edge-touching triangle, it's pointing at opposite side of the selected edge - take this opposite corner, and select all 4 triangles into a square.
  - Turn it to square field: now as you have all squares, choose one square as top-left square; from one side, count to four squares which follow in order, each next touching the same side of previous one; this is the first row: now, each cell which you selected, count with 90-degree edge compared to the initial edge to get this row, and naturally get a chessboard shape, n*n square of perfect squares.

Is this simple? Can you follow every step?
