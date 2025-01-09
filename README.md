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
```
prefix dbo: <http://dbpedia.org/ontology/>
SELECT ?artist ?dob
WHERE {
  ?artist dbo:birthDate  ?dob.
  filter("1600-01-01"^^xsd:date <= ?dob)
  filter("1651-01-01"^^xsd:date >= ?dob)
}
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

[Lecture Notes](https://docs.craft.do/editor/d/f8eb6938-2344-78ed-f381-eeb1c9b2cc77/d3b618d6-8eb8-4bc2-aee2-661e11133ea9/b/04B63A13-630D-49CE-8E17-FCA109144169/SPARQL?s=Zx3rAdYMuZyHPNU9dkoWrfTdyrPNMadmeNg6wstAKaCQ)

[Colab references](https://colab.research.google.com/drive/1ltYGjWM0GnyV4ajzvNPx-hYtF7jQAQJC?usp=sharing)

# Colab
## Indexelés, kulcsszó keresés, regex

```python
#letöltés után
with open('text.txt') as f:
  data = f.read()
print("Műrészlet:\n", data[7000:7300])

#regex
import re
#Keresem az összes olyan szót, amelyben csak a római számok szerepelnek, majd egy pont. A fejezetcímek pont ezt a formátumot veszik fel.
parts = re.split(r'\b[IVX]+\.', data)
parts[2][0:100]
#regex dokumentáció: https://docs.python.org/3/library/re.html

#indexelés
import re

index = {}
#Létrehozok egy fejezet számlálót, mivel a későbbiekben csak string tömbökkel foglalkozik a program, nem tudja majd melyik fejezetet vizsgálja
part = 1

for d in parts[1:11]:
  #Veszem az adott fejezetet és szavakra bontom
  words = d.split()
  #Minden egyes szót megvizsgálok és indexelem megfelelően!
  for i in words:
    #Amennyiben nincsen még ez a szó benne az indexünkben, akkor létrehozom a hozzá tartozó key-t.
    #Mindegyik szóhoz egy set-et generálok, mivel nem szeretném többször látni ugyanazt a fejezetet egy szóhoz, amennyiben az többször fordul elő egy fejezetben.
    if i not in index:
      index[i] = set()
    #Mindenképp jelzem, hogy az adott szót (i) megtaláltuk a fejezetben (part). Mivel set-et használok, a szóismétlődések nem okoznak problémát.
    index[i].add(part)
  #Ha minden szóval végeztem, akkor lépek a következő fejezetre, és megnövelem a fejezetszámlálót is.
  part += 1

#kulcsszó keresés
def search(pattern):
  print(f"{pattern} szó a következő fejezetekben található meg: {index[pattern]}")
  return index[pattern]

search("fiatal")
search("fiatalember")
search("macska")
```

### NLP feldolgozólánc, tokenizáció, stopszavak, adatkinyerés HTML-ből, spacy

```python
#Adatkinyerés htmlből
import requests
from bs4 import BeautifulSoup
html = requests.get("https://mek.oszk.hu/09000/09000/html/mikes1.htm", {}).text
raw = BeautifulSoup(html, 'html.parser')
korpusz = raw.get_text() #kinyerjük a szöveget a htmlból

import nltk
nltk.download('punkt')

#Tokenizáció mondatonként
from nltk import sent_tokenize
sentences = sent_tokenize(korpusz) #bontsuk a szöveget mondatokra
sentences[10:20]

#Tokenizáció szavanként
from nltk import word_tokenize
words = word_tokenize(korpusz) #bontsuk a korpuszt szavakra
words[50:60]

#Gyakoriságvizsgálat
from nltk.probability import FreqDist
fdist = FreqDist(words) #vizsgáljuk meg az egyes szavak előfordulásának gyakoriságát a szövegben
fdist.most_common(20)

#stopszavak
nltk.download('stopwords')
from nltk.corpus import stopwords
stopwords.words('hungarian')[0:10]

#plusz szavak hozzáadása és szűrés
import string
stop_words = set(stopwords.words('hungarian'))
stop_words.update(string.punctuation) # kiszűrjük az írásjeleket is
older_words = ["ugy", "illyen", "ollyan", "mind", "ha", "valo", "is", "vagyon", "bé"]
stop_words.update(older_words) #A régies szavak hozzáadása
fwords = [w for w in words if not w in stop_words] # Csak akkor adjuk hozzá a szavak listájához az adott szavat, ha nincs benne a stoplist-ben, azaz nem olyan gyakori szó amit nem szeretnénk számításba venni
fdist = FreqDist(fwords) # A szűrt listával vizsgálunk gyakoriságot
fdist.most_common(20)

#Konkordanciavizsgálat
from nltk.text import Text
t = Text(words)
t.concordance("néném") #Megvizsgáljuk, hogy a néném szó mikkel van gyakori kapcsolatban

