---
title: 'An ETL pipeline to construct IDPcentral Knowledge Graph using Bioschemas JSON-LD data feeds'
title_short: 'An ETL pipeline to construct IDPcentral Knowledge Graph using Bioschemas JSON-LD data feeds'
tags:
  - pipeline
  - workflow
  - knowledge graph
  - bioschemas
  - ETL
authors:
  - name: Ammar Ammar
    orcid: 0000-0002-8021-9162
    affiliation: 1
  - name: Alasdair Gray
    orcid: 0000-0003-1460-8327
    affiliation: 2
  - name: Ivan Mičetić
    orcid: 0000-0003-1691-8425
    affiliation: 3
affiliations:
  - name: Department of Bioinformatics (BiGCaT), Maastricht University, The Netherlands
    index: 1
  - name: Heriot-Watt University, Edinburgh, UK
    index: 2
  - name: BioComputing UP Lab, University of Padua, Italy
    index: 3

date: 11 November 2022
cito-bibliography: paper.bib
event: BioHackathon Europe 2022
biohackathon_name: "BioHackathon Europe"
biohackathon_url:   "https://biohackathon-europe.org/"
biohackathon_location: "Paris, France, 2022"
group: Project 23
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/ammar257ammar/biohackathon2022-bioschemas-consumer
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Ammar Ammar & Alasdair Gray \emph{et al.}
---


<!--

The paper.md, bibtex and figure file can be found in this repo:

  https://github.com/ammar257ammar/biohackathon2022-bioschemas-consumer

To modify, please clone the repo. You can generate PDF of the paper by
pasting above link (or yours) in

  http://biohackrxiv.genenetwork.org/

-->

# Introduction

As part of the one week Biohackathion Europe 2022 in Paris France, a group was formed to work on project 23 titled: Publishing and Consuming Schema.org DataFeeds.
Bioschemas is a lightweight vocabulary aims at making web pages contents machine-readable where software agents can consume that content and understand it in an actionable way. Due to the time needed to process each page, extracting markup by visiting each page of a site is not practical for huge sites. This approach imposes processing requirements on the publisher and the consumer. 
The Schema.org community proposed a method for exchanging markup from various pages as a DataFeed published at a recognized address in February 2022. The feed could consist of a single file containing the entire information or it could be divided into different files based on different aspects of the dataset, such as proteins and molecular entities, as in the case of ChEMBL. This would ease publisher and customer processing requirements and accelerate data collection.
The aim of Project 23 is to explore the implementation of the Schema.org proposal from both a producer and consumer perspective, for a variety of resources implementing different Bioschemas profiles. This report focuses on the consumer part of the project proposal where we explored an ETL pipeline (Extract-Transform-Load) approach and implemented a consumption pipeline that enables data feeds to be ingested into knowledge graphs (KG).


<!--
# Results
-->

## The construction of IDPcentral Knowledge Graph as a use case

