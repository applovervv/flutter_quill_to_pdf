# Flutter Quill to PDF

This package allow us create PDF's using `Deltas` from `Quill`.

Some options that can be configured:

- `DeltaAttributesOptions` (this attributes will be applied to whole delta)
- We can use custom fonts. Using `onRequestFont` functions in `PDFConverter` we can detect the font family detected, and use a custom implementation to return a `Font` valid to `pdf` package _Just works automatically with the default library implementation_
- `CustomWidget`, which helps you create custom `PDF` widgets using the `Paragraph` implementation from `flutter_quill_delta_easy_parser`.
- Optional front matter and back matter
- We can set a default directionality for the PDF widgets using `textDirection` from `PDFConverter`
- Page format using `PDFPageFormat` class
- `PDFWidgetBuilder` functions in `PDFConverter` that let us customize the detected style, and create a custom pdf widget implementation
- `ThemeData` optional theme data that let us changes the theme for to pdf document

> By default, the delta is processed by a local implementation that uses `DeltaAttributesOptions` to apply custom attributes (if it is not null), making it easier to add an attribute to the entire delta. If you want to create your own implementation or simply use a default delta, use `PDFConverter(...params).createDocument(shouldProcessDeltas: false)`.

![Delta in editor](https://github.com/CatHood0/flutter_quill_to_pdf/blob/master/example/assets/delta_to_convert.jpg)
![Delta converted in PDF](https://github.com/CatHood0/flutter_quill_to_pdf/blob/master/example/assets/delta_converted.jpg)

### Add dependencies

```yaml
dependencies:
  flutter_quill_to_pdf: ^2.2.4
```

### Import package

```dart
import 'package:flutter_quill_to_pdf/flutter_quill_to_pdf.dart':
```

### Personalize the settings of the page (height, width and margins)

We can use two types differents constructors of the same `PDFPageFormat` class

##### The common, with all set params:

```dart
final PDFPageFormat pageFormat = PDFPageFormat(
   width: ..., //max width of the page
   height: ..., //max height of the page,
   marginTop: ...,
   marginBottom: ...,
   marginLeft: ...,
   marginRight: ...,
);
```

##### The factory to marginize all `PDFPageFormat`

```dart
final PDFPageFormat pageFormat = PDFPageFormat.all(
   width: ..., //max width of the page
   height: ..., //max height of the page,
   margin: ..., //will set the property to the others margins
);
```

### Using PDFConverter to create finally our document

```dart
PDFConverter pdfConverter = PDFConverter(
    backMatterDelta: null,
    frontMatterDelta: null,
    textDirection: Directionality.of(context), // set a default Direction to your pdf widgets
    document: QuillController.basic().document.toDelta(),
    fallbacks: [...your global fonts],
    onRequestBoldFont: (String fontFamily) async {
        // this is optional
       ...your local font implementation
    },
    onRequestBoldItalicFont: (String fontFamily) async {
        // this is optional
       ...your local font implementation
    },
    onRequestFallbackFont: (String fontFamily) async {
        // this is optional
       ...your local font implementation
    },
    onRequestItalicFont: (String fontFamily) async {
        // this is optional
       ...your local font implementation
    },
    onRequestFont: (String fontFamily) async {
        // this is optional
       ...your local font implementation
    },
    pageFormat: pageFormat,
);
```

## To create it, we have three options:

#### `createDocument` _returns the PDF document associated_

```dart
final pw.Document? document = await pdfConverter.createDocument();
```

#### `createDocumentFile` _makes the same of the before one, write in the selected file path_

```dart
// [isWeb] is used to know how save automatically the PDF generated
await pdfConverter.createDocumentFile(path: filepath, isWeb: kIsWeb,...other optional params);
```

#### `generateWidget` _returns a Widget that you can use to draw on a PDF - giving you full control of the PDF_

```dart
import 'package:flutter_quill_to_pdf/flutter_quill_to_pdf.dart';
import 'dart:io';
import 'package:pdf/pdf.dart';

// Generate a widget from the PDF converter
final pw.Widget? pwWidget = await pdfConverter.generateWidget(
    maxWidth: pwWidgetWidth,
    maxHeight: pwWidgetHeight,
);

// Create a new PDF document with specific settings
final pw.Document document = pw.Document(
    compress: true,
    verbose: true,
    pageMode: PdfPageMode.outlines,
    version: PdfVersion.pdf_1_5,
);

// Create A4 page without margins
final PdfPageFormat pdfPageFormat = PdfPageFormat(
    PDFPageFormat.a4.width, PDFPageFormat.a4.height,
    marginAll: 0);

// Add a page to the document with custom layout
document.addPage(
    pw.Page(
    pageFormat: pdfPageFormat,
    build: (pw.Context context) {
        return pw.Stack(children: [
        // Create a full-page blue background
        pw.Expanded(
            child: pw.Rectangle(
            fillColor: PdfColor.fromHex("#5AACFE"),
            ),
        ),
        // Position the editor content in the top-left corner
        pw.Positioned(
            top: PdfPageFormat.a4.marginTop,
            left: PdfPageFormat.a4.marginLeft,
            child: pwWidget!),
        // Position a copy of the editor content in the bottom-right corner
        pw.Positioned(
            bottom: PdfPageFormat.a4.marginBottom,
            right: PdfPageFormat.a4.marginRight,
            child: pwWidget!),
        ]);
    },
    ),
);
// Save the document to a file
await file.writeAsBytes(await document.save());
```

## Supported

- Font family
- Size
- Bold
- Italic
- Strikethrough
- Underline
- Link
- Color
- Background Color
- Line-height
- Code block
* Direction
- Blockquote
- Align
- Embed image (Base64, URL and Storage Paths)
- Embed video (Just the URL of the Video will be pasted as a text)
- Header
- List (Multilevel List too)
- Indent

## No supported

- Superscript/Subscript (Not planned since is not supported by pdf package)
- Embed formula (Not planned)

You can contribute reporting issues or requesting to add new features on [flutter_quill_to_pdf](https://github.com/CatHood0/flutter_quill_to_pdf)
