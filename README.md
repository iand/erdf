# Introduction

This document describes how a subset of RDF can be embedded into XHTML or HTML by using common idioms and attributes. No new elements or attributes have been invented and the usages of the HTML attributes are within normal bounds. This scheme is designed to work with CSS and other HTML support technologies.

Note: hereafter the term HTML will be used to include both XHTML and HTML except where otherwise stated.

# Embeddable RDF

The subset of RDF that is used in this embedding scheme is called HTML Embeddable RDF. It allows some very important parts of the RDF model to be embedded but does not attempt to extend this to the full RDF model. This is very deliberate. Other attempts at embedding RDF in HTML have required the introduction of new syntax to express all the various RDF concepts.

The relationship is: all HTML Embeddable RDF is valid RDF, not all RDF is Embeddable RDF.

## An Example

This example shows how the rules later in this document can be used to embed FOAF into an HTML document. The HTML source is:

 
    <html>
      <head profile="http://purl.org/NET/erdf/profile">
        <title>Anna's Homepage</title>
        <base href="http://example.org/about" />
        <meta name="dc.creator" content="Anna Wilder" />
        <meta name="dc.title" content="Anna's Homepage" />
        <link rel="schema.dc" href="http://purl.org/dc/elements/1.1/" />
        <link rel="schema.foaf" href="http://xmlns.com/foaf/0.1/" />
        <link href="#anna" rev="foaf-homepage foaf-made" rel="foaf-maker" />
      </head>
      <body>
        <h2>About me...</h2>
        <p id="anna">

          Hi, I'm <span class="foaf-name"><span class="foaf-firstName">Anna</span> <span class="foaf-surname">Wilder</span></span>. 

          <img style="float: right" src="pic.jpg" class="foaf-depiction" alt="A picture of me"/>
       
          You might know me from IRC as <span class="foaf-nick">wildling</span> or sometimes <span class="foaf-nick">wilda</span>. 

          You can email me at <span class="foaf-mbox_sha1sum" title="69e31bbcf58d432950127593e292a55975bc66fd">anna {at} example.org</span>.
        </p>

      </body>
    </html>

This document specifies rules to turn that HTML into these RDF triples:

 
    <http://example.org/about> dc:creator "Anna Wilder" .
    <http://example.org/about> dc:title "Anna's Homepage" .
    <http://example.org/about> foaf:maker <http://example.org/about#anna> .
    <http://example.org/about#anna> foaf:homepage <http://example.org/about> .
    <http://example.org/about#anna> foaf:made <http://example.org/about> .
    <http://example.org/about#anna> foaf:name "Anna Wilder" .
    <http://example.org/about#anna> foaf:firstName "Anna" .
    <http://example.org/about#anna> foaf:surname "Wilder" .
    <http://example.org/about#anna> foaf:depiction <http://example.org/pic.jpg> .
    <http://example.org/about#anna> foaf:nick "wildling" .
    <http://example.org/about#anna> foaf:nick "wilda" .
    <http://example.org/about#anna> foaf:mbox_sha1sum "69e31bbcf58d432950127593e292a55975bc66fd" .


# Getting Started

For an HTML document to be recognised as containing Embeddable RDF it must declare conformance to a special profile. The profile declares that the document conforms to the conventions for embedding RDF. This is accomplished by adding the URI http://purl.org/NET/erdf/profile to the profile attribute on the document's head element like this:

 
    <html>
      <head profile="http://purl.org/NET/erdf/profile">
        ...
      </head>
      <body>
        ...
      </body>

    </html>

The profile attribute can include multiple profile URIs by separating them with spaces, e.g.:

 
    <head profile="http://example.com/other http://purl.org/NET/erdf/profile">

Applications desiring to extract the embedded RDF should examine the profile attribute for the presence of the string http://purl.org/NET/erdf/profile.

# Declaring Schemas To Be Used

Once the profile has been assigned, the author should add meta tags to the document declaring which schemas are to be used to embed triples. Each schema represents a collection of property names. References later in the document to undeclared schemas will be ignored.

The schema declaration associates a short schema prefix with a full URI, in a analagous way to namespaces in XML documents. The schema is declared using a link tag whose rel attribute encodes one or more schema prefix declarations using the following pattern:

    <link rel="schema.prefix" href="uri" />

For example, the following declaration:

    <link rel="schema.foaf" href="http://xmlns.com/foaf/0.1/" />

Would declare a schema prefix called foaf which represents the URI http://xmlns.com/foaf/0.1/. This method of declaring schemas is compatible to that used by Dublin Core to embed metadata in meta and link elements.

