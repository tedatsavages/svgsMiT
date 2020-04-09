xR Skillbuilder Lesson Plan

1 - what do we do, when?
1.	as xR designers, what we are doing combines scenography, prop design, VFX direction, and lighting design in that order of priority. It’s a lot.
2.	at Savages we approach the stack by chunking it into two parts, pre render and render/post. Pre-render is usually houdini and adobe CS, render/post is usually either Notch or the Unreal Engine. With real-time what’s most special is the ability to make new stuff with a render engine, traditional render engines (like renderman) only render. Sometimes for very specific reasons we will lean heavy on tools internal to a render engine to create core assets for a look, but this is infrequent.
3.	collaboration is key. 3D is by far the most difficult media to work with - hard to maintain the 80/20 rule. Style frames are important for aligning a team, which often means deciding “who learns what.”

2 - Thinking in Houdini
A - HDA definition – HDAs are chunks of Houdini code that can run in other editors

B - procedurality for four reasons:
1.	generate new assets to place on your set quickly and easily
2.	fix assets that are already on set quickly if they need adjustments
3.	save labor on repetitive tasks, like Sorting UVs
4.	integrate into future asset creation processes. HDAs are an investment of present time into future time.

C - what’s a UV?
A UV is a vec3 with it’s Z vector set to the normal and X and Y distributed on a 2D map corresponding to the pixels in a texture. It’s how an image gets mapped to a model. UV maps for real-time come in two flavors - repeating and normalized. Most of the time you want non-overlapping normalized UVs (with X and Y coordinates between 0.0 and 1.0) for real-time light maps. UVs over 1.0 on either major axis will cause the lightmap to overlap and cause errors. Textures and light maps can have different UVs - the texture UV is called “uv” and the lightmap UV is called “uv1.”

D - why do UVs matter?
UVs determine how your geometry will look at render - everything has some sort of texture, with very few exceptions. There are literally none around you in life, making color and reflectance only materials is a quick path to gaminess.


E - thinking about creating geometry three ways
1.	1 - made from foam and clay, which you add to and cut off from as needed using a specific set of modelling tools.
2.	2 - made from an understanding of volumes in space, like smoke or a cloud
3.	3 - made from lines generated via math or clicks and fleshed out into a form with either more math or more clicks or both.

Internalizing a relationship with each of these three forms is where personal definition as a 3D artist comes from. It’s what’s behind “the jazz.” Each have their own methods for generating UVs.

Each of the three forms can be used separately or together. Each of the three forms can also be converted to any of the other three forms. As you get better at houdini, and carve out paths for your use, you will find some operations make sense in one form more than another. For myself, I mainly use a combination of 1 and 3, with bits of 2 brought in to handle detailed work I’d like to approach with a significant amount of control in a very procedural fashion. For example, aging a surface and it’s edges.

F - Thinking about working with geometry two ways
1.	individual copies. Lots of control, as all elements are an island and can be freely addressed without consequence to other assets. However, this is the heaviest way to process data - all elements are processed for each instance of geometry.
2.	instanced copies - an “instance” is a presentation of an asset to the graphics card for rendering. Consider an herb planter in the window - you’ve made a mesh for the planter, and you want to grow some herbs in an animation. The herbs can each be their own geo, thereby each getting their own lane through the graphics card to the monitor. They can also all be grouped together, so points of data they share can transfer from one to the other. In general this is better.

G - what’s geometry?
Geometry refers to the collected pile of data the render engine refers to in order to draw a model to the screen.
Points - special data class internal to Houdini (for the most part). Used to define where the corners of polygons are, with one point per corner. Can also be drawn on their own. Commonly used to hold attributes for driving processing.
Edges - connect points, and end in vertices
Vertices - live at the end of every edge. Where two edges make a corner there are two vertices, three edges makes three vertices, all on the same Point. For this reason it’s better to write attributes to points than vertices.
Polygons - when two edges share one vertice, the form a polygon. Technically this is an impossible polygon, and would be considered “non-manifold.” Manifold geometry is sealed, many situations will require manifold geometry to function properly. More common is a three sided polygon (a “tri”) or a four sided polygon (a “quad.”) All real-time engines I’m aware of will triangulate any polygons with higher counts into triangles. Quad conversion is another issue called “retopology” - if you think about it it’s pretty mellow to draw as few triangles as possible inside of a regular (or even close to it) n-gon, there are n-2 of them. With quads what you are doing is more like rasterization - instead of keeping the original edges and vertices and then adding wedges and vertices, you are resampling edges to make new quads and then meshing between them to fill the polygon.
Normals are how a polygon face knows which side of itself is pointing out, so it can be shaded by light. Non-manifold geometry and manifold geometry both have normals - in purposefully non-manifold geometry, such as a funky screen setup, normals can face any way they need to. With manifold geometry, in general you will want them all to be facing “outwards the same way.” The construction of the normal itself is pretty cool - it’s a vec3, just like mostly everything else, but it’s got some special characteristics. Each vertice can be thought of as an origin, and the Z vector in the origin is pointing directly out from the face of the model. X and Y are generally pointing “across” and “up.” The vec3 that comes out is very useful for relating sets of vertices (continuous areas across the mesh) to each other, especially when you’re looking to describe small amounts of curvature. Here’s some advanced application of normals - making a toon shader. If you have a model with UVs and textures set up, you can scale the UVs up for the normal map and then invert the normal map, and the result is dark shading where the normal map has detail in it. This will all have context when we go over materials, but in the meantime think of normals as a great way to describe your mesh to processing in a way that goes beyond coordinate data. Normals are a part of every 3D format (unlike custom attributes, which are more difficult to work with). If you’re doing Artsy 3D, you can also encode additional data here - there’s no reason you can’t take “normal normals” and composite them with normals derived by noise, for example, to make it look like light is animating without having to actually animate light.

