---
title: OneNote API HTML tag support
TOCTitle: HTML tag support
description: Describes the tags recognized by the API in a page POST, and the markup for images, embedded file data, and rendered HTML blocks.
ms:assetid: f7e1d780-2876-4772-a2f6-8dcbeb4ec1ee
ms:mtpsurl: https://msdn.microsoft.com/en-us/library/Dn575442(v=office.15)
ms:contentKeyID: 60952868
ms.date: 10/02/2018
mtps_version: v=office.15
---

# OneNote API HTML tag support

The Microsoft OneNote API can accept a limited set of HTML tags in the Presentation content. The tags you can use are those that can be mapped to the features available on OneNote pages, and are described in this topic.

HTML that defines the OneNote page structure has the following requirements:

- The HTML must be UTF-8 encoded.

- The HTML should be well-formed XML for reliability. All tags with content require matching closing tags (for example, `<p>All paragraphs need closing tags</p>`), and all empty tags must include the trailing slash (for example, `<br/>` and `<img src="..."/>`).

- All attribute values must be surrounded by double- or single-quote marks like in the preceding `img` tag.

When your app provides HTML to the OneNote API in the HTTP POST body, or in a MIME part named "Presentation," be aware of the following general limitations:

- JavaScript code, included files, and CSS are removed.

- HTML forms are removed in their entirety.

In addition to the HTML that becomes the OneNote page structure and content, you can also pass HTML to the page-rendering feature via the `<img data-render-src="...">` tag. When you use the page-rendering feature, the set of supported HTML tags is closer to a standards-compliant browser, and the resulting webpage is inserted into the OneNote page as a bitmapped image. 

This topic describes the tags recognized by the API in a page POST, and the markup for images, embedded file data, and rendered HTML blocks.

## Supported HTML tags and limitations