The order in which the schemas are declared is not significant. Where two identical schema prefixes are declared the first takes precedence. Any subsequent declarations are ignored.

The schema prefix should be considered to be case-insensitive, i.e. schema.foaf and schema.Foa F should be considered to be delaring the same prefix.

# Encoding Properties

Once declared, the schema prefix can be used to encode property UR Is defined within that schema by concatenating the schema prefix, a hypen (-) or dot (.) and the local name of the property. For example supposing the schema http://xmlns.com/foaf/0.1/ has been assigned the prefix foaf using a link tag as described above. Then the RDF property http://xmlns.com/foaf/0.1/name could be encoded to as foaf-name or as foaf.name.

The reason why two alternate forms are specified is to retain compatibility with the Dublin Core specification and also to ensure compatibility with CSS which uses dots to separate element names from classes for HTML documents. It is recommended that authors use the dot separator in the head of the document and the hyphen separator in the body of the document.

# Writing Metadata About The Current Document

In general, metadata about the current document is placed in the <head> section. Two elements can be used here to represent triples. The first is the meta element which can be used to embed triples with literal values. The property is encoded in the name attribute and the literal value is placed in the content attribute.

For example:

    <meta name="dc-creator" content="Ian Davis" />

would generate the following triple:

    <> dc:creator "Ian Davis" .


The second HTML element used in the head is link. link can be used to embed triples with a resource value. The rel attribute encodes the property name and the href attribute representes the URI of the resource being referenced. For example:

    <link rel="foaf.maker" href="#ian" />

would generate the following triple:

 
    <> foaf:maker <#ian> .


The rel attribute allows multiple values delimited by spaces. Each of these values should be interpreted as a separate property for a new triple. For example:

    <link rel="foaf.maker dc.creator foaf.topic" href="#ian" />

would generate the following three triples:

    <> foaf:maker <#ian> .
    <> dc:creator <#ian> .
    <> foaf:topic <#ian> .

link can also be used with the rev attribute to swap the subject and values of the embedded triples. The href of the link element encodes the subject and the document's URL becomes the value. For example:

    <link rev="foaf.made" href="#ian" />

would generate the following triple:

    <#ian> foaf:made <> .


It is also possible to embed metatada about the current document in the body. Any element that does not have an ancestor with an id attribute can potentially embed metadata about the document. Triples with literal values are embedded using elements with class attributes. Anchor elements with rel or rev attributes are used to embed triples with resource values.

When embedding a triple with a literal value, the class attribute is used to encode the property name in the same way that the rel attribute on the link element was used above. The literal value of the triple can be embedded in two ways: either in the title attribute of the element, or as the string-value of the element's content. Only one of these methods can be used at a time. If a title attribute is present, the element's string content is not considered to be part of the embedded triple. The string-value of an element is equal to the concatenation of the contents of all it's descendant text nodes. The resulting string should be the same as if the XPath string function had been used.

For example:

    <div><address class="dc-creator">Ian Davis</address> wrote this</div>

would be equivalent to the following triple:

    <> dc:creator "Ian Davis" .


Nested tags are removed when computing the string-value of the element, so the following would produce exactly the same triple:

    <div><address class="dc-creator"><strong>Ian <em class="surname">Davis</span></strong></address> wrote this</div>

Another way of embedding the same triple, using the title attribute would be:

    <div><span class="dc-creator" title="Ian Davis">I</span> wrote this</div>

The a tag is used to embed triples with resource values. As in the case for link the rel attribute encodes the property name and the href attribute representes the URI of the resource being referenced. For example:

    <a rel="foaf.homepage" href="http://example.org/home">my home page</a>

represents the following triple:

    <> foaf:homepage <http://example.org/home> .


Additionally, when anchor tags are used in this way a further rdfs:label property is inferred from the content of the anchor or from the title attribute. As usual the title attribute takes precedence over the anchor's content. In the previous example an additional triple would be generated:

 
    <http://example.org/home> rdfs:label "my home page" .


If the anchor had a title attribute

    <a rel="foaf.homepage" href="http://example.org/home" title="about me">my home page</a>

then the following triples would be generated:

    <> foaf:homepage <http://example.org/home> .
    <http://example.org/home> rdfs:label "about me" .


The img tag can also be used to embed triples with resource values. In this case the src attribute represents the URI of the resource being referenced and the class attribute encodes the property as normal:
 
    <img rel="foaf.img" src="http://example.org/picture" />

represents the following triple:

 
    <> foaf:img <http://example.org/picture> .


