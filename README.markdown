ASQC provides a simple command-line SPARQL query client.

The intent is that this client can be used in Unix-style pipeline operations to perform sequences of query operations that pass information as RDF (from CONSTRUCT queries) or variable bindings (from SELECT queries).

# Installation

Assumes Python 2.7 installed; not yet tested with other versions.

Installation is from Python Package Index (PyPI).

## MacOS / Linux

### Temporary installation

This option assumes that the virtualenv package (http://pypi.python.org/pypi/virtualenv) has been installed.

Select working directory, then:

    virtualenv testenv
    source testenv/bin/activate
    pip install asqc

When finished, from the same directory:

    deactivate
    rm -rf testenv

### System-wide installation (needs root privileges)

    sudo pip install asqc

If older versions of rdflib and/or other utilities are installed, it may be necessary to force an upgrade, thus:

    sudo pip install --upgrade asqc

# Documentation

Right now, this is pretty much it.  For a usage summary:

    asq --help

See also the examples described below.

Currently, RDF data is supported as RDF/XML only, and SPARQL SELECT query results as JSON.  Support for other formats is on the TODO list.

## Usage

This information is displayed by "asq --help":

```
Usage:
  asq [options] [query]
  asq --help      for an options summary
  asq --examples  to display the path containing example queries

A sparql query client, designed to be used as a filter in a command pieline.
Pipelined data can be RDF or query variable binding sets, depending on the
options used.

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  --examples            display path of examples directory and exit
  -q QUERY, --query=QUERY
                        URI or filename of resource containing query to
                        execute. If not present, query must be supplied as
                        command line argument.
  -p PREFIX, --prefix=PREFIX
                        URI or filename of resource containing query prefixes
                        (default ~/.asqc-prefixes)
  -b BINDINGS, --bindings=BINDINGS
                        URI or filename of resource containing incoming query
                        variable bindings (default none). Specify '-' to use
                        stdin. This option works for SELECT queries only when
                        accessing a SPARQL endpoint.
  -r RDF_DATA, --rdf-input=RDF_DATA
                        URI or filename of RDF resource to query (default
                        stdin or none). May be repeated to merge multiple
                        input resources. Specify '-' to use stdin.
  -e ENDPOINT, --endpoint=ENDPOINT
                        URI of SPARQL endpoint to query
  -o OUTPUT, --output=OUTPUT
                        URI or filename of RDF resource for output (default
                        stdout).Specify '-'to use stdout.
  -t QUERY_TYPE, --type=QUERY_TYPE
                        Type of query output: SELECT (variable bindings,
                        CONSTRUCT (RDF) or ASK (status)
  -v, --verbose         display verbose output
```


# Example queries

The directory "examples" contains some sample files containing queries and prefix declarations that can be used with the following commands.

To obtain the full path name of the examples directory, enter:

    asq --examples

Commands below for running the examples assume this is the current working directory.

## Query DBpedia endpoint

This example comes from the DBpedia front page.  It returns a list of musicians born in Berlin, by sending a SPARQL query to the DBpedia SPARQL emndpoint.

    asq -e http://dbpedia.org/sparql -p dbpedia.prefixes -q dbpedia-musicians.sparql 

## Query SKOS ontology

This example retrieves the SKOS ontology RDF file and runs the SPARQL query locally.  It returns a list of classes defined by the ontology.

    asq -r http://www.w3.org/2009/08/skos-reference/skos.rdf -p skos.prefixes \
      "SELECT DISTINCT ?c WHERE { ?c rdf:type owl:Class }"

A similar query using CONSTRUCT returns the information as an RDF graph:

    asq -r http://www.w3.org/2009/08/skos-reference/skos.rdf -p skos.prefixes \
      "CONSTRUCT { ?c rdf:type owl:Class } WHERE { ?c rdf:type owl:Class }"

## Composition of queries to different data sources

This example shows how ASQ can be used to fetch results from different sources and combine the results.  SELECT query results from one query can be used to constrain the results returned by a second query.

This example uses DBpedia and BBC Backstage SPARQL endpoints to create a list of actors from Japan who appear in BBC television programmes:

    asq -e http://dbpedia.org/sparql -p dbpedia.prefixes \
      -q dbpedia-people-from-japan.sparql \
      >dbpedia-people-from-japan.json
    asq -e http://api.talis.com/stores/bbc-backstage/services/sparql -p dbpedia.prefixes \
      -b dbpedia-people-from-japan.json \
      -q bbc-people-starring-in-television-shows.sparql

or, equivalently, piping bindings from one asq command straight to the next:

    asq -e http://dbpedia.org/sparql -p dbpedia.prefixes \
      -q dbpedia-people-from-japan.sparql | \
    asq -e http://api.talis.com/stores/bbc-backstage/services/sparql -p dbpedia.prefixes \
      -b - \
      -q bbc-people-starring-in-television-shows.sparql

Notes:
* The query to the BBC backstage endpoint can take a little time to complete (about 30 seconds)
* These queries work in part because BBC backstage makes extensive use of the DBpedia ontologies
* It is possible that this particular result could have ben obtained from BBC backstage alone, as it replicates information from DBpedia, but the example has been constructed to use information from the different endpoints.
* Joining queries in this way when sending queries to different endpoints is *not* scalable in the current implementation of ASQ: all available results are retrieved from both services, then joined in the ASQ client.  (I am thinking about possible ways to use the results from one query to limit what comes from the next.  When querying RDF resources, results from one query are used directly to constrain the results of the next query.)


