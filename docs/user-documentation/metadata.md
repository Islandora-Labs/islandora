# Metadata in Islandora 8

> TL;DR: In Islandora 8, metadata is stored in Drupal, in _fields_ attached to _entities_ (nodes or media). This allows us to interact with metadata (add, edit, remove, display, index in a search engine...) almost entirely using standard Drupal processes. If exporting this metadata to Fedora and/or a triplestore, the values are serialized to RDF using mappings that can be set for each bundle.

!!! note "Drupal 8 Terminology"
    In Drupal 8, Fields can be attached to _bundles_ (sometimes called _entity sub-types_ -- e.g. Content types, Media types, Vocabularies) or _entities_ (e.g. Users). For more on Fields, see ["2.3 Content Entities and Fields"](https://www.drupal.org/docs/user_guide/en/planning-data-types.html) and ["6.3 Adding Basic Fields to a Content Type"](https://www.drupal.org/docs/user_guide/en/structure-fields.html) in the Official Drupal Guide.

<!-- Next revision: check status of changing 'bundles' to 'entity sub-types' (https://www.drupal.org/project/drupal/issues/1380720). -->

As described in the [resource nodes section](resource-nodes.md), Islandora 8 digital objects are comprised of _Drupal nodes_ for descriptive metadata, _Drupal media_ for technical metadata, and _Drupal files_ for the binary objects. This section describes how Islandora 8 uses and extends Drupal fields to manage descriptive metadata.

## Content Types

In Drupal, _Nodes_ come in different sub-types called _Content Types_ (e.g. Article, Basic page, Repository item). Content types contain fields, and configurations for how those fields can be edited or displayed. Each content type is essentially a _metadata profile_ that can be used for a piece of web content, or to describe a digital resource. For each field in a content type, an administrator can configure how data is entered, how it can be displayed, how many values can be stored, and how long the value can be. Some configurations, such as data entry and display, can be changed at any time. Others, such as how long a value can be or what options are available in a select list, cannot be changed once content has been created without first deleting all content of that type. However, fields can be added to existing content types with no consequence.


For example, the 'islandora_defaults' module provides a _Repository Item_ content type that defines many fields including "Alternative Title" and "Date Issued". Under the management menu for Repository Item you can see a list of the fields it includes ("Manage fields") as well as tabs for changing the input forms ("Manage form display") and display modes ("Manage display"). See the "[Create / Update a Content Type](content_types.md)" section for more details on creating and configuring fields.

![Screenshot of the "Manage fields" page for the "Repository Item" content type from islandora_defaults.](../assets/metadata_content_type_screenshot.png)

!!! tip "Titles aren't fields."
    Note that the "Title" field does not appear in this list. It is a built-in part of every content type. You can edit the label of this "field" if you want it called something other than "Title" under the "Edit" tab for that content type. This built-in title "field" is limited to 255 characters; if your content has longer titles it is encouraged to create a separate long_title field to store the full title and reserve the default title field for a display title. There is a contributed module called [Node Title Length](https://www.drupal.org/project/title_length), which allows an administrator to configure the length of the title field in the core node table. However, this only works on nodes (not media or other entities) and involves meddling in a core Drupal database schema, which makes some people uneasy.

!!! tip "7.x Migration Note: What about my XML?"
    In 7.x, metadata were usually stored using an XML schema such as MODS or DC, as datastreams attached to an object. In Islandora 8, metadata is stored as fields.
    This means we are breaking out individual elements from a hierarchical structure to being individual independent values. Where some hierarchy or field grouping
    is necessary, this can be done in Drupal using [Paragraphs](https://www.drupal.org/project/paragraphs), a widely-used Drupal contrib module. At the moment
    (Nov 2019) we are working on the technical challenge of mapping data from paragraphs into RDF in Fedora.
    The Metadata Interest Group has developed a default mapping ([spreadsheet](https://docs.google.com/spreadsheets/d/18u2qFJ014IIxlVpM3JXfDEFccwBZcoFsjbBGpvL0jJI/edit#gid=0), [guidance document](https://docs.google.com/document/d/15qSO9YcALtYSqd6CUuGx0t8FwUJ5pPwVPz0PA5rU898/edit?ts=5c5852f3#)) which provides a basic, yet customizable, transform between MODS metadata and Drupal fields in Islandora Defaults. It is suggested that individual institutions customize the mapping to meet their unique needs.

    That said, if keeping the "legacy" XML metadata from 7.x is important to you, it can be attached to an Islandora 8 resource node as a Media entity.
    However, there is no mechanism in Islandora 8 for editing XML in a user-friendly way.

When you create a node (i.e. an instantiation of a content type, such as by using Drupal's "Add Content" workflow), the fields defined by its content type become available. Once a node is created, its content type cannot be changed. To "switch" a node to a different content type, a repository manager would need to create a new node of the target content type, map the field values (programmatically or by copy-paste), and update any Media or children that refer to the old node to refer to the new one.

Not all content types in your Drupal site need be Islandora "resource nodes". A "resource node" content type will likely have behaviours (such as syncing to Fedora or causing derivatives to be generated) associated with it. This configuration, and the communication to the user of which content types are and are not considered to be Islandora resource nodes is left to the discretion of the site manager. In Islandora, a "resource node" is usually considered a descriptive record for "a thing", and is conceptually similar to an "Islandora Object" in 7.x, i.e. a "Fedora Object" in Fedora 3.x and below.

## Vocabularies

See also: [MIG Presentation on Taxonomies](https://docs.google.com/presentation/d/1LfpU6H4qxXtnYQPFntwMNtsgtU30yzp2MxwKKAllUOc/edit?usp=sharing) by Kristina Spurgin, 2021-07-19

In Drupal, _Taxonomy Vocabularies_ (or simply _Vocabularies_) are also entity sub-types that define a set of fields and their configurations. Whereas instances of content types are called _nodes_, items in a vocabulary are called _taxonomy terms_ (or simply _terms_). Traditionally, taxonomy terms are used to classify content in Drupal. For instance, the Article content type includes a field `field_tags` that can refer to terms in the Tags vocabulary.

There are two ways that users can interact with taxonomies: they can be "closed," e.g. a fixed list to pick from in a dropdown, or "open," e.g. `field_tags` where users can enter new terms, which are created on the fly. This is not set on the _vocabulary_ itself, but in the configuration of the field (typically on a node). Terms within vocabularies have an ordering, and can have hierarchical structure, but do not need to.

Islandora (through the Islandora Core Feature) creates the 'Islandora Models' vocabulary which includes the terms 'Audio', 'Binary', 'Collection', 'Digital Document', 'Image', 'Page', 'Paged Content', 'Publication Issue', and 'Video'. Islandora Defaults provides contexts that cause certain actions (e.g. derivatives to happen, or blocks to appear) based on which term is used.

<!-- Is it possible to add your own terms to this vocabulary? Is it recommended? -->

<!-- field_model is a "special field" in terms of the RDF mapping, because the drupal URI gets replaced by the 'external URI' but I'm not sure if this is where to mention this.-->

The Controlled Access Terms module provides additional vocabularies:
- Corporate Body
- Country
- Family
- Form
- Genre
- Geographic Location
- Language
- Person
- Resource Types
- Subject

Each of these vocabularies has its own set of fields allowing repositories to further describe them. The Repository Item content type has fields that can reference terms in these vocabularies. See 'Entity Reference fields' in the 'Field Types' section below.

The vocabularies provided by default are a starting point, and a repository administrator can create whatever vocabularies are desired.

## Field Types

Fields are where Drupal entities store their data. There are different _types_ of fields including boolean, datetime, entity reference, integer, string, text, and text_with_summary. These field types also have _widgets_ (controlling how data is entered) and _formatters_ (controlling how data is displayed). The [Drupal 8 documentation on FieldTypes, FieldWidgets, and FieldFormatters](https://www.drupal.org/docs/8/api/entity-api/fieldtypes-fieldwidgets-and-fieldformatters) includes a list of the core field types. Modules can provide their own field types, formatters, and widgets. The Controlled Access Terms module provides two additional types for use with Islandora: ETDF, and Typed Relation. These are described below.

_Entity Reference_ fields are a special type of field built into Drupal that creates a relationship between two entities. The field's configuration options include which kind of entities can be referenced. The 'Repository Item' content type, provided by islandora_defaults, includes several entity reference fields that reference vocabularies defined by the islandora and controlled_access_terms modules.

The 'Member Of' field is an entity reference field, defined by Islandora, which is the Islandora way of imposing a hierarchical order on resource nodes. This can be used to show membership in a collection, for pages that are members of a paged item, and for members of a complex object.

### EDTF

The EDTF field type is for recording dates in [Extended Date Time Format](https://www.loc.gov/standards/datetime/edtf.html), which is a format based off of the hyphenated form of ISO 8601 (e.g. 1991-02-03 or 1991-02-03T10:00:00), but also allows expressions of different granularity and uncertainty. The Default EDTF widget has a validator that only allows strings that conform to the EDTF standard. The Default EDTF formatter allows these date string to be displayed in a variety of human readable ways, including big- or little-endian, and presenting months as numbers, abbreviations, or spelling month names out in full.

!!! tip "Nerd note:"
    Big-endian = year, month, day. Little-endian = day, month, year. Middle-endian = month, day, year.
	
When configuring the EDTF widget for a field in a content type, you can choose to allow date intervals, but doing this prevents the widget from accepting values that include times. (The EDTF standard states that date intervals cannot contain times, but the field should be able to accept either a valid EDTF range or a valid EDTF datetime, so this is a bug.)

Example of valid inputs in a multi-valued EDTF Date field:
![Screenshot of valid dates ('2019', '2019-11', '2019-22', and '2019-02-02T02:22:22Z') in an EDTF form widget.](../assets/metadata_edtf_valid.png)

Example of the same EDTF dates displayed using little-endian format:
![Screenshot of dates displayed as '2019', 'November 2019', 'Summer 2019', and '2 February 2019 02:22:22Z'.](../assets/metadata_edtf_display.png)

Example of a valid EDTF value ('1943-05') and an invalid value ('1943 May') with the corresponding error message:
![Screenshot of both a valid ("1943-05") and an invalid ("1943 May") EDTF entry. Displays the error message "Could not parse the date 'May 1943' Years must be at least 4 characters long."](../assets/metadata_edtf_invalid.png)

The configuration for the Default EDTF Widget (including options for extra strict date validation, and allowing date intervals and date sets). This configuration can be set per field by clicking the gear icon next to any Default EDTF Widget field at **Administration** >> **Structure** >> **Content types** >> **Repository Item** >> **Manage form display** (admin/structure/types/manage/islandora\_object/form-display)
![Screenshot of the configuration screen for the EDTF Widget](../assets/metadata_edtf_widget_settings.png)

Example of how the EDTF formatter settings can change the display of an EDTF value:
![Combined screenshots displaying the EDTF default formatter settings, default on top and modified settings below, with an example formatted EDTF value displayed for each.](../assets/metadata_edtf_formatters.png)

The EDTF formatter settings can be changed per field by clicking the gear icon next to any Default EDTF formatter field at **Administration** >> **Structure** >> **Content types** >> **Repository Item** >> **Manage display** (admin/structure/types/manage/islandora\_object/display).

By default, EDTF date values are indexed in Solr as string values. The entered value (not the displayed value) is indexed.

!!! note "Solr note:"
    The Solr string data type requires the full field value to match the query in order to count as a match. This means that searching for `2014` will not retrieve a record where the recorded date value is `2014-11-02`.

EDTF date fields may be configured as sort fields in your search results Views. Currently by default, this results in a simple ordering by the literal EDTF date string. 

An EDTF date field with multiple or unlimited number of allowed values may be set as a sort field. In this case, the first occurrence of the field value is used as the sorting value. 

### Typed Relation

A Typed Relation field is an extension of Drupal's Entity Reference field, which allows the user to qualify the relation. It was created for describing a resource's contributors (modelled as taxonomy terms or some other Drupal entities) as well as their roles in this resource node (such as 'author', 'illustrator', or 'architect'). With only Drupal's Entity Reference fields, we would need individual fields for 'author', 'illustrator', 'architect', and any other roles that may need to be made available. Using a Typed Relation field, we can have one field for "Contributors" and let the user pick the role from a dropdown list.

!!! tip "Nerd note:"
    The parts of a field are called properties, so 'entity reference' and 'relation type' are properties of the Typed Relation field type.

Islandora Defaults provides a 'Linked Agent' field as part of the Repository Item content type, and populates the available relationships from the MARC relators list.
![Screenshot of adding a value into a typed relation field](../assets/metadata_typed_relation_field.png)

The list of available relations for this field is configurable at '/admin/structure/types/manage/islandora_object/fields/node.islandora_object.field_linked_agent'. If you re-use this existing field on another content type, you can define different available relations for that instance of the field. Relations are defined in the format `key|value`, and the key is used in the RDF mapping (see below).

![Screenshot of the 'Available Relations' configuration text box for the 'Linked Agent' field.](../assets/metadata_available_relations_config.png)

!!! tip "Typed Relation fields and mapping to RDF:"
    Unlike other fields, which can be assigned RDF predicates in RDF Mapping yaml files, a typed relation field uses a different predicate depending on the chosen "type". These predicates are assigned using the 'keys' in the `key|value` configuration. The key must be formated `namespace:predicate`, e.g. `relators:act`. The namespace ('relators') must be pre-defined in code, using a hook. See the RDF Mapping section of [Create / Update a Content Type](content_types.md) for more details, including a list of available pre-defined namespaces.


# Getting Metadata into Fedora and a Triple-store

The above sections described how Drupal manages and stores metadata, but Islandora 8 provides for pushing that metadata into a Fedora 4+ repository and a triple-store. Islandora does this by using Drupal's serialization capabilities to provide a JSON-LD serialization that can be ingested by Fedora 4+ repository and triple-stores. In response to actions taken as a result of if-then configurations in [Contexts](context.md), Islandora sends notifications to the repository and triple-store that an entity available to ingest.

The JSON-LD module (written for Islandora) takes an entity and its corresponding RDF mapping (a structure defined by the contributed RDF module) to create a JSON-LD serialization. An RDF mapping for a content type, media type, or vocabulary lists its fields and the RDF predicates that should be used for them.

The JSON-LD serialization for an entity is available by appending `?_format=jsonld` to the entity's URL. This particular syntax takes advantage of the REST API module. Below is an example JSONLD document representing the RDF serialization of a Repository item node created in a standard islandora-playbook based vagrant VM:

```
{
  "@graph":[
    {
      "@id":"http://localhost:8000/node/1",
      "@type":[
        "http://pcdm.org/models#Object"
      ],
      "http://purl.org/dc/terms/title":[
        {
          "@value":"New York, New York. A large lobster brought in by the New England fishing boat [Fulton Fish Market]",
          "@language":"en"
        }
      ],
      "http://schema.org/author":[
        {
          "@id":"http://localhost:8000/user/1"
        }
      ],
      "http://schema.org/dateCreated":[
        {
          "@value":"2019-03-14T19:05:24+00:00",
          "@type":"http://www.w3.org/2001/XMLSchema#dateTime"
        }
      ],
      "http://schema.org/dateModified":[
        {
          "@value":"2019-03-14T19:20:51+00:00",
          "@type":"http://www.w3.org/2001/XMLSchema#dateTime"
        }
      ],
      "http://purl.org/dc/terms/date":[
        {
          "@value":"1943-05",
          "@type":"http://www.w3.org/2001/XMLSchema#string"
        },
        {
          "@value":"1943-05",
          "@type":"http://www.w3.org/2001/XMLSchema#gYearMonth"
        }
      ],
      "http://purl.org/dc/terms/extent":[
        {
          "@value":"1 negative",
          "@type":"http://www.w3.org/2001/XMLSchema#string"
        }
      ],
      "http://purl.org/dc/terms/identifier":[
        {
          "@value":"D 630714",
          "@type":"http://www.w3.org/2001/XMLSchema#string"
        }
      ],
      "http://purl.org/dc/terms/type":[
        {
          "@id":"http://localhost:8000/taxonomy/term/11"
        }
      ],
      "http://purl.org/dc/terms/rights":[
        {
          "@value":"No known restrictions. For information, see U.S. Farm Security Administration/Office of War Information Black & White Photographs(http://www.loc.gov/rr/print/res/071_fsab.html)",
          "@type":"http://www.w3.org/2001/XMLSchema#string"
        }
      ],
      "http://purl.org/dc/terms/subject":[
        {
          "@id":"http://localhost:8000/taxonomy/term/26"
        }
      ],
      "http://schema.org/sameAs":[
        {
          "@value":"http://localhost:8000/node/1"
        }
      ]
    },
    {
      "@id":"http://localhost:8000/user/1",
      "@type":[
        "http://schema.org/Person"
      ]
    },
    {
      "@id":"http://localhost:8000/taxonomy/term/11",
      "@type":[
        "http://schema.org/Thing"
      ]
    },
    {
      "@id":"http://localhost:8000/taxonomy/term/26",
      "@type":[
        "http://schema.org/Thing"
      ]
    }
  ]
}
```

Because the Repository item's title field is mapped to 'dc:title' in the RDF mapping, the node's title value appears like this in the JSON-LD output:
```
"http://purl.org/dc/terms/title":[
  {
    "@value":"New York, New York. A large lobster brought in by the New England fishing boat [Fulton Fish Market]",
    "@language":"en"
  }
],
```

Also note that the URI (`@id`) value is 'http://localhost:8000/node/1' (without the `?_format=jsonld`). Old versions of Islandora 8 included the `?_format=jsonld`, and dealing with discrepancies is described at "[Adding back ?_format=jsonld](../technical-documentation/adding_format_jsonld.md)".

# Batch editing metadata in fields

If you are editing multiple resources in order for them to have the same metadata value, the Views Bulk Edit module can help. Here is a video of [creating a view using Views Bulk Operations](https://www.youtube.com/watch?v=ZMp0lPelOZw) to apply a subject term to multiple resources simultaneously. 

For more complex changes, or when the values need to differ for each value, an export-modify-reimport method may be needed. Use a view to export CSV or other structured data (including an identifier such as a node id), modify the values as necessary, then use [migrate csv](../technical-documentation/migrate-csv.md) or [Workbench](../technical-documentation/migration-overview.md) to re-import and update the values. 