The example pipeline that we developed in this work is based on a [previous work](https://github.com/BioComputingUP/IDP-KG) developed during the ELIXIR sponsored BioHackathon-Europe 2020 and reported in BioHackrXiv \cite{Gray_2021}.
In that work, several notebooks were developed to generate the IDPcentral Knowledge Graph based on data harvested from three sources: 
[DisProt](https://disprot.org/), [MobiDB](https://mobidb.org/), and [ProteinEnsemble (PED)](https://proteinensemble.org/).

More specifically, we aimed at reprocuding [one of the notebooks](https://github.com/BioComputingUP/IDP-KG/blob/main/notebooks/ETLProcess.ipynb) that did the ETL processing in order to create the knowledge graph, but this time in the form of a pipeline.

The pipeline is supposed to load scraped JSON-LD from the three aformentioned sources, convert it to RDF, apply SPARQL construct queries to map the source RDF to a unified Bioschemas-based model and store the resulting KG as a ttl file.

## Exploring the LinkedPipes linked data suite

We explored a suite for linked data called [LinkedPipes](https://etl.linkedpipes.com/), specifically, the ETL (Etract-Transform-Load) part of it.

LinkedPipes ETL is an RDF based, lightweight ETL tool. It has a modular design providing a large collection of components to be used in building ETL pipelines.
Everything in LinkedPipes is in RDF. The ETL pipelines, component setups, and messages indicating pipeline progress are included in this. 

We found LinkedPipes to be feature-rich and suitable for our aim. Some of the capabilities of LinkedPipes includes:

1. Loading data from different sources like: 
	- Download files over HTTP (multiple download in parallel from a list of URLs is supported).
	- Download from FTP
	- Load data from a SAPRQL endpoint using a construct query.
		- Supports chunked loading from SPARQL through a series of CONSTRUCT queries (it can be helpful to avoid timeouts) 
		- Supports extracting triples using scrollable cursor technique for OpenLink Virtuoso
1. Convert the downloaded files to RDF (details are specified by user) from multiple formats (CSV, TSV, JSON, XML, HDT) and then merge all the RDFs in a single graph.
1. Manipulate/map the RDF graph using SPARQL update/construct queries.
1. zip/unzip/hash/rename/filter files.
1. Export/store the resulted graph into a SPARQL endpoint, a virtuoso instance, a Jena TBD model, push to FTP or just save as a file locally (supports most known formats like nt, ttl, nq, rdf/xml, JSON-LD, trig and others)
1. Scrape URLs and extract information from the HTML using CSS selectors and create a triples from them on the fly (which then can be mapped to another model).
1. Validates input data using SHACL shapes
1. Generates text files using the {{ mustache }} templates.
1. Create VoID and DCAT dataset/distribution metadata and add it to the graph or save it as a separate file (the user controls that in the pipeline).
1. Other features:
	- Provides geographical projection transformation features.
	- Translates literals using Bing machine translation.
	- Checks data with a SPARQL ask query and stops pipeline execution on success or fail.


## Starting LinkedPipe using Docker

We used the Dockerized version of LinkedPipes which can be run using ```docker-compose``` and the following commands:

```
git clone https://github.com/linkedpipes/etl.git
cd etl
docker-compose up
```

## Runtime Configuration for LinkedPipes components

Some of LinkedPipes components requires a runtime configuration to function. For example, the [HTTP get list](https://etl.linkedpipes.com/components/e-httpgetfiles) component, that download multiple files using HTTP requests, requires the URLs to be downloaded and the target file names to be provided using a runtime configuration representated in RDF.

Below is an example of the structure of this RDF configuration:

```
@prefix httpList: <http://plugins.linkedpipes.com/ontology/e-httpGetFiles#> .

<http://localhost/resource/configuration> a httpList:Configuration ;
    httpList:reference <http://localhost/resource/ref/1> ;

<http://localhost/resource/ref/1> a httpList:Reference ;
    httpList:fileUri "https://github.com/BioComputingUP/IDP-KG/raw/main/scraped-data/2022-10-27_subset/jsonld/disprot/DP00186.jsonld" ;
    httpList:fileName "DP00186.jsonld" ;
```

The ttl snippet above indicates three main pieces of information:

1. Create a httpList:Configuration entity.
1. Create a httpList:Reference entity for each URL needs to be downloaded, each entity having two predicates at least:
	- httpList:fileUri where the object is a string literal of the URL of the file to be downloaded.
	- httpList:fileName where the object is the name of the file to be stored locally after the download.
	
Fortunately, this configuation can be constructured from a list of URLs using a SPARQL CONSTRUCT query.

## The pipeline explained

![An overview of the developed pipeline using  \label{fig}](./figures/the-pipeline.png)


The figure above shows the built pipeline in this work which downloads JSON-LD files scraped from three sources and stored on GitHub, converts them to RDF, map the RDF to a unifed model and store the resulting graph to a ttl file. It will also calculate some statistics on the convereted RDF and store it in a CSV file next to the output ttl file.


## Testing the pipeline

The pipeline can be tested with a local running instance of LinkedPipes or using the online [demo instance](https://demo.etl.linkedpipes.com/).

By using the upload function under the "piplines" tab, users can load the pipeline either as a file or as a URL, and then, it be executed to get the output RDF. All the files needed for the pipeline to execute (the input URL list and the JSON-LD files) are hosted online in GitHub. Therfore, the pipeline is portable and reproducible on any machine.
 

## References
