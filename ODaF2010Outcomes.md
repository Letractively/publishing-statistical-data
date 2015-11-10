# SDMX-RDF progress at ODaF 2010 #

This page summarizes outcomes from the SDMX-RDF working discussions at
the Open Data Foundation (ODaF) 2010 meeting at Tilburg.

## Summary points ##

The meeting reinforced the value of establishing
standards and practice now for how to publish statistical and related data
in linked data form to avoid the chaos of multiple incompatible approaches evolving.

There are further potential use cases for linked data in statistics beyond the core
aggregated data targeted by SDMX-RDF. In particular, representing micro-data (survey information as supported by [DDI](http://www.ddialliance.org/)) or rather metadata for it would complement SDMX-RDF and, for example, support deep provenance track back.

We identified a need for a common core vocabulary for linked data publication of
data cubes for web-dissemination. The cube models for SDMX and DDI are already compatible, by design, so a single vocabulary for cubes could be extended by further
modular vocabularies for SDMX specific, microdata/DDI specific and other usages (e.g.
reference metadata, classifications). We partitioned the existing SDMX-RDF proposal
identifying the core data cube vocabulary and clarified a number of the associated
open issues.

Several of the current open issues are either not relevant to the data cube core, were resolved, or are simply a matter of doing the work. The primary open issues requiring further investigation before final resolution are:

  * flattening - whether dimension and attribute values should (always) be flattened down to the observation level
  * standardized URI schemes for identifying elements below DataSets
  * validation

## Data Cube vocabulary ##

The concepts from the current SDMX-RDF draft which should be in a core Data Cube vocabulary are:

  * DataSet
  * Observation
  * Dimension/Attribute/Measure and the associated packaging as ComponentProperties
  * DataStructureDefinition
    * enumeration of dims/attributes
    * code lists (via URI reference)
    * optionality
    * ordering
    * maybe attachment level (relates to _Flattening_ issue)
  * ConceptRoles (as subclasses of Dimension, no need for role properties)
  * Slice (an m-dimensional sub-cube)

### Note on Slices ###

It was felt that the ability to identify a slice of a DataSet, often but not always a 1-dimensional vector, is needed in the core. This is necessary to enable publishers to call out and annotate slices and for UIs to use these slices as the basis for presentation.

It was felt that emphasis on TimeSeries slices is specific to the statistics usage and should be in the SDMX vocabulary whereas the core Data Cube vocab should have a generic mechanism supporting m-dimensional slices.

Representing slices will require both a concept to represent the sub-key in the DSD and an concept for the collection of observations which make up a given slice instance. However, the slice is not intended to represent an arbitrary sparse pick from observations.

### SDMX extension ###

The concepts which should be in an extension SDMX vocabulary are everything else, specifically including:

  * DataFlow
  * Reporting Taxonomies
  * Section (1-dim slice)
  * TimeSeries (1-dim slice, over time dim)
  * Group
  * Constraints (arb. bounded/projected sub cubes)
  * DataProvider, Provision agreement ...
  * Annotation

## Substantive Issues list ##

### Flattening ###

Some attributes are common across all observations in a dataset or a slice,
some dimensions are (by definition) common across all observations in a slice.
In RDF it is preferable to be able to immediately find the dimension and attribute
values of each observation for both ease of query and to ensure the entire context of an extracted observation is known independent of the data organization above it.

The current SDMX-RDF proposal allows for both flattened and non-flattened forms. This felt like an awkward compromise to some delegates. An alternative proposal was discussed that you always flatten down to observations but also retain this information at the slice level since those drive UIs.

The problems with this are (a) lack of compactness and (b) developer surprise ("you want me to duplicate information?"). The latter seemed like a presentational issue.

Regarding compactness there was some feeling that that is the point of the compact XML representations and using RDF is already highly non-compact. For transfer of very large data sets use a representations optimized for that.

There was also concern over whether the ability to query across datasets in this fine grained way might, of itself, encourage users into a trap of making inappropriate comparisons.

The agreed way forward for this issue is to work through a concrete test case involving at least two datasets which are comparable (e.g. indicator datasets based on common admin geography and time) and two which are potentially incomparable (e.g. with non-overlapping population ages).

### Validation ###

The target use case of web dissemination means that we expect to normally be taking
well-structured data sets and simply making them available. So validation in the
sense of testing internal consistency in the source data is out of scope.

However, it is also important for consuming clients to be sure that they have obtained an intact and appropriate projection
from the source dataset. The inclusion of DSD information in the core Data Cube vocabulary supports this by declaring the dimensions and attributes which are required.

Further discussion is needed to clarify what precise semantics and processing behaviour should be implied and enforced as a result of such declarations.

### URI schemes ###

Underlying SDMX (or similar format) datasets do not natively have a URI associated with
all entities in the model. If URIs can be assigned arbitrarily then round tripping of a RDF cube data through SDMX would require annotations to record each URI. Similarly, tool chains which provide linked data views over SDMX sources will need to record or dynamically reconstruct URI assignment. There is likely to be a common need to synthesize URIs for elements below the DataSet level.  There was some discussion on whether such a URI assignment algorithm should be a mandated piece of a standard or a encouraged practice which can be adapted to local needs.

Note that URIs for identification are a different issue from URLs for addressing (e.g. via a RESTful query API) but two are related so the SDMX RESTful Web Services URI scheme is relevant.

The W3C [Media Fragment](http://www.w3.org/TR/media-frags/) scheme may also be relevant since observations and slices can be regarded as fragments of a single dataset object.

The agreed way forward on this issue is develop a proposed set of URI guidelines first and then decide how strongly to make these standards as opposed to guides. Note that this discussion and resulting action was part of the overall workshop plenary and not specific to the immediate Data Cube vocabulary.

## Other issues ##

The other issues mentioned during the discussions were:

  * classification and comparability  - this could be an opportunity for another modular vocabulary.
  * property reuse - current SDMX-RDF approach was seen as reasonable
  * COG usage - COG does include statements of the roles that each concept can play which can form the basis for a refined version of the COG RDF mapping, need to schedule and resource work on this
  * round tripping - largely resolved by partitioning to core data cube vocab
  * datatypes
  * metadata & MSDs - moved to either SDMX partition or separate reference metadata vocabulary
  * groups - moved to SDMX partition
  * reports - moved to SDMX partion
  * attachment level - part of flattening issue
  * cross-sections/constraints - moved to SDMX partition

## Next steps ##

  * Draft and review Data Cube vocabulary
  * Draft and review documentation for Data Cube vocabulary
  * Test case driven investigation into flattening issue
  * Refine (subset) COG RDF mapping