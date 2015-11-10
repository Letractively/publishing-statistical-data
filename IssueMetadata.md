# SDMX Metadata sets #

## Background ##

SDMX-RDF (or Stats RDF or whatever it gets renamed to) currently supports publication of data sets and associated structure definitions but doesn't yet cover the publication of Reference Metadata. This is a feature of SDMX that we need to support. This page gives outline background and working space in which to develop a proposal.

## SDMX structures ##

SDMX supports publication of _reference metadata_, that is metadata which is not integral with the data but independent from the datasets themselves. Such metadata can be attached at all levels from observations, through time series and datasets, up to data flows and agencies though it is typically used for the dataset and above levels.
> the SDMX Information Model reference metadata is supported by a structure which directly mirrors the DataSet structure, indeed both are specializations of the generic Structure/StructureUsage pattern which is part of the SDMX Base Layer.

The key concepts are:

  * Metadata Structure Definition (MSD), analogous to Dataset Structure Definition, which defines:
    * where the metadata can be attached
    * the index key that defines that attachment
    * the structure of a Metadata Report which comprises a nestable hierarchical set of metadata attributes

  * MetadataSet, analogous to a DataSet, which comprises:
    * an attachment key which connects the MetadataSet to the allowed attachment level in the corresponding data flow
    * a series of MetadataAttributeValues which give values for the attributes allowed by the MSD
    * an effectiveDate stamp for the metadata

  * MetadataAttribute, defines an individual attribute that can be included in a metadata report
    * can have an associated concept defining the the semantics of the attribute
    * can have a local type declaration to override the type of the concept
    * can have a local representation declaration to override the representation of the concept
    * can be Coded or Uncoded
    * can be arrange in parent/child hierarchy - this is purely for presentation purposes and does not affect semantics, represented in SMDX-ML as nesting of the MetadataAttribute declarations
    * have have associated AttributeProperty annotations  which "allows the MetadataAttribute to have identifiable text such as a URL"
    * can be declared, within the MSD, as  optional or mandatory

## Proposed changes to SDMX-RDF ##

The proposed approach is to directly mirror the DSD structure to handle MSDs and use the same machinery of component properties for MetadataAttributes including generating COG-based predefined attributes.

### Metadata Structure Definition ###

The proposed top level structure is:

```
sdmx:MetadataStructureDefinition a rdfs:Class, owl:Class;
    rdfs:label "Metadata structure definition"@en;
    rdfs:comment "Defines the structure of a MetadataSet"@en;
    rdfs:subClassOf sdmx:Structure;
    .

sdmx:MetadataReportStructure a rdfs:Class, owl:Class;
    rdfs:label "Metadata report structure"@en;
    rdfs:comment "Defines the structure of a single report within an MSD"@en;
	.
	
sdmx:metadataReportStructure a rdf:Property, owl:ObjectProperty;
    rdfs:label "metadata report structure"@en;
    rdfs:comment """designates a single report structure within an overall metadata structure definition"""@en;
    rdfs:domain sdmx:MetadataStructureDefinition;
    rdfs:range  sdmx:MetadataReportStructure ;
	.
	
sdmx:MetadataProperty a rdfs:Class, owl:Class;
    rdfs:label "Metadata property"@en;
    rdfs:comment """Class of properties used to provide metadata values in 
                    metadata reports (sdmx:MetadataSet)"""@en;
    rdfs:subClassOf sdmx:ComponentProperty;
    owl:disjointWith sdmx:MeasureProperty;
    owl:disjointWith sdmx:AttributeProperty;
    owl:disjointWith sdmx:DimensionProperty;
    .

```

Note that the is some disagreement here between the SDMX Information Model and SDMX-ML.
According to the information model there is a 1:1 correspondence between the MetadataStructure definition and the ReportStructure which would mean that in SDMX-RDF we could merge those two structures. However, SDMX-ML usages allows multiple nested reports we cater for that here.

### Structure and Structure Usage ###

