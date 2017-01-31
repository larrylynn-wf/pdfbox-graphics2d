# pdfbox-graphics2d
Graphics2D Bridge for Apache PDFBox

## Intro
Using this library you can use any Graphics2D API based SVG / graph / chart library 
to embed those graphics as vector drawing in a PDF.

The following features are supported:

- Drawing any shape using draw???() and fill???() methods from Graphics2D.
- Drawing images. The default is to always lossless compress them. You could plugin 
  your own Image -> PDImageXObject conversion if you want to encode the images as jpeg. 
  **Note:** At the moment PDFBox only writes 8 bit images. So even if you draw 
  a 16 bit image it will be reduced to 8 bit. Depending on the colorspaces this may be 
  bad and cause colorshifts in the embedded image (e.g. with 16 Bit ProPhoto color profile).
- All BasicStroke attributes
- Paint:
	- Colors. You can specify your own color mapping implementation to special map the (RGB) 
	colors to PDColor. Beside using CMYK colors you can also use spot colors.
	- GradientPaint, LinearGradientPaint and RadialGradientPaint. There are some restrictions:
	  - GradientPaint always generates acyclic gradients. 
	  - LinearGradientPaint and RadialGradientPaint always assume even split fractions. 
	  The actual given fractions are ignored at the moment.
- Drawing text. At the moment all text is drawn as vector shapes, so no fonts are embedded. 
  The following TextAttribute features are supported:
   - Underline
   - Strikethrough
   - Kerning / Lignature
   - Foreground/Background paint. Can be any of the paints supported.

The following features are not supported:

- copyArea(). This is not possible to implement.
- hit(). Why would you want to use that?

## Example Usage

See [manual drawing](src/test/java/de/rototor/pdfbox/graphics2d/PdfBoxGraphics2dTest.java) 
and [drawing SVGs](src/test/java/de/rototor/pdfbox/graphics2d/RenderSVGsTest.java).
**TODO:** Add some inline code how to use this library.

## Rendering text using fonts vs vectors

When rendering a text in a PDF file you can choose two methods:
- Render the text using a font as text.
- Render the text using vector shapes through its GlyphVector.

Rendering a text using a font is the normal and preferred way to display a text:
- The text can be copied and is searchable.
- Usually it takes less space then when using vector shapes.
- When printing in PrePress (Digital / Offset Print) the RIP usually handles text special to ensure 
the best possible reading experience. E.g. RGB Black is usually mapped to a black 
with some cyan. This gives a "deeper" black, especially if you have a large black area. 
But if you use a RGB black to render text it is usually mapped to pure black to avoid 
any printing registration mismatches, which would be very bad for reading the text.
- Note: When rendering a text using a font you should always embed the needed subset of the font into the PDF. 
  Otherwise not every (=most) PDF viewers will be able to display the text correctly, if they don't have the font or
  have a different version of the font, which can happen across different OS and OS versions.
- Note: Not all PDF viewer can handle all fonts correctly. E.g. PDFBox 1.8 was not able to handle fonts right. 
But nowadays all PDF viewers should be able to handle fonts fine.

On the other site rendering a text using vector shapes has the following properties:
- The text is always displayed the same. They will be no differences between the PDF viewers.
- The text is not searchable and can not be copied.
- Note: Vector shapes take more space than a embedded font.
- Note: You may want to manually alter the color mapping to e.g. ensure a black text is printed using pure CMYK black. 
If you not plan to print the PDF in offset or digital print you can ignore that. This will make no difference for 
your normal desktop printer.

At the moment only rendering text as vector shapes is implemented. To embed a text using its font into a PDF direct 
access to the underling font file is required, because PDFBox needs to build the embedded font subset. Using the normal 
Java font API it is not possible to access the underlying font file. So a mapping Font -> PDFont is needed. At the 
moment there is a hook to implement that, but its not working yet.

## Licence

Licenced using the Apache Licence 2.0.