# Writing Metadata About Other Things

The id attribute can be used to define things within the current document with their own metadata. Any triples embedded within descendants of an element with an id attribute have a subject composed from the current document URI, a hash (#) and the value of the id attribute.

For example

    <p id="ian"><span class="foaf-name">Ian Davis</span> wrote this</p>

would generate the following triple:

    <#ian> foaf:name "Ian Davis" .


When a tag has multiple ancestors with id attributes, only the nearest is used to generate triples. For example:

    <div id="outer"><p id="inner"><span class="dc-title">A Value</span></p></div>

would generate the following triple:

    <#inner> dc:title "A Value" .

Anchors can be used to express relationships between things in the document and other resources. For example:

 
    <p id="ian"><a rel="foaf.homepage" href="http://example.org/home">my home page</a></p>

represents the following triple:

 
    <#ian> foaf:homepage <http://example.org/home> .


# RDF Classes

For tags with an id attribute and for anchors any tokens in the class attribute beginning with a hyphen are considered to be RDF class names. So the following XHTML:

  <p id="ian" class="-foaf-Person">I am a person</p>

generates the following triple:

 
  <#ian> rdf:type foaf:Person .


A more complex example:

    <p id="ian" class="-foaf-Person">
      <span class="foaf-name">Ian Davis</span> has a homepage
      <a href="http://purl.org/NET/iand" rel="foaf-homepage" class="-foaf-Document">here</a>
    </p>

Generates these triples:

    <#ian> rdf:type foaf:Person .
    <#ian> foaf:name "Ian Davis" .
    <#ian> foaf:homepage <http://purl.org/NET/iand> .
    <http://purl.org/NET/iand> rdf:type foaf:Document .


Or one with mixed properties and classes in the class attribute:

    <p id="ian" class="-foaf-Person">
     Ian is owed $1 by <span class="foaf-knows -foaf-Person" id="eric">Eric Miller</span>
    </p>

Which embeds the following triples:
 
  <#ian> rdf:type foaf:Person .
  <#ian> foaf:knows <#eric> .
  <#eric> rdf:type foaf:Person .

# Combinations

This section discusses situations in which combinations of the rel, rev, class and id attributes are used on elements and why you might want to combine them in these ways.

## Elements With Both class And id attributes

    <p><span class="dc-creator" id="ian">Ian Davis</span> wrote this</p>

generates a triple like this:

    <> dc:creator <#ian> .

Nested elements then have a subject derived from the id attribute, for example:
 
    <p><span class="dc-creator" id="ian"><span class="foaf-name">Ian Davis</span></span> wrote this</p>

generates two triples like this:

    <> dc:creator <#ian> .
    <#ian> foaf:name "Ian Davis" .

## Anchors With class And rel Attributes

The rel attribute defines the relationship between the subject (the document, or something identified within it) and the resource identified in the href attribute.

The class attribute defines the relationship between the subject (the document or something defined within it) and a literal value derived from the title attribute or element content. The class attribute never applies to the resource identified by the href attribute.

So this example encodes a vcard:url relationship between the resource #ian and the url http://example.com, plus it provides the literal text 'Ian' as the value of #ian's vcard-name property:

    <p id="ian">
      <a href="http://example.com/" rel="vcard-url" class="vcard-name">Ian</a>
    </p>

In triples:

    <#ian> vcard:url <http://example.com/> .
    <#ian> vcard:name "Ian" .

## Anchors With rel And rev Attributes

This usage pattern allows properties and their inverses to be embeded very succinctly. For example:

    <p id="ian">
      <a href="http://example.com/" rel="foaf-made" rev="foaf-maker">Ian</a>
    </p>

generates the following triples:

    <#ian> foaf:made <http://example.com> .
    <http://example.com> foaf:maker <#ian> .


## Anchors With rel And id Attributes

The rel attribute defines the relationship between the subject (the document, or something identified within it) and the resource identified in the href attribute.

The id attribute defines the start of a new referenceable resource, however the presence of an href attribute overrides the id attribute. The result is that the href attribute is used wherever the id would have been and the id is ignored.

So, for example:

    <p id="ian">
      <a rel="foaf-homepage" href="http://example.com/" id="homepage">
        <span class="dc-title">My home page</span>
      </a>
    </p>

generates two triples like this:
 
    <#ian> foaf:homepage <http://example.com/> .
    <http://example.com/> dc:title "My home page" .

## Anchors With rel, rev, class And id Attributes

The above rules still apply. The id attribute is ignored in the presence of an href attribute. The class attribute, as always, relates to the enclosing subject (document or part of document):

    <p id="ian">
      <a href="http://example.com/" rel="foaf-made" 
         rev="foaf-maker" id="homepage" class="foaf-homepageTitle">My home page</a>
    </p>

generates the following triples:
 
    <#ian> foaf:made <http://example.com> .
    <http://example.com/> foaf:maker <#ian> .
    <http://example.com/> foaf:homepageTitle "My home page" .

## Anchors With rel Or rev But No href Attribute

In the absence of an href attribute both rel and rev are ignored.

# Best Practice Notes

## Idiomatic HTML

Although this document does not assign any special meanings to HTML elements wherever possible elements with specific semantics as provided by the HTML specification should be preferred over the generic div and span elements. For example, when writing a name it is preferable to use the address element.

## Specifying the Base URI for the Document

It is recommended that the base element be used in the head of a document to specify the base URI for the document. This URI is used for determining the subject of triples.

For example:
 
    <html>
      <head profile="http://purl.org/NET/erdf/profile">
        <title>Homepage</title>
        <base href="http://example.org/about" />
      </head>
      <body>
        <p id="ian"><a rel="foaf.homepage" href="http://example.org/home">my home page</a></p>
      </body>
    </html>

would set the base URI for the document to http://example.org/about and generate the following triple:

 
    http://example.org/about#ian foaf:homepage <http://example.org/home> .


# Notes

## Compatibility With GRDDL

A GRDDL-capable client tool, when visiting a page which contains that profile reference will first visit the profile document (this document), apply the transformation declared here (using the data-view profile above). The results of that transformation will contain a statement identifying the transformation that should be applied to convert the document into RDF/XML. The GRDDL client has everything it needs to get RDF/XML from the original XHTML document, while the author of the document need not concern themselves with the details of the profile, merely including the reference to its URI is enough.

## Common Schemas

This section lists some common schemas in a format suitable for including in an HTML document:

 
    <link rel="schema.dc" href="http://purl.org/dc/elements/1.1/" />
    <link rel="schema.terms" href="http://purl.org/dc/terms/" />
    <link rel="schema.foaf" href="http://xmlns.com/foaf/0.1/" />
    <link rel="schema.bio" href="http://vocab.org/bio/0.1/" />
    <link rel="schema.contact" href="http://www.w3.org/2000/10/swap/pim/contact#" />
    <link rel="schema.air" href="http://www.daml.org/2001/10/html/airport-ont#" />
    <link rel="schema.rss" href="http://purl.org/rss/1.0/" />
    <link rel="schema.rel" href="http://purl.org/vocab/relationship/" />
    <link rel="schema.geo" href="http://www.w3.org/2003/01/geo/wgs84_pos#" />
    <link rel="schema.doap" href="http://usefulinc.com/ns/doap#" />
    <link rel="schema.rdfs" href="http://www.w3.org/2000/01/rdf-schema#" />
    <link rel="schema.wn" href="http://xmlns.com/wordnet/1.6/" />

# Implementations

**Editorial Note:** Due to the age of this document, these links are no longer functioning but are left here for historical referenece.

The following implementations of the embedding scheme described in this document are available:

* [An XSLT stylesheet implementing the rules described in this document](http://purl.org/NET/erdf/extract)
* [A transformation service using the above stylesheet](http://purl.org/NET/erdf/extract)
* [Embeddable RDF parser](http://arc.web-semantics.org/documentation/erdf_parser) in [ARC](http://arc.web-semantics.org/), see [webservice](http://arc.web-semantics.org/demos/erdf-extract)

# Further Reading

**Editorial Note:** These links have been moved to the github repository wiki

* Limitations of Embedding RDF in HTML
* Summary of Triple Production Rules
* FOAF in HTML
* DOAP in HTML
* RSS in HTML
* RDF Schemas in HTML
* Embedded RDF Profile Document 

# References

This document has drawn on several long-standing conventions:

* [A Proposed Convention for Embedding Metadata in HTML.](http://web.archive.org/web/20110429155116/http://www.w3.org/Search/9605-Indexing-Workshop/ReportOutcomes/S6Group2.html) - a report from the [W3C](http://web.archive.org/web/20110429155116/http://research.talis.com/2005/erdf/wiki/Main/W3C) Distributed Indexing and Searching Workshop, May 28-29, 1996
* [Expressing Dublin Core in HTML/XHTML meta and link elements](http://web.archive.org/web/20110429155116/http://www.dublincore.org/documents/dcq-html/) - embedding Dublin Core metadata into HTML documents 