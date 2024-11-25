# Markdown Guideline

These guideline specifies the markup elements of Markdown from [CommonMark](https://spec.commonmark.org) spec that can be used in `.md` files and parsed correctly in our documentation.
To cover our needs we also use customised markdown elements.

> **WARNING**
>
> When writing in Markdown escape characters shall be used for special symbols (see the details [here](https://en.wikipedia.org/wiki/Escape_character) and via the related links in the article).


## Heading

Two lines before each heading.

Each subsequent level is one level lower than the previous one:

```markdown
# H1
## H2
### H3
#### H4
##### H5
###### H6
```


## Horizontal Rule

```markdown
___

---

***
```

___


## Paragraphs

To create paragraphs, use a blank line before and after the text.

```markdown
I really like using Markdown.

I think I'll use it to format all of my documents from now on.
```


## Section

To make a link to a particular paragraph of a doc page, use the following construction: `[Title](#the-first-title)`.


## Font

**Bold**:

```markdown
**Bold text**
```

*Italic*:

```markdown
*Italicized text*
_Italicized text_
```

~~Strikethrough~~:

```markdown
~~Strikethrough~~
```


## List


### Ordered List

```markdown
1. First item.
2. Second item.
3. Third item.
```

### Unordered List

```markdown
* first item;
* second item;
* third item.
```

* first item;
* second item;
* third item.


### Nested List

```markdown
* first item;
* second item;
* third item;
    * indented item;
    * indented item.
      * indented item;
      * indented item.
* fourth item.
```

* first item;
* second item;
* third item;
    * indented item;
    * indented item.
        * indented item;
        * indented item.
* fourth item.


### Check List

```markdown
* [x] write the press release;
* [ ] update the website;
* [ ] contact the media.
```

* [x] write the press release;
* [ ] update the website;
* [ ] contact the media.Other elements for unordered list:

```markdown
+
-
```


## Table

Use colons to align text in tables:

```markdown
| Syntax | Description || :-----------: | :-----------: || Header | Title |
| Paragraph | Text |
```

| Syntax | Description |
| --- | --- |
| Header | Title |
| Paragraph | Text |

Default alignment in tables (without use of colons):

```markdown
| Syntax | Description || ----------- | ----------- || Header | Title |
| Paragraph | Text |
```


## Link

```markdown
[Link title](https://www.example.com).
```

[Link title](https://www.example.com).


## Image

```markdown
![Figma snippet example](../image/snippet-example-figma.png "Figma snippet example")
```

![Figma snippet example](../image/snippet-example-figma.png "Figma snippet example")

```plaintext
![Image name][id]

[id]: https://octodex.github.com/images/dojocat.jpg  "The Dojocat"
```

![Image name](https://octodex.github.com/images/dojocat.jpg "The Dojocat")


## Blockquote

```markdown
> Blockquote
>> Blockquote
>>> Blockquote
```

> Blockquote> Blockquote> > Blockquote


## Callouts

```markdown
> **NOTE**
> Used to highlight information blocks that you need to pay attention to.

> **WARNING**
> It is used to highlight information related to the prohibition, incorrect information, how not to do.

> **CAUTION**
> Used to warn of something important.

> **SUCCESS**
> Use to beautifully highlight / design any positive information.
```

>**NOTE**
>
> Used to highlight information blocks that you need to pay attention to.

>**WARNING**
>
> It is used to highlight information related to the prohibition, incorrect information, how not to do.

>**CAUTION**
> 
> Used to warn of something important.

>**SUCCESS**
>
> Use to beautifully highlight / design any positive information.


## Code


### Syntax Highlighted Code

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```


### Code Block

```plaintext
The text
```


### Inline Code

```markdown
`./docs` directory
```

`./docs` directory.


## Emoji

💡
See also ["Copying and Pasting Emoji"](https://www.markdownguide.org/extended-syntax/#copying-and-pasting-emoji).


## Documentary Flavored Markdown


### Snippet

This is an extension to the standard markdown to render beautiful snippets on Documentary Platform.

![](../image/snippet-example-file.png)

![](../image/snippet-example-figma.png)

> **NOTE**
>
> Use a relative path to a file or a file URL in the repo. It uses the same syntax as for embedding images:

```markdown
![]()
```

Snippet can be used for rendering the files from repo:

* document files (`.txt`, `.pdf`, `.doc`, `.docx`);
* CSV files are rendered as a table;
* JSON, `.sql` files are rendered as a code block;
* code files will be rendered as a code block as well with a link to the source file;
* plantuml (`.puml`) files including C4 diagrams are rendered as an interactive svg;
* video files (`.webm, .mp4`).You can use it for arbitrary URLs:

![](../image/snippet-example-atlassian.png)

![](../image/snippet-example-url.png)

Snippet contains 3 key attributes:

* `title`;
* `description`;
* `link`.The attributes can be defined in the following syntax:

```markdown
![]() { title="" description="" }
```

For images text in the square brackets used as an `alt` attribute of `img` html tag, but for snippets it used as a `title` value.
So you can define a `title` in square brackets or as an attribute. Attribute value has a precedence.

Example:

```markdown
![Some Title](link) { title="This title rewrites the previous one" description="Description goes here" }
```

> **NOTE**
>
> The `link` cannot be defined as an attribute — write it in parentheses
For the code files you can provide exact line number you want to display: `{ lines=23 }`, or the range of lines: `{ lines=1:8 }`
Example:

```markdown
![FFT Function](./path/to/the/code.go) { lines="13:42" }
```

For CSV files a default delimiter is `,`. If your file has different delimiter, for correct parsing provide as following:

```markdown
![csv file title](./file.csv) { delimiter="|" }
```

The available options are also: `,`, `;`, `|` .


### Video


> **NOTE**
>
> The `WebM` format is rendered as a snippet link in Safari browser.




