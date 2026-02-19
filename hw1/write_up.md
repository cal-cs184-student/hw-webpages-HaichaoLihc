# task 1

I rasterize triangles by first finding the triangle’s bounding box (min/max x,y) and clamping it to the screen. Then I scan every pixel in that box and test the pixel center
(x+0.5, y+0.5) with three edge functions. If all three edge values have the inside sign, I color that pixel. I update edge values incrementally across x/y, so each next test
is just adds, and I write directly to sample_buffer.

This is no worse than standard bounding-box sampling because it does the same amount of traversal: one pass over pixels inside the bounding box, with constant work per
pixel. So time complexity is O(bounding_box_width * bounding_box_height), same as any method that checks each sample in the triangle’s bounding box.

![screenshot_2-17_0-42-0.png]

Special optimizations 

Because the edge function is affine in input x and y, I can precompute edge-function coefficients, using them to incrementally find the value of the edge funtion at each input pixel position instead of recomputing full expressions each sample.

Also, I determined winding sign once (from signed area) so the inside test is a single consistent sign check.

![Screenshot 2026-02-17 at 12.43.28 AM.png]

unoptimized

![Screenshot 2026-02-17 at 12.42.53 AM.png]

# Task 2

- For each triangle, I still find its bounding box.
- For each pixel in that box, I split the pixel into a small grid.
- I test each small sample point to see if it is inside the triangle.
- If inside, I color that sample in sample_buffer.
- After all drawing is done, I average all samples of each pixel and write that average to the final framebuffer.

Data structure:

- sample_buffer is a 1D array.
- Layout is: each pixel’s samples are stored together.
- Pixel (x, y) starts at (y * width + x) * sample_rate.

Why supersampling helps:

- Normal 1-sample drawing makes triangle edges look jagged.
- With many samples, edge pixels can be partly covered.
- Averaging those samples gives in-between colors on edges.
- That makes edges look smoother (less stair-step effect).

Pipeline changes I made:

- set_sample_rate and set_framebuffer_target now resize sample_buffer to width * height * sample_rate.
- fill_pixel now fills all samples of that pixel (for points/lines).
- rasterize_triangle now writes per-subsample instead of one color per pixel.
- resolve_to_framebuffer now averages each pixel’s samples into one final color.

Comparison of sample rate 1, 4, 16.

![screenshot_2-17_1-27-18.png]

![screenshot_2-17_1-27-20.png]

![screenshot_2-17_1-27-22.png]

# Task 3

For Task 3, I modified svg/transforms/robot.svg so cubeman is waving instead of standing in a neutral pose. I used hierarchical transforms to rotate and reposition the
limbs: the right arm is rotated sharply upward and the lower right arm is additionally bent to form a clear wave gesture. I then adjusted the left arm to angle downward
slightly so the pose feels asymmetric and more natural. I also added small rotations to the legs with slight horizontal offsets to suggest a weight shift, making the
character look balanced while waving rather than perfectly rigid. 

![screenshot_2-17_2-11-16.png]

### Extra credit

I added a viewport-rotation feature with two keys: J rotates the view by +5° and K rotates by -5°. To do this, I inserted one extra transform in the rendering pipeline. Before, points were mapped as SVG -> NDC -> screen; now they go SVG -> NDC -> rotated NDC -> screen. NDC is just a normalized intermediate coordinate space between SVG units and window pixels. I rotate in NDC around the center (first tranlsate to x=0.5,y=0.5, then rotate, then translate back), so the whole scene rotates like a camera view while pan/zoom still behave normally, and I applied the same transform to the canvas border so it stays aligned with the SVG.

![screenshot_2-17_2-16-5.png]