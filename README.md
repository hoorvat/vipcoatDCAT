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
    dcterms:title <"Title of the resource">;
```
## dcat:Record
```
vipcoat:record/recordName1
    a dcat:CatalogRecord;
    dcterms:description <"Description of the catalog record">;
    dcterms:title <"Title of the record">;
    dcterms:issued <"Time when issued">;
    dcterms:modified <"Time when last modification">;
    foaf:primaryTopic vipcoat:resource/resourceName1;  # This is the connection to the dcat:Resource
```
## dcat:Distribution
```
vipcoat:distribution/distributionName
    a dcat:Distribution;
    dcat:accessURL <"https://vipcoat-oip.com/resourcepage">;  # The page where the data is rendered
    dcat:downloadUrl <"https://vipcoat-bucket.s3.eu-west-2.amazonaws.com/uploads/filename">;  # Where the file is stored
    dcat:mediaType <"text/csv">; # The list https://mimetype.io/all-types/
    dcat:description <"description of the distribution">;
    dcat:issued <"Time when uploaded">;
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
