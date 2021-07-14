---
layout: post
title:  "[TRNS-EN] How browsers Work: Behind the scenes of modern web browsers (last update: 07.12.2021.)"
date:   2021-07-05 10:55:00 +0900
categories:
---

## [TRNS-EN] How browsers Work: Behind the scenes of modern web browsers (last update: 07.12.2021.)
`05.07.2021`
@yb park 

---

CONTENTS

[1. Introduction](#1-introduction)
    [1-1. The browsers we will talk about](#1-1-the-browsers-we-will-talk-about)
    [1-2. The browser's main functionality](#1-2-the-browsers-main-functionality)
    [1-3. The browser's high level structure](#1-3-the-browsers-high-level-structure)
[2. The rendering engine](#2-the-rendering-engine)
    [2-1. Rendering engines](#2-1-rendering-engines)
    [2-2. The main flow](#2-2-the-main-flow)
    [2-3. Main flow examples](#2-3-main-flow-examples)
[3. Parsing and DOM tree construction](#3-parsing-and-dom-tree-construction)
    [3-1. Parsing-general](#3-1-parsing-general)
    [3-2. Grammars](#3-2-grammars)
    [3-3. Parser-Lexer Combination](#3-3-parser-lexer-combination)
    [3-4. Translation](#3-4-translation)
    [3-5. Parsing example](#3-5-parsing-example)
    [3-6. Formal definitions for vocabulary and syntax](#3-6-formal-definitions-for-vocabulary-and-syntax)
    [3-7. Types of parsers](#3-7-types-of-parsers)
    [3-8. Generating parsers automatically](#3-8-generating-parsers-automatically)
[4. HTML Parser](#4-html-parser)
    [4-1. The HTML grammar definition](#4-1-the-html-grammar-definition)
    [4-2. Not a context free grammar](#4-2-not-a-context-free-grammar)
    [4-3. HTML DTD](#4-3-html-dtd)
    [4-4. DOM](#4-4-dom)
    [4-5. The parsing algorithm](#4-5-the-parsing-algorithm)
    [4-6. The tokenization algorithm](#4-6-the-tokenization-algorithm)
    [4-7. Tree construction algorithm](#4-7-tree-construction-algorithm)
    [4-8. Actions when the parsing is finished](#4-8-actions-when-the-parsing-is-finished)
    [4-9. Browser's error tolerance](#4-9-browsers-error-tolerance)
        [</br> instead of](#br-instead-of-br)
        [A stray table](#a-stray-table)
        [Nested form elements](#nested-form-elements)
        [A too deep tag hierarchy](#a-too-deep-tag-hierarchy)
        [Misplaced html or body tags](#misplaced-html-or-body-tags)
[5. CSS parsing](#5-css-parsing)
    [5-1. WebKit CSS parser](#5-1-webkit-css-parser)
[6. The order of processing  scripts and style sheets](#6-the-order-of-processing--scripts-and-style-sheets)
    [6-1. Scripts](#6-1-scripts)
    [6-2. Speculative parsing](#6-2-speculative-parsing)
    [6-3. Style sheets](#6-3-style-sheets)
[7. Renter tree construction](#7-renter-tree-construction)
    [7-1. The render tree relation to the DOM tree](#7-1-the-render-tree-relation-to-the-dom-tree)
    [7-2. The flow of constructing the ree](#7-2-the-flow-of-constructing-the-ree)
    [7-3. Style Computation](#7-3-style-computation)
    [7-4. Sharing style data](#7-4-sharing-style-data)
    [7-5. Firefox rule tree](#7-5-firefox-rule-tree)
        [7-5-1. Division into structs](#7-5-1-division-into-structs)
        [7-5-2. Computing the style contexts using the rule tree](#7-5-2-computing-the-style-contexts-using-the-rule-tree)
[8. Manipulating the rules for an easy match](#8-manipulating-the-rules-for-an-easy-match)
    [8-1. Applying the rules in the correct cascade order](#8-1-applying-the-rules-in-the-correct-cascade-order)
    [8-2. Style sheet cascade order](#8-2-style-sheet-cascade-order)
    [8-3. Specificity](#8-3-specificity)
    [8-4. Sorting the rules](#8-4-sorting-the-rules)
    [8-5. Gradual process](#8-5-gradual-process)
[9. Layout](#9-layout)
    [9-1. Dirty bit system](#9-1-dirty-bit-system)
    [9-2. Global and incremental layout](#9-2-global-and-incremental-layout)
    [9-3. Asynchronous and Synchronous layout](#9-3-asynchronous-and-synchronous-layout)
    [9-4. Optimizations](#9-4-optimizations)
    [9-5. The layout process](#9-5-the-layout-process)
    [9-6. Width calculation](#9-6-width-calculation)
    [9-7. Line Breaking](#9-7-line-breaking)
[10. Painting](#10-painting)
    [10-1. Global and Incremental](#10-1-global-and-incremental)
    [10-2. The painting order](#10-2-the-painting-order)
    [10-3. Firefox display list](#10-3-firefox-display-list)
    [10-4. WebKit rectangle storage](#10-4-webkit-rectangle-storage)
    [10-5. Dynamic changes](#10-5-dynamic-changes)
    [10-6. The rendering engine's threads](#10-6-the-rendering-engines-threads)
    [10-7. Event loop](#10-7-event-loop)
[11. CSS2 visual model](#11-css2-visual-model)
    [11-1. The canvas](#11-1-the-canvas)
    [11-2. CSS Box model](#11-2-css-box-model)
    [11-3. Positioning scheme](#11-3-positioning-scheme)
    [11-4. Box types](#11-4-box-types)
    [11-5. Positioning](#11-5-positioning)
        [11-5-1. Relative](#11-5-1-relative)
        [11-5-2. Floats](#11-5-2-floats)
        [11-5-3. Absolute and fixed](#11-5-3-absolute-and-fixed)
    [11-6. Layered representation](#11-6-layered-representation)
    
---


# 1. Introduction
Web browsers are the most widely used software. In this primer, I will explain how they work behind the scenes. We will see what happens when you type `google.com` in the address bar until you see the Google page on the browser screen.


## 1-1. The browsers we will talk about
There are five major browsers used on desktop today: Chrome, Internet Explorer, Firefox, Safari and Opera. On mobile, the main browsers are Android Browser, iPhone, Opera Mini and Opera Mobile, UC Browser, the Nokia S40/S60 browsers and Chrome-all of which, except for the Opera browsers, are based WebKit. I will give examples from the open source browsers Firefox and Chrome, and Safari(which is partly open source). According to `StatCounter statistics`(as of June 2013) Chrome, Firefox and Safari make up around 71% of global desktop browser usage. On mobile, Android Browser, iPhone and Chrome constitute around 54% of usage.


## 1-2. The browser's main functionality
The main function of a browser is to present the web resource you choose, by requesting it from the server and displaying it in the browser window. The resource is usually an HTML document, but may also be a PEF, image, or some other type of content. The location of the resource is specified by the user using a URI(Uniform Resource Identifier).

The way the browser interprets and displays HTML files is specified in the HTML and CSS specifications. The specifications are maintained by the W3C(World Wide Web Consortium) organization, which is the standards organization for the web. For years browsers conformed to only a part of the specifications and developed their own extensions. That caused serious compatibility issues for web authors. Today most of the browsers more or less conform to the specifications.

Browser user interfaces have a lot in common with each other. Among the common user interface elements are:

**- Address bar for inserting a URI**
**- Back and forward buttons**
**- Bookmarking options**
**- Refresh and stop buttons for refreshing or stopping the loading of current documents**
**- Home button that takes you to your page**

Strangely enough, the browser's user interface is not specified in any formal specification, it just comes from good practices shaped over years of experience and by browsers imitating each other. The HTML5 specification doesn't define UI elements a browser must have, but lists some common elements. Among those are the address bar, status bar and tool bar. There are, of course, features unique to a specific browser like Firefox's downloads manager.

## 1-3. The browser's high level structure
The browser's main components are (1.1):

1. **The user interface :** this includes the address bar, back/forward button, bookmarking menu, etc. Every part of the browser display except the window where you see the requested page.
2. **The browser engine :** marshals actions between the UI and the rendering engine.
3. **The rendering engine :** responsible for displaying requested content. For example if the requested content is HTML, the rendering engine parses HTML and CSS< and displays the parsed content on the screen.
4. **Networking :** for network calls such as HTTP requests, using different implementations for different platform behind a platform-independent interface.
5. **UI backend :** used for drawing basic widgets like combo boxes and windows. This backed exposes a generic interface that is not platform specific. Underneath it uses operating system user interface methods.
6. **JavaScript interpreter :** Used to parse and execute JavaScript code.
7. **Data storage :** This is persistence layer. The browser may need to save all sorts of data locally, such as cookies. browsers also support storage mechanisms such as localStorage, IndexedDB, WebSQL and FileSystem.
<br/>
<br/>
![high-level](/uploads/layers%20(1).png)
*Figure: Browser components*

It is important to note that browsers such as Chrome run multiple instances of the rendering engine: one for each tab. Each tab runs in a separate process.

<br/>

# 2. The rendering engine
The responsibility of the rendering engine is well... Rendering, that is display of the requested content on the browser screen. 
By default the rendering engine can display HTML and XML documents and images. It can display other types of data via plug-ins or extension; for example, displaying PDF documents using a PDF viewer plug-in. However, in the chapter we will focus on the main use case: displaying HTML and images that are formatted using CSS.

## 2-1. Rendering engines
Different browsers use different rendering engines: Internet Explorer uses Trident, Firefox uses Gecko, Safari uses WebKit. Chrome and Opera (from version 15) use Blink, a fork of WebKit. WebKit is an open source rendering engine which started as an engine for the Linux platform and was modified by Apple to support Mac and Windows. See [webkit.org](https://www.webkit.org) for more details.

## 2-2. The main flow
The rendering engine will start getting the contents of the requested document from the networking layer. This will usually be done in 8kB chunks. After that, this is the basic flow of the rendering engine:

![renderingflow](/uploads/renderingflow.png)
*Figure: Rendering engine basic flow*

The rendering engine will start parsing the HTML document and convert elements to DOM nodes in a tree called the "Content Tree". The engine will parse the style data, both in external CSS filles and in style elements. Styling information together with visual instructions in the HTML will be used to create another tree: the render tree.
The render tree contains rectangles with visual attributes like color and dimensions. The rectangles are in the right order to be displayed on the screen.
After the construction of the render tree it goes through a "layout" process. This means giving each node the exact coordinates where it should appear on the screen. The next stage is painting the render tree will be traversed and each node will be painted using the UI backend layer.

It's important to understand that this is a gradual process. For better user experience, the rendering engine will try to display contents on the screen as soon as possible. It will not wait until all HTML parsed before starting to build and layout the render tree. Parts of the content will be parsed and displayed, while the process continues with the rest of the contents that keeps coming from the network.

## 2-3. Main flow examples
![webkitmainflow](/uploads/webkitmainflow.png) *Figure: WebKit main flow*

![mozillamainflow](/uploads/mozilamainflow.jpg) *Figure: Mozilla's Gecko rendering engine main flow*

From figures 3 and 4 you can see that although WebKit and Gecko use slightly different terminology, the flow is basically the same.
Gecko calls the tree of visually formatted elements a "Frame Tree". Each element is a frame. WebKit uses the term "Render Tree" and it consists of "Render Objects". WebKit uses the term "layout" for the placing of elements, while Gecko calls it "Reflow". "Attachment" is WebKit's term for connecting DOM nodes and visual information to create the render tree. A minor non-semantic difference is that Gecko has an extra layer between the HTML and the DOM tree. It is called the "content sink" and is a factory for making DOM elements. We will talk about each part of the flow:

<br/>

# 3. Parsing and DOM tree construction
## 3-1. Parsing-general
Since parsing is a very significant process within the rendering engine, we will go into it a little more deeply. Let's begin with a little introduction about parsing.
Parsing a document means translating it to a structure the code can use. The result of parsing is usually a tree of nodes that represent the structure of the document. This is called a pase tree or a syntax tree.

For example, parsing the expression `2 + 3 - 1` could return this tree:

![mathematicalexpression](/uploads/mathematicalexpression.png) *Figure: mathematical expression tree node*

## 3-2. Grammars
Parsing is based on the syntax rules the document obeys: the language or format it was written in. Every format you can parse must have deterministic grammar consisting of vocabulary amd syntax rules. It is called a context free grammar. Human languages are not such languages and therefore cannot be parsed with conventional parsing techniques.

## 3-3. Parser-Lexer Combination
Parsing can be separated into two processes: lexical analysis and syntax analysis.
Lexical analysis is the process of breaking the input into tokens. Tokens are the language vocabulary: the collection of valid building blocks. In human language it will consist of all the words that appear in the dictionary for the language.
Syntax analysis is the applying of the language syntax rules.

Parsers usually divide the work between two components: the **lexer** (sometimes called tokenizer) that is responsible for breaking the input into valid tokens, and the **parser** that is responsible for constructing the parse tree by analyzing the document structure according to the language syntax rules. The lexer knows how to strip irrelevant characters like white spaces and line breaks.

![parsetrees](/uploads/parsetrees.png) *Figure: from source document to parse trees*

The parsing process is iterative. The parser will usually ask the lexer for a new token and try to match the token with one of the syntax rules. If a rule is marched, a node corresponding to the token will be added to the parse tree and the parser will ask for another token.
If no rule matches, the parser will store the token internally, and keep asking for tokens until a rule matching all the internally stored tokens is found. If no rule is found then the parser will raise an exception. This means the document was not valued and contained syntax errors.

## 3-4. Translation
In many cases the parse tree is not the final product. Parsing is often used in translation: transforming the input document to another format. An example is compilation. The compiler that compiles source code into machine code first parses it into a parse tree and then translates the tree into a machine code document.

![compilationflow](/uploads/compilationflow.png) 

## 3-5. Parsing example
In figure 5 we built a parse tree from a mathematical expression. Let's try to define a simple mathematical language and see the parse process.
Vocabulary: Our language can include integers, plus signs and minus signs.

Syntax:
1. The language syntax building blocks are expressions, terms and operations.
2. Our language can include any number of expressions.
3. An expressions is defined as a "term" followed by an "operation" followed by another term.
4. An operation is a plus token or a minus token.
5. A term is an integer token or an expression

Let's analyze the input `2 + 3 - 1`
The first substring that matches a rule is `2` : according to rule #5 it is a term. The second match is `2 + 3`: this matches the third rule: a term followed by an operation followed another term. The next match will only be hit at the end of the input. `2 + 3 - 1` is an expression because we already know that `2 + 3` is a term, so we have a term followed by an operation followed another term. `2 + +` will not match any rule and therefore is an invalid input.

## 3-6. Formal definitions for vocabulary and syntax
Vocabulary is usually expressed by regular expressions.
For example our language will be defined as:

~~~
INTEGER: 0|[1-9]|[0-9]*
PLUS: +
MINUS: -
~~~

As you see, integers are defined by a regular expression.
Syntax is usually defined in a format called BNF. Our language will be defined as:
~~~
expression := term operation term
operation := PLUS | MINUS
term := INTEGER | expression
~~~

We said that a language can be parsed by regular parsers if its grammar is a context free grammar. An intuitive definition of a context free grammar is a grammar that can be entirely expressed in BNF. For a formal definition see [Wikipedia's article on Context-free grammar](https://en.wikipedia.org/wiki/Context-free_grammar).

## 3-7. Types of parsers
There are two types of parsers: top down parsers and bottom up parsers. An intuitive explanation is that top down parsers examine the high level structure of the syntax and try to find a rule match. Bottom up parsers start with the input and gradually transform it into the syntax rules. starting from the low level rules until high level rules are met.

Let's see how the tow types of parsers will parse our example.

The top down parser will start from the higher level rule: it will identify `2 + 3` as an expression. It will then identify `2 + 3 - 1` as an expression (the process of identifying the expression evolves, matching the other rules, but the start point is the highest level rule).

The bottom up parser will scan the input until a rule is matched. It will then replace the matching input with the rule/ this will go on until the end of the input. The partly matched expression is placed on the parser's stack.

![stackinput](/uploads/stackinput.png)

This type of bottom up parser is called a shift-reduce parser, because the input is shifted to the right (imagine a pointer pointing fist at the input start and moving to the right) and is gradually reduced to syntax rules.

## 3-8. Generating parsers automatically
There are tools that can generate a parser. You feed them the grammar of your language-its vocabulary and syntax rules-abd they generate a working parser. Creating a parser requires a deep understanding of parsing and it's not easy to create an optimized parser by hand, so parser generators can be very useful.

WebKit uses two well known parser generators: Flex for creating a lexer and Bison for creating a parser (you might run into them with the names Lex and Yacc). Flex input is a file containing regular expression definitions of the tokens. Bison's input us the language syntax rules in BNF format.

<br/>


# 4. HTML Parser
The job of the HTML parser is to parse the HTML markup into a parse tree.

## 4-1. The HTML grammar definition
The vocabulary and syntax of HTML are defined in specification create by the W3C organization.

<br/>


## 4-2. Not a context free grammar
As we have seen in the parsing introduction, grammar syntax can be defined formally using formats like BNF.
Unfortunately all the conventional parser topics do not apply to HTML (I didn't bring them up just for fun- they will be used in parsing CSS and JavaScript). HTML cannot easily be defined by a context free grammar that parsers need. There is a formal format for defining HTML-DTD (Document Type Definition) - but it is not a context free grammar.

This appears strange at first sight; HTML is rather close to XML. There are lots of available XML parsers. There is an XML variation of HTML - XHTML- so what's the bit difference?

The difference is that the HTML approach is more "forgiving": it lets you omit certain tags (which are then added implicitly), or sometimes omit start and end tags, and so on. On the whole it's a "soft" syntax, as opposed to XML's stiff and demanding syntax.
This seemingly small detail makes a world of a difference. On one hand this is the main reason why HTML is so popular: it forgives your mistakes and makes life easy for the web author. On the other hand, it makes it difficult to write a formal grammar. So to summarize, HTML cannot be parsed easily by conventional parsers, since its grammar is not context free. HTML cannot be parsed by XML parsers.

<br/>


## 4-3. HTML DTD
HTML definition is in a DTD format. This format is used to define languages of the [SGML(Standard Generalized Markup Language)](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language) family. The format contains definitions for all allowed elements, their attributes and hierarchy. As we say earlier, the HTML DTD doesn't form a context free grammar.

There are a few variations of the DTD. The strict mode conforms solely to the specifications but other modes contain support for markup used by browsers in the past. The purpose is backward compatibility with older content. The current strict DTD is here: [www.w3.org/TR/html4/strict.dtd](www.w3.org/TR/html4/strict.dtd). 


<br/>

## 4-4. DOM
The output tree (the "parse tree") is a tree of DOM element and attribute nodes. DOM is short for Document Object Model. It is the object presentation of the HTML document and the interface of HTML elements to the outside world like JavaScript. 
The root of the tree is the "Document" object.

The DOM has an almost one-to-one relation to the markup. e.g.:
{% highlight html %}
{% raw %}
<html>
    <body>
    <p>
    Hello World
    </p>
    </body>
    <div><img src="example.png"></div>
</html>
{% endraw %}
{% endhighlight %}

This markup would be translated to the following DOM tree:

![dommarkup](/uploads/dommarkup.png) *Figure: DOM tree of the example markup**

Like HTML, DOM is specified by the W3C organization. See [https://www.w3.org/DOM/DOMTR](https://www.w3.org/DOM/DOMTR). It is generic specification for manipulating documents. A specific module describes HTML specific elements. The HTML definition can be found here: [www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html](www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html).

When I say the tree contains DOM nodes, I mean the tree is constructed of elements that implement one of the DOM interfaces. Browsers use concrete implementations that have other attributes used by the browser internally.

<br/>


## 4-5. The parsing algorithm
As we saw in the previous sections, HTML cannot be parsed using the regular top down or bottom up parsers.

The reasons are:

1. The forgiving nature of the language.
2. The fact that browsers have traditional error tolerance to support well known cases of invalid HTML.
3. The parsing process is reentrant. For other languages the source doesn't change during parsing, but in HTML, dynamic code (such as script elements containing `document.write()` calls) can add extra tokens, so the parsing process actually modifies the input.

Unable to use the regular parsing techniques, browsers create custom parsers for parsing HTML.

The [parsing algorithm is described in detail by the HTML5 specification](https://html.spec.whatwg.org/multipage/parsing.html). The algorithm consists of two stages: tokenization and tree construction.

Tokenization is the lexical analysis, parsing the input into tokens. Among HTML tokens are start tags, end tags, attribute names and attribute values.
The tokenizer recognizes the token. gives it to the tree constructor, and consumes the next character for recognizing the next token, and so on until the end of the input.

![htmlparsingflow](/uploads/htmlparsingflow.png) *Figure: HTML parsing flow(taken from HTML5 spec)*

<br/>


## 4-6. The tokenization algorithm
The algorithm's output is an HTML token. The algorithm is expressed as a state machine. Each state consumes one or more characters of the input stream and updates the next state according to those characters. The decision in influenced by the current tokenization state and by the tree construction state. This means the same consumed character will yield different results for the correct next state, depending on the current state. The algorithm is too complex to describe fully, so let's see a simple example that will help us understand the principle.

Basic example-tokenizing the following HTML:

{% highlight html %}
{% raw %}
<html>
    <body>
    Hello World
    </body>
</html>
{% endraw %}
{% endhighlight %}

The initial state is the "Data state". When the &lt; character is encountered, the state is changed to **"Tag open state"**. Consuming an `a-z` character causes creation of a **"Start tag token"**, the state is changed to "Tag name state". We stay in this state until the &gt; character is consumed. Each character is appended to the new token name. In our case the created token is an `html` token.

When the &gt; tag is reached, the current token is emitted and the state changes back to the **"Data state"**. The `<body>` tag will be treated by the same steps. So far the `html` and `body` tags were emitted. We are now back at the **"Data state"**. Consuming the `H` character of `Hellow World` will cause creation and emitting of a character token, this goes on until the &lt; of `</body>` is reached. We will emit a character token for each character of `Hello World`.

We are now back at the **"Tag open state"**. Consuming the next input `/` will cause creation of an `end tag token` and a move to the **"Tag name state"**. Again we stay in this state until we react &gt;. Then the new tag token will be emitted and we go back to the **"Data state"**. The `</html>` input will be treated luke the previous case.

![tokenizingthe](/uploads/tokenizingthe.png) *Figure: Tokenizing the example input*

<br/>


## 4-7. Tree construction algorithm

When the parser is created the Document object is created. During the tree construction stage the DOM tree with the Document in its root will be modified and elements will be added to it. Each node emitted by the tokenizer will be processed by the tree constructor. For each token the specification defines which DOM element is relevant to it and will be created for this token. The element is added to the DOM tree, and also the stack of open elements. This stack is used to correct nesting mismatches and unclosed tags. The algorithm is also described as a state machine. The states are called "insertion modes".

Let's see the tree construction process for the example input:

{% highlight html %}
{% raw %}
<html>
    <body>
    Hello World
    </body>
</html>
{% endraw %}
{% endhighlight %}

The input to the tree construction stage is a sequence of tokens from the tokenization stage. The first mode is the **"initial mode"**. Receiving the "html" token will cause a move to the **"before html"** mode and a reprocessing of the token in that mode. This will cause creation of the HTMLHtmlElement element, which will be appended to the root Document object.

The state will be changed to **"before head"**. The "body" token is then received. An HTMLHeadElement will be created implicitly although we don't have a "head" token and it will be added to the tree.

We now move to the **"in head"** mode and then to **"after head"** The body token is reprocessed, an HTMLBodyElement is created and inserted and the mode is transferred to **"in body"**.

The character tokens of the "Hello world" string are not received. The first one will cause creation and insertion of a "Text" node and the other characters will be appended to that node.

The receiving of the body end token will cause a transfer to **"after body"** mode. We will now receive the html end tag which will move us to **"after after body"** mode. Receiving the end of file token will end the parsing.

![treeconstruction](/uploads/treeconstruction.jpg) *Figure: tre construction of example html*

<br/>



## 4-8. Actions when the parsing is finished
At this stage the browser will mark the document as interactive and start parsing scripts that are in "deferred" mode: those that should be executed after the document is parsed. The document state will be then set to "complete" and a "load" event will be fired.

You can see [the full algorithms for tokenization and tree construction in the HTML5 specification](https://html.spec.whatwg.org/multipage/parsing.html#html-parser).

<br/>


## 4-9. Browser's error tolerance

You never get an "Invalid Syntax" error on an HTMl page. Browsers fix any invalid content and go on.

Take this HTML for example:

{% highlight html %}
{% raw %}
<html>
    <mytag></mytag>
    <div>
        <p>
    </div>
    Really lousy HTML
    </p>
</html>
{% endraw %}
{% endhighlight %}

I must have violated about a million rules ("mytag" is not a standard tag, wrong nesting of the "p" and "div" elements and more) but the browser still shows it correctly and doesn't complain. So a lot of the parser code is fixing the HTML author mistakes.

Error handling is quite consistent in browsers, but amazingly enough it hasn't been part of HTML specifications. Like bookmarking and back/forward buttons it's just something that developed in browsers over the years. There are known invalid HTML construct repeated on many sites, and the browsers try to fix them in a way conformant with other browsers.

The HTML5 specification does define some of these requirements. (WebKit summarizes this nicely in the comment at the beginning of the HTML parser class.)

> The parser parses tokenized input into the document, building up the document tree.
> If the document is well-formed, parsing it is straightforward.
>  Unfortunately, we have to handle many HTML documents that are not well-formed, 
> so the parser has to be tolerant about errors.
> We have to take care of at least the following error conditions:

> 1. *The element being added is explicitly forbidden inside some outer tag. In this case we should close all tags up to the one which forbids the element, and add it afterwards.*
> 2. *We are not allowed to add the element directly. It could bge that the person writing the document forgot some tag in between (or that the tag in between is optional). This could be the case with the following tags: HTML HEAD BODY TBODY TR TD LI.*
> 3. *We want to add a block element inside an inline element. Close all inline elements up to the next higher block element.*
> 4. *If this doesn't help, close elements until we are allowed to add the element-or ignore the tag.*

<br/>
Let's see some WebKit error tolerance examples: 

<br/>


### &lt;/br&gt; instead of &lt;br&gt;

Some sites use `</br>` instead of `<br>`. In order to be compatible with IE and Firefox, WebKit treats this like `<br>`.

The code:
~~~java
if (t-> isCloseTag(brTag) && m_document->inCompatMode()){
    reportError(MalformedBRError);
    t->beginTag = true;
}
~~~
Note that the error handling is internal: it won't be presented to the user.

<br/>

### A stray table

A stray table is a table inside another table, but not inside a table cell.
For example:
{% highlight html %}
{% raw %}
 <table>
    <table>
        <tr>
            <td>inner table</td>
        </tr>
    </table>
    <tr>
        <td>outer table</td>
    </tr>
 </table>
{% endraw %}
{% endhighlight %}

WebKit will change the hierarchy to two sibling tables:

{% highlight html %}
{% raw %}
  <table>
        <tr>
            <td>inner table</td>
        </tr>
    </table>
<table>
    <tr>
        <td>outer table</td>
    </tr>
 </table>
{% endraw %}
{% endhighlight %}

The code:
~~~java
if (m_inStrayTableContent && localName == tableTag)
    popBlock(taleTag);
~~~

<br/>

### Nested form elements
In case the user puts a form inside another form, the second form is ignored.
The code:
~~~java
if (!m_currentFormElement){
    m_currentFormElement = new HTMLFormElement(formTag, m_document);
}
~~~

<br/>

### A too deep tag hierarchy
The comment speaks for itself.

> www.liceo.edu.mx is an example of a site that archives a level of nesting of about 1500 tags, all from a bunch of &lt;b&gt;s. We will only allow at most 20 nested tags of the same type before just ignoring them all together.

~~~java
bool HTMLParser::allowsNestedRedundantTag(const AtomicString& tagName){
    unsigned i = 0;
    for (HTMLStackElem* curr = m_blockStack;
        i < cMaxRedundantTagDepth && curr && curr->tagName == tagName;
            curr = curr->next, i++) {}
    return i != cMaxRedundantTagDepth;
}
~~~

<br/>

### Misplaced html or body tags
Again-the comment speaks for itself.

> Support fo really broken HTML. We never close the body tag, since some stupid web pages close it before the actual end of the doc. Let's rely on the end() call to close things.

~~~java
if (t->tagName == htmlTag || t->tagName == bodyTag) return;
~~~
  
  
So web authors beware-unless you want to appear as an example in a WebKit error tolerance code snippet-write well formed HTML.

<br/>

# 5. CSS parsing

Remember the parsing concepts in the introduction? Well, unlike HTML, CSS is a context free grammar and can be parsed using the types of parsers described in the introduction. In fact the CSS specification defines CSS lexical and syntax grammar.
Let's see some examples:
The lexical grammar (vocabulary) is defined by regular expressions for each token:

~~~
comment \/\*[^*]*\*+([^/*][^*]*\*+)*\/
num [0-9]+|[0-9]*"."[0-9]+
nonascii [\200-\377]
nmstart [_a-z] | {nonascii} | {escape}
nmchar [_a-z0-9-] | {nonascii} | {escape}
name {npchar} +
ident {nmstart}{nmchar}*
~~~

"ident" is short for identifier, like a class name. "name" is an element id (that is referred by "#")
The syntax grammar is described in BNF.

~~~
ruleset
    : selector [',' S* selector ]*
    '{'S* declaration [';' S* declaration ]* '}' S*
    ;
selector
    : simple_selector [ combinator selector | S+ [ combinator? selector]? ]?
    ;
simple_selector
    : element_name [ HASH | class | attrib | pseudo ]*
    | [ HASH | class | attrib | pseudo ]+
    ;
class 
    : '.' IDENT
    ;
attrib 
    : '[' S* IDENT S* [ [ '=' | INCLUDES | DASHMATCH ] S*
    [ IDENT | STRING ] S* ] ']'
    ;
pseudo
    : ':' [ IDENT | FUNCTION S* [IDENT S*] ')' ]
    ;
~~~

Explanation: A ruleset is this structure:
~~~
div.error, a.error {
    color: red;
    font-weight: bold;
}
~~~

div.error and a.error are selectors. This part inside the curly braces contains the rules that are applied by the ruleset. This structure is defined formally in this definition:
~~~
ruleset
: selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S* 
;
~~~

This means a ruleset is a selector or optionally a number of selectors separated by a comma and spaces (S stands for white space). A ruleset contains curly braces and inside them a declaration or optionally a number of declarations separated by a semicolon. "declaration" and "selector" will be defined in the following BNF definitions.

<br/>

## 5-1. WebKit CSS parser
WebKit uses Flex and Bison parser generators to create parsers automatically from the CSS grammar files. As you recall from the parser introduction, Bison creates a bottom up shift-reduce parser. Firefox users a top down parser written manually. In both cases each CSS file is parsed into a StyleSheet object. Each object contains CSS rules. The CSS rule objects contain selector and declaration objects and other objects corresponding to CSS grammar.



![parserCSS](/uploads/parserCSS.png) *Figure: parsing CSS*

<br/>

# 6. The order of processing  scripts and style sheets
## 6-1. Scripts
The model of the web is synchronous. Authors expect scripts to be parsed and executed immediately when the parser reaches a `<script>` tag. The parsing of the document halts until the script has been executed. If the script is external then the resource must first be fetched from the network-this is also done synchronously, and parsing halts until the resource is fetched. This was the model for many years and is also specified in HTML4 and 5 specifications. Authors can add the "defer" attribute to a script, in which case it will not halt document parsing and will execute after the document is parsed. HTML5 adds an option to mark the script as asynchronous so it will be parsed and executed by a different thread.

<br/>


## 6-2. Speculative parsing
Both WebKit and Firefox do this optimization. While executing scripts, another thread parses the rest of the document and finds out what other resources need to be loaded from the network and loads them. In this way, resources can be loaded on parallel connections and overall speed is improved. *Note: the speculative parser only parses references to external resources like external scripts, style sheets and images: it doesn't modify the DOM tree-that is left to the main parser.*


<br/>

## 6-3. Style sheets
Style sheets on the other hand have a different model. Conceptually it seems that since style sheets don't change the DOM tree, there is no reason to wait for them and stop the document parsing. There is an issue, though, of scripts asking for style information during the document parsing stage. If the style is not loaded and parsed yet, the script will get wrong answers and apparently this caused lots of problems. It seems to be an edge case but is quite common. Firefox blocks all scripts when there is a style sheet that is still being loaded and parsed. WebKit blocks scripts only when they try to access certain style properties that may be affected by unloaded style sheets.

# 7. Renter tree construction
While the DOM tree is being constructed, the browser construct another tree, the render tree. This tree is of visual elements in the order in which they will be displayed. It is the visual representation of the document. The purpose of this tree is to enable painting the contents in their correct order.

Firefox calls the elements in the render tree "frames". WebKit uses the term renderer or render object.
A renderer knows how to lay out and paint ifself and its children.
WebKit's RenderObject class, the base class of the renderers, has the following definition:

~~~java
class RenderObject{
    virtual void layout();
    virtual void paint(PaintInfo);
    virtual void repaintRect();
    Node* node; // the DOM node
    RenderStyle* style; // the computed style
    RenderLayer* containingLayer; // the containing z-index layer
}
~~~

Each renderer represents a rectangular area usually corresponding to a node's CSS box, as described by the CSS2 spec. it includes geometric information like width, height and position. The box type is affected by the "display" value of the style attribute that is relevant to the node (see the style computation section). Here is WebKit code for deciding what type of renderer should be created for a DOM node, according to the display attribute:

~~~java
RenderObject* RenderObject::createObject(Node* node, RenderStyle*style){
    Document* doc = node->document();
    RenderArena* arena = doc->renderArena();
    ...
    RenderObject* o = 0;

    switch(style->display()){
        case NONE:
            break;
        case INLINE:
            o = new (arena) RenderInline(node);
            break;
        case BLOCK: 
            o = new (arena) RenderBlock(node);
            break;
        case INLINE_BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case LIST_ITEM:
            o = new (arena) RenderListItem(node);
            break;
        ...  
    }
    return o;
}
~~~

The element type is also considered: for example, form controls and tables have special frames. 
InWebKit if an element wants to create a special renderer, it will override the `createRenderer()` method. The renderers point to style objects that contains non-geometric information.

<br/>


<br/>


## 7-1. The render tree relation to the DOM tree
The renderers correspond to DOm elements, but the relation is not one to one. Non-visual DOm elements will not be inserted in the render tree. An example is the "head" element. Also elements whose display value was assigned to "none" will not appear in the tree (whereas elements with "hidden" visibility will appear in the tree).

There are DOm elements which correspond th several visual objects. There are usually elements with complex structure that cannot be described by a single rectangle. For example, the "select" element has three renderers: one for the display area, one for the drop down list bos and one for the button. Also when text is broken into multiple lines because the width is not sufficient for one line, the new lines will be added as extra renderers.

Another example of multiple renderers is broken HTML. According to the CSS spec an inline element must contain either only block elements or only inline elements. In the case of mixed content, anonymous block renderers will be created to wrap the inline elements.

Sone render objects correspond to a DOM node but not in the same place in the tree. Floats and absolutely positioned elements are out of flow, placed in a different part of the tree, and mapped to the real frame. A place holder frame is where they should have been.

![therendertree](/uploads/therendertree.png) *Figure: The render tree and the corresponding DOM tree (3.1). The "Viewport" is the initial containing block. In WebKit it will be the "RenderView" object.*

<br/>


## 7-2. The flow of constructing the ree
In Firefox, the presentation is registered as a listener for DOM updates. The presentation delegates frame creation to the `FrameConstructor` and the constructor resolves style (see style computation) and creates a frame.

InWebKit the process of resolving the style and creating a renderer is called "attachment". Every DOm node has an "attach" method. Attachment is synchronous, node insertion to the DOm tree calls the new node "attach" method.

Processing the html and body tags results in the construction of the render tree root. The root render object corresponds to what the CSS spec calls the containing block: the top most block that contains all other blocks. Its dimensions are the viewport: the calls it `RenderView`. This is the render objects that the document points to. The rest of the tree is constructed as a DOM nodes insertion.

See the CSS2 spec on the processing model.

<br/>


## 7-3. Style Computation
Building the render tree requires calculating the visual properties of each render object. 
This is done by calculating the style properties of each element.

The style includes style sheets of various origins, inline style elements and visual properties in the HTML (like the "bgcolor" property). The later is translated to matching CSS style properties.

The origins of style sheets are the browser's default style sheets, the style sheets provided by the page author and user style sheets-these are style sheets provided by the browser user (browsers let you defined your favorite styles. InFirefox, for instance, this is done by placing a style sheet in the "Firefox Profile" folder).

Style computation brings ip a few difficulties:
1. Style data is a very large construct, holding the numerous style properties, this can cause memory problems.
2. Finding the matching rules for each element can cause performance issues if it's not optimized. Traversing the entire rule list for each element to find matches is a heavy task. Selectors can have complex structure that can cause the matching process to start on a seemingly promising path that is proven to be futile and another path has to be tried.
Means the rules apply to a `<div>` who is the descendant of 3 divs. Suppose you want to check if the rule applies for a given `<div>` element. You choose a certain path up the tree for checking. You may need to traverse the node tree up just to find out there are only two divs and the rule does not apply. you then need to try other paths in the tree.
3. Applying the rules involves quite complex cascade rules that defined the hierarchy of the rules.

(2.)For example-this compound selector:
~~~html
div div div div {
    ...
}
~~~


Let's see how the browsers face these issues:

<br/>


## 7-4. Sharing style data
WebKit nodes references style objects (RenderStyle). These objects can be shared by nodes in some conditions. The nodes are siblings of cousins and:

1. The element must be in the same mouse state (e.g., one can't be in :hover while the other isn't it.)
2. Neither element should have an id
3. The tag names should match
4. The class attributes should match
5. The set of mapped attributes must be identical
6. The link states must match
7. The focus states must match
8. Neither element should be affected by attribute selectors, where affected us defined as having any selector match that uses an attribute selector in any position within the selector at all
9. There must be no inline style attribute on the elements
10. There must be no sibling selectors in use at all. WebCore simply throws a global switch when any sibling selectors in use at all. WebCore simply throws a global switch when any sibling selector is encountered and disables style sharing for the entire document when they are present. This includes the `+` selector and selectors like `:first-child` and `:lst-child`.
    
<br/>    

## 7-5. Firefox rule tree
Firefox has two extra trees for easier style computation: the rule and style context tree. WebKit also has style objects but they are not stored in a tree like the style context tree, only the DOm node points to its relevant style.


![firefoxcontexttree](/uploads/firefoxcontexttree.png) *Figure: Firefox context tree(2.2)*

The style context contain end values. The values are computed by applying all the matching rules in the correct order and performing manipulations that transform them from logical to concrete values. For example, if the logical value is a percentage of the screen it will be calculated and transformed to absolute units. The rule tree idea is really clever. it enables sharing these values between nodes to avoid computing them again.
This also saves space.

All the matched rules are stored in a tree. The bottom nodes in a path have higher priority. The tree contains all the paths for rule matches that were found. Storing the rules is done lazily. The tree isn't calculated at the beginning for every node, but whenever a node style needs to be computed the computed paths are added to the tree.

The idea is to see the tree paths as words in a lexicon. Lets say we already computed this rule tree:


![computedtree](/uploads/computedtree.png)

Suppose we need to match rules for another element in the content tree, and find out the matched rules (in the correct order) are B-E-I. We already have this path in the tree because we already computed path A_B_E_I_L. We will now have less work to do.

Let's see how the tree saves us work.
### 7-5-1. Division into structs
The style contexts are divided into structs. Those structs contain style information for a certain category like border or color. All the properties in a struct are either inherited or non inherited. Inherited properties are properties that unless defined by the element, are inherited from its parent. Non inherited properties (called "reset" properties) use default values if not defined. 

The tree helps us by caching entire structs (containing the computed end values) in the tree. The idea is that if the bottom node didn't supply a definition for a struct, a cached struct in an upper node can be used.

### 7-5-2. Computing the style contexts using the rule tree
When computing the style context for a certain element, we first compute a path in the rule tree or use an existing one. We them begin to apply the rules in the path to fill the struct in our new style context. We start at the bottom node of the path-the one with the highest precedence (usually the most specific selector) and traverse the tree up until our struct is full. If there is no specification for the struct in that rule node, then we can greatly optimize-we go up the tree until we find a node that specifies it fully and simply point to it-that's the best optimization- the entire struct is shared. This saves computation of end vales and memory.
if we find partial definitions we go up the tree until the struct is filled.

If we didn't find any definitions for our struct then, in case the struct is an "inherited" type, we point to the struct of our parent in the **context tree**. In this case we also succeeded in sharing structs. If it's a reset struct then default values will be used.

If the most specific node does add values then we need to do some extra calculations for transforming it to actual values. We then cache the result in the tree node so it can be used by children.

In case an element has a sibling or a brother that points to the same tree node then the **entire style context** can be shared between them.
Lets see an example: Suppose we have this HTML

~~~HTML
<html>
    <body>
        <div class="err" id="div1">
            <p>
                this is <span class="big"> big error </span>
                this is also a 
                <span class="big"> very big error </span> error
            </p>
        </div>
    </body>
</html>
~~~

And the following rules:
~~~css
1. div { margin: 5px; color: black; }
2. .err { color: red }
3. .big { margin-top: 3px }
4. div span { margin-bottom: 4px }
5. #div1 { color: blue }
6. #div2 { color: green }
~~~

To simplify things let's say we need to fill out only two struct: the color struct and the margin struct. The color struct contains only one member: the color
The margin struct contains the four sides.
The resulting rule tree will look like this (the nodes are marked with the node name: the number of the rule they point at):

![theruletree](/uploads/theruletree.png) *Figure: The rule tree*

The context tree will look like this (node name: rule node they point to):

![thecontexttree](/uploads/thecontexttree.png) *Figure: The context tree*

Suppose we parse the HTML and get to the second `<div>` tag. We need to create a style context for this node and fill its style structs.
create a style context for this node and fill its style structs.
We will match the rules and discover that the matching rules for the `<div>` are 1, 2 and 6. This means there is already an existing path in the tree that our element can use and we just need to add another node to it for rule 6 (node F in the rule tree).
We will create a style context and put it in the context tree. The new style context will point to node F in the rule tree.

We now need to fill the style structs. We will begin by filling out the margin struct. Since the last rule node (F) doesn't add to the margin struct, we can to up the tree until we find a cached struct computed in a previous node insertion and use it. We will find it on node B. which is the uppermost node that specified margin rules.

We do have a definition for the color struct, so we can't use a cached struct. rules and come to the conclusion that it points to rule G, like the previous span. Since we have siblings that point to the same node. we can share the entire style context and just point to the context of the previous span.

For struct that contain rules that are inherited from the parent, caching is done on the context tree (the color property is actually inherited, but Firefox treats it as reset and caches it on the rule tree).
For instance if we added rules for fonts in a paragraph:

~~~css
p { font-family: Verdana; font-size: 10px; font-weight: bold }
~~~

Then the paragraph element, which is a child of the div in the context tree, could have shared the same font struct as his parent. This is if no font rules were specified for the paragraph.

In WebKit, who does not have a rule tree, the matched declarations are traversed four times. First non-important high priority properties are applied (properties that should be applied first because others depend on them, such as display), then high priority important rules. This means that properties that appear multiple times will be resolved according to the correct cascade order. The last wins.

So to summarize: sharing the style objects (entirely or some of the structs inside them) solves issues 1 and 3. The Firefox rule tree also helps in applying the properties in the correct order.


<br/>


## 8. Manipulating the rules for an easy match
There are several sources for style rules:
* CSS rules, either in external style sheets or in style elements.
~~~css
p { color: blue }
~~~

* Inline style attributes like
    
~~~html
<p style="color: blue"></p>
~~~

* HTML visual attributes (which are mapped to relevant style rules)

~~~
<p bgcolor="blue"> </p>
~~~

The last two are easily matched to the element since he owns the style attributes and HTML attributes can be mapped using the element as the key.

As noted previously in **issue #2**, the CSS rule matching can be trickier. To solve the difficulty, the rules are manipulated for easier access.

After parsing the style sheet, the rules are added to one of several hash maps, according to the selector. There are maps by id, by class name, by tag name and a general map for anything that doesn't fit into those categories. If the selector is an id, the rule will be added to the id map, if it's a class it will be added to the class map etc.


The manipulation makes it much easier to match rules. There is no need to look in every declaration: we can extract the relevant rules for an element from the maps. This optimization eliminates 95+% of the rules, so that they need not even be considered during the matching process(4.1.).

Let's see for example the following style rules:

~~~css
p.error { color: red }
#messageDiv { height: 50px }
div { margin: 5px }
~~~

The first rule will be inserted into the class map. The second into the id map and the third into the tag map.
For the following HTMl fragment;

~~~html
<p class="error">an error occurred </p>
<div id="messageDiv">this is a message </div>
~~~

We will first try to find rules for the p element. The class map will contain an "error" key under which the rule for "p.error" is found. The div element will have relevant rules in the id map (the key is the id) and the tag map. So the only work left is finding out which of the rules that were extracted by the keys really match.

For example if the rule for the div was

~~~css
table div { margin: 5px }
~~~

it will still be extracted from the tag map, because the key is the rightmost selector, but it would not match our div element, who does not have a table ancestor.

Both WebKit Firefox do this manipulation.

<br/>



## 8-1. Applying the rules in the correct cascade order
The style object has properties corresponding to every visual attribute (all CSS attributes but more generic). If the property is not defined by any of the matched rules, then some properties can be inherited by the parent element style object. Other properties have default values.

The problem begins when there is more than one definition- here cones the cascade order to solve the issue.


<br/>

## 8-2. Style sheet cascade order

A declaration for a style property can appear in several style sheets, and several times inside a style sheet. This means the order of applying the rules is very important. This is called the "cascade" order. According to CSS2 spec, the cascade order is (from low to high):

    1. Browser declarations
    2. User normal declarations
    3. Author normal declarations
    4. Author important declarations
    5. User important declarations

The browser declarations are lease important and the user overrides the author only if the declaration marked as important. Declarations with the same order will be sorted by specificity. and then the order they are specified. The HTML visual attributes are translated to matching CSS declarations. They are treated as author rules with low priority.


<br/>

## 8-3. Specificity

The selector specificity is defined by the CSS specification as follows:

* count 1 if the declaration it is from is a 'style' attribute rather than a rule with a selector, 0 otherwise (= a)
* count the number of ID attributes in the selector (= b)
* count the number of other attributes and pseudo-classes in the selector (= c)
* count the number of element names and pseudo-elements in the selector (= d)

Concatenating the four numbers a-b-c-d (in a number system with a large base) gives the specificity.

The number base you need to use is defined by the highest count you have in one of the categories.
For example, if a=14 you can use hexadecimal base. In the unlikely case where a=17 you will need a 17 digits number base. The later situation can happen with a selector like this: html body div div p ... (17 tags in your selector.. not very likely).

Some examples:

~~~css
*{} /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
li{} /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */  
li:first-line{} /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */  
ul li{} /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */  
ul ol+li{} /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */  
h1+*[rel=up]{} /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */  
ul ol li.red{} /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */  
li.red.level{} /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */  
#x34y{} /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
style="" /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */  
~~~

<br/>


## 8-4. Sorting the rules

After the rules are matched, they are sorted according to the cascade rules. WebKit uses bubble sort for small lists and merge sort fot big ones. WebKit implements sorting by overriding the ">" operator for the rules:

~~~java
static bool operator >(CSSRuleData& r1, CSSRuleData& r2){
    int spec1 = r1.selector()->specificity();
    int spect2 = r2.selector()->specificity();
    return (spec1 == spec2) : r1.position() > r2.position() : spec1 > spec2;
} 
~~~

<br/>


## 8-5. Gradual process

WebKit uses a flag that marks if all top level style sheets (including @imports) have been loaded. If the style is not fully loaded when attaching, place holders are used and it is marked in the document, and they will be recalculated once the style sheets were loaded.

<br/>

# 9. Layout
When the renderer is created and added to the tree, it does not have a position and size. Calculating these values is called layout or reflow.

HTMl uses a flow based layout model, meaning that most of the time it is possible to compute the geometry in a single pass. Elements later "in the flow" typically do not affect the geometry in a single pass. Elements later "in the flow" typically do not affect the geometry of elements that are earlier "in the flow", so layout can proceed let-to-right, top-to-bottom through the document. There are exceptions: for example, HTML tables may require more than one pass (3.5).

The coordinate system is relative to the root frame. Top and left coordinates are used.

Layout is a recursive process. It begins at the root renderer, which corresponds to the `<html>` element of the HTML document. Layout continues recursively through some or all of the frame hierarchy, computing geometric information for each renderer that requires it.

The position of the root renderer is 0,0 and its dimensions are the viewport-the visible part of the browser window.

All renderers have a "layout" or "reflow" method, each renderer invokes the layout method of its children that need layout.


<br/>

## 9-1. Dirty bit system
In order not to do a full layout for every small change, browsers use a "dirty bit" system. A renderer that is changed or added marks itself and its children as "dirty": needing layout.

There are two flags: "dirty", and "children are dirty" which means that although the renderer itself may be OK, it has at least one child that needs a layout.

<br/>



## 9-2. Global and incremental layout

Layout can be triggered on the entire render tree-this is "global" layout. This can happen as a result of:

1. A global style change that effects all renderers, like a font size change.
2. As a result of a screen being resized

Layout can be incremental, only the dirty renderers will be laid out (this can cause some damage which will require extra layouts). 
Incremental layout is triggered (asynchronously) when renderers are dirty. for example when new renderers are appended to the render tree after extra content came from the network and was added to the DOM tree.


![incrementallayout](/uploads/incrementallayout.png) *Figure: Incremental layout-only dirty renderers and their children are laid out (3.6.)*

<br/>


## 9-3. Asynchronous and Synchronous layout

Incremental layout is done asynchronously. Firefox queues "reflow commands" for incremental layouts and a scheduler triggers batch execution of these commands. WebKit also has a timer that executes an incremental layout -the tree is traversed and "dirty" renderers are layout out.
Scripts asking for style information, like "offsetHeight" can trigger incremental layout synchronously.


<br/>

## 9-4. Optimizations

When a layout is triggered by a "resize" or a change in the renderer position (and not size), the renders sizes are taken from a cache and not recalculated..

In some cases only a sub tree is modified and layout does not start from the root. This can happen is cases where the change is local and does not affect its surroundings-like text inserted into text fields (otherwise every keystroke would trigger a layout starting from the root).

<br/>


## 9-5. The layout process

The layout usually has the following pattern:

1. Parent renderer determines its own width
2. Parent goes over children and:
   1. Place the child renderer (sets its X and Y)
   2. Calls child layout if needed-they are dirty or we are in a global layout, or for some other reason-which calculates the child's height
3. Parent uses children's accumulative heights and the heights of margins and padding to set its own height-this will be used by parent renderer's parent.
4. Sets its dirty bit to false

Firefox uses a "state" object (nsHTMLReflowState) as a parameter to layout (termed "reflow"). Among others the state includes the parent width.
The output of the Firefox layout is a "metrics" object (nsHTMLReflowMetrics). It will contain the renderer computed height.

<br/>


## 9-6. Width calculation

The renderer's width is calculated using the container block's width, the renderer's style "width" property. the margins and borders.
For example the width of the following div:
~~~html
<div style="width: 30%">
</div>
~~~

Would be calculated by WebKit as the following (class RenderBox method calcWidth)

* The container width is the maximum of the containers availableWidth and 0.
* The availableWidth in this case is the contentWidth which is calculated as:

~~~js
clientWidth() - paddingLeft() - paddingRight()
~~~

clientWidth and clientHeight represent the interior of an object excluding border and scrollbar.
* The elements width is the "width" style attribute. It will be calculated as an absolute value by computing the percentage of the container width.
* The horizontal borders and paddings are now added.

So far this was the calculation of the "preferred width". Now the minimum and maximum widths will be calculated.
If the preferred width is greater then the maximum width, the maximum width is used. If it is less then the minimum width (the smallest unbreakable unit) then the minimum width is used.

The values are cached in case a layout is needed, but the width does not change.

<br/>


## 9-7. Line Breaking

When a renderer in the middle of a layout decides that is needs to break, the renderer stops and propagates to the layout's parent that it needs to be broken. The parent creates the extra renderers and calls layout on them.


<br/>

# 10. Painting

In the painting stage, the render tree is traversed and the renderer's "paint()" method is called to display content on the screen. Painting uses the UI infrastructure component.

<br/>


## 10-1. Global and Incremental

Like layout, painting can also be global-the entire tree is painted-or incremental. In incremental painting, some of the renderers change in a way that does not affect the entire tree. The changed renderer invalidates its rectangle on the screen. This causes the OS to see it as a "dirty region" and generate a "paint" event. The OS does it cleverly and coalesces several regions into one. in Chrome it is more complicated because the renderer is in a different process then the main process. Chrome simulates the OS behavior to some extent. The presentation listens to these events and delegates the message to the render root. The tree is traversed until the relevant renderer is reached. It will repaint itself (and usually its children).

<br/>


## 10-2. The painting order

CSS2 defined the order of the painting process. This is actually the order in which the elements are stacked in the stacking contexts. This order affects painting since the stacks are painted from back to front. The stacking order of a block renderer is:

1. background color
2. background image
3. border
4. children
5. outline

<br/>


## 10-3. Firefox display list

Firefox goes over the render tree and builds a display list for the painted rectangular. it contains the renderers relevant for the rectangular, in the right painting order (backgrounds of the renderers, then borders etc.). That way the tree needs to be traversed only once for a repaint instead of several times- painting all backgrounds, then all images, then all borders etc.

Firefox optimizes the process by not adding elements that will be hidden, like elements completely beneath other opaque elements.


<br/>

## 10-4. WebKit rectangle storage

Before repainting, WebKit saves the old rectangle as a bitmap. It then paints only the delta between the new and old rectangles.

<br/>


## 10-5. Dynamic changes

The browsers try to do the minimal possible actions in response to a change. So changes to an element's color will cause only repaint of the element. Changes to the element position will cause layout and repaint of the element, its children and possibly siblings. Adding a DOM node will cause layout and repaint of the node. Major changes, like increasing font size of the "html" element. will cause invalidation of caches, relayout and repaint of the entire tree.


<br/>

## 10-6. The rendering engine's threads

The rendering engine is single threaded. Almost everything, except network operations, happens in a single thread. In Firefox and Safari this the main thread of the browser. In Chrome it's the tab process main thread.
Network operations can be performed by several parallel threads. The number of parallel connections is limited. (usually 2-6 connections)

<br/>


## 10-7. Event loop

The browser main thread is an event loop. it's an infinite loop that keeps the process alive. it waits for events (like layout and paint event) and processes them. This is Firefox code for the main event loop:

~~~java
while (@mExiting)
    NS_ProcessnextEvent(thread);
~~~

<br/>

# 11. CSS2 visual model

## 11-1. The canvas

According to the CSS2 specification, the term canvas describes "the space where the formatting structure is rendered": where the browser paints the content. The canvas is infinite for each dimensions of the space but browsers choose an initial width based on the dimensions of the viewport.

According the [https://www.w3.org/TR/CSS2/zindex.html](https://www.w3.org/TR/CSS2/zindex.html), the canvas is transparent if contained within another, and given a browser defined color it it is not.

<br/>


## 11-2. CSS Box model

The Css box model describes the rectangular boxes that are generated for elements in the document tree and laid out according to the visual formatting model.
Each box has a content area (e.g. text, an image, etc.) and optional surrounding padding, border, and margin areas.

![boxmodel](/uploads/boxmodel.jpg) *Figure: CSS2 box model*

Each node generates 0..n such boxes.
All elements have a "display" property that determines the type of box that will be generated.
Examples:

~~~
block: generates a block box.
inline: generates one or more inline boxes.
none: no box is generated.
~~~

The default is inline but the browser style sheet may set other defaults. For example: the default display for the "div" element is block.
you can find a default style sheet example here:
[www.w3.org/TR/CSS2/sample.html](www.w3.org/TR/CSS2/sample.html)

<br/>


## 11-3. Positioning scheme

There are three schemes:

1. Normal: the object is positioned according to its place in the document. This means its place in the render tree is like its place in the DOM tree and laid out according to its box type and dimensions
2. Float: the object is first laid out like normal flow, then moved as far left or right as possible
3. Absolute: the object is put in the render tree in a different place than in the DOM tree

The positioning scheme is set by the "position" property and the "float" attribute.
* static and relative case a normal flow
* absolute and fixed cause absolute positioning

In static positioning no position is defined and the default positioning is used. In the other schemes, the author specifies the position: top, bottom, left, right.

The way the box is laid out is determined by:

* Box type
* Box dimensions
* positioning scheme
* External information such as image size and the size of the screen

<br/>


## 11-4. Box types 

Block box: forms a block-has its own rectangle in the browser window.

![blockbox](/uploads/blockbox.png) *Figure: Block box*

Inline box: does not have its own block, but is inside a containing block.

![inlineboxes](/uploads/inlineboxes.png) *Figure: Inline boxes*

Blocks are formatted vertically one after the other. Inline are formatted horizontally.

![blockand](/uploads/blockand.png) *Figure: Block and Inline formatting*

Inline boxes are put inside lines or "line boxes". The lines are at least as tall as the tallest box but can be taller, when the boxes are aligned "baseline" -meaning the bottom part of an element is aligned at a point of another box other then the bottom. If the container width is not enough, the inlines will be put on several lines. 
This is usually what happens in a paragraph.


![lines](/uploads/lines.png) *Figure: Lines*

<br/>


## 11-5. Positioning
### 11-5-1. Relative
Relative positioning-positioned like usual and then moved by the required delta.

![relativepositioning](/uploads/relativepositioning.png) *Figure: Relative positioning*

### 11-5-2. Floats

A float box is shifted to the left or right of a line. The interesting feature is that the other boxes flow around it. The HTML:

~~~html
<p>
    <img style="float: right" src="images/image.gif" width="100" height="100">
    Lorem

</p>
~~~

Will look like:

![float](/uploads/float.png) *Figure: Float*

### 11-5-3. Absolute and fixed

The layout is defined exactly regardless of the normal flow. The element does not participate in the normal flow. The dimentions are relative to the container. In fixed, the container is the viewport.

![fixedpositioning](/uploads/fixedpositioning.png) *Figure: Fixed positioning*

*Note: the fixed box will not move even when the document is scrolled!*

<br/>


## 11-6. Layered representation

This is specified by the z-index CSS property. It represents the third dimension of the box: its position along the "z axis".

The boxes are divided into stacks (called stacking contexts). Ineach stack the back elements will be painted first and the forward elements on top, closer to the user. In case of overlap the foremost element will hide the former element.
the stacks are ordered accodring to the z-index property. Boxes with "z-index" property form a local stack. The viewport has the outer stack.

Example:
~~~html
<style type="text/css">
      div {
        position: absolute;
        left: 2in;
        top: 2in;
      }
</style>

<p>
    <div
         style="z-index: 3;background-color:red; width: 1in; height: 1in; ">
    </div>
    <div
         style="z-index: 1;background-color:green;width: 2in; height: 2in;">
    </div>
 </p>
~~~

The result will be this: 

![fixedpositioning2](/uploads/fixed.png) *Figure: Fixed positioning*

Although the red div precedes the green one in the markup, and would have been painted before in the regular flow, the z-index property is higher, so it is more forward in the stack held by the root box.

