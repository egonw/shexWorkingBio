# ShEx for the Working Bioinformatician

License: CC-BY 4.0 Int.

Author: Egon Willighagen <br />

With thanks to Andra Waagmeester for contributions.

# Introduction

[ShEx](http://shex.io/) or Shape Expressions is a formal language to describe ("express")
how a piece of RDF can take form ("shape"). The people that have worked with eXtensible Markup Language (XML)
in the past know the approach as schema, as in Document Type Definitions (DTD), XML Schema, or Schematron.
The people that know SPARQL, a shape expression is quite similar, but instead of asking for data, the
structure describes what data is expected. The people that do not know either, a shape expression basically
explains what information we expect for a certain resource.

This document gives a number of example shapes for frameworks bioinformaticians may work with.
It does not intend to replace these documents:

* [Micro-tutorial](https://rawgit.com/shexSpec/shex.js/wikidata/doc/micro-tutorial.html)
* [Primer](http://shex.io/shex-primer/index.html)
* [ShEx spec](http://shex.io/shex-semantics/index.html)

## WikiPathways RDF

We start with a basic shape expression for a WikiPathways RDF `DataNode` for a gene:

```turtle
<http://identifiers.org/ncbigene/5604>
        a                   wp:GeneProduct , wp:DataNode ;
        rdfs:label          "MAP2K1"^^xsd:string ;
        dc:identifier       <http://identifiers.org/ncbigene/5604> ;
        dc:source           "Entrez Gene"^^xsd:string ;
        dcterms:identifier  "5604"^^xsd:string ;
        dcterms:isPartOf    <http://identifiers.org/wikipathways/WP4806_r110852> ;
        wp:bdbEnsembl       <http://identifiers.org/ensembl/ENSG00000169032> ;
        wp:bdbEntrezGene    <http://identifiers.org/ncbigene/5604> ;
        wp:bdbHgncSymbol    <http://identifiers.org/hgnc.symbol/MAP2K1> ;
        wp:bdbUniprot       <http://identifiers.org/uniprot/Q02750> , <http://identifiers.org/uniprot/A4QPA9> , <http://identifiers.org/uniprot/H3BRW9> ;
        wp:isAbout          <http://rdf.wikipathways.org/Pathway/WP4806_r110852/DataNode/b64f4> .
```

We can define a shape expression that requires that all gene products have an identifier
and a Ensembl identifier mapped with BridgeDb:

```shex
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX wp: <http://vocabularies.wikipathways.org/wp#>

<gene> {
  dc:identifier IRI ;
  dc:source xsd:string ;
  wp:bdbEnsembl IRI
}
```

See also this [readme](wprdf/README.md).

### Interactions

For interactions in the pathways we can also define shape expressions. For example, a general interaction would look like this:

```turtle
@prefix wp:    <http://vocabularies.wikipathways.org/wp#> .
@prefix dcterms: <http://purl.org/dc/terms/> .

<http://rdf.wikipathways.org/Pathway/WP4806_r110852/WP/Interaction/ae20c>
        a                 wp:DirectedInteraction , wp:Interaction ;
        dcterms:isPartOf  <http://identifiers.org/wikipathways/WP4806_r110852> ;
        wp:isAbout        <http://rdf.wikipathways.org/Pathway/WP4806_r110852/Interaction/ae20c> ;
        wp:participants   <http://identifiers.org/ncbigene/2549> , <http://identifiers.org/ensembl/ENSG00000146648> ;
        wp:source         <http://identifiers.org/ensembl/ENSG00000146648> ;
        wp:target         <http://identifiers.org/ncbigene/2549> .
```

We can then define a simple shape for an interaction:

```shex
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX wp: <http://vocabularies.wikipathways.org/wp#>

<interaction> {
  wp:participants   IRI {1,} 
}
```

This shape can be extended to expect a `wp:source` and `wp:target` for directed interactions:

```shex
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX wp: <http://vocabularies.wikipathways.org/wp#>

<interaction> {
  wp:participants   IRI {2,} ;
  wp:source         IRI ;
  wp:target         IRI
}
```

See also this [readme](interaction/README.md).

## Databases in Wikidata



## Adverse Outcome Pathways

Another interesting aspect of ShEx is that shapes can reference other shapes. For example, take the
following adverse outcome pathway RDF (thx to Marvin for the example):

```turtle
@prefix aop.relationships: <http://identifiers.org/aop.relationships/> .
@prefix aop: <http://identifiers.org/aop/> .
@prefix aop.events: <http://identifiers.org/aop.events/> .
@prefix aopo: <http://aopkb.org/aop_ontology#> .
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

aop.relationships:597
        a       aopo:KeyEventRelationship ;
        dc:identifier   aop.relationships:597 ;
        rdfs:label      "KER 597" ;
        foaf:page       <http://identifiers.org/aop.relationships/597> ;
        dcterms:created "2016-11-29T18:41:34" ;
        dcterms:modified        "2016-12-03T16:37:58" ;
        aopo:has_upstream_key_event     aop.events:593 ;
        aopo:has_downstream_key_event   aop.events:585 ;
        dcterms:isPartOf        aop:95 .

aop.events:593
        a       aopo:KeyEvent ;
        dc:identifier   aop.events:593 ;
        rdfs:label      "KE 593" ;
        foaf:page       <http://identifiers.org/aop.events/593> ;
        dc:title        "Inhibition, Ether-a-go-go (ERG) voltage-gated potassium channel " ;
        dcterms:alternative     "Inhibition, Ether-a-go-go (ERG) voltage-gated potassium channel " ;
        dc:source       "AOPWiki" ;
        dcterms:isPartOf        aop:95 .
```

Here, each key event relationship requires two key events. We can then define two shapes and
say that the object of, in this case, the `aopo:has_upstream_key_event` predicate should be
something that adheres to the other shape:

```shex
prefix aopo: <http://aopkb.org/aop_ontology#>

<ker> {
  aopo:has_upstream_key_event     @<ke> ;
  aopo:has_downstream_key_event   @<ke>
}

<ke> {
   a [ aopo:KeyEvent ]
}
```

See also this [readme](aop/README.md).

## NanoSafety RDF

