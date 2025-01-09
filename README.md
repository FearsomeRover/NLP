# NLP

Notes for the Natural Language Processing Course (BMEVIMIAC22)

## SPARQL queries

### Megszámlálás

    SELECT (COUNT(DISTINCT ?o) AS ?count) WHERE {
        ?s rdf:type ?o.
    }

### Filter

``` SPARQL
SELECT ?s ?p ?o WHERE {
    ?s ?p ?o
    FILTER(CONTAINS(LCASE(STR(?o)), "tilalom"))
}
# vagy:
FILTER(STRSTARTS(STR(?s), "http://frvr.hu/resource/feny"))
```

### 4. Készítsen SPARQL lekérdezést, ami megtalálja az összes alkotót

``` SPARQL
select ?alkoto where {
    ?alkoto <Chttp://www.w3.org/1999/02/F22-rdf-syntax-ns#type> dbo:Artist
}
```

### 5.Készítsen SPARQL lekérdezést, ami visszaadja a "Giovanni" nevű alkotókat (akiknek a nevében szerepel a "Giovanni" string). A kapott táblázat két oszlopa legyen a művész azonosítója és a neve

    select ?alkoto where {
       ?alkoto <Chttp://www.w3.org/1999/02/F22-rdf-syntax-ns#type> dbo:Artist
       FILTER regex(?name, "giovanni", "i")
    }

### 1. Szépművészeti Múzeum és DBpedia adatok kapcsolódása

```
select DISTINCT ?id  WHERE {
    ?id
}
```
    

### 16. századi olaszok költők akik Florenzében hunytak el

    PREFIX dbo: <http://dbpedia.org/ontology/>
    PREFIX dbr: <http://dbpedia.org/resource/>
    SELECT (COUNT(DISTINCT ?s) as ?count) WHERE {
        ?s dbo:deathPlace dbr:Florence.
        ?s ?p <http://dbpedia.org/class/yago/Wikicat16th-centuryItalianPainters>.
    }
---
    PREFIX dbo: <http://dbpedia.org/ontology/>
    PREFIX dbr: <http://dbpedia.org/resource/>
    SELECT (COUNT(DISTINCT ?s) WHERE {
        ?s dbo:deathPlace dbr:Florence.
        ?s <http://purl.org/dc/terms/subject> dbr:Category:16th-century_Italian_painters.
    }

### 16. szazadi, Florenzeben elhunyt olasz koltok alkotasainak nevei (prefixek elhagyva)

    SELECT ?name WHERE {
        ?s dbo:deathPlace dbr:Florence.
        ?s <http://purl.org/dc/terms/subject> dbr:Category:16th-century_Italian_painters.
        ?instance owl:sameAs ?s.
        ?creation ecrm:P11_had_participant ?instance.
        ?creation a ecrm:E65_Creation.
        ?thing ecrm:P12i_was_present_at ?creation.
        ?thing rdfs:label ?name
    }

### Olasz muveszeknek alkotasainak szama a szepmuveszeti muzeumban

    SELECT (COUNT(DISTINCT ?thing) AS ?count) WHERE {
        ?s dbo:nationality dbr:Italy.
        ?instance owl:sameAs ?s.
        ?creation ecrm:P11_had_participant ?instance.
        ?creation a ecrm:E65_Creation.
        ?thing ecrm:P12i_was_present_at ?creation.
    }

### Szepmuveszeti adatok Jan Lievensrol

    SELECT ?subject ?predicate ?object WHERE {
        ?instance owl:sameAs dbr:Jan_Lievens
        {
            ?subject ?predicate ?instance.
        }
        UNION
        {
            ?instance ?predicate ?object.
        }
    }

### Literals

    SELECT ?s {
        ?s <IRI> "LITERAL"
    }

### Binding

    bind("TEST" as ?test)

### Joining

    select ?pokemon ?cry {
        ?pokemon <https://triplydb.com/academy/pokemon/vocab/colour> ?color.
        ?pokemon <https://triplydb.com/academy/pokemon/vocab/cry> ?cry.
    }
    limit 10

### Abbreviations

    base <https://triplydb.com/academy/pokemon/vocab/>
    prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
    select ?color ?cry ?name ?type1Name ?type2Name {
        ?pokemon a ?class.
        ?pokemon <colour> ?color.
        ?pokemon <cry> ?cry.
        ?pokemon rdfs:label ?name.
        ?pokemon <type> ?type1.
        ?pokemon <type> ?type2.
        ?type1 rdfs:label ?type1Name.
        ?type2 rdfs:label ?type2Name.
    }
    limit 10

