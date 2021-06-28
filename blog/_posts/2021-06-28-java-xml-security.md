---
layout: post
title: Java, XML og sikkerhet, juni 2021
description: >
  TL;DR
  JAXP er Oracle Javas XML API som kan brukes for lesing, skriving og prosessering av XML. Alle metodene i JAXP er “vulnerable by design” 🤯 og sårbare for blant annet XXE.
  Som Javautvikler må du som regel forholde deg til XML, man bør bruke defence-in-depth tankegang, og skru på alle sikkerhets-features for XML. XXE er sårbarheten det fokuseres på i denne artikkelen, men det finnes flere. Ikke tro! Ha håndfaste bevis for deg selv basert på det du selv vet.
  Dessverre er det ingen shortcuts, fyr opp en kanne kaffe og les [JAXP Security Guide](https://docs.oracle.com/en/java/javase/11/security/java-api-xml-processing-jaxp-security-guide.html). 
  **Eller enda bedre, les denne artikkelen, forhåpentligvis ikke så knusktørr!**
---
# Java, XML og sikkerhet, juni 2021
* this unordered seed list will be replaced by toc as unordered list
{:toc}

## Hvem har nytte av lese dette? 🎯
Dette er **ikke** en introduksjonsartikkel, jeg regner med du har erfaring med Java og at du har snust i Javas XML-implementasjon. Du er kanskje også klar over at det finnes sårbarheter omkring XML som XXE og deserialiserings-angrep.
* Backend Javautviklere som leser/skriver/prosesserer XML
* Pentestere som ønsker å vite mer om Java og XML sårbarheter
* Teamledere (send til teamet hvis de bruker XML)
### Hvem bruker XML da, kan vi ikke bare bruke JSON? 😇
XML benyttes i forferdelig mange bransjer i Norge, til forferdelig mye forskjellig. Det brukes gjerne som overføringsformat for at forskjellige systemer skal kunne snakke sammen, API-er kan eksponeres ved hjelp av XML [(SOAP)](https://en.wikipedia.org/wiki/SOAP), konfigurasjoner kan leses og skrives vha. XML, import/export av data i forskjellige systemer kan gjerne bruke XML, En del systemer har standarisert på forskjellige XML-formater i henhold til et [XML Schema](https://en.wikipedia.org/wiki/XML_schema). 

Så hvis du jobber innenfor, eller oppimot bransjer som bank, finans, helse, utdanning, kartdata, regnskap, transport, offentlige institusjoner/etater så er det stor sannsynlighet for at du enten må lese eller skrive XML, eller det benyttes XML-biblioteker her og der.
**Som Javautvikler må du som regel forholde deg til XML.**
### XML-prosessering i Java
Dette er alternativene man har for XML prosessering med bruk av standard Java API-er:
* JAXP - Java API for XML Processing
    * DOM - Document Object Model
    * SAX - Simple API for XML
    * TrAX - Transformation API for XML + xpath
    * StAX - Streaming API for XML
    * Validation - XSD Schema and Validation
    * DOMLS - DOM Load and Save
* JAXB - Java Architecture for XML Binding

Det finnes også en del andre biblioteker som kan håndtere XML, både for enkel XML, og serialisering/deserialisering av objekter til/fra XML ([med sine egne sikkerhetsproblemer](https://owasp.org/www-project-top-ten/2017/A8_2017-Insecure_Deserialization), stolt medlem av [OWASP Top 10 Web Application Security Risks](https://owasp.org/www-project-top-ten/)).

Det er en del å tenke på hvis man velger å inkludere og bruke et eksternt bibliotek, eksempelvis utbredelse, dokumentasjon, sårbarheter, oppdateringsfrekvens, alder, egne rutiner for oppdatering, egne policies ++. Sjekk [cvedetails](https://www.cvedetails.com/), [National Vulnerability Database](https://nvd.nist.gov/), Stack Overflow, Google, [exploit-db](https://www.exploit-db.com/), deres egne nettsider for dokumentasjon for å researche om det er et bibliotek man har lyst til å ta i bruk.

**For denne artikkelens del så ser jeg ikke på andre biblioteker, kun standard Java-API, eksplisitt JAXP, ikke JAXB.**

## Er ikke Java sikkert som banken da, trenger man å tenke på det? 🙈🙉🙊
### Introduksjon
Dette eksempelet er et eksempel hvor stort sett alt er gjort *"getting started way"*, så enkelt som mulig. Det er satt opp ved hjelp av de seneste versjoner som er kompatible pr. mai 2021 av: 
* [Docker 20.10.6](https://www.docker.com/), 
* [Spring Boot 2.4.6](https://spring.io/projects/spring-boot) med Web MVC
* [Java SE 11](https://openjdk.java.net/projects/jdk/11/) 
 
Intensjonen er å lage en simpel applikasjon på en server som skal lese XML fra request body og plukke ut en verdi. Har kan man tenke seg at verdien som plukkes ut brukes i f.eks et databaseoppslag, eller for å berike et objekt el.l. men for enkelhets skyld returnerer jeg bare verdien som en tekststreng i en enkel [JSON-respons](https://www.json.org/json-en.html).
### Forutsetninger
Jeg stoler kun på offisielle kilder så jeg benytter meg av [tutorials fra Oracle](https://www.oracle.com/java/technologies/jaxp-introduction.html), og jeg bryr meg ikke om error håndtering eller validering (Det er ikke noe schema) for å gjøre koden så enkel som mulig. Eksemplet som jeg viser her benytter DOM, men github kildekoden inneholder også kode som demonstrerer XXE vha SAX, StAX, TrAX, xpath, Schema, Validation og DOMLS.
Du kan prøve ut koden selv ved å klone dette prosjektet og starte docker-containeren (testet på Ubuntu 20.04):
~~~bash
apt install openjdk-11-jdk
git clone https://github.com/fjank/vulnerable-java-xml-1.git
cd vulnerable-java-xml-1
./gradlew build
docker build --build-arg JAR_FILE=build/libs/\*.jar -t fjank/vuln-xml .
docker run -p 8080:8080 fjank/vuln-xml .
~~~

### Kode
* Initialiseringen er gjort ved hjelp av [Spring Initializr](https://start.spring.io/)
* [Spring Boot with docker tutorial](https://spring.io/guides/gs/spring-boot-docker/) 
* [Serving Web Content with Spring MVC tutorial](https://spring.io/guides/gs/serving-web-content/)
* [Java API for XML Processing (JAXP) Tutorial](https://www.oracle.com/java/technologies/jaxp-introduction.html)
![Spring Initializr](/assets/img/blog/2021-java-xml/initializer.png)
DOM koden som kjøres i kontrolleren uten validering, errorhåndtering etc. skal se slik ut ihht tutorial:

~~~java
// DocumentBuilderFactory er starten for DOM parsing
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();

// .newDocumentBuilder() lager DocumentBuilder som er klassen som står for parsing av XML.
DocumentBuilder db = dbf.newDocumentBuilder();

// .parse(...) for å parse til et XML Document, her ut ifra en streng via InputSource og StringReader.
Document doc = db.parse(new InputSource(new StringReader(xml)));

// Resten er kun API kall for å plukke ut verdien vi er på jakt etter, ikke så viktig for denne artikkel.
...
~~~

For å teste ut at mini-applikasjonen fungerer slik den skal (hent ut tekst fra en XML-tag som postes til serveren med en HTTP request body), kan man for eksempel bruke [Postman](https://www.postman.com/), Eller hvis du har sjekket ut prosjektet nevnt over, så finner du en form på http://localhost:8080 hvor du kan modifisere XML slik at den blir som under.

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <test>It works!</test>
</root>
~~~
![It works](/assets/img/blog/2021-java-xml/it-works.png)
1. Her er request body som er en XML (`Content-Type: application/xml`)
2. Svaret ifra applikasjonen som en `application/json`
3. [curl](https://curl.se/) equivalent (hvis du er mer en `curl` fan)
4. rå request og response (hvis du vil se alt som sendes fram og tilbake)

Dette ser jo fint og flott ut, teksten ble hentet ut som forventet, og levert tilbake som en JSON. 😎

🏴‍☠️ La oss modde litt på requesten... 🏴‍☠️
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>
    <test>&xxe;</test>
</root>
~~~
![Ouch!](/assets/img/blog/2021-java-xml/ouch.png)

## Nasty stuff, hva er dette? 🥶
Jeg utnyttet her en XXE for LFI mot DOM-koden. Det samme skjer også mot alle de andre API-ene i JAXP, sjekk gjerne ut koden og prøv selv for å verifisere at dette er tilfelle.
### [XXE](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing) for [LFI](https://en.wikipedia.org/wiki/File_inclusion_vulnerability) 👀
Lo and behold, jeg fikk lest ut filen `/etc/passwd`. Heldigvis så er applikasjonen startet med en spesifikk bruker (*spring*, definert i Dockerfile) hvis jeg ikke hadde spesifisert en bruker, hadde applikasjonen blitt startet som *root*, og jeg hadde hatt full lesetilgang til alle filer på hele docker-containeren. Nå er det iallefall kun begrenset til filer som bruker *spring* har tilgang til. 
Det er ille nok, jeg kan f.eks. lese ut konfigurasjonen til applikasjonen med brukernavn, passord, tokens, kanskje også SSH-nøkler, laste ned klassefilene og dekompilere de for å pløye gjennom kildekoden etter flere sårbarheter, pløye gjennom loggfiler, historikkfiler osv. osv. Jeg kan lese alt som brukeren *spring* har tilgang til.
*Fun fact: hvis serveren kjører på en windows-boks, og du spesifiserer en mappe i XML exploit isteden for en fil, så får du ut directory listingen!*
#### [OOB-XXE](https://www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/)
Hvis man ikke får resultatet tilbake direkte kan man bruke out-of-band teknikker for likevel å lese filer. Det er mulig å benytte OOB-XXE i `javax.xml.validation.Validator` i Java 11 for å lese enkle filer uten linjeskift.
#### [Error based XXE](https://niravgadhiya.blogspot.com/2020/03/blind-error-based-xxe.html)
Hvis man får feilmeldinger tilbake fra serveren, kan man prøve å injecte i selve feilmeldingen.
### XXE for [DoS](https://en.wikipedia.org/wiki/Denial-of-service_attack) 💣
Jeg kan prøve å kræsje serveren, bare for å være ond med f.eks. [billion laughs attack](https://en.wikipedia.org/wiki/Billion_laughs_attack), men det er ikke noe gøy, dessuten har java 11 default beskyttelse for å unngå dette. 😅

### XXE for [portscanning](https://en.wikipedia.org/wiki/Port_scanner) og nettverksscanning 🔎
Eksempelvis ved å bruke `<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://internal-server.intra.net:22">]>` så vil XML-parser prøve å connecte til den serveren på port 22, typisk åpen på Linux-maskiner, eller vi kan bruke port 139, typisk åpen på Windows-maskiner. Finnes maskinen, så vil det gå raskt (DNS-oppslaget returnerer med en gang), finnes den ikke går DNS-oppslaget tregere, ergo kan vi bruke timing og gigantiske ordlister for å mappe opp maskiner på samme nettverk (**bak brannmuren, husk at det er serveren som hoster applikasjonen som utfører kallene**). Jakting på tjenester og porter ved hjelp av XXE er gøy.

### XXE for [SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) 🐱‍👤
Er vi heldige så får vi svaret vårt tilbake i en respons, da kan vi etter å ha kjørt hostscanning og portscanning, forespørre interne nettverksressurser uten å tenke på brannmurregler. [Her er et eksempel ifra virkeligheten](https://www.aon.com/cyber-solutions/aon_cyber_labs/ssrf-and-xxe-vulnerabilities-in-pdfreactor/).

### XXE to [RCE](https://en.wikipedia.org/wiki/Arbitrary_code_execution) 🦸‍♀️
XXE alene er ikke så gøy, det er når man kan kombinere med andre sårbarheter at det begynner å bli gøy. [Her er et nasty eksempel fra 2018](https://www.ambionics.io/blog/oracle-peoplesoft-xxe-to-rce) på en XXE som ble utnyttet til å få `nt authority/system` shell på alle servere med Oracle PeopleSoft installert.

### Når må man tenke på XML og XXE? Hva kan vi stole på? 🕵️‍♀️
Alle XML-er som man ikke har kontroll på bør man ikke stole på. 
* XML-er som kommer inn som en request ifra internet: Åfy! 
* XML-er ifra en partner: sikker på at partneren har kontroll? Ikke stol på det. 
* XML-er fra en server i samme VLAN: Sikker på at maskinen ikke er kompromittert? Don't trust it! 
* XML-er på disk: Sikker på at ingen har tuklet med de?
* Osv osv. 

Ikke stol på noen, ha som utgangspunkt at XML som skal prosesseres er ondskapens verk som kun er ute etter å lage bråk.

## Javel, hva er den beste måten å lese XML på i Java ifølge Oracle? 😑
Å kun lese og bruke [Java API for XML Processing (JAXP) Tutorial](https://www.oracle.com/java/technologies/jaxp-introduction.html) er ikke nok for å prosessere XML på en sikker måte. **Man må også lese [sikkerhetsguide for utviklere](https://docs.oracle.com/en/java/javase/11/security/index.html),** spesifikt kapittel 12 hvor vi finner en god del mer informasjon. Man må lese denne med lupe, for her står det at [FSP](https://docs.oracle.com/en/java/javase/11/security/java-api-xml-processing-jaxp-security-guide.html#GUID-88B04BE2-35EF-4F61-B4FA-57A0E9102342) er default på for å beskytte mot DoS, men det tilgjengeliggjør fremdeles eksterne entiteter, [den må eksplisitt settes for å blokkere XXE](https://docs.oracle.com/en/java/javase/11/security/java-api-xml-processing-jaxp-security-guide.html#GUID-94ABC0EE-9DC8-44F0-84AD-47ADD5340477). **JAXP er "vulnerable by design"** 🤯.

For å beskytte seg mot 0-days, ukjente bugs eller framtidige bugs som enda ikke er laget, bør man bruke [defence-in-depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)) tankegang, og skru på alle sikkerhets-features for XML-parsere, selv om de tilsynelatende ikke er nødvendig. Det kan for eksempel være en spesifikk kodebane i JAXP/JAXB som gjør at den ene sikkerhets-featuren ikke blir trigget. Har man kun den beskyttelsen, så har angripere funnet en åpning. Derimot hvis man har skrudd på flere lag med beskyttelse, så vil forhåpentligvis angrepet bli stoppet av neste lag.

Etter min mening er sikkerhetsguiden forferdelig. Komplisert, ikke komplett, ustrukturert og kommer med vage anbefalinger, så den bør ikke være eneste kilde til sannheten.

Ei heller har ikke alle factories støtte for de samme features og properties, så man må finne ut hva som er støttet og ikke støttet for hver enkelt type ved hjelp av API-dokumentasjon, JAXP Security Guide, inspisering av JDK-kildekode (f.eks. debugging) og andre kilder (pentestrapporter, sårbarhetsrapporter, bug bounty-rapporter etc.). Under følger min anbefaling for de forskjellige delene av JAXP, tror jeg har fått med meg alle sammen.
*Jeg skal få sjekket om buggen jeg fant (DOMLS) er rapportert eller ikke, og rapportere den.*


Selvfølgelig så bør man bruke standard OO teknikker f.eks. factories, factory methods, singletons e.l. for å få config på en plass. Dette gjelder for programmatisk konfigurasjon av JAXP factories. Det er også mulig å bruke jaxp.properties og/eller System properties, men det er utenfor scope til denne artikkelen.

**Bruk eksemplene under som det de er, eksempler! Les [viktig info under eksemplene.](#Viktig-info)**

### Defence in depth-innstillinger for DOM
~~~java
// For DOM, skru på sikkerhetsinnstillinger via SAXParserFactory
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();

// skru på FSP
dbf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);

// skru av mulighetene for at XML kan ha en DOCTYPE
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
// skru av mulighetene for eksterne DTDs
dbf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
// skru av mulighetene for eksterne entiteter
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
// skru av mulighetene for eksterne parameter-entiteter
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities",  false);

// Sett tillatte protokoller for ekstern DTD til ingen
dbf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
// Sett tillatte protokoller for eksterne schema til ingen
dbf.setAttribute(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");

// skru av xinclude støtte 
dbf.setXIncludeAware(false);
dbf.setFeature("http://apache.org/xml/features/xinclude", false);
// skru av entity resolution
dbf.setExpandEntityReferences(false);

// Nå kan du lage en DocumentBuilder og parse på en trygg måte.
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(new InputSource(new StringReader(xml)));
~~~
### Defence in depth-innstillinger for SAX
~~~java
// Defence in depth innstillinger for SAX
// For SAX, skru på sikkerhetsinnstillinger via SAXParserFactory
SAXParserFactory spf = SAXParserFactory.newInstance();

// skru på FSP
spf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);

// skru av mulighetene for at XML kan ha en DOCTYPE
spf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
// skru av mulighetene for eksterne DTDs
spf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
// skru av mulighetene for eksterne entiteter
spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
// skru av mulighetene for eksterne parameter-entiteter
spf.setFeature("http://xml.org/sax/features/external-parameter-entities",  false);


// skru av xinclude støtte 
spf.setXIncludeAware(false);
spf.setFeature("http://apache.org/xml/features/xinclude", false);
//Nå kan du lage en SAXParser med en ContentHandler og parse på en trygg måte.
~~~
### Defence in depth-innstillinger for StAX
~~~java
// Beware! StAX støtter ikke FSP!
// For StAX, skru på sikkerhetsinnstillinger via XMLInputFactory
XMLInputFactory f = XMLInputFactory.newInstance();

// Skru av støtte for DOCTYPE
f.setProperty(XMLInputFactory.SUPPORT_DTD, false);
// Skru av støtte for eksterne entiteter
f.setProperty(XMLInputFactory.IS_SUPPORTING_EXTERNAL_ENTITIES, false);

// Sett tillatte protokoller for ekstern DTD til ingen
f.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
// Sett tillatte protokoller for eksterne schema til ingen
f.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");

// Skru på ignorering av eksterne DTD's.
f.setProperty("http://java.sun.com/xml/stream/properties/ignore-external-dtd", true);
//Nå kan du lage en XMLStreamReader/XMLEventReader og plukke verdier på en trygg måte.
~~~
### Defence in depth-innstillinger for Schema og Validator
~~~java
// XXE i Schema/Validator er som regel error-basert eller OOB
SchemaFactory sf = SchemaFactory.newDefaultInstance();

// skru på FSP
sf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);

// Sett tillatte protokoller for ekstern DTD til ingen
sf.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
// Sett tillatte protokoller for eksterne schema til ingen
sf.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");

// Schema vil nå få sine innstillinger fra SchemaFactory
Schema schema = sf.newSchema(new StreamSource(new StringReader(xml)));
// Validator vil også få sine innstillinger fra SchemaFactory
Validator validator = schema.newValidator();
/* Det er også mulig å hente en ValidatorHandler med newValidatorHandler(),
 * men jeg har ikke brukt den og vet heller ikke hvordan den fungerer, 
 derfor har jeg heller ingen eksempelinstillinger for denne.*/
~~~

### Defence in depth-innstillinger for TransformerFactory/SAXTransformerFactory
~~~java
// Her er det mulig å injecte både i xml og i stylesheet
TransformerFactory tf = TransformerFactory.newDefaultInstance();

// skru på FSP
tf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);

// Sett tillatte protokoller for ekstern DTD til ingen
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
// Sett tillatte protokoller for eksterne stylesheets til ingen
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");

// Her brukes StreamSource, bruker man DOMSource, SAXSource eller StAXSource, 
// sørg også for å sette defence in depth-innstillinger for disse kildene.
StringWriter writer = new StringWriter();
Result result = new StreamResult(writer);
Transformer t = tf.newTransformer(new StreamSource(new StringReader(xslt)));
t.transform(new StreamSource(new StringReader(xml)), result);
~~~
### Defence in depth-innstillinger for XPathFactory
~~~java
XPathFactory xpf = XPathFactory.newDefaultInstance();
// Skru på FSP, men den skrur kun av eksterne xpath funksjoner. XXE via xml er fremdeles mulig!
xpf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
XPath xPath = xpf.newXPath();
XPathExpression xpe = xPath.compile(expression);
// Ikke bruk xpe.evaluate(inputSource, XPathConstants.STRING), siden den bruker en utrygg DocumentBuilderFactory
// Lag en DocumentBuilderBuilderFactory som hindrer XXE, parse deretter til org.w3c.dom.Document og send dette til evaluate.
DocumentBuilderFactory dbf = DocumentBuilderFactory.newDefaultInstance();
dbf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
dbf.setAttribute(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
dbf.setXIncludeAware(false);
dbf.setProperty("http://apache.org/xml/features/xinclude", false);
dbf.setExpandEntityReferences(false);

DocumentBuilder db = dbf.newDocumentBuilder();
Document document = db.parse(new InputSource(new StringReader(xml)));
Object result = xpe.evaluate(document, XPathConstants.STRING);
~~~
### Defence in depth-innstillinger for DOMLS
~~~java
DOMImplementationRegistry registry = DOMImplementationRegistry.newInstance();
DOMImplementationLS impl = (DOMImplementationLS) registry.getDOMImplementation("LS");
LSParser builder = impl.createLSParser(DOMImplementationLS.MODE_SYNCHRONOUS, null);
DOMConfiguration config = builder.getDomConfig();

// FSP er støttet, men en subtil bug hindrer oss i å aktivere FSP.
//dbf.setParameter(XMLConstants.FEATURE_SECURE_PROCESSING, true);

// skru av mulighetene for at XML kan ha en DOCTYPE
// ekvivalente verdier.
config.setParameter("disallow-doctype", true);
config.setParameter("http://apache.org/xml/features/disallow-doctype-decl", true);
// skru av mulighetene for eksterne DTDs
config.setParameter("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
// skru av mulighetene for eksterne entiteter
config.setParameter("http://xml.org/sax/features/external-general-entities", false);
// skru av mulighetene for eksterne parameter-entiteter
config.setParameter("http://xml.org/sax/features/external-parameter-entities",  false);

// Skru av XInclude:
config.setParameter("http://apache.org/xml/features/xinclude", false);

// Nå kan du lage en LSInput og parse på en trygg måte.
LSInput lsInput = impl.createLSInput();
lsInput.setCharacterStream(new StringReader(xml));
Document doc = builder.parse(lsInput);
~~~

### Viktig info
Uansett om applikasjonen fungerer, eller hvis applikasjonen din gjør noe litt annerledes en de enkle eksemplene over, og/eller hvis applikasjonen ikke fungerer etter dette enten fordi den bruker for mye ressurser, trenger eksterne/interne entiteter eller du absolutt skal gjøre noe fancy, anbefaler jeg å gjøre følgende: 
* Hold tunga rett i munnen
* Les API-dokumentasjonen 
* Les [JAXP Security Guide](https://docs.oracle.com/en/java/javase/11/security/java-api-xml-processing-jaxp-security-guide.html)
* Følg Oracle's anbefalinger for JAXP og sikkerhet som du finner [helt nederst i sikkerhetsguiden](https://docs.oracle.com/en/java/javase/11/security/java-api-xml-processing-jaxp-security-guide.html#GUID-82A5172C-C75E-4276-93C4-C3D2774C97E1) som en en kort sjekkliste.
    * Skru på featuren FSP, juster deretter de individuelle features og egenskaper i henhold til dine spesifikke krav.
    * For prosesseringsbegrensninger, juster de slik at de er akkurat store nok til å håndtere maksimum som applikasjonen krever.
    * For eksterne tilgangsrestriksjoner, reduser eller fjern applikasjonens avhengigheter av eksterne ressurser inkludert bruken av "resolvere", deretter begrens disse restriksjonene.
    * Sett opp en lokal katalog og åpne opp Catalog API for alle XML parsere for å ytterligere redusere applikasjonens avhengighet av eksterne ressurser.
* Sjekk andre kilder for sårbarheter
* Kjør code review med fokus på sikkerhet 
* Kjør statisk kodeanalyse f.eks. [SonarQube](https://www.sonarqube.org/), [Find Security Bugs](https://find-sec-bugs.github.io/), [semgrep](https://semgrep.dev/) 
* Kjør eksplisitt pentesting av delen som har med XML å gjøre

## Fallgruver
* **Java-versjon:** Denne artikkelen og tilhørende eksempler er basert på Oracle Java versjon 11.0.9 og 11.0.11, testet på Windows 10 og Ubuntu 20.04. Eldre eller nyere versjoner av Java kan ha annen funksjonalitet, deprekerte klasser/metoder, andre defaults, bugs, etc. Sjekk ut API-dokumentasjon og Security guide for tilhørende versjon, og gjør dine egne tester. 
* **JVM-type:** På samme måte som med Java-versjon, så er også [JVM-typen](https://en.wikipedia.org/wiki/List_of_Java_virtual_machines) viktig å ta hensyn til, forskjellige implementasjoner kan ha forskjellig underliggende funksjonalitet, forskjellige defaults, andre bugs etc. Sjekk implementasjonsspesifikke guider vedrørende sikkerhet.
* **Ikke default parser:** Det er også mulig å bytte ut default implementasjonen for JAXP med en annen XML-parser, (Enten på egen hånd, eller en applikasjonsserver kan trylle inn sin egen parser), som igjen kan ha forskjellig underliggende funksjonalitet, forskjellige defaults osv. Sjekk implementasjonsspesifikke guider vedrørende sikkerhet hvis dette er gjort.


## Oppsummering
Jobber man med Java, og har med XML å gjøre, så er sikkerhet noe man må tenke på og sette seg godt inn i. 
Jeg anbefaler å ikke stole på noen andre enn deg selv (*ikke på meg og denne artikkelen heller*), utforsk sikkerhetsinnstillinger, prøv å bryt deg inn i egen kode, få hjelp av en pentester, viktigst av alt: **ikke tro! Ha håndfaste bevis for deg selv basert på det du selv vet.**

Denne artikkelen har kun snakket om en spesifikk XML-sårbarhet (XXE), men det finnes også andre sårbarheter som man må vite om og ta høyde for under utvikling (f.eks. deserialiserings-angrep). Gode rutiner for sikker utvikling og kunnskap om sårbarheter er nødvendig for å beskytte applikasjonen(e) man skriver mot angrep, både målrettede og tilfeldige.

Finner du noen feil/svakheter/forbedringspotensiale i artikkelen eller koden på GitHub så vil jeg gjerne vite det! Legg igjen kommentarer, send epost, whatnot, jo før jo heller.

Hvis denne artikkelen var interessant og dere vil at jeg skal skrive mer, gjerne om noe spesifikt innenfor Java/PHP/Node/Python/C# -verdenen og en eksplisitt sårbarhet, legg igjen en kommentar så skal jeg se hva jeg får til. 🥰