While not necessary to make this work, the cleanest way to plumb this in seems to be to follow the SDMX architecture in having an abstraction for the notion of a Structure and Structure Usage, make both DSDs and MSDs into specializations of this structure and change the domain/range of `sdmx:structure` to match.

```
# --- Structure pattern  --------------------------

sdmx:Structure a rdfs:Class, owl:Class;
    rdfs:label "Structure"@en;
    rdfs:comment "Abstract class for defining the structure (ComponentProperties) that can be used in some data or metadata set"@en;
    .

sdmx:DataStructureDefinition rdfs:subClassOf sdmx:Structure .
sdmx:MetadataStructureDefinition rdfs:subClassOf sdmx:Structure .

sdmx:Structured a rdfs:Class, owl:Class;
    rdfs:label "Structured"@en;
    rdfs:comment """Abstract class for things whose Structure can be defined by an sdmx:Structure declaration, alternative name: StructureUsage"""@en;
    .

sdmx:DataSet rdfs:subClassOf sdmx:Structured .
sdmx:DataFlow rdfs:subClassOf sdmx:Structured .

sdmx:structure a rdf:Property, owl:ObjectProperty;
    rdfs:label "structure"@en;
    rdfs:domain sdmx:Structured;
    rdfs:range sdmx:Structure;
    .
    
sdmx:component a rdf:Property, owl:ObjectProperty;
    rdfs:label "component"@en;
    rdfs:domain sdmx:Structure;
    rdfs:range sdmx:ComponentProperty;
    .

sdmx:componentOrder a rdf:Property, owl:ObjectProperty;
    rdfs:label "component order"@en;
    rdfs:domain sdmx:Structure;
    rdfs:range rdf:List;
    rdfs:comment """Optional specification of the order in which dimensions 
            (and optionally other components) should be used. Value is 
            an rdf:List of sdmx:ComponentProperties or sdmx:ComponentSpecifications"""@en;
    .
```

### Nesting ###

One feature of MetadataSets is that they support grouping of properties. For example, a CONTACT metadata field might group together NAME, ADDRESS and PHONE metadata fields. In SDMX this is used purely for presentational purposes.

To support this in SDMX-RDF we need to at least represent the hierarchical structure of metadata fields as part of the MSD. The proposal is to do this as part of the component order machinery. So, as presaged in the modified comment on `sdmx:componentOrder`, the component order list can contain structured entries which indicate nesting. Conveniently this gives us a hook for attaching additional annotations to `sdmx:ComponentProperty` occurrences within the `Structure`. In particular, we also need to be able to indicate whether a metadata field is required or optional.

```
# --- Extended structure entries --------------------------

sdmx:ComponentSpecification a rdfs:Class, owl:Class ;
    rdfs:label "Component specification"@en;
    rdfs:comment """Used to define properties of a component (attribute, dimension etc)
           which are specific to its usage in a DSD or MSD. This is used by including 
           instances of sdmx:COmponentSpecification in place of the 
           sdmx:ComponentProperty in an sdmx:componentOrder list"""@en;
    .

sdmx:specifiedComponent a rdf:Property, owl:ObjectProperty;
    rdfs:label "specified component"@en;
    rdfs:comment """Links a component specification to the component property
                    that is being qualified"""@en;
    rdfs:domain sdmx:ComponentSpecification;
    rdfs:range  sdmx:ComponentProperty;
    .

sdmx:commponentRequired a rdf:Property, owl:DatatypeProperty;
    rdfs:label "component required"@en;
    rdfs:comment """Indicates whether a component property is required (true) or 
                    optional (false) in the context of a DSD or MSD"""@en;
    rdfs:domain sdmx:ComponentSpecifition;
    rdfs:range  xsd:boolean;
    .
```

