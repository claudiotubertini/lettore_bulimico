# Il lettore Google (CUBO Unipol 1 aprile 2019)
### Questo testo accompagna ma non sostituisce le slides presentate in sala (disponibili sempre in questo repo) 
Negli utlimi anni abbiamo assistito a una crescita esponenziale della produzione di dati e documenti tra cui pagine Web, notizie giornalistiche, letteratura scientifica, e-mail, documenti aziendali e social media come blog, forum, recensioni di prodotti e tweet. 
Questi dati sono presenti sia all'interno delle reti aziendali, in ERP, CRM, ecc., sia sul web. 
Gli strumenti tipici della business intelligence e della gestione dei database incrementano e gestiscono questa "galassia" di dati. Per definizione la produzione editoriale è un'estrazione di questi dati, la loro analisi e sintesi da parte di un operatore esperto.
Noi ci occuperemo di come questi dati possono a volte essere estratti e riutilizzati. Partiremo da semplici esempi di web scraping fino ad arrivare a individuare l'utilizzo del web semantico.
Per seguire il titolo di questa conferenza potremmo dire che il "lettore google" è un lettore automatico di dati e testi e soprattutto di dati strutturati.
Le tecnologie di cui parleremo sono solo di scraping e query. Non toccheremo tutte le tecnologie XML che consentono il recupero di questi dati e la loro pubblicazione. 
Perché questa strana idea della "galassia dei dati"?
Un esempio: un'azienda ci chiede di pubblicare dati tratti in larga parte da un loro gestionale, ci viene consegnato un file XML ma gli autori insistono che per correggerlo devono averne una versione in word. In questo modo i dati non potranno più essere riutilizzati, escono dalla "galassia" e possono solo finire in una pubblicazione. Conservati in XML possono invece essere aggiornati e costituire materiale per future pubblicazioni.


Una delle attività di Google è raccogliere i link dei diversi siti. Ecco come si potrebbe fare: [NOTA: il lavoro principale di Google non è questo ma è rispondere alle domande degli utenti, risposte di cui questi link costituiscono solo una parte, il resto è profilazione delle caratteristiche dell'utente]

Salvare in un file link_crawler.py le seguenti righe
```
from urllib.request import urlopen, urljoin
import re

def download_page(url):
    return urlopen(url).read().decode('utf-8')

def extract_links(page):
    link_regex = re.compile('<a[^>]+href=["\'](.*?)["\']', re.IGNORECASE)
    return link_regex.findall(page)

if __name__ == '__main__':
    target_url = 'https://clueb.it/'
    clueb = download_page(target_url)
    links = extract_links(clueb)
    for link in links:
        print(urljoin(target_url, link))
```   
Da lanciare in questo modo:
```
python3 link_crawler.py > output.csv
```

Per costruire un vero e proprio link crawler dovremmo memorizzare tutti i link che troviamo e poi passarli uno a uno alla ricerca dei link contenuti in quelle pagine in una sorta di ricerca ricorsiva. Naturalmente ci ricorderemo delle pagine già viste e non torneremo a visitarle se non dopo un certo periodo di tempo.

Scendiamo ora di livello ed entriamo in una pagina alla ricerca di quelli che abbiamo chiamato "dati".
```
<!DOCTYPE html> 
<html> 
    <head>
  <meta charset="UTF-8">
  <title> I Promessi Sposi</title>
   </head>
<body>
  <div itemscope="" itemtype="http://schema.org/Book" itemprop="mainEntity">
  <img itemprop="image" src="https://en.wikipedia.org/wiki/File:Francesco_Hayez_040.jpg"
     alt="Alessandro Manzoni"/>
    <span itemprop="name">I promessi sposi</span> —
    <link itemprop="url" href="https://it.wikipedia.org/wiki/I_promessi_sposi" /><br />
    di <a itemprop="author" href="https://it.wikipedia.org/wiki/Alessandro_Manzoni">Alessandro Manzoni</a>
  </div>
  

    <div itemtype="http://schema.org/Offer" itemscope="" itemprop="offers">
      <span itemprop="name">I promessi sposi</span><br />
      <meta itemprop="priceCurrency" content="EURO" />
      <span itemprop="price">€ 19.95</span>
      <link itemprop="availability" href="http://schema.org/InStock">In Stock
    </div>
   
  
</div>
</body>
</html>

```
Questo è un esempio di testo che fa uso di un linguaggio taggato molto comune: l'HTML5. E' un caso particolare di XML (affermazione approssimativa ma sufficiente in questo contesto).
Oltre a tag strutturali ne vediamo altri che individuano e illustrano aspetti contenutistici. E' un esempio, *in nuce*,  di web semantico.

Utilizziamo lxml in un esempio di web scraping. 
Prendiamo ad esempio una pagina di un sito di e-commerce 

```
from lxml import html
from lxml import etree
link = 'https://www.libreriauniversitaria.it/macroeconomia-prospettiva-europea-blanchard-olivier/libro/9788815265715'
response = requests.get(link)
sourceCode = response.content
html_elem = html.fromstring(sourceCode)
for e in html_elem.xpath('//ul[@class="dettagli-prodotto"]/li')]:
      print(e.text_content)

# Se avessimo utilizzato un parser XML avremmo ottenuto un errore perché, come spesso accade le pagine web non sono file XML "ben formati", in genere vengono # dimenticati # i tag di chiusura
# root = etree.fromstring(response.content, base_url="http://where.it/is/from.xml") 
# genera un errore

# possono essere utilizzati anche i css selectors
from lxml.cssselect import CSSSelector
for e in htmlElem.cssselect("h1[itemprop]"):
     print(e.text_content())

htmlElem.xpath('//h1[@itemprop="name"]')[0].text_content()
# oppure anche
[e.get('itemprop') for e in htmlElem]


[ e.text_content() for e in htmlElem.xpath("//span[@itemprop]")]
['I promessi sposi', 'Alessandro Manzoni', 'I promessi sposi', '€ 19.95']

``` 
Abbiamo scaricato una pagina html e ne abbiamo estratto i dati che cercavamo. Per individuarli abbiamo (manualmente) dovuto usare gli strumenti da sviluppatore presenti in tutti i principali browser. Trovato il dato che interessa scorrendo la pagina web, cliccate con il tasto destro su ‘Inspect element’ e ancora su ‘Copy XPath’

Sempre più spesso le pagine html includono "dati strutturati". Questi dati possono essere estratti facilmente.

```
import extruct
import requests
import pprint
from w3lib.html import get_base_url

r = requests.get('https://www.libreriauniversitaria.it/macroeconomia-prospettiva-europea-blanchard-olivier/libro/9788815265715')
base_url = get_base_url(r.text, r.url)
data = extruct.extract(r.content, base_url=base_url)
pp = pprint.PrettyPrinter(indent=2)

pp.pprint(data)
with open('libreria.json', 'w') as file:
    file.write(res)
```
```
'https://www.nytimes.com/2019/03/07/us/politics/paul-manafort-sentencing.html?action=click&module=Top%20Stories&pgtype=Homepage'

r = requests.get('https://www.nytimes.com/2019/03/14/science/pi-math-geometry-infinity.html')
base_url = get_base_url(r.text, r.url)
data = extruct.extract(r.content, base_url=base_url)
import json
res = json.dumps(data)
with open('nytimes.json', 'w') as file:
    file.write(res)
```  
Ora sappiamo cos'è un dato: una tripletta RDF: un soggetto, un predicato, un oggetto. Ciascuno di questi è in genere un URI.
