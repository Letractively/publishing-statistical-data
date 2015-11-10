# Requirements and issues for representing code values #

[Issue 22](https://code.google.com/p/publishing-statistical-data/issues/detail?id=22) concerns the representation of code lists. It turns out there are several requirements here. This page is a place to iteratively create a write up of those requirements and constraints so we can match them up to the possible solutions.

## SDMX Code lists ##

Dimensions in SDMX are represented as _Coded values_ drawn from a controlled list. As part of defining the structure of a dataset (the DSD) you define what code list will be used for each coded property. This supports validation (you can detect illegal codes) and comparison across datasets which use the same code list.

In RDF it is natural to represent an individual code as a resource (i.e. by a URI). The question is how do we represent the collection of resources that make up a given controlled code list.  Among the options are:

  * SKOS concept schemes
  * OWL classes
  * a customized vocabulary

These are not disjoint. A common design pattern is to use an OWL class to represent all the codes within a given `skos:conceptScheme`, e.g. in order to declare the range of a property.

Some code lists used in statistical data are "naturally" thought of as concepts in the sense of classification schemes, controlled term lists and thesauri. An example is the SDMX COG concept of gender (CL\_SEX). This is not a code list for the real world concept of gender but a controlled set of labels for use in recording gender information in data. For example, the CL\_SEX code list includes terms like "Not Specified". This is a perfect match to SKOS.

However, for some dimensions then there are existing sets of identifiers (URIs) which could reasonably be used to encode the dimension's value. Examples, of these include geographical or administrative regions, time periods and the various URI Sets being developed by the UK Government Data Project (e.g. to identify schools, companies, hospitals etc). In these cases the identifiers identify the entity itself not a thesaurus term for the entity - a school in the Schools URI Set has a location (for example) and one would not have a term like "not specified" as part of the Schools URI Set.

## Requirements ##

  * **Validation**. It should be possible to validate instance data, to test that the presented value for a dimension is legal with respect to the definition of that dimension.

  * **SDMX COG**. It should be easy to use code lists from the SDMX Content Oriented Guidelines.

  * **Reuse**. Where sets of identifiers already exist it should be easy to reuse them. It would be preferable to be able to directly use identifiers from URI Sets such as time periods, administrative regions and Schools without have to create a duplicate code list for them.

  * **Enumeration**. For finite code lists it should be possible to discover the all the legal codes in that code list, particular for using in UIs. Not that some code lists (e.g. reference time periods) are unbounded.

  * **SDMX compatibility**. Translation of domain specific SDMX code lists should be straightforward.

  * **Simplicity**. The solution should be simple in terms of authoring and consuming SDMX-RDF (ideally the information is written down once rather than duplicated, no intended consequences).

Others?

_In the discussion I think Richard might also have been asking for the ability to define subsets of external code lists. I haven't added that explicitly because that doesn't seem to differentiate any of the choices - by definition you have to do some duplication then._

_We could require the ability to "close" the list so you can guarantee the enumeration is complete and can't be affected by finding more RDF statements in some other document. That isn't possible with SKOS or simple "x rdf:type t" statements, you have to use lists with all the attendant problems of SPARQL query._

Happy to add any of these.

## Discussion ##

### Current design ###

The current design uses `sdmx:codeList` to attach a code list to an `sdmx:CodedProperty` and the code list itself is represented as a `skos:ConceptScheme`.

_Validation_. Partial. It is easy to navigate from the DSD to the code list but for hierarchical code lists the validator then needs to traverse the `skos:ConceptScheme`. Existing RDF validators such as Eyeball or Pellet-IC would not work and SPARQL based validators (Schemarama, SPIN) could not perform the hierarchy traversal.

_COG_. Easy, we've already translated to the COG code lists to SKOS.

_Reuse_. Problematic. The terms in the URI Sets we wish to reuse in data.gov.uk are generally not `skos:Concepts` - both in the sense that they are not declared that way at the moment and in the sense that that would be a surprising modelling. This would mean either creating a parallel SKOS thesaurus corresponding to the terms in the identifier sets or accepting the cognitive dissonance of e.g. finding that your town is also a SKOS Concept.

_Enumeration_. Enumeration is possible but requires hierarchy traversal and so is not a simple lookup. A user interface would, however, tend to present the hierarchy in any case and so this may not be extra work.

_SDMX Compatibility_. Good. The code list is directly attached to the Dimension declaration.

_Simplicity_. Fine.