MAKING SETS AND STUFF: A GUIDE

1 - Identify what you are making, and go find some references. 3D modelling is very literally filling a void, just like a painter making a realistic artwork will tend to look at the scene you should gather some references to pin your filled void against.
2 - Identify the major shapes in your references, and think about how you’d make them. Would you be able to define some math that creates what you’re seeing, or thinking about seeing? If so write it out, as much as you’re able. For example - “a window cut out from a wall, with four panes evenly dividing the window” can be cleanly broken down as:
•	Wall Size - a vec3 defining width, height and depth, simply set
•	Window Size - a vec3 that can either be set or be a ratio of the wall size. The ration can be the same for X and Y, or change for each of the major axes of the look
•	Window Pane Size - a vec3 that can be a simple even split of Window Pane x and y, or set with a more complicated scheme that uses the sizes of the left panes to determine the sizes of the right panes, or an even more complicated scheme that treats left and right panes individually.”
•	Mullion Size - a single float value defining overall thickness of the mullions (the pieces dividing the panes) that can either be simply defined, or established as a ratio of Wall Size z, which represents the depth of the cut out window from the wall.

Each of the latter three bullet points offers a choice: define the value outright, or define the value based on another value. “All roads lead to Rome” - at some point, all of the parameters you define can be traced back to at least one variable that’s been defined specifically. In the case above, this is Wall Size, with all the other parameters able to be established from just this vec3.

3 - Savages are a procedural design house first and foremost, so for us this step is extra-tricky - from what you’ve done above, determine what you’d want to set based on another parameter and what you’d want to define specifically. The more you want to define specifically, the more work you need to do before you see results. This assumes you’re going to want to make more than one of what you’re making - think about how many staircases you’ve experienced in your life. How different were they from one another? How many different instruction sets do you think you would need to define every set of stairs you’ve ever experienced? This number is probably not so high, once you remove the very fine details that define how a staircase has specifically lived in situ. Our studio’s modelling mindset embraces this tendency for designers and architects to think similarly to one another, at the end of the day, in many cases - we will use the modelling process to define HDAs, and if it’s possible our preference is to refine an existing HDA rather than make a new one to cover a similar task. As a consequence, the consideration of how much an artist needs to define to see desired results from an asset becomes a question of how future time is used, not only for ourselves but also collaborators that will use the asset. If you’re modelling a one-off piece, you can absolutely bang it together and not sweat this, but this class will be using Houdini - it’s a bit silly to skip the procedural part.

4 - Determine an approach to making the core geometry. Is it reasonable to start from geometric primitives, like cubes and cones and spheres and cylinders? The overwhelming majority of architectural forms especially can be reliably constructed from combined geometries, and then further treated with detail as a whole to complete the presentation of the model. If you’re making something more organic, like a character, the modelling process can be more like pushing and pulling clay. This second method is more advanced and a bit beyond the scope of this course, but it is definitely there for you.

5 - Make your geometry. This step takes a while!

6 - Prepare your UVs. This step is usually pretty mellow, but it can be time-consuming too. Don’t forget that these are as important as your geometry!

7 - Prepare textures. How is this thing getting it’s materials put on? Do different sets of polygons represent different general surfaces, like “walnut veneer” or “chrome”? In this case, you can grab assets that represent these materials from many many places, and go to town. It’s possible, though, that your geometry is much closer to blockout and you will be specifying surfaces purely through texture - if you played any Red Dead Redemption 2, you’ve seen this. You can prepare these specificied composite textures in Photoshop or the GIMP, or a painting software such as Quixel Mixer or Substance Painter. In either case, it is very important that all of the UVs on your model have the same scale.

8 - Render it!
