# Long read Assembly

Congratulations, we have now done the short read assembly. 
We can assemble the long reads, using the tool named **Flye**.

 **2. OLC (Overlap–Layout–Consensus)**
- Find overlaps between all pairs of reads.  
- Build a graph of overlaps.  
- Resolve paths to produce contigs.  
- Used for long-read assemblers.

# Theory on Long Read assembly

The Flye assembly is based on finding overlapping reads with variable length with high error tolerance.

Show with a scheme how **Flye** works.

The setup: The genome is a book that was shredded into strips, and the book repeats some paragraphs many times. The strips are the reads.
Step 1, draft chunks: You tape strips together into rough pages. Some are taped wrongly, but no text is lost.
Step 2, road map: You notice the repeated paragraphs and merge them into one shared stretch of road. The book becomes a map of roads and junctions.
Step 3, pick the road: A long strip that runs through a repeated paragraph, with text visible on both sides, shows which road leads in and which leads out.
