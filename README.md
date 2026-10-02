Canvas Studio
A lightweight browser-based drawing studio starter. No build step or dependencies are required.
Run locally
Open index.html in a modern browser, or serve this folder with any static web server.
Included
Raster canvas with brush and eraser
Line, rectangle, and ellipse tools
Layer creation, visibility, opacity, rename, and deletion
Undo and redo
SVG import and SVG export
Image import
Project save/open as JSON
PNG export
Responsive dark UI
Important limitations
This is a starter foundation, not a full Infinite Painter or Adobe Illustrator replacement. Imported SVG is retained as SVG markup and displayed, but there is not yet a full path/node editor, boolean geometry engine, or complete SVG document parser. The initial PNG export currently exports the raster canvas; SVG export includes recorded strokes and imported SVG markup.
1. Separate each layer into its own offscreen canvas so layer visibility and opacity affect rendering correctly.
3. Add SVG DOM sanitization and a selectable vector-object model.
5. Add autosave, touch gestures, and performance improvements.