<table>
<colgroup>
<col style="width: 30%" />
<col style="width: 70%" />
</colgroup>
<thead>
<tr class="header">
<th><p>HTML tag</p></th>
<th><p>Limitations and comments</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>Structure tags</strong></p></td>
<td><p></p></td>
</tr>
<tr class="even">
<td><p>&lt;html&gt;</p></td>
<td><p>Appropriate to enclose the content, but not required.</p></td>
</tr>
<tr class="odd">
<td><p>&lt;head&gt;</p></td>
<td><p></p></td>
</tr>
<tr class="even">
<td><p>&lt;title&gt;</p></td>
<td><p>The contents of this tag will be used as the page title, if it exists. The tag is optional.</p></td>
</tr>
<tr class="odd">
<td><p>&lt;meta&gt;</p></td>
<td><p>This tag is generally ignored by the API, except when the name attribute is &quot;created&quot;. See the [&lt;meta&gt; tag details](#meta-tag-details) section.</p></td>
</tr>
<tr class="even">
<td><p>&lt;body&gt;</p></td>
<td><p></p></td>
</tr>
<tr class="odd">
<td><p>&lt;div&gt;, &lt;span&gt;</p></td>
<td><p>Styling on these tags is ignored.</p></td>
</tr>
<tr class="even">
<td><p><strong>Content tags</strong></p></td>
<td><p></p></td>
</tr>
<tr class="odd">
<td><p>&lt;a&gt;</p></td>
<td><p>Contents of the tag can only be text at this time. Images cannot be linked.</p></td>
</tr>
<tr class="even">
<td><p>&lt;p&gt;, &lt;br/&gt;</p></td>
<td><p>See important information about the &lt;p&gt; tag in the [&lt;p&gt; tag details](#p-tag-details) section.</p></td>
</tr>
<tr class="odd">
<td><p>&lt;h1&gt;...&lt;h6&gt;</p></td>
<td><p>Mapped to the corresponding OneNote paragraph styles.</p></td>
</tr>
<tr class="even">
<td><p>&lt;ul&gt;, &lt;ol&gt;, &lt;li&gt;</p></td>
<td><p>Unordered and ordered lists are supported, including nested lists. Type attribute is ignored.</p></td>
</tr>
<tr class="odd">
<td><p>&lt;img&gt;</p></td>
<td><p>See important information about this tag in the [&lt;img&gt; tag details](#img-tag-details) section.</p></td>
</tr>
<tr class="even">
<td><p>&lt;object&gt;</p></td>
<td><p>Allowed for embedding file objects (see the [&lt;object&gt; tag details](#object-tag-details) section). Otherwise ignored.</p></td>
</tr>
<tr class="odd">
<td><p>&lt;table&gt;, &lt;tr&gt;, &lt;td&gt;</p></td>
<td><p>Nested tables are allowed. These are mapped to OneNote tables. Border attribute with single value (border=&quot;2&quot;) is supported. &lt;table&gt; and &lt;td&gt; accept width as either number of pixels (for example, width=&quot;100px&quot;) or percent of page width (for example, width=&quot;60%&quot;.</p></td>
</tr>
<tr class="even">
<td><p>&lt;b&gt;, &lt;em&gt;, &lt;strong&gt;, &lt;i&gt;, &lt;u&gt;, &lt;strike&gt;, &lt;del&gt;, &lt;sup&gt;, &lt;sub&gt;, &lt;cite&gt;, &lt;font&gt;</p></td>
<td><p>Mapped to the available character style in OneNote. The &lt;font&gt; tag only accepts the face and color attributes.</p></td>
</tr>
</tbody>
</table>


## \<meta\> tag details

The following code shows how to use the `meta` tag in a simple POST.

```html
  Content-Type:multipart/form-data; boundary=BlockPartBoundary
  --BlockPartBoundary
  Content-Disposition:form-data;name="Presentation"
  Content-Type:application/xhtml+xml
  <?xml version="1.0">
  <html>
    <head>
      <title>Example showing how to set the creation data-time</title>
      <meta name="created" content="2013-06-11T06:30:00-08:00"/>
    </head>
    <body>
      <p>The head tag sets the created data and time of the capture.</p>
    </body>
  </html>
  --BlockPartBoundary--
```

The OneNote API recognizes a particular kind of `meta` tag to set the created date and time of the new OneNote page. The `name="created"` attribute must be included, and the `content` attribute must be an ISO 8601 standard time stamp. The time stamp should include a UTC time zone offset as appropriate. If this `meta` tag is not given, the page created time will be blank. 

Optionally, the OneNote API can also accept a time zone offset without date and time to set the page creation to the time in that time zone when the page was created. The following code shows this usage.

```html
  <meta name="created" content="-08:00" />
```

## \<p\> tag details

With modern CSS implemented in browsers, the `<p>` tag can have a wide variety of style attributes. A few of those are supported on the `<p>` tag when you POST to the OneNote API to create a new OneNote page. Style attributes not listed are ignored:

- `background-color:#HEXCODE` will highlight the text. Both hexadecimal \#RRGGBB format and named colors (for example, red) are accepted.

- `color:#HEXCODE` gives the color for the text. Both hexadecimal \#RRGGBB format and named colors (for example, red) are accepted.

- `font-family:fontName1, fontName2,…` indicates the font to use. The font names must be available on the device, and OneNote only uses the first name in the list.

- `font-size:XXpx` specifies the font character size, in pixels. Point-size (for example, 12pt) and other ways of specifying font sizing are not recognized.

The following code shows a paragraph tag using the supported style attributes.

```html 
  <p style="color:#00FFFF; font-family:Garamond; font-size:28px; background-color:#000000">
    This paragraph shows yellow, 28-pixel Garamond text on a black background</p>
```

## \<img\> tag details

Use the `img` tag to include images in your capture. There are three ways to include images: by referencing Internet images as you would in a standard HTML page, by including the image data directly in the capture POST, and by having the OneNote API embed a thumbnail of a webpage.

### Image attributes

The OneNote API supports the following attributes in `img` tags:

- `src="http://..."` is the normal HTML way of indicating the image source, which will be included in the OneNote page if it is publicly accessible on the Internet.

- `src="name:BlockName"` indicates the MIME part name in the POST that includes the binary image data to be used for this image in the OneNote page.

- `data-render-src="http://…"` is an attribute recognized by the OneNote API. When given a URL, the page-rendering service inserts a bit-mapped image of the webpage as it would appear in a web browser, into the new OneNote page.

- `data-render-src="name:BlockName"` is an attribute recognized by the OneNote API. When the attribute value is of the `name:...` form, it indicates the MIME part name in the POST that includes HTML to render as an image. The resulting bit-mapped image is inserted into the new OneNote page. When the part is a PDF file (content-type:application/pdf), each page will be rendered as a separate image on the page.

- `alt` is the alternative text used by screen-readers and if the image can't be rendered.

- `height` is the height in pixels that the image will be inserted into the OneNote page. The unit is always pixels. When the image comes from a multi-page PDF file, the height specifies the total height for all the pages together. To specify the size of each image, multiply the desired height by the number of pages in the PDF file.

- `width` is the width in pixels that the image will be inserted into the OneNote page.

- `style` can be used to set maximum values for width and height: `style="max-width:150px; max-height:200px"`

### Limits when using images

Remember these limits when you're embedding images in a OneNote API capture. We're working to remove or expand these limits, but for now they are:

- **Total POST size limit is ~70 MB**, including file and other data. Captures with data more than that limit may make your app and captures unreliable, so be careful.

- **MIME part size limit is 25 MB**. Larger data blocks will be rejected by the API. This applies to both images and file-data parts, and the size includes the part headers.

- **Image limit is 150 per page**. When using the `src="internetURL"` attribute, the API ignores `<img>` tags beyond the limit.

- **MIME parts limit is 6 per POST**. That includes the Presentation HTML part.

- **Maximum number of `<img/>` and `<object/>` tags using data-render-src is 5…or 1**. That is, images can have up to 5. Only one PDF document can be displayed via an `<img data-render-src="name:blockName">` tag. Additional rendered images and embedded files are ignored.

- **File-type icons are predefined**. The OneNote API recognizes a wide variety of common file types, and embeds the file using predefined icons. If the API doesn't recognize the file type, it uses a generic file icon.

### Referencing an image by URL

When your capture HTML includes a standard `<img src="..."/>` tag, the `src` attribute gives the normal URL of the image. The OneNote API gets a copy of that image from the Internet, and inserts it directly into the page. If the `img` tag includes `height` or `width` attributes (always in pixels), those values are used to size the image on the new OneNote page. The following code shows an example.

```html
<img src="http://officeimg.vo.msecnd.net/en-us/files/018/949/ZA103278226.png" 
    height="180" width="345"/>
```

If you reference images by URL, ensure that they are available to the OneNote on the public Internet. If you provide a URL to an image that the API can't get, it won't show up on the captured page.

### Including image data in the POST

You can also send the image data for up to 150 images directly inside the OneNote API POST. This is common when the images are from a mobile device camera, or if your app is running on a hardware device like a scanner or camera. Even if the image is also available on the Internet, embedding the data in your POST can sometimes make the captured page available to OneNote users sooner by reducing the total number of round-trips needed to build the OneNote page.

To embed an image into your POST, you use the `src` attribute with a value of `"name:BlockName"`. The name prefix tells the OneNote API that the rest of the attribute value is the name of a MIME part in the POST. (For more information, see the [Pages REST API](onenote-create-page.md).

The following is an example of an embedded image (HTTP headers and code not involved with the image have been removed).

```html
Content-Type:multipart/form-data; boundary=NewPart12345

--NewPart12345
Content-Disposition:form-data; name="Presentation"
Content-Type:text/html
<!DOCTYPE html>
<html>
  <body>
    <img src="name:imageDataPart1" width="50" height="50"/>
  </body>
</html>

--NewPart12345
Content-Disposition:form-data; name="imageDataPart1"
Content-Type:image/jpeg
...binary image data...

--NewPart12345--
```

It's important to note that you don't need to Base64 or otherwise encode the binary image data. That saves time and bytes over-the-wire for both your app and the OneNote API. Also, remember that the OneNote API only uses the first 150 image data blocks included in the POST.

### Capturing a webpage as an image

You can have the OneNote API include a screen-shot of a webpage as an image inside your captured page. The webpage can either be a publicly-available Internet webpage that you reference by URL, or you can provide the HTML in a MIME part. This is useful when the user wants to capture a webpage they're viewing, like when they're doing research for a on-sale purchase, or want a record of what a page looked like at that moment.

When capturing webpages this way, the `height` and `width` attributes tell the OneNote how large to display the image inside the captured OneNote page. This isn't necessarily the same as the actual page. If you want the image of the webpage to appear in OneNote at the original size, don’t include `height` and `width` attributes in the `img` tag.

To have OneNote render the page as an image, you include the `data-render-src` attribute in an `img` tag. When a browser displays the `img` tag, the `data-render-src` attribute has no meaning for it, so instead the browser uses the normal `src` attribute. When you post that `img` tag to the OneNote API, it uses the information in the `data-render-src` attribute to decide how to create a screen-shot of the information.

There are two ways to use the `data-render-src` attribute. If the value is like:

  - `data-render-src="http://www.onenote.com"`, the API generates a JPEG preview of the webpage, and inserts that into the captured page.

  - `data-render-src="name:imageDataPartName"`, the API takes the HTML from the POST data part that has the indicated `imageDataPartName` value, generates a JPEG preview of the HTML, and inserts that JPEG image into the page.

The following is an example that shows both ways of including a rendered image of a webpage.

```html
Content-Type:multipart/form-data; boundary= NewPart12345
--NewPart12345
Content-Disposition:form-data; name="Presentation"
Content-Type:text/html

<!DOCTYPE html>
<html>
  <body>
    <p>This first image embeds a screen-shot of the onenote.com web site.</p>
    <img data-render-src="http://www.onenote.com" width="500"/>

    <p>This second image embeds a bit of HTML in the next multi-part 
      block as a screen-shot</p>
    <img data-render-src="name:htmlPart" width="500"/>
  </body>
</html>

-- NewPart12345
Content-Disposition:form-data; name="htmlPart"
Content-Type:text/HTML

<html>
  <body>
    <h1>This will be rendered in an image<h1>
    <p style="font-weight:bold">and so will this text. Don't try to embed another 
      data-render-src type image inside the HTML part; it won't 
      work. Instead, use URL-based real images like this:</p>
    <img src="http://officeimg.vo.msecnd.net/en-us/files/018/949/ZA103278226.png" 
    height="180" width="345"/>
  </body>
</html>

-- NewPart12345--

```

The HTML you provide in a separate MIME part for rendering is not limited to the set of tags listed in this topic. The rendering engine is much closer to a standard web browser, with some limitations around browser plug-ins and active content. If you have a complex layout that you want displayed on the captured page, including that complex HTML as an image might be a good option.

The webpage rendering doesn't recognize the `data-render-src` attribute, so you can't include them inside the HTML part.


> [!TIP]
> The page-rendering has no ability to sign in for the user, so pages that require being signed in will likely not turn out as your users expect if you send just the URL in the `data-render-src` attribute; for example, if someone is looking at their purchase history or account status and they try to capture the page.
> 
> If your app includes an `img` tag with the `data-render-src` attribute set to the current URL, the thumbnail will usually be either the site sign-in page, or worse, an "access denied" page. In those cases, it's best to grab the page's HTML and send that in a named part of your POST. However, be aware that there are some risks with that, especially if the HTML contains images that can only be retrieved by a signed-in user.



## \<object\> tag details

Use the `object` tag to embed files directly on the OneNote page, displaying them as an icon representing the file type and not showing the actual file contents, as happens with the `img` tag. The data referenced by the `object` tag must be provided in a MIME part of the POST request, and cannot be a URL reference.

### Object attributes

The `object` tag is used in different ways by different browser plug-ins, and has an open standard definition. The OneNote API takes advantage of that flexibility and supports the following attributes in object tags:

- `data-attachment="DisplayedFileName.extension"` specifies the file name and extension displayed on the OneNote page for the file. That value will also be used if the user saves the file by activating it from the OneNote page.

- `data="name:PartName"` indicates the MIME part name in the POST that includes the file data to be embedded in the OneNote page. Embedded files can only be added to a page by sending the data directly in the multi-part POST request.

- `type="standardMIMEtype"` tells the OneNote API what type of file is being embedded. Be sure that the MIME type you specify is the same type as the file data, or the OneNote API could return an error for the POST request.

The icon displayed in the OneNote page for the embedded file is determined in the following precedence:

1. **Filename Extension** provided in the `data-attachment` attribute. If that isn't recognized, then…

2. **Type attribute** is checked for a recognized MIME type. If that isn't recognized, then…

3. **Content-type header** of the file data MIME part is checked. If that isn't recognized, then…

4. **The default "document"** icon is used.

Remember these limits when you're embedding files in a OneNote API capture. We're working to remove or expand these limits, but for now they are:

- **Total POST size limit is ~70 MB**, including file and other data. Captures with data more than that limit may make your app and captures unreliable, so be careful.

- **MIME part size limit is 25 MB**. Larger data blocks will be rejected by the API. This applies to both images and file-data parts, and the size includes the part headers.

- **MIME parts limit is 6 per POST**. That includes the Presentation HTML part.

- **Maximum number of `<img/>` and `<object/>` tags using data-render-src is 5**. Additional rendered images and embedded files are ignored.

- **File-type icons are predefined**. The OneNote API recognizes a wide variety of common file types, and embeds the file by using predefined icons. If the API doesn't recognize the file type, it uses a generic file icon.

The following code shows how to embed a file by using REST.

```html
Content-Type:multipart/form-data; boundary=MyAppPartBoundary
Authorization:bearer tokenString

--MyAppPartBoundary
Content-Disposition:form-data; name="Presentation"
Content-type:text/html

<!DOCTYPE html>
<html>
  <head>
    <title>A simple page with an embedded file</title>
  </head>
  <body>
    <p>This is a simple Presentation block.</p>
    <p>This next object tag specifies the file data in this POST request.</p>
    <object data-attachment="Logo.jpg" data="name:MyAppFilePartName" 
      type="image/jpeg" />
  </body>
</html>

--MyAppPartBoundary
Content-Disposition:form-data; name="MyAppFilePartName"
Content-type:image/jpeg

... embedded file binary data ...

--MyAppPartBoundary--
```

## Example block of HTML

The following is an HTML block that shows many tags supported by the OneNote API. (The style attribute on the `body` tag is there just so it doesn't look awkward if you display it in a browser.)

```html
<html>
  <head>
    <title>HTML sample block</title>
    <meta name="created" content="2013-06-11T06:30:00-08:00"/>
  </head>
  <body style="font-family:Arial;">
    <h1>HTML sample block</h1>

    <h2>The basics</h2>
    <p>For the most part, try to keep the HTML simple, and be 
      sure to properly close all tags.</p>
    <p>Character encoding must be in UTF-8; the API doesn't 
      accept other encodings</p>

    <h2>Overall structure</h2>
    <p>The normal &lt;html&gt;, &lt;head&gt; and &lt;body&gt; tags are 
      expected, but the API is usually okay if they aren't included.</p>
    <p>Inside the &lt;head&gt; tag, we recognize only the &lt;title&gt; and
      One form of the &lt;meta&gt; tags.</p> 
    <p>All includes, CSS, script, etc. inside the &lt;head&gt; and &lt;body&gt;
      blocks are ignored.</p>
    <p>As you can tell from the &lt;body&gt; tag's style attribute, in this
      HTML, the API ignores CSS styles</p>

    <h2>Block formatting</h2>
    <p>Paragraph (&lt;p&gt;) and line-break (&lt;br/&gt;) tags are handled 
      properly. They get the &quot;Normal&quot; OneNote styling</p>
    <p>The &lt;div&gt; and &lt;span&gt; tags are recognized but generally 
      don't have any significant effect, since CSS styling is ignored. A
      &lt;div&gt; tag acts like a &lt;p&gt; tag.</p>
    <p>The header tags &lt;h1&gt; through &lt;h6&gt; are recognized, and
      map to the OneNote Heading 1 through Heading 6 styles.</p>

    <h2>Simple character formatting</h2>
    <p>This paragraph shows how the API handles <b>HTML4+ bold 
      &lt;b&gt;</b> tags</p>
    <p>...and <strong>HTML4+ strong &lt;strong&gt;</strong> tags</p>
    <p>...and <i>HTML4+ italics &lt;i&gt;</i> and &lt;cite&gt; tags</p>
    <p>...and <em>HTML4+ emphasis &lt;em&gt;</em> tags</p>
    <p>...and <u>HTML underline &lt;u&gt;</u> tags</p>
    <p>...and <strike>HTML strikeout &lt;strike&gt; and &lt;del&gt;</strike> tags</p>
    <p>...and Super<sup>script</sup> (&lt;sup&gt;) and Sub<sub>script</sub> (&lt;sub&gt;) tags</p>
    <p style="background-color:#FFBBBB; color:#EEEEFF; font-family:Garamond; font-size:30px">...and styled text</p>
    <p>...and <a href=".">HTML anchor &lt;a&gt;</a> tags.</p>

    <h2>Images</h2>
    <p>The &lt;img&gt; tag is supported. The src attribute can be either
      a URL to an Internet image, or can be a reference to an image provided
      in the data sent to the OneNote API in the request. For
      example, here&apos;s a referenced image: </p>
    <img 
      src="http://officeimg.vo.msecnd.net/en-us/files/018/949/ZA103278226.png"
      alt="Everyone loves OneNote!"/>
    <p>The OneNote API supports JPEG, GIF, TIFF and PNG images</p>
    <p>This next tag inserts a screenshot thumbnail image of the OneNote.com web
      page into the captured page, displayed as 500 pixels wide.</p>
    <img data-render-src="http://www.onenote.com" width="500"/>
    <h2>Tables</h2>
    <p>Tables are understood, but not table headers. Table headers are treated
      like normal rows. You can nest tables, but their content-layout ability
      is limited. </p>
    <p>Tables have to be &quot;regular&quot;, in that the API assumes all 
      rows have same number of columns. Similarly, all columns have the same 
      number of rows. More specifically, the API ignores colspan and rowspan 
      attributes.</p>
    <p>Table border and other styling is ignored, but instead get the default 
      OneNote table styling, so a simple table like the below will have no
      visible borders on the notebook page, until the user adds them.</p>
    <table border="0px">
      <tr>
        <td width="20%">First row First column</td>
        <td width="40%">First row Second column</td>
      </tr>
      <tr>
        <td style="background-color:#EEEEFF">Second row First column in blue</td>
        <td style="background-color:#EEFFEE">Second row Second column in green</td>
      </tr>
    </table>

    <p>Lists of both types (&lt;ol&gt; and &lt;ul&gt;) are supported, and can
      be nested. The &quot;type=&quot; attribute is ignored.</p>
    <ul>
      <li>First unordered list item</li>
      <li>Second unordered list item<br>which contains another list
        <ol>
          <li>First (nested) ordered list item</li>
          <li>Second (nested) ordered list item</li>
        </ol>
      </li>
    </ul>
    <h2>Not (yet) supported</h2>
    <p>Nested tables are supported, but table header rows (&lt;th&gt;) 
      are treated like normal table rows (&lt;tr&gt;)</p>
    <p>CSS styles are ignored at this time.</p>
    <p>Scripts are ignored.</p>
    <p>Other tags (for example, &lt;article&gt;, &lt;header&gt;,
      &lt;footer&gt;, &lt;section&gt; and &lt;aside&gt;) are largely
      ignored, but their otherwise-valid contents are included.</p>
  </body>
</html>
```

## See also

- [Create OneNote pages](onenote-create-page.md)
- [Input and output HTML in OneNote pages](onenote_input_output_html.md)
- [Add images, videos, and files to OneNote pages](onenote_images_files.md)
- [OneNote Developer blog](http://go.microsoft.com/fwlink/?LinkID=390183)
- [OneNote development questions on Stack Overflow](http://go.microsoft.com/fwlink/?LinkID=390182)
- [OneNote GitHub repos](http://go.microsoft.com/fwlink/?LinkID=390178) 
