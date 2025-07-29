# 01. Image Layering & Optimization MAIN TAKEWAY:
      - A Docker image is built in layers, each representing one step in your Dockerfile. Layers stack like sheets of paper when you change
        one step, only that layer is rebuilt, speeding builds and reusing unchanged parts. Optimizing layer order and combining commands shrinks image size and accelerates builds.


# 02. What Is Image Layering?
      - An Image layer is a read-only filesystem snapshot created by each Dockerfile instructions (e.g., RUN, COPY).

      - When you build an image, Docker applies each instruction in sequence, producing one layer per step.


# 03. Real-World Analogy:
      - Imagine painting a wall in stages:
        a. Primer coat
        b. Base Color
        c. Accent stripes
        d. Sealer
      
      - Each coat is its own layer. If you change the accent stripes, you only need to redo that layer—not primer or base color. Likewise, Docker 
        only rebuilds the layer whose instruction changed.


# 04. How Layers Work
      a. Base Layer: From FROM ubuntu:20.04 or node:18-alpine.

      b. Instruction Layers:
         1. RUN apt-get update → Layer A
         2. RUN apt-get install curl → Layer B
         3. COPY . /app → Layer C
         4. RUN npm install → Layer D
      
      c. Final Layer: CMD ["node","app.js"]

      d. Docker caches layers. On rebuild, unchanged layers are reused, so only later layers rerun.


# 05. Why Layering Matters
      a. Build Speed: Cached layers skip re-execution.

      b. Storage Efficiency: Multiple images share common layers.

      c. Incremental Updates: Small changes rebuild minimal layers.


# 06. Image Optimization Techniques
      a. Order Instructions by Change Frequency
         - Place rarely changed commands first, and volatile ones later.
         If you edit code, only the last COPY layer and CMD rebuild, reusing earlier cached layers.
      
      b. Combine RUN Steps
         - Merging multiple RUN commands into one reduces layer count and removes intermediate files:

      c. Use Multi-Stage Builds
         - Separate build and runtime stages to keep final image small
         - The resulting image contains only the compiled binary, not Go toolchain layers.
      
      d. Choose Minimal Base Images
         - Use slim or Alpine variants to minimize base layer size (e.g., python:3.11-slim vs. python:3.11).



