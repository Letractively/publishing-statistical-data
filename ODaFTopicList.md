**Scope discussion**

## High level topics ##

  * DDI and micro data
  * registry
  * naming, standardization, maintainability
  * classification and comparability

## SDMX-RDF closure ##

  * Property reuse
  * COG usage
  * round tripping
  * validation
  * datatypes
  * metadata & MSDs
  * groups
  * reports
  * attachment level, optionality
  * cross-sections
  * URI construction patterns (new)
  * versioning/maintenance (new)

## Breakout 1 - raw notes ##

**Standardization.** Broad agreement that it would be good if SDMX-RDF sticks as a name and the vocabulary ends up within the SDMX organization. This would be as a profile and associated "syntax" in an annexe, important that no changes to the information model be required.

Some concerns over whether this would affect timeliness.

**Scoping.** Broad agreement that we are looking for a web profile of SDMX for dissemination, with an initial phase that is _good enough_ for this purpose. So we can separate features into not required at all for such a profile and maybe required not but critical path.

Arofan gave examples Structure Sets and process as definitely not in in phase 1, though some solution for mapping might be required within the profile by phase 2. Constraints are an intermediate case, they _are_ useful for UI purposes but are complex and weren't implemented for a long time in SDMX tools.

We also made a distinction between the ability to _cast_ between the SDMX-ML and SDMX-RDF representations within the profile and full round tripping at a document level.

**Flattening.** Discussed the flattening of attributes and dimension assertions from groups/series/sections down to observations for ease of querying and cross-set combination in the RDF world. The current SDMX-RDF proposal allows for both flattened and non-flattened forms. This felt like an awkward compromise to some delegates. Alternative proposal that you always flatten down to observations but also retain series/sections with their annotations since those drive UIs (but not Groups). This flattening would avoid a lot of complexities around section attachments.

The problems with this are (a) lack of compactness and (b) developer surprise ("you want me to duplicate information?").  The latter seemed  like a presentational issue.

Regarding compactness there was some feeling that that is the point of the compact XML representations and using RDF is already highly non-compact. For transfer of very large data sets use the representations optimized for that.

We did not close out this issue.

**URIs and round tripping.** The compactness discussion led on to a discussion of round tripping from SDMX-RDF through SDMX-ML. The current proposal allows arbitrary URI structures for the observations and series with an encouragement for locally defined (rather than globally standardized) patterns of URI construction. Observations and series are addressable via query and URI synthesis is not required, nor desirable, for retrieval. This would mean round tripping would require an annotation within SDMX-ML for preservation of URI assignments.

The issue of standardization of URI patterns, how that interacts with versioning but also tracability and provenance for recombined information is a new topic for the topic list (added above). We had some preliminary discussion on this but did not reach an initial consensus.