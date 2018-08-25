# Free Design Software - Desired Capabilities

Some of us may at one point have had access to Adobe software, and now do not due to financial challenges or no longer working
for a company that provides access to it. This article is written from the perspective of formerly having Adobe Software and
needing to perform as much as the same tasks as possible but using free software. Those starting out fresh in graphic design
who may want to utilize free software may find the information here helpful as it provides ideas of what to expect.

We will only list some capabilities that we feel that a free alternative may not have.

# Adobe Illustrator
So, we want to create vector graphics but we want to use free software to do it. Before finding software, let's gather some
features that Illustrator has that may not be possible (or very difficult to do) in a free alternative.

1. **Symbols.** We may want to create a particular object and save it as a symbol, both in-project and to a library. This way,
we may easily place "copies" of them in multiple places in a project. If ever we need to update the symbol's color or any other
physical attribute, the change will be reflected in all "copies". This is useful, for example, when we are trying to draw a
map with a lot of "copies" of objects. In addition, there should be the capability to unlink a "copy" from the symbol, which
means that it would be made free so that editing the "master copy" will no longer affect it.
2. **Styles and Appearances.** This feature is one of the neatest and powerful in Illustrator. We usually have a single simple
path or shape, and we usually want to shade, drop-shadow, bevel, apply visual effects, etc. to it. When we want to apply the 
same thing to another shape, we have to perform it all over again, and again for each additional shape. We will probably want 
to decouple the "look" of a shape from the shape itself. This way, we can save just the "look" separately, and apply it to
any shape we need to with a mere few clicks.
3. **Brushes.** These are ways to alter the width of a segment across its length. Also, we would be able to create a long
object (or a series of objects like a trail of sparkles, for example) and it would be applied across the length of a line,
curve, or path. This would be useful for creating decorative borders. It would be neat if we get to specify whether to
stretch the object all the way, repeat a number of times along the whole length, or repeat when its length is shorter than
the curve/line we're applying it to. It would also be neat if we get to define what a path would look like at the ends or
on a corner. The nice thing about these is they could be easily swapped for a given curve.
4. **Non-Destructive Shape Operations.** It is useful to group basic shapes together for the purpose of creating a more
complex shape, but we don't want to "commit" it. We still want to be able to move each shape around within the group or edit
them across the lifetime of the project. If we "commit" the group, this means that we now get a single path and we've lost the
ability to move/edit each individual simple shape.
5. **Clone with Rotate.** In order to create radial symmetry, we may want to be able to pick a shape, pick a pivot point, and
specify degrees. Shortcut keys would be nice to make this operation easier.
6. **Grouping Objects.** An object hierarchy would be extremely beneficial for organization, and for a single manipulation
as a group. It'd be nice to choose objects as one group, move, scale, etc. as one unit instead of individually. We could also
apply effects to a group as a whole instead of for each individual object.
7. **Offset Path.** When we have an odd shape, we may want to create another object behind it to make it look like we are
outlining it without using a stroke. No, this is not duplicating a shape and slightly scaling up the object, repositioning it,
which works only for two shapes - squares and circles, not to mention, very tedious and inaccurate. This is "create me a new
shape that is larger or smaller by x pixels uniformly along the edge of the original shape". This effect is quite commonly
done with illustrative text. Also, such a feature should also know whether to round or cap off sharp angles.
8. **Live Preview Curve.** This may be helpful, especially when working with the pen tool to see what the curve will look like first
before we drop/commit the anchor point. This would also be helpful for spiral or arc tools.
9. **Seamless Patterns.** We sometimes want to be able to create patterns that can be tilable, and we often do NOT want to see
the blank spaces that are the obvious bounds of the repeated pattern. In a perfect seamless pattern, there should not be a single
straight line that could be drawn across the whole canvas that would not intersect any object.
10. **Expand Shape.** So we've got a shape with various effects applied to it, including stroke, fill, any shadows, etc. Sometimes,
we will want to break everything down into individual paths for individual control of each component.
11. **Smoothen Tool.** Many have only a mouse to draw, and it results in very jagged paths. Even with a pen and tablet, the
results may still be rough. The smoothen tool, when dragged in a close proximity to a jagged piece of path, would decrease
the number of anchor points, make some adjustments, and intelligently guess the intended smooth curve. This tool isn't
perfect, but it is far more often effective than ineffective.

## Adobe Photoshop
1. **Content Aware Fill.** This tool allows us to select an area with something in it that we want gone. When the operation
is complete, the area would be intelligently filled with content according to the surrounding area, and the effect would
look like as if we photographed the same picture without the person or object there. This tool isn't absolutely perfect
and would work only when the surrounding area is fairly detailed and random (like pebbles and shrubs). Also, as a bonus,
we would like this kind of filling when we slightly rotate an entire photo in a canvas, and fill the 4 resulting acute
triangular areas along the edge, with content according to what's in the image.
2. **Prespective Correction.** Many photographs (especially of buildings) cannot help but be taken from awkward angles and
distances, resulting in slightly wrong or undesired perspective. We would like a tool that could "straighten out" the perspective.
We realize that such an operation could warp the entire picture and could result in blank areas, so we would also like to
be able to content-aware-fill those areas.
3. **Feather Selection.** Sometimes, we might want to select an area and affect it visually, however, we want some smooth transition
between the inside and outside of the selection when we're done applying our desired effect. We should also be able to define
the feather radius to control how smooth or abrupt the transition.
4. **Save and Load Selection.** Selection is tough, so it may be handy to be able to save them and later retrieve them. Saved
selections would be part of the file.
5. **Selection by Approximate Color.** We would like to select regions that have the same approximate color. We should be able to
set a tolerance value to determine cutoff points (what color is different enough not to be selected).
6. **Boolean Selection.** We should also be able to add to selection, and at times, subtract from selection.
7. **Image Offset.** This tool first tiles the image, and then offsets the viewing frame so that we will be able to see the edges
and a corner of the original image. This is helpful for creating seamless background images where we can effectively "erase" the
original seams with various tricks depending on the image.
