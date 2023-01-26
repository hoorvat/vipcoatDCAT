# My thoughts on how to add records to a data catalog

## ...so far:

If a user wants to add a catalog record the order of operations is as follows:

1. Create a dcat:Resource\
   this is done first because it's not dependent on other objects.
2. Create a dcat:Record\
   this basically indicates what has been added. it also has a connection to the dcat:Resource that was created before
3. Create a dcat:Distribution\
   this is an object that gives access to a file that was uploaded somewhere or is accessible via api
4. Create a dcat:Dataset\
   dataset merges all the necessary information about the data. It combines the resource and data distribution that were created above.
5. Finally, update the dcat:Catalog with the new dcat:Record, dcat:Dataset, dcat:Resource

## Initial VIPCOAT DCAT

```
PREFIX vipcoat: <https://vipcoat-oip.com/dcat/>

vipcoat:catalog 
    a dcat:Catalog;
    foaf:homepage <"https://vipcoat-oip.com/dcat">  # The search engine
```

## dcat:Resource
```
vipcoat:resource/resourceName1
    a dcat:Resource;
    dcterms:title "Title of the resource";
```
## dcat:Record
```
vipcoat:record/recordName1
    a dcat:CatalogRecord;
    dcterms:description "Description of the catalog record";
    dcterms:title "Title of the record";
    dcterms:issued "Time when issued";
    dcterms:modified "Time when last modification";
    foaf:primaryTopic vipcoat:resource/resourceName1;  # This is the connection to the dcat:Resource
```
## dcat:Distribution
```
vipcoat:distribution/distributionName
    a dcat:Distribution;
    dcat:accessURL "https://vipcoat-oip.com/resourcepage";  # The page where the data is rendered
    dcat:downloadUrl "https://vipcoat-bucket.s3.eu-west-2.amazonaws.com/uploads/filename";  # Where the file is stored
    dcat:mediaType "text/csv">; # The list https://mimetype.io/all-types/
    dcat:description "description of the distribution";
    dcat:issued "Time when uploaded";
    dcat:byteSize 
```

## dcat:Dataset
```
vipcoat:dataset/datasetName
    a dcat:Dataset;
    dcat:distribution vipcoat:distribution/distributionName;  # Connection to the distribution
    dcat:resource vipcoat:resource/resourceName;  # Connection to the resource
```

## Update dcat:Catalog
```
vipcoat:catalog
    dcat:record vipcoat:record/recordName1;  # Connection to the record
    dcat:dataset vipcoat:dataset/datasetName;  # Connection to the dataset
    dcat:resource vipcoat:resource/resourceName;  # Connection to the resource

```


# Versioning

The versioning is done at the level of resources (subjects and objects are uris of resources). Each time a resource is initially created, create two of them.
1. A base resource; e.g. https://vipcoat-oip.com/resource/resource-uuid. This one is always pointing to the latest version with a predicate dcat:hasCurrentVersion. It also keeps track of all the available versions with a predicate dcat:hasVersion. I would not give this one all the inputs but would treat it as a version controller.
2. An initial version resource; e.g. https://vipcoat-oip.com/resource/resource-uuid-issueTime. This one is the real resource. When it comes to versioning it should always point to the base resource with a predicate dcat:isVersionOf. It should also point the previous version with a predicate dcat:previousVersion

Each time a new version is made update the base resource, add another version. No need to update the previous version.


```
vipcoat:resource/resourceName
    a dcat:Resource;  # this is a base resource
    dcat:hasVersion
        vipcoat:resource/resourceName-v1  # this is a list of versions that were created
        vipcoat:resource/resourceName-v2
    dcat:hasCurrentVersion vipcoat:resource/resourceName-v2  # this is the latest version


vipcoat:resource/baseResource-v1
    [...]  # for this one we need all the additional information such as title, description, distribution
    a dcat:Resource;  # this is a version resource
    dcat:isVersionOf vipcoat:resource/resourceName;  # this connects it to the base resource


vipcoat:resource/baseResource-v2
    [...]
    a dcat:Resource;  # this is a version resource
    dcat:isVersionOf vipcoat:resource/resourceName;  # this connects it to the base resource
    dcat:previousVersion vipcoat:resource/baseResource-v1;  # this points to the previous version
```

## Using checksum when creating a new version

We can utilize checksum of a resource's distribution to check if a file that is being uploaded a new file. If the checksum of the new file is the same as the one in the graph we should not allow a new version to be created.

To take this to a new level we can always before creating check if a distribution exists with the same checksum. No need to clutter the database with same files.

## Traversing the graph with versions

From every resource we can check what is its dcat:isVersionOf pointing to. The base resource in subject will return nothing. All the version resources will point to the base resource. With SELECT DISTINCT in sparql query this will return a single, base, resource. From the base we can easily fetch the latest version for the query search, or all versions for the resource page where versioning is shown.