Note that by applying the Structure pattern and extending it this way this provides a solution to the core of [Issue 30](https://code.google.com/p/publishing-statistical-data/issues/detail?id=30).

Note that, as currently, we would still have `sdmx:component` links for each metadata property in the MSD, including nested properties. This gives a flat easy to query structure as a parallel to the complex lists of `sdmx:componentOrder`.

Now we have this generic machinery for annotating component properties then to represent nesting we just need:

```
sdmx:childComponents a rdf:Property, owl:ObjectProperty;
    rdfs:label "child components"@en;
    rdfs:comment """Used primarily in sdmx:MetadataStructureDefinitions to indicate a
            grouping of the metadata properties used for presentational purposes.
            The sub components should still be listed as sdmx:components of the 
            structure but in the sdmx:componentOrder the group is represented by a 
            single sdmx:ComponentSpecification which in turn lists the child 
            components by means of this property"""@en;
    rdfs:domain sdmx:ComponentSpecification;
    rdfs:range  rdf:List;
    .
```

Which completes the representation of Metadata Structure Definitions.

### MetadataSet ###

Then an actual metadata record would be represented using:

```
sdmx:MetadataSet a rdfs:Class, owl:Class;
     rdfs:label "Metadata set"@en;
     rdfs:comment """A report containing metadata via attached MetadataProperties"""@en;
     .

sdmx:metadata a rdf:Property, owl:ObjectProperty;
    rdfs:label "metadata"@en;
    rdfs:comment """Links an entity (from Observation up to DataFlow) to a corresponding metadata record."""@en;
    rdfs:range sdmx:MetadataSet;
    .

sdmx:effectiveDate a rdf:Property, owl:DatatypeProperty;
    rdfs:label "effective date"@en;
    rdfs:comment """The date on which all the effectiveDate metadata in the metadata set is effective."""@en;
    rdfs:subPropertyOf dcterms:date;
    rdfs:range xsd:dateTime;
    .

sdmx:MetadataReport a rdfs:Class, owl:Class;
     rdfs:label "Metadata report"@en;
     rdfs:comment """A metadata report (within an overall Metadata set) containing metadata via attached MetadataProperties"""@en;
     .

sdmx:metadataReport a rdf:Property, owl:ObjectProperty;
    rdfs:label "metadata report"@en;
    rdfs:comment """Links a Metadata set to one or more component reports"""@en;
    rdfs:domain sdmx:MetadataSet;
    rdfs:domain sdmx:MetadataReport;
    .
    
sdmx:reportStructure a rdf:Property, owl:ObjectProperty;
    rdfs:label "report structure"@en;
    rdfs:comment """Links an individual metadata report to the report structure it corresponds to"""@en;
    rdfs:domain sdmx:MetadataReport ;
    rdfs:range  sdmx:MetadataReportStructure ;
    .
```

So an instance of `sdmx:MetadataSet` would link to the corresponding MSD via `sdmx:structure` and to metadata values via instances of the specified `sdmx:MetadataProperty` properties.

In the case of nested metadata properties the parent property instance would point to a bNode which in turn would have property values for each of the child properties.

_Alternatively, the MetadataSet could be entirely flat and the nesting structure in the corresponding MSD consulted when formatting reports. That seems more cumbersome to work with._

### Flows ###

Finally we need the notion of a MetadataFlow analogous to a DataFlow. Again it seems easiest to implement a generic Flow pattern, which the metadata and data cases then instantiate:

```

sdmx:MetadataFlow a rdfs:Class, owl:Class;
    rdfs:label "Metadata flow"@en;
    rdfs:comment """A flow of Metadata sets"""@en;
    rdfs:subClassOf sdmx:Structured, sdmx:Flow;
    .

sdmx:metadataFlow a rdf:Property, owl:ObjectProperty;
    rdfs:label "metadata flow"@en;
    rdfs:comment """gives the metadata flow associated with a metadata set"""@en;
    rdfs:domain sdmx:MetadataSet;
    rdfs:range  sdmx:MetadataFlow;
    .

# --- Flow pattern --------------------------

sdmx:Flow a rdfs:Class, owl:Class;
    rdfs:label "Flow"@en;
    rdfs:comment """Abstract information flow, can be either data or metadata"""@en;
    rdfs:subClassOf sdmx:Structured;
    .

sdmx:DataFlow     rdfs:subClassOf  sdmx:Flow .
sdmx:MetadataFlow rdfs:subClassOf  sdmx:Flow .

sdmx:controllingAgreement a rdf:Property, owl:ObjectProperty;
    rdfs:label "controlling agreement"@en;
    rdfs:domain sdmx:Flow;
    rdfs:range sdmx:ProvisionAgreement;
    .
```

### Metadata properties ###

Many of the COG (Concept Oriented Guidelines) concepts are suited to use in specifying metadata. I've generated a set of `sdmx:MetadataProperty` instances, analogous to the existing dimension, attribute and measure instances in the vocab area.

For some common metadata components there are existing RDF vocabularies that consumers would expect to see used. My default proposal is that publishers use COG, or properties corresponding to their own existing schemes, as the default. Where they can identify equivalences they be declared via `owl:equivalentProperty` or `rdfs:subPropertyOf` assertions and corresponding inferred assertions should also be made.

The draft sdmx-metadata.ttl file in vocabs contains a set of example such mappings (rdfs:comment plus the contact details). However, those should be reviewed since specifying the format for contact details as using vcard is an extra burden on format converters.

Alternatively we could generate a short list of common RDF properties that could be linked in this way.

## Omissions ##

### Attachment level ###

The current proposal does not include attachment levels. In principle a key or partial key defining the legal attachment restrictions could be added to `sdmx:MetadataStructureDefinition`. However, such information seems to be more useful as a guide for publication. For this initial pass at SDMX-RDF we are aiming at support for consumption and free use of `sdmx:metadata` at any level may be a sufficient starting point.

### AttributeProperty annotations ###

SDMX also supports attachment of attribute properties to metadata properties within an MSD. This could easily be supported via the `sdmx:ComponentSpecification` machinery but we would need additional guidance and examples to provide a concrete proposal.

## Worked example ##

### Example source ###

We start with the example metadata samples provided in SDMX 2.0 section 3b "SDMX-ML Schemas and Samples".

The MSD sample from `ContactMDStructureSample.xml` is:

```
<structure:MetadataStructureDefinition id="SDDS_CONTACT_INFO" agencyID="IMF">
  <structure:Name>SDDS Contact Info</structure:Name>
  <structure:TargetIdentifiers> ...  </structure:TargetIdentifiers>
  <structure:ReportStructure id="PRIMARY_CONTACT_REPORT_COMPONENT" target="COMPONENT">
    <structure:Name>Report for Primary Contact Information</structure:Name>
    <structure:MetadataAttribute conceptRef="PRIMARY_CONTACT" ... usageStatus="Conditional">
      <structure:MetadataAttribute conceptRef="NAME" ... usageStatus="Conditional">
        <structure:TextFormat/>
      </structure:MetadataAttribute>
      <structure:MetadataAttribute conceptRef="ADDRESS" ... usageStatus="Conditional">
        <structure:TextFormat/>
      </structure:MetadataAttribute>
      <structure:MetadataAttribute conceptRef="TELEPHONE" ... usageStatus="Conditional">
        <structure:TextFormat/>
      </structure:MetadataAttribute>
     <structure:MetadataAttribute conceptRef="E-MAIL" ... usageStatus="Conditional">
       <structure:TextFormat/>
     </structure:MetadataAttribute>
   </structure:MetadataAttribute>
  </structure:ReportStructure>
  ... other ReportStructures ...
</structure:MetadataStructureDefinition>
```

Each of the concepts used for the MetadataAttributes is taken from an IMF scheme included in the sample file.
However, all of the leaf concepts are also present in the COG, we will reuse those and so the only concept scheme information we require in the translation is:

```
imf:sdds-contacts-concept-scheme a skos:ConceptScheme, sdmx:CodeList;
    skos:prefLabel "Contact information"@en;
    rdfs:label "Contact information"@en;
    skos:notation "CONTACT" ;
    skos:hasTopConcept imf:sdds-contacts-concept-primary-contact .

imf:sdds-contacts-concept-primary-contact a skos:Concept, sdmx:Concept;
    skos:topConceptOf imf:sdds-contacts-concept-scheme;
    skos:inScheme     imf:sdds-contacts-concept-scheme;
    skos:notation "PRIMARY_CONTACT" .
    skos:prefLabel "Primary contact"@en .

imf:sdds-contacts-primary-contact a sdmx:MetadataComponent;
    sdmx:concept imf:sdds-contacts-concept-primary-contact;
    rdfs:label  "Primary contact"@en .
```

Then the structure defintion would translate as:

```
imf:sdds-contact-info-msd a sdmx:MetadataStructureDefinition;
    rdfs:label "SDDS Contact Info";
    sdmx:metadataReportStructure  imf:primary-contact-report-structure .

imf:primary-contact-report-structure a sdmx:MetadataReportStructure;
    rdfs:label "Report for Primary Contact Information";
    sdmx:component imf:sdds-contacts-concept-primary-contact,
                   sdmx-metadata:contactName,
                   sdmx-metadata:contactMail,
                   sdmx-metadata:contactPhone,
                   sdmx-metadata:contactEmail;
    sdmx:componentOrder (
        [a sdmx:ComponentSpecification;
         sdmx:specifiedComponent imf:sdds-contacts-primary-contact;
         sdmx:childComponents (
             [a sdmx:ComponentSpecification; sdmx:specifiedComponent sdmx-metadata:contactName;  sdmx:componentRequired false];
             [a sdmx:ComponentSpecification; sdmx:specifiedComponent sdmx-metadata:contactMail;  sdmx:componentRequired false];
             [a sdmx:ComponentSpecification; sdmx:specifiedComponent sdmx-metadata:contactPhone; sdmx:componentRequired false];
             [a sdmx:ComponentSpecification; sdmx:specifiedComponent sdmx-metadata:contactEmail; sdmx:componentRequired false];
         )
        ]
    ) .
```


The corresponding MetadataSet from `MetadataReportSample.xml` is:

```
<imfc:MetadataSet>
  <metadatareport:MetadataStructureRef>SDDS_CONTACT_INFO</metadatareport:MetadataStructureRef>
  <metadatareport:MetadataStructureAgencyRef>IMF</metadatareport:MetadataStructureAgencyRef>
  <imfc:PRIMARY_CONTACT_INFO>
    <imfc:PRIMARY_CONTACT_INFOTarget>
      <imfc:METADATAFLOW>SDDS</imfc:METADATAFLOW>
      <imfc:CATEGORY>EXT_DEBT</imfc:CATEGORY>
      <imfc:DATA_PROVIDER>MX</imfc:DATA_PROVIDER>
    </imfc:PRIMARY_CONTACT_INFOTarget>
    <imfc:NAME>Maria Rodrigues</imfc:NAME>
    <imfc:ADDRESS>5335 Zocalo, Mexico DF, Mexico 46578</imfc:ADDRESS>
    <imfc:TELEPHONE>675-908-9999</imfc:TELEPHONE>
    <imfc:E-MAIL>mrodriguez@bmex.org</imfc:E-MAIL>			
  </imfc:PRIMARY_CONTACT_INFO>
  ... other reports ...
</imfc:MetadataSet>
```

which would in turn translate as:

```
imf:sdds-contact-info-metadata-set a sdmx:MetadataSet;
    sdmx:metadataReport imf:sdds-contact-info-metadata-primary-contact-info .

imf:sdds-contact-info-metadata-primary-contact-info a sdmx:MetadataReport;
    sdmx:reportStructure imf:primary-contact-report-structure;
    imf:sdds-contacts-primary-contact [
        sdmx-metadata:contactName "Maria Rodrigues";
        sdmx-metadata:contactMail "5335 Zocalo, Mexico DF, Mexico 46578";
        sdmx-metadata:contactPhone "675-908-9999";
        sdmx-metadata:contactEmail <mailto:mrodriguez@bmex.org>;
    ] .
```

Here we are not assuming vCard compatibility for the sdmx-metadata properties which would require information not included in the sample data.

The `imf:sdds-contact-info-metadata-set` would then be attached to the resource corresponding to the Metadataflow designated in the key, using the `sdmx:metadata` property.