#Szófelhő ábrázolás
text = ' '.join(fwords)
wordcloud = WordCloud().generate(text)
plt.figure(figsize = (12, 12))
plt.imshow(wordcloud)
plt.axis("off")
plt.show


#Spacy
#https://course.spacy.io/en/
import spacy

# Létrehozunk egy angol nyelvű üres NLP objektumot.
nlp = spacy.blank("en")

# Nem tartalmaz feldolgozóláncot
nlp.pipeline

# Létrehozunk egy egyszerű dokumentumot
doc = nlp("Hello world!")

# Majd kiírjuk a tokeneket
for token in doc:
    print(token.text)

# A tokenek sorszámozottak.
doc = nlp("It costs $5.")
print("Index:   ", [token.i for token in doc])
print("Text:    ", [token.text for token in doc])

# A karakterfüzér elemei alapján típust is meghatározunk
print("is_alpha:", [token.is_alpha for token in doc])
print("is_punct:", [token.is_punct for token in doc])
print("like_num:", [token.like_num for token in doc])

from prettytable import PrettyTable

doc = nlp("Liz was smiling while she ate two pizzas yesterday")

table = PrettyTable(['Text', 'Lemma', 'POS', 'DEP', 'HEAD'])

# Iterate over the tokens
for token in doc:
    table.add_row([token.text, token.lemma_, token.pos_, token.dep_, token.head.text])

#vizuális ábrázolás
from spacy import displacy
displacy.render(doc, style="dep", jupyter=True)

#Ki mit evett? Infókinyerés spacyvel
from spacy.matcher import Matcher

# A mintaillesztő inicializálása (a nyelvi modell szókészletére támaszkodunk)
matcher = Matcher(nlp.vocab)

# Ezek a keresési minták
# Figyeljük meg, hogy nem szavakra keresünk!
matcher.add("PIZZA_PATTERN", [[ {"DEP": "nummod"},{"POS": "NOUN"} ]])
matcher.add("PERSON_PATTERN", [[ {"DEP": "nsubj", "POS": "PROPN"} ]])

# Ugyanazt a mondatot fogjuk feldolgozni.
doc = nlp("Liz was smiling while she ate two pizzas")

# Jön a keresés
matches = matcher(doc)

print("Találatok száma:", len(matches))

# Kiírjuk a találatokat
for match_id, start, end in matches:
    # A span a start és end pozíciók közötti szövegrész
    matched_span = doc[start:end]
    print(matched_span.text)

```

### ChatGPT

```python
#letöltés
!pip install openai >/dev/null

#Ezt a tokent fogom használni (colab bal oldal kulcs user secretsnél megadni!!)
from openai import OpenAI
from google.colab import userdata
openai = OpenAI(api_key = userdata.get('openai-token'))

openai_modell = 'gpt-3.5-turbo'

#Minta api call
def openai_complete(content):
  completion = openai.chat.completions.create(
    model=openai_modell,
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {
            "role": "user",
            "content": content
        }
    ]
  )
  return completion.choices[0].message.content

print(openai_complete("Írj egy rövid Python kódot, amely megszámolja egy szöveg szavait!"))

#Openai tokenizáció
import tiktoken
enc = tiktoken.encoding_for_model(openai_modell)
tokens = enc.encode("waaaaat")
print(tokens)

for tok in tokens:
  print(enc.decode([tok]))
```

## DL query

```
#DL nyelv:
#Azon egyedek halmaza, melyek valamely adat-tulajdonsága a megadott intervallumból vehet fel értéket:
data_property some xsd:decimal[>= 50, <= 100]

#Azon egyedek halmaza, melyek valamely objektum-tulajdonsága a megadott halmazból vehet fel
#értéket (a “class” helyére DL Query kifejezés is írható):
object_property some class

#Azon egyedek halmaza, melyek valamely tulajdonságának értéke a megadott érték:
object_property value instance
data_property value “100”^^xsd:decimal

#Halmazok metszete, uniója, komplementere:
class1 and class2
class1 or class2
not class

#A kifejezések zárójelezhetők.


E18_Physical_Thing and
(P43_has_dimension some (E54_Dimension and (P90_has_value some xsd:decimal[<= 100])))

#17. századi alkotások
E65_Creation
  and (P11_had_participant some E39_Actor)
  and (dbo:creationYear some xsd:integer[ >= 1599 , <= 1700 ])


#Osztály létrehozása (vagy kattintgatva és DL query-t másolva):
Class: :17thCenturyCreation
    EquivalentTo:
        E65_Creation
        and ( ecrm:P11_had_participant some E39_Actor )
        and ( dbo:creationYear some xsd:integer[ >= 1599 , <= 1700 ] )
```