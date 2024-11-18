# Writing Guideline

We recommend to apply the [Google developer documentation styleguide](https://developers.google.com/style).

Below are the guidelines for easy searching, customized for writing inDrive documentation.


## General Principles

* use present tense;
* use active voice;
* document the current version of a product or feature, avoid trying to document future features or products, even in innocuous ways. The exemptions: blog posts and release notes. Avoid the words: currently, existing, latest, now, old, presently, at present, soon;
* avoid directorial language (for example, "above" and "below");
* avoid negative constructions when possible (consider whether it's necessary to tell the reader what they can't do instead of what they can);
* write short sentences;
* address the reader directly (use "you", instead of "they", "the user", etc.);
* define abbreviations in the section "Abbreviations and Acronyms" if they are used in the documentation;
* be consistent — use one exact term in all places of a document.


## Formatting and Organization


### Lists

Introduce a list with the appropriate context. In most cases, precede a list with an introductory sentence. The sentence can end with a colon or a period. If the list doesn't need any additional context other than the heading, it's possible to not introduce a list with an introductory sentence.

> **CAUTION**
>
> Don't use a list to show only one item; a single item isn't really a list. If you want to set a single item off from surrounding text, then use some other formatting.


#### Numbered and Bulleted List

A sequence of steps to be performed in order. Each sentence in an ordered list ends with a period:

```plaintext
1. Open the tab.
2. Click the button.
3. Check the result.
```

Nested sequential lists are labeled with lowercase letters or lowercase Roman numerals. The following is an example of a nested sequential list. Each sentence in a bullet list ends with a semicolon. The last sentence of a list or nested list ends with a period:

```plaintext
1. Open the tab.
   * enter the values:
     * 1;
     * 2;
     * 3.
   * enter the text. 
2. Click the button.
3. Check the result.
```

A bulleted list is a set of items that's neither a sequence nor options.


### Punctuation

* a colon indicates that closely-related information follows;
* use commas to separate items in a series, and use commas to separate certain kinds of clauses;
* don't use exclamation points in text except when they're part of a code example;
* avoid using slashes except in file paths and URLs.


### Notes and Notices

When you decide to use a notice, use one of the following types:

Note:

> **NOTE**
> An ordinary aside or tip. Provides information that is useful but not critical to the reader.
 
Caution:

> **CAUTION**
> Tells the reader to proceed carefully. For example, "We don't recommend using ... ."
 
Warning:

> **WARNING**
> Stronger than a caution notice; it means "Don't do this" or that this step might be irreversible, such as leading to permanent data loss.
 
Success:

> **SUCCESS**
> Describes a successful action or an error-free status. Used only in interactive or dynamic content; don't use this notice type in ordinary static pages.


### Numbers

Spell out numbers:

* whole numbers from zero through nine;
* a number that starts a sentence;
* all ordinal numbers in text.

Use numerals:

* numbers 10 and greater;
* version numbers;
* technical quantities, such as amounts of memory, amounts of disk space, numbers of queries, or usage limits;
* page numbers;
* chapter numbers, sections, pages, etc.;
* prices;
* numbers without units, such as numbers used in mathematical expressions;
* numbers less than 10 when they appear in the same sentence with numbers greater than 9;
* negative numbers;
* percentages;
* dimensions;
* numbers containing decimal points;
* measurements;
* numbers in a range.


### Tables

Table columns heads:

* use sentence case;
* write concise headings and omit articles (a, an, the);
* don't end with punctuation, including a period, an ellipsis, or a colon;
* use table headings for the first column and the first row only.


### Headers

* when possible, avoid using -ing verb forms as the first word in any heading or title;
* don't include numbers in headings to indicate a sequence of sections;
* don't use punctuations: the sentences shall be as simple as possible;
* don't use abbreviations. If it is unavoidable, spell out the abbreviation in the first paragraph.


#### Levels

* the first-level section — H1 header;
* the following sections — Hn+1 lower — than the previous one;
* two empty lines before each heading.


#### Capitalization and Lowercase

* capitalize nouns, pronouns, adjectives, verbs, adverbs, and subordinate conjunctions (example: after, because, before etc.);
* lowercase articles (a, an, the), coordinating conjunctions, and prepositions (regardless of length).
* lowercase the second word after a hyphenated prefix (e.g., Mid-, Anti-, Super-, etc.) in compound modifiers (e.g., Line-level, Self-Compound, etc.);
* if any of the doc sections is under development, discussion etc., specify the uncompleted stage in the heading brackets.


### Font


#### Underline

Please do not underline.


#### Italics

This font is used for the following cases:

* introducing technical terms or key terms;
* introducing key terms;
* use of a particular word or phrase itself;
* titles of books and articles.


#### Bold

This font is used for the following cases:

* UI elements;
* hostnames and usernames;
* term lists;
* emphasis when changing context.


### Code


#### Code Block

Use code formatting for code examples.


#### In-Line Code

In-line code formatting should be used for:

* command names: e.g. `unzip`;
* package names: e.g. `mysql-server`;
* optional commands;
* file names and paths: e.g. `~/.ssh/authorized_keys`;
* URLs: e.g. `http://your_domain`;
* ports: e.g. `3000`;
* key presses, which should be in ALL CAPS, like `ENTER`. If keys need to be pressed simultaneously, use a plus symbol (+), such as `CTRL+C`;
* method names;
* attribute names and values (including constant);
* class names;
* HTTP status code. e.g:
    * an HTTP `400 Bad Request`;
    * an HTTP `2xx or 400 status code`.
* command output (e.g. `nameserver 169.254.169.254`);
* command-line utility names, such as `gcloud`, `gsutil`, `kubectl`, and `bq`;
* data types;
* DNS record types;
* environment variable names;
* element names (XML and HTML);
* filenames, filename extensions (if used), and paths;
* folders and directories;
* HTTP verbs;
* HTTP status codes;
* HTTP content-type values;
* IAM role names (for example, `roles/storage.admin`);
* method and function names;
* namespace aliases;
* placeholder variables;
* query parameter names and values.

In-line code is not used for the following elements (if they are not a part of code):

* email addresses;
* domain names;
* URLs that the reader is supposed to follow in a browser;
* names of products, services, and organizations.


### Links

* use meaningful link text. Links should make sense when read out of context;
* don't use often "click here" or "read this document". Some people who use screen readers jump from link-to-link to scan a page and need to understand what a link contains;
* use "see" to refer to links and cross-references;
* if a link downloads a file, indicate this action and the file type in the link text;
* when mentioning an inDrive colleague provide a link to Teammate internal information portal. If you mention any other more or less outstanding person (outside inDrive), provide a link to a public account (for example, [LinkedIn](https://www.linkedin.com)).


### UI Elements and Interaction

When you're documenting tasks that involve UI, focus on a task, not how a user interacts with a UI element.

If you document UI elements, put UI element names in bold, and use appropriate nouns and verbs to describe how to interact with them.


### Names and Naming


#### Filenames

See the details [here](repo-files-storage.md).


#### Domain Names

Don't use real domain names, as well as email addresses, or people's names in your examples.


#### Service Names

The service names shall be the same as in GitHub repositories.

> **NOTE**
>
> There are cases when a service has internal (unofficial) established name (ex. New Order).
> 
> The first letter of the names of such services shall be capitalized, the names are written without quotes.


### Visuals


#### Diagrams (`.puml`)

System diagrams, container diagrams and diagrams of a service should be created and located in the `architecture-docs` repo.

ER-diagrams and sequence (flow) diagrams should **preferably** be placed in a repo next to API and database tables, but they also can be placed in the `architecture-docs` repo, if required.

Types of `.puml` diagrams:

* system diagram. `README.md` includes a snippet link to the service system `.puml` diagram at the very beginning under the short purpose description of the service;
* container diagram;
* service database section should include a snippet link to an ER-diagram;
* service operation algorithm section should include sequence `.puml` diagrams.


#### Gifs, Images and Screenshots

> **CAUTION**
>
> Use gifs, images ot screenshots where it is impossible to create documentation without them. Applying them it is always expensive for the following documentation support, so please think twice.

* for every image provide alt text (the context before an image) that summarize the intent of an image;
* each image should have a figure caption — a sentence under an image without a period at the end of describing sentence;
* don't use images of text, code samples, or terminal output;
* don't repeat images unless absolutely necessary;
* all personal data, including test data, on these files should be blurred;
* do not use images instead of diagrams — it's difficult to support them.

For the screenshots you can use the ["Take a screenshot on your MAC"](https://support.apple.com/en-us/HT201361) guide.

`.gif` files should be **680 × 394 pixels**. To create `.gif` files use [Gifski](https://gif.ski) converter.