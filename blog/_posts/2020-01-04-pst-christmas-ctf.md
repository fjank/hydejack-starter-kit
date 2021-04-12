---
layout: post
title: PST Julekalender
description: >
  PST (Politiets Sikkerhetstjeneste) hadde i 2019 en julekalender. Den ble annonsert via en fiktiv annonse for
  NPST (Norpolar Sikkerhetstjeneste), som hadde en jobbannonse etter midlertidig stilling som snikende alvebetjent.
  Dette er en gjennomgang av julekalenderen som var en full CTF. 
---
# 2019 - NPST utlyser en midlertidig stilling som snikende alvebetjent
* this unordered seed list will be replaced by toc as unordered list
{:toc}
## Introduksjon
![Eksempel på annnonse](/assets/img/blog/2020-npst/ad.jpeg)
Mot slutten av november i 2019 ble det sluppet en del stillingsannonser og [artikler](https://www.vg.no/nyheter/innenriks/i/JowQQb/pst-soeker-snikende-alvebetjent-paa-svalbard) 
om NPST som utlyser en midlertidig stilling som snikende alvebetjent. Etter å ha gått igjennom en del av stillingsannonsene, 
Kunne man finne en [faktisk link](https://npst.no/) til starten av en veldig gøy CTF arrangert av [PST](https://www.pst.no/).
Man kunne nok ha gjettet seg fram til domenet også.

![Forsiden til NPST julekalender](/assets/img/blog/2020-npst/npst-start.png)
Her var en liten intro video, en link til en webcruiter annonse, og en link til den faktiske kalenderen.
![kalender startside](/assets/img/blog/2020-npst/kalender.png)
Hver dag kl 00:00 åpnes en ny luke og en ny oppgave. Jeg tenkte til å begynne med å løse oppgavene når de åpnet, 
men så raskt at jeg ikke hadde nok erfaring, for det var alltid noen som hadde løst oppgavene på betydelig kortere
tid enn meg, så jeg sluttet raskt med å løse oppgaver på natten. Jeg var fornøyd hvis jeg fikk løst alle oppgavene.

Python koden er kun testet at fungerer, den er ikke ryddet opp i, så den er ikke akkurat optimal. 🤪  

## Luke 1
###  Velkommen til NPST (10)
    De dagene det blir ytret ønske om din ekspertise, trenger vi at du svarer i NPSTs intranett. 
    Svarene vi ofte er på jakt etter pleier å være på formen PST{noenTilfeldigeBokstaver}. 
    Avvik vil tydelig annonseres. For å kunne legge inn besvarelser der må du opprette en bruker. 
    Pass på å benytte en epost der det er mulig å kontakte deg dersom du ønsker å delta i kåringene ukens ansatt 
    og årets alvebetjent (se praktisk info lenger nede).
    
    Intranettet finner man her: https://intranett.npst.no.
    
    Etter du har laget deg en bruker i intranettet trenger vi også at du logger deg på et 
    litt eldre system du finner her: 
    https://login.npst.no. Vi har ikke helt fått klarhet i hva dette systemet brukes til, 
    men vi tror du skal kunne logge inn med følgende brukernavn og passord:
    
    Brukernavn: bruker
    Passord: Advent2019


Første oppgave er å lage seg en bruker i CTF løsningen som brukes, deretter får man oppgitt brukernavn og passord til 
[https://login.npst.no](https://login.npst.no). Dette gir deg flagget.  
    
    ███╗   ██╗██████╗ ███████╗████████╗  
    ████╗  ██║██╔══██╗██╔════╝╚══██╔══╝  
    ██╔██╗ ██║██████╔╝███████╗   ██║    
    ██║╚██╗██║██╔═══╝ ╚════██║   ██║   
    ██║ ╚████║██║     ███████║   ██║   
    ╚═╝  ╚═══╝╚═╝     ╚══════╝   ╚═╝   
    
    Brukernavn: bruker
    Passord: Advent2019
    
    Velkommen!
    Løsningen på oppgaven er PST{a7966bf58e23583c9a5a4059383ff850}.
    
    Du må endre passordet ditt.
    
    Nytt passord: 

Flagg: PST{a7966bf58e23583c9a5a4059383ff850}
{:.note}

### Velg passord (15)
    Gratulerer!    
    Nå er det på tide å bytte passord på brukeren din. 
    Gjeldende passord policy er kjent for å være litt kranglete, så lykke til!
      
Det ser her ut til at det er regler som må følges, etter litt prøving og feiling ser det ut til at regelsettet er:

    Passordet må inneholde minst ett tegn fra hver av de følgende gruppene:
     - [a-z]
     - [A-Z]
     - [0-9]
     - [*@!#%&()^~{}]
    Tegnene i passordet må skrives i stigende ASCII-verdi.
    Summen av ASCII-verdier modulo 128 må være lik 24.

Et lite python program er en løsning or å finne et nytt gyldig passord, 

~~~python
# file: "findpw.py"
import string
pw = ''

def checkGroups(attempt):
    lower = False
    upper = False
    number = False
    special = False
    for c in attempt:
        if c in string.lowercase:
            lower = True
            continue
        if c in string.uppercase:
            upper = True
            continue
        if c in string.digits:
            number = True
            continue
        if c in '*@!#%&()^~{}':
            special = True
            continue

    return lower and upper and number and special

def checkAscending(attempt):
    last = 0
    for c in attempt:
        o = ord(c)
        if o <= last:
            return False
        last = o
    return True

def checkModulo(attempt):
    # now check the modulo according to the rules.
    sum = 0
    for c2 in attempt:
        sum += ord(c2)
    if sum % 128 == 24:
        return True
    return False

for c in string.printable:
    for c2 in string.printable:
        for c3 in string.printable:
            for c4 in string.printable:
                pw = c + c2 + c3 + c4
                asc = checkAscending(pw)
                if not asc:
                    continue
                modCorrect = checkModulo(pw)
                if not modCorrect:
                    continue
                groupCorrect = checkGroups(pw)
                if not groupCorrect:
                    continue
                print(pw)
~~~

Det første og beste passordet `0@Ag` gir oss flagg 2.

    Passordet kunne ikke bli endret på nåværende tidspunkt.
    Prøv igjen på nyåret!
    
    PST{6a0f0731d84afa4082031e3a72354991}

Flagg: PST{6a0f0731d84afa4082031e3a72354991}
{:.note}

### Passordgjenoppretting (20)
    En tidligere ansatt måtte slutte etter å ha endt opp på listen over slemme barn. 
    Dessverre glemte vi å be han kopiere ut filene sine før han sluttet, og vi har følgelig ikke passordet.  
    Dette er det vi har av info:  
          Brukernavn: admin  
          ???: 42f82ae6e57626768c5f525f03085decfdc5c6fe  
    Klarer du å logge inn på kontoen?

Her har vi fått oppgitt en hash, [hash-identifier](https://code.google.com/archive/p/hash-identifier/) sier at det sannsynligvis er en [SHA-1](https://en.wikipedia.org/wiki/SHA-1), 
så her bruker jeg [hashcat](https://hashcat.net/hashcat/). 
Min mangel på erfaring gjør her at jeg begynner i feil ende og kaster bort mye tid.
Jeg prøver å cracke den med [rockyou](https://en.wikipedia.org/wiki/RockYou), alle passordlister i [seclists](https://github.com/danielmiessler/SecLists), alle [rules](https://www.notsosecure.com/one-rule-to-rule-them-all/) som ligger default i hashcat på alle 
på alle foregående passordlister, men får ikke noe resultat. 
Som et siste desperat forsøk setter jeg på en bruteforce, med en plan om å la den stå over natten. 
2 sekunder senere har den funnet passordet. 😒

~~~bash
$ hashcat -m 100 -a 3 -1 ?a hash ?1?1?1?1
~~~
    Velkommen! 
    Systemet er deaktivert grunnet julestri. 
    Frem til nyttår kan du nå filene dine her:
    - løsning.txt
    - notater.jpg
Løsning.txt inneholder flagget vi er på jakt etter:

    ███████╗██╗      █████╗  ██████╗  ██████╗    
    ██╔════╝██║     ██╔══██╗██╔════╝ ██╔════╝ ██╗
    █████╗  ██║     ███████║██║  ███╗██║  ███╗╚═╝
    ██╔══╝  ██║     ██╔══██║██║   ██║██║   ██║██╗
    ██║     ███████╗██║  ██║╚██████╔╝╚██████╔╝╚═╝
    ╚═╝     ╚══════╝╚═╝  ╚═╝ ╚═════╝  ╚═════╝                                                 
    PST{36044221cd3e991ffc56eb2f1e368ca0}


Flagg: PST{36044221cd3e991ffc56eb2f1e368ca0}
{:.note}
notater.jpg er et bilde av en pult med en datamaskin, en kopp, en grøtskål 
og en gul lapp med informasjon `IV PS ⚙️` det skal vise seg at vi får bruk for senere.

## Luke 2
    Oppdatert reiseråd

    Det vurderes som høy risiko å medbringe tjenestetelefon til SYDPOLARE strøk. 
    Det er lett for fremmede makters etterretningstjenester å ta seg inn i mobiltelefoner og nettbrett. 
    En alvebetjents telefon kan gi god innsikt i NPSTs virke, og inneholde sensitiv informasjon.

    I et utenlandsk mobilnett vil all kommunikasjon til og fra enheten kunne avlyttes, 
    uten at brukeren av mobiltelefonen kan oppdage det. NPST anser det som rimelig temmelig sannsynlig 
    at slik kapasitet er etablert på permanent basis i mobilnettverkene og internett i SYDPOLARE områder. 
    NPST tilbyr sikkerhetsbriefer til alvebetjenter 
    og andre offentlige tjenestepersoner i forkant av reiser til høyrisikoområder.

    Barn i områder under SYDPOLAR kontroll bør sende sine ønskelister 
    til Jule Nissen gjennom trygge krypterte kanaler.
    
Ingen oppgaver denne dagen.
## Luke 3 - PPK (0)
    Elektronisk krypteringsmaskin
    
    NPST har avdekket at den norske overvåkingstjenesten overtok 
    en rekke ENIGMA maskiner etter den tyske okkupasjonsmakten. 
    ENIGMA ble i stort omfang benyttet av de fleste tyske militære styrker til både kryptering og dekryptering 
    av informasjon under andre verdenskrig. Den var enkel å bruke, og antatt å være umulig å knekke. 
    Dette var hovedårsaken til dens utbredelse. 
    ENIGMA ble benyttet av den norske overvåkingstjenesten som en reserveløsning frem til 1960.
    
    Som ledd i skjermingen av ønskelister har NPST vurdert 
    å ta i bruk ENIGMA som erstatning for dagens krypteringsprosedyre. 
    Grunnet manglende EDB-entusiasme i grasrota møter dette stor motstand, 
    da medbragt EDB-utstyr ute i felt er svært lite populært. 
    Det har blitt ytret ønske om å beholde dagens penn-og-papir-kryptering (PPK), 
    men det er uvisst hvor sikkert dette er. Kunne noen forsøkt å knekke følgende PPK-kode:
    
    KNO fmwggkymyioån 30å6ø8432æå54710a9æ09a305å7z9829 fmwggkymyioån ngpoo
    
    Skriv gjerne svaret i intranettet.

Penn og papir kryptering er høyst sannsynlig en enkel krypteringsmåte, [cyberchef](https://gchq.github.io/CyberChef/) er mitt goto tool for exploration.
Her viser det seg å være en veldig enkel [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher): ROT5. 
Cyberchef har ikke støtte for æøå, men det er raskt å manuelt ta de bokstavene som havner i det området.  

Flagg: PST{30e6d8432ce54710f9c09f305e7b9829}
{:.note}

## Luke 4
    Ønskelister på avveie
    Takket være mange tilbakemeldinger har NPST konkludert med at 
    krypteringen brukt på ønskelistene ikke er tilstrekkelig. 
    Alvebetjent Methi viste med sin kløkt at det var mulig å knekke koden på under 10 minutter! 
    Jule NISSEN vil også takke følgende alvebetjenter for rask utførelse (under 60 minutter):

    unblvr, fjank, ameneon, Furris, kristaver, TrustedInstaller, bjarneUbetjent, lokling, 
    endrelem, sayco, StianS, Olga, ColdCoffeeCup, vidarli, stiffi, potetsos, jallasprit, 
    57ur14, zup, pask, grobert, Noldus og atletiskj.

    Med dette som bakteppe har NPST mistanke om at ønskelistene til 453 barn er på avveie. 
    Det er grunn til å tro at sydpolare hackere står bak.
    NISSEN er på joggreise, men vil ta tak i problemet så snart han har returnert.

    Overtidspepperkaker
    Det har den siste tiden vært stor pågang etter normal arbeidstid, spesielt rundt midnatt. 
    Dette har gått hardt utover overtidsbudsjettet. NISSEN vil vurdere tiltak for å redusere disse kostnadene.
    
Ingen oppgaver denne dag, men jeg har fått en takk fra Jule NISSEN, og ligger i hovedfeltet! 😂 

## Luke 5 - 🐒i (5)
    Oppdatering på ønskeliste-lekkasje

    Ytterligere 72 ønskelister ser ut til å være på avveie.
    NISSEN jobber på spreng for å utvikle en ny uknekkelig PPK.

    Kranglete API
    Våre analytikere har gravd litt i et system en tidligere ansatt jobbet med før avskjedigelse. 
    Noen av adventsvikarene er kjent med personen fra tidligere 
    gjennom uthenting av filer og gjenoppretting av passord. 
    Det ser ut til at systemet er skrevet i node.js, 
    men analytikerne våre greier ikke helt skjønne hvordan det fungerer. 
    Det finnes heller ikke noen dokumentasjon. 
    Kunne noen EDB-kompetente alvebetjenter sett nærmere på dette, 
    og rapportert tilbake i intranettet om det ligger noe av interesse der?

    https://npst.no/api/🙃.js

    Julevurdering
    Med bakgrunn i ønskelistelekasjen er julevurderingen nedgradert. Det er nå MULIG at det blir en GOD JUL.

Ved å åpne denne url får vi en json tilbake:
~~~json
{
  "error":false,
  "state":"[>🍕<, 🍉, 🐴, 🐟, 🚀, 🚩]",
  "message":"Bruk /api/🙃.js?commands=🤷 for en 📃 over 👌 ,n🚽er."
}
~~~
Oversatt til: hjelp for en liste over ok kommandoer. Javel, la oss prøve `/api/🙃.js?commands=🤷 `.
~~~json
{ "error":false,
  "state":"[>🍕<, 🍉, 🐴, 🐟, 🚀, 🚩]",
  "message":"Tilgjengelig ,n🚽er: ✨, ⚡, 🔑, 🤷. Eksempel: /api/🙃.js?commands=✨⚡✨"
}
~~~
Jaha, la oss prøve `/api/🙃.js?commands=✨⚡✨` da.
~~~json
{
  "error":false,
  "state":"[🍕, >🐟<, 🚩, 🚀, 🍉, 🐴]",
  "message":""
}
~~~
Så, her kan man forsøke å forstå hvordan dette gøye api'et fungerer, for det er jo et flagg i state verdien, 
det ser ut til at en av verdiene i state er valgt, så oppgaven er nok å få valgt flagget.
Jeg forsøker _ikke_ å forstå dette, jeg lager meg et rudimentært brute force python program.
Etter kort tid finner jeg flagget på bl.a. command `⚡⚡⚡⚡✨🔑`                                                                                  
~~~json
{
  "error":false,
  "state":"[🐟, 🚀, 🍉, 🐴, >🚩<, 🍕]",
  "message":"PST{ba323c3f5b3f1b536461d41cc7f1ba60}"
}
~~~

Flagg: PST{ba323c3f5b3f1b536461d41cc7f1ba60}
{:.note}


~~~python
# file: "brute-flag.py"
# -*- coding: utf-8 -*-
import requests, codecs, sys
UTF8Writer = codecs.getwriter('utf8')
sys.stdout = UTF8Writer(sys.stdout)

validChars = u'✨⚡🔑🤷'


for c in range(0, 3):
    print validChars[c]
    response = requests.get(u'https://npst.no/api/🙃.js?commands='+ validChars[c])
    print(response.text)
    for d in range(0, 3):
        print validChars[c] + validChars[d]
        response = requests.get(u'https://npst.no/api/🙃.js?commands='+ validChars[c]+validChars[d])
        print(response.text)
        for e in range(0, 3):
            print validChars[c] + validChars[d] + validChars[e]
            response = requests.get(u'https://npst.no/api/🙃.js?commands='+ validChars[c]+validChars[d]+validChars[e])
            print(response.text)
            for f in range(0,3):
                print validChars[c] + validChars[d] + validChars[e]+validChars[f]
                response = requests.get(u'https://npst.no/api/🙃.js?commands='+ validChars[c]+validChars[d]+validChars[e]+validChars[f])
                print(response.text)
                for g in range(0,3):
                    print validChars[c] + validChars[d] + validChars[e]+validChars[f]+validChars[g]
                    response = requests.get(u'https://npst.no/api/🙃.js?commands='+ validChars[c]+validChars[d]+validChars[e]+validChars[f]+validChars[g])
                    print(response.text)
                    for h in range(0,3)
                        print validChars[c] + validChars[d] + validChars[e]+validChars[f]+validChars[g]+validChars[h]
                        response = requests.get(u'https://npst.no/api/🙃.js?commands='+ validChars[c]+validChars[d]+validChars[e]+validChars[f]+validChars[g]+validChars[h])
                        print(response.text)
~~~

## Luke 6 - PPKv2 (5)
    Ny pr053dyr3 f0r kryp73ring

    NI553N h4r få77 på p1455 ny PPK. K4n ni553n3 hj31p3r3 v3rifi53r3 4t k0d3n 3r ukn3kk31ig?

    KNO fmw55k8m7i179 z98øyåz8æy67aåy0å6æ7aø1å1438åa5a fmw55k8m7i179 95p11

Her ser det også ut til at det er en standard Caesar Cipher, og et sterkt hint til at [l33t](https://simple.wikipedia.org/wiki/Leet) er brukt i tillegg.
Standard rotering gir oss imidlertid bare søppel, men roterer vi små/store bokstaver for seg selv (ROT5) og tall for seg selv, 
så kommer løsningen av seg selv: `PST krø11p4r3n735 b54daeb4ca23fea6e2c3fd7e7094ef1f krø11p4r3n735 51u77` skrevet i l33t.
Det var forøvring ikke mulig å løse denne til å begynne med, iom at det faktisk var en feil i hashen som var lagt ut. 
Alikevel var det noen som hadde løst den, se en fantastisk forklaring på hvordan [her](https://github.com/myrdyr/ctf-writeups/tree/master/npst).

Flagg: PST{b54daeb4ca23fea6e2c3fd7e7094ef1f}
{:.note}

~~~python
# file: "solver.py"
destriptio2 = 'Ny pr053dyr3 f0r kryp73ring\
NI553N h4r få77 på p1455 ny PPK. K4n ni553n3 hj31p3r3 v3rifi53r3 4t k0d3n 3r ukn3kk31ig?'


description = 'Ny pr053dyr3 f0r kryp73ring\
NI553N h4r få77 på p1455 ny PPK. K4n ni553n3 hj31p3r3 v3rifi53r3 4t k0d3n 3r ukn3kk31ig?'

# Seems like a standard rot 5, but the devil is in the details...
codeo = 'kno fmw55k8m7i179 z98øyåz8æy67aåy0å6æ7aø1å1038åa5a fmw55k8m7i179 95p11'
coded = 'KNO fmw55k8m7i179 z98øyåz8æy67aåy0å6æ7aø1å1438åa5a fmw55k8m7i179 95p11'

original = 'abcdefghijklmnopqrstuvwxyzæøåABCDEFGHIJKLMNOPQRSTUVWXYZÆØÅ'
rotated = 'fghijklmnopqrstuvwxyzæøåabcdeFGHIJKLMNOPQRSTUVWXYZÆØÅABCDE'

leetOriginal =  '0123456789'
leetTranslat1 = '0123456789'

def rotCharsAndNUmbers(chars):
    rv = ''
    for pos, c in enumerate(chars):
        # skip space
        if c == ' ':
            rv += c
            continue
        # skip numbers
        if c in '0123456789':
            num = int(c) + 6
            if num > 9:
                num = num - 10


            rv += str(num)
            continue
        # skip unknown
        if (c < 'a' or c > 'z') and c not in 'æøåÆØÅ' and (c < 'A' or c > 'Z'):
            rv += c
            continue
        idx = original.index(c)
        rv += rotated[idx]
    return rv


def antiLeet(chars, leetTranslat):
    rv = ''
    for pos, c in enumerate(chars):
        # skip space
        if c == ' ':
            rv += c
            continue
        # skip numbers
        if c not in '0123456789':
            rv += c
            continue

        idx = leetOriginal.index(c)
        result = leetTranslat[idx]
        if result not in '0123456789abcdefEAB':
            rv += c
        else:
            rv += result
    return rv


print(coded)
rot = rotCharsAndNUmbers(coded)
print(rot)

trans = antiLeet(rot, leetTranslat1)
#print(trans)
~~~


## Luke 7
### Nissens verksted (3)
    Ukens ansatt
    Jule NISSEN gratulerer unblvr som ukens ansatt, og takker for ekstraordinær innsats. 
    NISSENS sekretariat har tatt kontrakt via elektronisk post.

    Korrigering av gårsdagens dagsbrief
    NISSEN vil også takke en dyktig alvebetjent som har gjort NPST oppmerksom på 
    en feil i den krypterte teksten gårsdagens dagsbrief. Denne er nå korrigert.

    Begrensinger i intranettet
    Som noen kanskje har fått med seg har vi innskrenket antall svar som kan skrives i intranettet. 
    Etter noen ivrige alvebetjenter hadde lagt inn så mange svar at NISSEN ikke orket å lese alle svarene, 
    så han seg nødt til å sette en begrensning på 1000 svar.

    Digitalisering
    Som del av nordpolar digitaliseringsstrategi har NPST i det siste jobbet med å digitalisere NISSENS verksted. 
    Der foregår alt av design og snekring for å lage gaver til snille barn, 
    men vi har misstanke om at vi har mistet et flagg der inne. 
    Kunne noen alvebetjenter tatt en titt og sett om de finner det?

    https://verksted.npst.no

    Julevurdering
    Det er nå SANNSYNLIG at det blir en GOD JUL.

På juleverkstedet finner vi en sinnsvak liste av bilder. Etter litt analysering, så kan man se at det er ikke bare forskjellige
bilder, det er en liste av ca 2340 bilder. Isteden for å lete etter det ene bildet av et flagg, så skriver jeg et program
som henter ut alle bilder det er kun ett av. Nedlasting av bilder gjør jeg enkelt og greit med firefox, "save as".
Når jeg nå har en liste med 3 bilder, er det enkelt. En sko, en pingvin og ett flagg. Flagget er løsningen.  

Flagg: PST{8798e1f0a271b09750a6531686fc621b}
{:.note}

Kode:
~~~python
# file: "solver.py"
# get the filenames
import hashlib
from os import listdir
from os.path import isfile, join

hashes = {""}
mypath = '.'
filenames = [f for f in listdir(mypath) if isfile(join(mypath, f))]

counts = {'': 0}
print(len(filenames))
for name in filenames:
    file = open(mypath + "/" + name, 'rb').read()
    hexdigest = hashlib.md5(file).hexdigest()
    if hexdigest not in counts:
        counts[hexdigest] = 1
    else:
        counts[hexdigest] = counts[hexdigest] + 1

for count in counts:
    if counts[count] == 1:
        print(count)
~~~

### Bedriftsspionasje (2)
    Våre analytikere mistenker at det også kan foregå spionasje mot Nissens verksted. 
    Kan du ta en ny titt og se om du finner noe muffens?

Tjah, kan jo være den teite pingvinen? Pingviner hører jo ikke til på Nordpolen. Jupp, det var det.  

Flagg: PST{b30b4add25b97721ebf0e7ad2eb26eb9}
{:.note}

## Luke 8
### 8. desember (10)
![strava.png](/assets/img/blog/2020-npst/8-strava.png)
    
    Lokalisering av isbjørn

    Den 4. desember dro Jule NISSEN og Rudolf RØDNESE på joggetur til postkontoret 
    for å hente juleønsker fra snille barn. 
    Etter en noe turbulent sledetur iførte de seg NPSTs nye treningsklær og jogget fra sledeplassen til postkontoret. 
    På turen tok Rudolf RØDNESE en liten omvei, hvor han observerte en mistenkelig isbjørn. 
    Uheldigvis greier ikke Rudolf RØDNISSE å huske hvor han befant seg da han observerte isbjørnen.

    Kan en alvebetjent se nærmere på dette, og rapportere tilbake lokasjonen til isbjørnen i intranettet?

    Rudolf RØDNESE vil helst ha lokasjonen til isbjørnen (så nøyaktig som mulig) 
    i uppercase, inklusive mellomrom, omgitt av PST{ og }.

    Eksempel: Julenissens verksted → PST{JULENISSENS VERKSTED}.

Etter en liten bomtur innom [steganografi](https://en.wikipedia.org/wiki/Steganography) på bildet, så er det på tide å ta en titt på 
[strava](https://www.strava.com/) siden det ser ut til at julenissen bruker strava.
Joda, både Rudolf Rødnese og Jule Nissen er på Strava, og de logger turene sine public. på den 4 desember ser det et til 
at rudolf faktisk tok en omvei via North Pole Expedition Museum (Google Maps). Det er faktisk en isbjørn foran der...  

Flagg: PST{NORTH POLE EXPEDITION MUSEUM}
{:.note}

### Spionaktivitet (15)
    Det har kommet tips om mulig spionaktivitet utført av sørpolare agenter på 
    Svalbard i tidsrommet Nissen og Rudolf var på løpetur.
    Kan du identifisere en agent?

Ved å bruke strava flyby, så kan vi se enda en teit pingvin (Pen Gwyn), som var ute på tur, fulgte etter Rudolf og Nissen
hele tiden.
På hans profil finner vi flagget.  

PST{69d26031ea5dbbeb56f22d9647f7c98e}
{:.note}

### Mystiske profiler (20)
    Våre analytikere mistenker at Pen Gwyn har rapportert hjem til sine kontakter på sydpolen.
    Klarer du å dekode noe av kommunikasjonen?

Her ble det et aldri så lite helvete på natten. Det bildet vi skulle ha fått for analyse, det hadde blitt fjernet, 
og noen begynte å trolle ved å lage enda flere falske stravaprofiler med fiktive flagg og fiktive gåter.
Å avholde konkurranser i et ikke-kontrollert miljø (f.eks. Strava) er gøy, men har potensiale til å bli en aldri så 
liten katastrofe for deltakerene.
Når jeg endelig fikk det korrekte bildet var det en annen sak.
Alt jeg har tilgjengelig av [stego-tools](https://github.com/DominicBreuker/stego-toolkit) ga meg nada.
Bakgrunnen i dette bildet har streker i forskjellig tykkelse. Med litt analyse kan man se at det er [morsekode](https://en.wikipedia.org/wiki/Morse_code),
en manuell dekoding av morse gir oss siste flagg.  

PST{e06531d19ff020a479520ef28c8d12c}
{:.note}
Profil bildet til Pen Gwyn inneholder også en gul lapp med informasjon `VII TF ⚙️`.
## Luke 9
    Ønskelistelekkasje
    NPST har funnet ut at ønskelistene befant seg på en minnepinne som en alvebetjent 
    hadde glemt igjen i Jule NISSENS reinsdyrslede. 
    Alveteknikere har overført filene på minnepinnen, og ønskelistene er nå forsvarlig oppbevart. 
    Alvebetjenten har blitt irettesatt for å ha oppbevart sensitiv informasjon på minnepenn.
    
    Fotoarkiv
    Takket være noen dyktige alvebetjenter ble det den 8. desember avdekket mystisk aktivitet fra en sydpolar agent. 
    Et bilde lastet opp av agenten har blitt sikret, og er nå tilgjengelig i NPSTs arkiv. 
    Det ryktes at dette kan komme til nytte når man forsøker å løse siste oppgave i gårsdagens luke.

Ingen oppgaver, men de har lagt ut det manglende bildet Luke 8. 😒

## Luke 10 - Vige vs. Nere (5)
    Sjakkmesterskap

    For tiden arrangeres polarmesterskap i sjakk på Nordpolen. 
    Flere av NPSTs ansatte bruker store deler av arbeidsdagen på å følge med på mesterskapet. 
    Da mange sitter i graderte nett uten mulighet til å strømme sjakk, 
    har NISSEN valgt å legge ved det siste partiet mellom to sørpolare spillere, Vige og Nere:
    PGN:

    1. a3 e6 2. b3 Ke7 3. Ra2 d5 4. h3 h6 5. f4 Qd6 6. Nc3 Qa6 7. Nf3 g5 8. Nb1 b5 9. d4 Qa5+ 10. Bd2 Kd6 11. g4 Bg7 
    12. Bc3 f5 13. Nh2 Ba6 14. Qd2 Bc8 15. Qc1 Qb6 16. Ba1 b4 17. Nd2 Ke7 18. Ndf3 Nd7 19. axb4 Ngf6 20. gxf5 Ne5 
    21. Ng4 c5 22. Nh4 Qd6 23. Nf2 Ng8 24. Bg2 a5 25. Ra4 Qd8 26. Be4 Qe8 27. Ra3 Kf7 28. b5 Qe7 29. O-O exf5 
    30. Nxf5 Ba6 31. Ng3 Qf6 32. Nd1 Ra7 33. Kg2 Nc4 34. bxa6 Qb6 35. Nb2 Nxa3 36. Nh5 Qxb3 37. Nxg7 Nb5 38. Nd1 Qe3 
    39. Rh1 Qa3 40. Re1 Rc7 41. Ne8 Kxe8 42. Qb1 Rhh7 43. dxc5 Rce7 44. Rg1 Re5 45. fxg5 Nf6 46. gxh6 Kf8 47. a7 Ng4 
    48. Kh1 Nc7 49. Bxd5 Qb4 50. Kg2 Qa3 51. c6 Qc5 52. Qb8+ Re8 53. Be5 Ne6 54. Nc3 Rf7 55. Qxe8+ Kxe8 56. Bf4 Rf6 
    57. Bg3 Kf7 58. Be1 1-0

    Kommentarer til partiet kan legges inn i intranettet.
    
Hm, første jeg tenker på er stegano og sjakk. Litt søk rundt omkring gir meg et lovende resultat: [Chess steganography](https://incoherency.co.uk/chess-steg/)
men alle mine bokstaver er "jumbled around". `HHL DJDWEDESKWCLXK u02s104y2s665t5v3w2619v6184su50t CGGXDAHTJTFMWH KEMIL` 
Heldigvis får jeg et kraftig hint av Roy Solberg før jeg graver meg for dypt ned i kaninhullet og begynner å analysere
biblioteket for å finne ut om de har modifisert det.

Sjakkspillerene heter Vige og Nere. Vigenere encryption, selvfølgelig. 
Siden vi vet teksten over skal være `PST KRØLLPARANTES` kan vi begynne med å putte inn PST i cyberchefs vigenere decoder.
Kjører man viegnere decoding med resultatet, får man nøkkelen.
De tre første bokstavene er `SPS`, educated guess gir oss nøkkelen `SPST` (kan også bruteforce hvis man vil).  
[Her er Cyberchef linken til løsningen](https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('SPST')&input=SEhMIERKRFdFREVTS1dDTFhLIHUwMnMxMDR5MnM2NjV0NXYzdzI2MTl2NjE4NHN1NTB0IENHR1hEQUhUSlRGTVdIIEtFTUlM).  

Flagg: PST{f02a104f2a665e5d3d2619d6184dc50a}
{:.note}
## Luke 11 - 1337 (5)
    Primære faktorer
    En av NPSTs primære oppgaver er å overvåke kommunikasjonen mellom personer 
    mistenkt for å utøve spionasje for sørpolarske aktører. 
    En alvebetjent har snappet opp en melding tiltenkt en spion. Grunnet flere faktorer trenger vi hjelp 
    med å finne ut av hva denne meldingen inneholder. Kan en alvebetjent se over meldingen og finne noen spor?

    Forsøk på smøring av alvebetjenter
    En betrodd alvebetjent har mottatt et parti med julebrus. Julebrusen var plassert utenfor NISSENS verksted 
    og adressert alvebetjenten. Det var ingen eksplisitte tegn til avsender, men mistanken er rettet mot Sydpolen 
    pga en sor pingvinfjær som ble funnnet i nærheten av partiet. I tråd med regelverk om smøring fra fremmed makt 
    har alvebetjent informert sin nærmeste leder Jule NISSEN og partiet med julebrus har blitt destruert. 
    Vi oppfordrer andre alvebetjenter til også å rapportere forsøk 
    der de mistenker fremmede makter forsøker å påvirke dem.

Meldingen er en rekke av 0 og 1. En binærrekke tydeligvis. En god del forsøk på å dekode den til ASCII
eller binærformat, morse osv gir ingen resultater. Men det finnes et mønster! 
Dette hintes til av oppgavens navn: `1337`. Hvis man lagrer meldingen i en tekstfil og åpner opp denne i en editor,
gjør bredden mindre og større, så vil man se et mønster tre fram. Eller du kan skrive et lite python program for å finne 
riktig linebreak.  

**Flagg: PST{LINEBREAK_IT_TILL_YOU_MAKE_IT!}**

     ███   ██  █████    █  █     ███ █   █ █████ ████  ████  █████   █   █   █       █ █████       █████ █ █     █           █   █  ███  █   █       █   █   █   █   █ █████       █ █████ █  █    
     █  █ █  █   █     █   █      █  ██  █ █     █   █ █   █ █      █ █  █  █        █   █           █   █ █     █           █   █ █   █ █   █       ██ ██  █ █  █  █  █           █   █   █   █   
     █  █ █      █     █   █      █  █ █ █ █     █   █ █   █ █     █   █ █ █         █   █           █   █ █     █            █ █  █   █ █   █       █ █ █ █   █ █ █   █           █   █   █   █   
     ███   ██    █    █    █      █  █ █ █ ████  ████  ████  ████  █   █ ██          █   █           █   █ █     █             █   █   █ █   █       █   █ █   █ ██    ████        █   █   █    █  
     █       █   █     █   █      █  █  ██ █     █   █ █ █   █     █████ █ █         █   █           █   █ █     █             █   █   █ █   █       █   █ █████ █ █   █           █   █   █   █   
     █    █  █   █     █   █      █  █   █ █     █   █ █  █  █     █   █ █  █        █   █           █   █ █     █             █   █   █ █   █       █   █ █   █ █  █  █           █   █       █   
     █     ██    █      █  █████ ███ █   █ █████ ████  █   █ █████ █   █ █   █ █████ █   █   █████   █   █ █████ █████ █████   █    ███   ███  █████ █   █ █   █ █   █ █████ █████ █   █   █  █    



~~~python
# file: "solver.py"
binStr = '0111000110011111000010010000011101000101111101111001111001111100010001000100000001011111000000011111010100000⏎
         10000000000010001001110010001000000010001000100010001011111000000010111110100100000100101001000100000100010000⏎
         00100110010100000100010100010100000010100100100000000100010000000000010001010000010000000000010001010001010001⏎
         00000001101100101001001001000000000001000100010001000010010100000010000010001000000100101010100000100010100010⏎
         10000010001010100000000010001000000000001000101000001000000000000101001000101000100000001010101000101010001000⏎
         00000000100010001000100001110001100001000010000100000010010101011110011110011110011110010001011000000000010001⏎
         00000000000100010100000100000000000001000100010100010000000100010100010110000111100000000100010001000010001000⏎
         00001000100000100010000001001001101000001000101010001000001111101010000000001000100000000000100010100000100000⏎
         00000000100010001010001000000010001011111010100010000000000010001000100010000100001001000100000100010000001001⏎
         00010100000100010100100100000100010100100000000100010000000000010001010000010000000000000100010001010001000000⏎
         01000101000101001001000000000001000100000001000010000011000010000001001111101110100010111110111100100010111110⏎
         10001010001011111010001000111110001000101111101111101111100010000111000111001111101000101000101000101111101111⏎
         101000100010010000'
print(len(binStr))
for i in range(0,7):
    line = binStr[i*191:i*191+191]
    for ch in line:
        if ch=='0':
            print(' ', end = "")
        else:
            print('█', end = "")
    print("")
~~~


## Luke 12 - Arbitrær kode (5)
    Evaluering av trusler
    NPST har oppdaget et endepunkt som er tilgjengelig på SPST sin offentlige nettside. 
    Det vites på dette tidspunktet ikke om dette endepunktet er tilgjengelig ved en feiltagelse, 
    eller om det er meningen at dette skal brukes av deres agenter. 
    En av NPSTs teknikere har påpekt at det ser ut til å være mulig å kjøre arbitrær kode via endepunktet. 
    Det er ønskelig at en alvebetjent undersøker dette endepunktet og rapporterer eventuelle flagg via intranettet.

    Url: https://api.spst.no/eval?eval=`<pre>${getFlag()}</pre>`

    Jule NISSEN er blitt syk
    Jule NISSEN er utilgjengelig de neste dagene grunnet sykdom. 
    Det mistenkes at sykdommen er influensa ettersom Jule NISSEN har sprøyteskrekk 
    og ikke tok vaksinen sammen med resten av NPST. 
    Ledelsen anbefaler alle alvebetjenter å praktisere god hygiene og benytte seg av antibak for å unngå videre smitte.

    Julevurdering
    Med bakgrunn i Jule NISSENS sykdom blir julevurderingen nedgradert. Det er nå mulig at det blir en GOD JUL.

Her får vi oppgitt en url, med to enorme hint om at her er det mulig å kjøre kode, og `eval=''`. 
Vi kan anta at det faktisk er en eval funksjon som kjøres, og node.js, et populært server-side språk har en slik funksjon.
[Burpsuite](https://portswigger.net/burp) er vår venn her, den gir oss full kontroll over hva som sendes til server,
uten at browseren krøller med hva vi prøver å sende. Ved å hente kildekoden til getFlag ser vi en kommentar om en encrypt
funksjon. Denne funksjonen henter et passord for en dato, et salt Natriumhydrogensulfat ($$ NaHSO_4 $$) og en tredje funksjon, decrypt.
Så, ved å se på passord funksjonen, ser vi at den kaller getSecretPasswordNumber med et tall som ser ut til å følge en
[Fibonacci følge](https://en.wikipedia.org/wiki/Fibonacci_number). Aller sist ser det ut til at den siste datoen de har brukt
er 2019-12-11.

Vi har nå alt vi trenger for å finne flagget. Vi kaller getSecretPasswordNumber med tallet 34, slik at vi får `password = passord-61`
Så kaller vi decrypt med passordet, saltet og det krypterte flagget. Voila!
~~~
https://api.spst.no//eval?eval=`<pre>${decrypt('passord-61','NaHSO4',getFlag())}</pre>`
~~~

Flagg: PST{24e592de8b20fe09938916d79b08854e}
{:.note}


~~~js
// file: "code.js"
function getFlag() {
  // Det er sikkert smartere å kryptere flagget først, og bare skrive inn det
  // krypterte resultatet her, enn å kryptere på serveren hver gang.
  // 11.12.19: Kryptert flagget nå. Vi kan sikkert slette encrypt-funksjonen?
  return "e5a8aadb885cd0db6c98140745daa3acf2d06edc17b08f1aff6daaca93017db9dc8d7ce7579214a92ca103129d0efcdd";
}
function encrypt(input) {
  // Bruk `decrypt` for å dekryptere

  const algorithm = "aes-192-cbc";
  // 06.12.19: husk å oppdatere denne hver dag!!!
  // 09.12.19: dette var sykt slitsomt. kan vi finne en bedre løsning?
  // 11.12.19: Krypteres permanent med dagens passord nå.
  // Denne funksjonen trengs vel ikke lenger?
  const password = getPassword("10.12.19");

  // 09.12.19: pepper er ikke et salt. Når vi på sikt krypterer utenfor serveren
  // burde vi oppdatere dette til noe mer vitenskapelig korrekt.
  // Natriumhydrogensulfat?
  // 11.12.19: Oppdatert med den kjemiske formelen ;)
  const key = crypto.scryptSync(password, formatSalt("pepper"), 24);

  const iv = Buffer.alloc(16, 0);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  
  let encrypted = cipher.update(input, "utf8", "hex");
  encrypted += cipher.final("hex");

  return encrypted;
}
function getPassword(date) {
  const passwords = {
    "06.12.19": "passord-" + getSecretPasswordNumber(3),
    "07.12.19": "passord-" + getSecretPasswordNumber(5),
    "08.12.19": "passord-" + getSecretPasswordNumber(8),
    "09.12.19": "passord-" + getSecretPasswordNumber(13),
    "10.12.19": "passord-" + getSecretPasswordNumber(21)
  };
  // 06.12.19: vi har ikke flere passord etter 10. Burde vurdere alternative
  // løsninger.
  return passwords[date] || `fant ikke passord for ${date}`;
}
function decrypt(password, salt, input) {
  const algorithm = "aes-192-cbc";
  
  const key = crypto.scryptSync(password, formatSalt(salt), 24);
  
  const iv = Buffer.alloc(16, 0);
  const decipher = crypto.createDecipheriv(algorithm, key, iv);
  
  let decrypted = decipher.update(input, 'hex','utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
function getSecretPasswordNumber(n) {
  return Math.PI.toFixed(48).toString().split(".")[1].slice(n, n+2);
}
~~~

## Luke 13 - Token effort (5)
    Nøkkel på avveie
    NPST sin driftsansvarlig er dessverre sykemeldt, og jobben hans er midlertidig overtatt av en alv 
    ved pakkeavdelingen til NISSENS verksted. Pakkealven har lagt merke til noe muffens som foregår på Jule NISSENS 
    liste over snille og slemme barn. Pakkealven, som ikke er veldig teknisk anlagt, 
    har behov for hjelp til å finne ut av hvem som snoker i sakene. Han har ikke så mange spor å gi, 
    men han har pakket ned bevismaterialet godt for å unngå bevisforspillelse.

    Alvebetjenten som overbrakte informasjonspakken til Jule NISSEN har dessverre glemt passordet som ble benyttet. 
    Hans små grå begynner dessverre å gå ut på dato, men han mente at passordet var nevnt som en av teknologiene 
    i annonsen fra vår samarbeidende tjeneste, PST.

    FILER:
    https://kalender.npst.no/logger.7z.001
    https://kalender.npst.no/logger.7z.002
    https://kalender.npst.no/logger.7z.003
    https://kalender.npst.no/logger.7z.004

    Kan en alvebetjent finne ut av hvilken API-nøkkel som er kompromittert, og legge det inn i intranettet på 
    formen PST{<den-kompromitterte-nøkkelen>}?

    Unngå lekkasje
    NISSEN vil også oppfordre alle ansatte til å beskytte informasjon bedre ved bruk av kryptering. 
    Sørg også for å bruke gode passord.
    
min kode for denne oppgave er på en annen maskin, derfor har jeg ikke inkludert koden jeg faktisk brukte på denne.
{:.note}

Aller først må vi finne nøkkelen for å dekryptere [7zip](https://www.7-zip.org/) arkivet. 
Her hintes det til en teknologi som er nevnt i en finn annonse for PST.
For å gjøre det enkelt for meg bruker jeg [cewl](https://github.com/digininja/CeWL/) for å lage en ordliste basert på finn annonsen.
Så bruker jeg [7zip-JTR Decrypt Script](https://gist.github.com/bcoles/421cc413d07cd9ba7855), 
og finner passordet `Graylog`. (Of course, misset hintet "små grå").
Arkivet inneholder enda en zip fil, som er enda en zip-fil osv. Uten et hint fra Roy Solberg om å passe på å få 
pakket ut absolutt alle loggfiler, hadde jeg nok ikke løst denne (jeg manglet den viktigste).
Ved slike oppgaver så er det IKKE lurt å gjøre dette manuelt. Så når jeg til slutt får laget meg et lite
bash script som rekursivt pakker ut alt som pakkes ut kan, ender vi opp med fem loggfiler.
Analysering av tokens viser at det er 1000 unike tokens som brukes. Det er ikke noe mønster på user agents, ei heller noe
access-pattern mønster, IP adressene er interne, så man kan ikke se om en av de kommer fra sydpolen f.eks.
ved å hente ut alle tokens for en dag, og lagre de i en liste, og så kjøre EXCEPT/minus for alle dager i alle kombinasjoner,
så kan man se at det er en dag som peker seg ut med at det er et extra token den dagen. Dette viste seg å
være Pen Gwyn (pingvinjævel) igjen.  

Flagg: PST{67e49727affdee991ec58180ee657b28}
{:.note}
## Luke 14 - Lekket data (5)
    Ukens ansatt
    Jule NISSEN gratulerer joey som ukens ansatt, og takker for at han har skapt et fantastisk arbeidsmiljø 
    tross den korte tiden han har jobbet med oss. NISSENS sekretariat vil ta kontakt via elektronisk post førstkommende virkedag. 
    NISSEN håper gårsdagens prøvelser gikk bra.

    Lekkasje fra SPST
    NPST har gjennom en temmelig hemmelig kilde fått tilsendt et dokument som stammer fra SPST sitt interne nettverk. 
    Kilden sier at filen kommer fra en harddisk som ikke ble makulert da datamaskinen ble kastet. 
    Harddisken var markert med U+2295/U+22BB. Utover dette har vi ingen andre spor om krypteringen som er brukt. 
    Kan en alvebetjent se om det finnes noen mulighet for å hente ut dataen her?
    
    Det krypterte dokumentet følger etter en viktig beskjed om julebordet!
    
    Julebord
    
    Det nærmer seg julebord for våre alvebetjenter! Det blir konkurranse i hammerkasting 
    og lengste hammerdistanse vil vinne en premie! Juleskinke vil også bli servert etter en forrett av julegrøt. 
    Mandel er så klart inkludert!
    
    Melding GSU/MWR0QXpUW1p7TGZvITUXJgQaHiUbCRdONxcgEQwAahgDEU4LIgcxYyclGEwODysBPw8MVCcUAgILZRoxF0kdaiY8NjplGjUXSRKvARh⏎
    FAyAWdBYME2oGiUUGJAB0NjknHlUKChwgBjURHVQlBQkXDzYYOwsMBmoFiUUgKgAwFQYYKwcfDgtlBjEXGx0+Gh4MGyhSPUUNESRVHwwdMRd0EQAQLx⏎
    tCRTsrFjEXSRKyGQsAHGUXOkUGBDoGGQgDIAA9Cw5UKwNMAQc2ATFFBgQvBw0WBCocMQsMWkA5CQELNxc6RQgCajs8NjppUh4QBRFqOwUWHSAceEUBE⏎
    TgQGBELN1I7CB0VJgFMFgEoUh8pKCEZVQQEHGUEshcdVCUXBgAFMVIyChtULBkJFwtlEyJFDR05BglFATUXJgQaHiUbCQsLa3heSEkgIwcfAQ8iUmRW⏎
    R0V4W11cTi4eNRcdEWoDiRcLZRYtDh0dLRBMDQ8mGTEXDFSvVQgABTcLJBEMBi9VlAsdLhc4DBoALwdMFgEoUiIEG1Q5EAIBGmUGPQlJPwY0OTZAZSQ⏎
    9RQMbKBcJF04jHSYRGhU+AUwNDzcWIEUEES5ViUUPKxM4HBoROBBMDAArGjsJDRE+VQVFCiwBJwBJjCQGBwACLAEgAAcRZH9BRSErATAEDlR6QUJUXG⏎
    tDbUUPGzgQGAoFZRc6RQgTLxsYRQsxUiIABRgzHgcAGmUBJAQHHSQSHwoeNRYmBA5UJxoYRSUJMwE2RVQmEAgAHCAcdAQfVAQlPzFAZTkYJDwnagMNF⏎
    04wBjFFGZFqHwMCCSAGIRdJACMZTBUBNgY/CgcAJQcJEU4oFzBFDBpqFBpFBiQcJ0UbESMbHwEXN1x0IB0ALwdMBAIxUrFFDYwnGAlFHaBSNgAHDT4B⏎
    CRZONR0nEQIbJAEDFwsxUjIKG1SvVR8AACEXdJ0HByEQAAwdMRcmRQ8GK1VOFgAsHjgAS1QoFB4LQGU2MRFJAiMHBwAcZQE7CEkbJ1UiNT0RUj0OAhF⏎
    qHQ0XTisdMQtJFSQQABYLZR05RQgAahEJERogUjIKGxEtkB5FHqBSMAAbETlVGAAcNxsgChsdL1tmSE4DADEBCBNqRVpLX3dcZVxJB69VHw4HIwYxEU⏎
    k6GiY4RRosHnQAB1QkDEwOHDwCIAAbHSQSHwQCIh0mDB0ZL1UfCgNlFT4KGxAvVQ0RTjMbdAwCHy9VAAAAIhcmRQ+ROFUYDAIiEzoCSQAjGUwMACsaO⏎
    wkNET5VBUWWKwE/AAUdOQEJCwtrUgKAGxFqHQ0GBSAAMUUDGygXCRdONZd0FhkGLxsLRQgqAHSASRIjGwIATiAcdBMMHWoHGQsKMVIwAAcaL1UCHAtl⏎
    GSYcGQAvBwULCSAcem9EVAaNHgEPIlJkUkdFeFtdXE4sHDIMBQA4EB4RC2UXOkUIEy8bGEUlCTMBNkkHIwEYRRggAD8WHREuVQMCTjcdIAAdVCwaHkW⏎
    LZQE1BwYALwcJRQQwHjEDBgYoEB4ACiAeJwAHEWZVAQAAZQQxFwIHPhAIABplBDUXSRUmGQkXCyEXdBMMGC4cC0UcKgYxEQxaajMDFwagAjELHRgjEh⏎
    oMHWUUOxcaHSQeCRdOIRcgEQxULBoeBws3FzAABQcvGwlFACoZdBEAGGoUGEUMIBQ7CQIaIxsLAABlEDgMG1Q/BgUOBSAAdBWMVCUYTAELMVI2CQAGa⏎
    hACRQkqFnQPHBhkf0FFOiwAJwEIE2pEXEtfd1xlXEkYLwMJFxogUjELSRUtEAIRTiAGdBUIBj4cTA8bKRc2FxwHagEFCU4gHHQVBgAvGx8MCykedA4A⏎
    GC4QTAxOCyIHMUdUDhAYEQtlEDgASRMgGh4RTiAGIAAbVCsBTBOLN1IgDA0YIxIJFwtlGT0JDRFqBYlFByscJwwNESRVCgwFLlInFQgGIRACRQsxBjE⏎
    XSRU+VQQEAGUQOABJGzoFCAQJIAZ6RT8dahAeRRs2Gz8XDFQ6kEwKA2UBOZ0bHSQSHwMBNwGsDgwAah0NF04jBzoCDAY+VQkRGiAAJwoEVCsZGgAAZQ⏎
    ExF0kBPlUYDAJll3QNCFQnEAABGmUWMREdEWocAhELNxwgS2NZaiEDFx0hEzNFWEZkRF5LX3xSJwsIBDoQGEUYLFI7FRlULxtMCAspFj0LDlQlGEwEG⏎
    mU5GCQ8J2odDRdOJx49ER1UOQwHS04DHSYNjAQvGxgJByIEPRZJAiMZTAELMQYxRQsRPgxMBBplBDEXAgc+EAgAGmUGPQlJPwY0OTZOLRMiCwwGahAC⏎
    AQ9lHjELDgYvVQ4EBWUCsUUPGzgXCRcLIRc4FgwaL1UfDAAgUiAMBVQgAABLZE94EAQdFSsbCxcLNVI5Ch1UGSU/MWQWBjsXDRU+FAEEHS4bOgAHVDy⏎
    QHkUGJAB0BwUdPgFMEBo2EyARSRIlB0wAGmUWNREIFSQSHgAeZRt0CwgAPlUfCgNlATEXSQE+VRgMAmWXdBYdFScYCUUINxN0AAdULQcZFR4gUjIXAA⏎
    IjGQAMCSBSJwoEVCAaDgcLN1IyChtUBCU/MUBlJD1FCxE4VQ0JAiBSNQsaFT4BCUUbKxwzgEmRahcJCxcxBjFFGhEtVQ0TTjEYMQsMBz4QAgBOIwA1R⏎
    RoAJQcIBBokHzUWAh0kEAJFCDcXOUUdHSZVDQEDLBw9Fh0GKwYGCgA2BjEEBBE+VQQEHGUUsREdVDkcBxcLMVI4nRoaIxsLAABreF5vPB8vGx9FDysB⏎
    NREdfhoQAkUpMgs6RQ8bOFUZEQMgAD8AHVQ/GwgAHCYdIgAbVCsHDgAHIVI5Ch1UBCU/MUBPeF4xCB8hVRgMAmUTOAkMVCwaHkUbMR8xFwIRPlUNFww⏎
    gGzBFDREkVR8MHTEXdBEAEC8bQm89LxcyRTokGSFARSUgGycAGwQjGwsTBysXOm9jOghPZi4PKwY9CwwaagMFCU4zlCYASQc+EAICGmUWMUUHETkBCU⏎
    UaKlIwBA4RJBBMABoxFyZFAhgrEgkXTjWXdAGMBiYcC0UFMxM4DB0RPlUcgE4jGycODBpkVToMTikXIAAbVC8BGAAcZRc6RQcNahkJEws3EzoBkQZqB⏎
    gMITi4TOkUFETwQHgBOJxcwFwxUOJAaBBwgAHpvOSceDg1WV3dLYlVdRntMXVZfc0dlXF5MfkAKVlonFGEBWBVyCA==
    
Ah, denne oppgaven er gøy. Noe er kryptert, og vi har fått et hint `U+2295/U+22BB`. Disse er unicode karakterer ⊕/⊻,
som er symboler brukt for [XOR](https://en.wikipedia.org/wiki/Exclusive_or). Så det er høyst sannsynlig en XOR encryption
som er brukt med en nøkkel av noe slag. 
Jeg husker ikke helt hvordan jeg faktisk kom fram til løsningen her, jeg jobbet en del med bruteforcing og søk etter 
korrekt lengde på nøkkel. Uansett, [xortool](https://github.com/hellman/xortool) er ganske fantastisk for å knekke xor encryption.
Aller først må vi hente ut de faktiske bytes (strengen er base 64 encoded).
Lagre strengen som en tekstfil `melding.b64`, deretter converter til bytes i bash.
Ser ut til at vi har fått en delvis match, teksten var ikke helt korrekt. enkelt å gjette seg fram til den korrekte 
nøkkelen `JulenErTeit`, og vi finner flagget.
[Cyberchef oppskrift](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',false)XOR(%7B'option':'UTF8','string':'JulenErTeit'%7D,'Standard',false)&input=R1NVL01XUjBRWHBVVzFwN1RHWnZJVFVYSmdRYUhpVWJDUmRPTnhjZ0VRd0FhaGdERVU0TElnY3hZeWNsR0V3T0R5c0JQdzhNVkNjVUFnSUxaUm94RjBrZGFpWThOanBsR2pVWFNSS3ZBUmhGQXlBV2RCWU1FMm9HaVVVR0pBQjBOamtuSGxVS0Nod2dCalVSSFZRbEJRa1hEellZT3dzTUJtb0ZpVVVnS2dBd0ZRWVlLd2NmRGd0bEJqRVhHeDArR2g0TUd5aFNQVVVORVNSVkh3d2RNUmQwRVFBUUx4dENSVHNyRmpFWFNSS3lHUXNBSEdVWE9rVUdCRG9HR1FnRElBQTlDdzVVS3dOTUFRYzJBVEZGQmdRdkJ3MFdCQ29jTVFzTVdrQTVDUUVMTnhjNlJRZ0NhanM4TmpwcFVoNFFCUkZxT3dVV0hTQWNlRVVCRVRnUUdCRUxOMUk3Q0IwVkpnRk1GZ0VvVWg4cEtDRVpWUVFFSEdVRXNoY2RWQ1VYQmdBRk1WSXlDaHRVTEJrSkZ3dGxFeUpGRFIwNUJnbEZBVFVYSmdRYUhpVWJDUXNMYTNoZVNFa2dJd2NmQVE4aVVtUldSMFY0VzExY1RpNGVOUmNkRVdvRGlSY0xaUll0RGgwZExSQk1EUThtR1RFWERGU3ZWUWdBQlRjTEpCRU1CaTlWbEFzZExoYzREQm9BTHdkTUZnRW9VaUlFRzFRNUVBSUJHbVVHUFFsSlB3WTBPVFpBWlNROVJRTWJLQmNKRjA0akhTWVJHaFUrQVV3TkR6Y1dJRVVFRVM1VmlVVVBLeE00SEJvUk9CQk1EQUFyR2pzSkRSRStWUVZGQ2l3Qkp3QkpqQ1FHQndBQ0xBRWdBQWNSWkg5QlJTRXJBVEFFRGxSNlFVSlVYR3REYlVVUEd6Z1FHQW9GWlJjNlJRZ1RMeHNZUlFzeFVpSUFCUmd6SGdjQUdtVUJKQVFISFNRU0h3b2VOUlltQkE1VUp4b1lSU1VKTXdFMlJWUW1FQWdBSENBY2RBUWZWQVFsUHpGQVpUa1lKRHduYWdNTkYwNHdCakZGR1pGcUh3TUNDU0FHSVJkSkFDTVpUQlVCTmdZL0NnY0FKUWNKRVU0b0Z6QkZEQnBxRkJwRkJpUWNKMFViRVNNYkh3RVhOMXgwSUIwQUx3ZE1CQUl4VXJGRkRZd25HQWxGSGFCU05nQUhEVDRCQ1JaT05SMG5FUUliSkFFREZ3c3hVaklLRzFTdlZSOEFBQ0VYZEowSEJ5RVFBQXdkTVJjbVJROEdLMVZPRmdBc0hqZ0FTMVFvRkI0TFFHVTJNUkZKQWlNSEJ3QWNaUUU3Q0VrYkoxVWlOVDBSVWowT0FoRnFIUTBYVGlzZE1RdEpGU1FRQUJZTFpSMDVSUWdBYWhFSkVSb2dVaklLR3hFdGtCNUZIcUJTTUFBYkVUbFZHQUFjTnhzZ0Noc2RMMXRtU0U0REFERUJDQk5xUlZwTFgzZGNaVnhKQjY5Vkh3NEhJd1l4RVVrNkdpWTRSUm9zSG5RQUIxUWtERXdPSER3Q0lBQWJIU1FTSHdRQ0loMG1EQjBaTDFVZkNnTmxGVDRLR3hBdlZRMFJUak1iZEF3Q0h5OVZBQUFBSWhjbVJRK1JPRlVZREFJaUV6b0NTUUFqR1V3TUFDc2FPd2tORVQ1VkJVV1dLd0UvQUFVZE9RRUpDd3RyVWdLQUd4RnFIUTBHQlNBQU1VVURHeWdYQ1JkT05aZDBGaGtHTHhzTFJRZ3FBSFNBU1JJakd3SUFUaUFjZEJNTUhXb0hHUXNLTVZJd0FBY2FMMVVDSEF0bEdTWWNHUUF2QndVTENTQWNlbTlFVkFhTkhnRVBJbEprVWtkRmVGdGRYRTRzSERJTUJRQTRFQjRSQzJVWE9rVUlFeThiR0VVbENUTUJOa2tISXdFWVJSZ2dBRDhXSFJFdVZRTUNUamNkSUFBZFZDd2FIa1dMWlFFMUJ3WUFMd2NKUlFRd0hqRURCZ1lvRUI0QUNpQWVKd0FIRVdaVkFRQUFaUVF4RndJSFBoQUlBQnBsQkRVWFNSVW1HUWtYQ3lFWGRCTU1HQzRjQzBVY0tnWXhFUXhhYWpNREZ3YWdBakVMSFJnakVob01IV1VVT3hjYUhTUWVDUmRPSVJjZ0VReFVMQm9lQndzM0Z6QUFCUWN2R3dsRkFDb1pkQkVBR0dvVUdFVU1JQlE3Q1FJYUl4c0xBQUJsRURnTUcxUS9CZ1VPQlNBQWRCV01WQ1VZVEFFTE1WSTJDUUFHYWhBQ1JRa3FGblFQSEJoa2YwRkZPaXdBSndFSUUycEVYRXRmZDF4bFhFa1lMd01KRnhvZ1VqRUxTUlV0RUFJUlRpQUdkQlVJQmo0Y1RBOGJLUmMyRnh3SGFnRUZDVTRnSEhRVkJnQXZHeDhNQ3lrZWRBNEFHQzRRVEF4T0N5SUhNVWRVRGhBWUVRdGxFRGdBU1JNZ0doNFJUaUFHSUFBYlZDc0JUQk9MTjFJZ0RBMFlJeElKRnd0bEdUMEpEUkZxQllsRkJ5c2NKd3dORVNSVkNnd0ZMbEluRlFnR0lSQUNSUXN4QmpFWFNSVStWUVFFQUdVUU9BQkpHem9GQ0FRSklBWjZSVDhkYWhBZVJSczJHejhYREZRNmtFd0tBMlVCT1owYkhTUVNId01CTndHc0Rnd0FhaDBORjA0akJ6b0NEQVkrVlFrUkdpQUFKd29FVkNzWkdnQUFaUUV4RjBrQlBsVVlEQUpsbDNRTkNGUW5FQUFCR21VV01SRWRFV29jQWhFTE54d2dTMk5aYWlFREZ4MGhFek5GV0Vaa1JGNUxYM3hTSndzSUJEb1FHRVVZTEZJN0ZSbFVMeHRNQ0FzcEZqMExEbFFsR0V3RUdtVTVHQ1E4SjJvZERSZE9KeDQ5RVIxVU9Rd0hTMDRESFNZTmpBUXZHeGdKQnlJRVBSWkpBaU1aVEFFTE1RWXhSUXNSUGd4TUJCcGxCREVYQWdjK0VBZ0FHbVVHUFFsSlB3WTBPVFpPTFJNaUN3d0dhaEFDQVE5bEhqRUxEZ1l2VlE0RUJXVUNzVVVQR3pnWENSY0xJUmM0Rmd3YUwxVWZEQUFnVWlBTUJWUWdBQUJMWkU5NEVBUWRGU3NiQ3hjTE5WSTVDaDFVR1NVL01XUVdCanNYRFJVK0ZBRUVIUzRiT2dBSFZEeVFIa1VHSkFCMEJ3VWRQZ0ZNRUJvMkV5QVJTUklsQjB3QUdtVVdOUkVJRlNRU0hnQWVaUnQwQ3dnQVBsVWZDZ05sQVRFWFNRRStWUmdNQW1XWGRCWWRGU2NZQ1VVSU54TjBBQWRVTFFjWkZSNGdVaklYQUFJakdRQU1DU0JTSndvRVZDQWFEZ2NMTjFJeUNodFVCQ1UvTVVCbEpEMUZDeEU0VlEwSkFpQlNOUXNhRlQ0QkNVVWJLeHd6Z0VtUmFoY0pDeGN4QmpGRkdoRXRWUTBUVGpFWU1Rc01CejRRQWdCT0l3QTFSUm9BSlFjSUJCb2tIelVXQWgwa0VBSkZDRGNYT1VVZEhTWlZEUUVETEJ3OUZoMEdLd1lHQ2dBMkJqRUVCQkUrVlFRRUhHVVVzUkVkVkRrY0J4Y0xNVkk0blJvYUl4c0xBQUJyZUY1dlBCOHZHeDlGRHlzQk5SRWRmaG9RQWtVcE1nczZSUThiT0ZVWkVRTWdBRDhBSFZRL0d3Z0FIQ1lkSWdBYlZDc0hEZ0FISVZJNUNoMVVCQ1UvTVVCUGVGNHhDQjhoVlJnTUFtVVRPQWtNVkN3YUhrVWJNUjh4RndJUlBsVU5Gd3dnR3pCRkRSRWtWUjhNSFRFWGRCRUFFQzhiUW04OUx4Y3lSVG9rR1NGQVJTVWdHeWNBR3dRakd3c1RCeXNYT205ak9naFBaaTRQS3dZOUN3d2FhZ01GQ1U0emxDWUFTUWMrRUFJQ0dtVVdNVVVIRVRrQkNVVWFLbEl3QkE0UkpCQk1BQm94RnlaRkFoZ3JFZ2tYVGpXWGRBR01CaVljQzBVRk14TTREQjBSUGxVY2dFNGpHeWNPREJwa1ZUb01UaWtYSUFBYlZDOEJHQUFjWlJjNlJRY05haGtKRXdzM0V6b0JrUVpxQmdNSVRpNFRPa1VGRVR3UUhnQk9KeGN3Rnd4VU9KQWFCQndnQUhwdk9TY2VEZzFXVjNkTFlsVmRSbnRNWFZaZmMwZGxYRjVNZmtBS1Zsb25GR0VCV0JWeUNBPT0)  

Flagg: PST{a392960421913165197845f34bf5d1a8}
{:.note}
Kode:
~~~bash
$ cat melding.b64 | base64 -d > melding.bin
$ xortool melding.bin
The most probable key lengths:
   1:   10.6%
   5:   9.6%
   9:   8.8%
  11:   24.4%
  18:   5.8%
  22:   14.4%
  25:   4.6%
  33:   9.7%
  44:   6.9%
  55:   5.4%
Key-length can be 3*n
Key-length can be 4*n
Most possible char is needed to guess the key!
$ xortool -l 11 -c 20 melding.bin
1 possible key(s) of length 11:
'Ju)enEr\x11eit
Found 1 plaintexts with 95%+ valid characters
See files filename-key.csv, filename-char_used-perc_valid.csv
~~~

## Luke 15 
### 15. desember (10)
    Beslag av minnepenn
    NPST har i all hemmelighet tatt beslag i en minnepenn som tilhører en sydpolarsk aktør ved navn Pen Gwyn. 
    Minnepennen ser i første øyekast ut til å være privat og inneholde feriebilder, 
    men det er også en kryptert zip fil lagret på den. 
    NPST trenger tilgang til denne zip filen og søker umiddelbar hjelp 
    fra alvebetjentene for å finne passordet. Merk: Passordet ønskes innlevert i klartekst på intranettet!
    
    Eksempel: Hunter2 -> PST{Hunter2}
    
    Jule NISSENS sykdomsforløp
    Jule NISSEN ser ut til å være på bedringens vei allerede 
    og han vil mest sannsynlig være tilbake på jobb i god tid før jul.
    
    Julevurdering
    Med bakgrunn i endringen av Jule NISSENS sykdom blir julevurderingen oppgradert. 
    Det er nå sannsynlig at det blir en GOD JUL.
    
Ooo. En minnepinne. Ok, la oss se hva vi har å jobbe med.
~~~bash
$ file spst_minnepinne.dd
spst_minnepinne.dd: DOS/MBR boot sector MS-MBR Windows 7 english at offset 0x163 
"Invalid partition table" at offset 0x17b 
"Error loading operating system" at offset 0x19a 
"Missing operating system", disk signature 0x75be3a56; 
partition 1 : ID=0x7, start-CHS (0x0,32,33), end-CHS (0x82,170,40), startsector 2048, 2097152 sectors
~~~
Hmm, vel kanskje vi bare skal mounte den for å se hva den inneholder?
~~~bash
$ losetup --partscan --find --show spst_minnepinne.dd
/dev/loop0
$ cd /media/root/SPST/
$ ls -al
total 5552
drwxrwxrwx  1 root root    4096 Nov 29 16:22  .
drwxr-x---+ 3 root root    4096 Jan  5 15:58  ..
-rwxrwxrwx  1 root root 5676023 Nov 29 16:22  feriebilder.zip
drwxrwxrwx  1 root root       0 Nov 29 16:21 'System Volume Information'
$ cd System\ Volume\ Information/
$ ls -al
total 5
drwxrwxrwx 1 root root    0 Nov 29 16:21 .
drwxrwxrwx 1 root root 4096 Nov 29 16:22 ..
-rwxrwxrwx 1 root root   76 Nov 29 16:21 IndexerVolumeGuid
-rwxrwxrwx 1 root root   12 Nov 29 16:21 WPSettings.dat
$ cat IndexerVolumeGuid 
{6BA8F788-A856-4200-A576-2F97CDF4A269}
$ cat WPSettings.dat 

. 
~~~
Så, der har vi feriebildene, og de er passordbeskyttet. Det er alt som vises på disken, ikke no gøy i andre filer. 
Da må passordet være skjult en plass, vi prøver med en `strings` på dd filen, og der kommer faktisk et password ut di gitt, base 64 encoded. 
"Et kjempelangt passord som aldri vil kunne gjettes av NPST! :)"  

Flagg: PST{Et kjempelangt passord som aldri vil kunne gjettes av NPST! :)}
{:.note}

Ett av bildene har nok en gul lapp med informasjon `VII TW ⚙️`.
### Alternativer (15)
    NPST har mottatt informasjon om at minnepennen var tiltenkt en nordpolar kilde ført av en sydpolar kildefører. 
    Kilden og kildeføreren skal ikke ha noe direkte kjennskap til hverandre, 
    men kommuniserer på en forhåndsavtalt måte. 
    Våre alveteknikere trenger en innføring i hvordan de skjuler dataen sin. Kan du hjelpe dem?
    
    Flagget er PST{md5(navnet på stedet du fant passordet i forrige trinn (ett tegn))}
    
    eks: echo -ne "?" | md5sum d1457b72c3fb323a2671125aef3eab5d -
    
    Flagget blir da PST{d1457b72c3fb323a2671125aef3eab5d}

Hrmph. Så vi trenger å finne ut hvordan de lagrer informasjonen. Ok, opp med "the big gun", [Sleuth kit - Autopsy](https://www.sleuthkit.org/autopsy/).
litt snusing rundt der eksponerer informasjonen er lagret i en `_`. Dette er en feature man finner i NTFS - [Alternate Data Streams](https://blogs.technet.microsoft.com/askcore/2013/03/24/alternate-data-streams-in-ntfs/).
![Autopsy sleuthkit](/assets/img/blog/2020-npst/15-autopsy.png)
Følger vi oppskriften over får vi flagget. 
~~~bash
echo -ne "_" | md5sum
b14a7b8059d9c055954c92674ce60032
~~~

Flagg: PST{b14a7b8059d9c055954c92674ce60032}
{:.note}
### Feriebilder (20)
    Bildene ser jo ut til å være helt normale sydpolare feriebilder, 
    men vi vet jo at sydpolare spioner liker å gjemme data inni andre ting. 
    Kan du hente ut et flagg fra ett av bildene?

![PST{md5(red_herring)}](/assets/img/blog/2020-npst/15_Blue_0.png)
Dette må jo bare være en stego oppgave, og jeg hadde rett. Ved å bruke [stegoveritas](https://github.com/bannsec/stegoVeritas) på 
`varm dag på stranda.png`, så dukker det opp informasjon i [LSB](https://itnext.io/steganography-101-lsb-introduction-with-python-4c4803e08041) i Blå og Grønn kanalen.
PST{md5(red_herring)}. Noe mer informasjon klarer jeg ikke å finne, men det ser jo ut til at kan spiser fisk på ett av bildene, 
vedder på at det er en red herring. Rød er den iallefall, og md5 av filen er det korrekte flagget.

~~~bash
$ md5sum måltid.png 
07385aacc9264738cd7c32e76f3b81a5  måltid.png
~~~

Flagg: PST{07385aacc9264738cd7c32e76f3b81a5}
{:.note}
## Luke 16
    Opptrapping av fremmed rekruttering
    SPST har trappet opp sine forsøk på å rekruttere informanter i både NPST og i den nordpolare alvebefolkningen. 
    NPST ber derfor alle alvebetjenter om å være oppmerksomme på tegn 
    på at de eller andre blir utsatt for påvirkningsoperasjoner. 
    Slike tegn kan for eksempel være at partier med pepperkaker eller julebrus blir levert til din adresse. 
    Dersom du opplever situasjoner der du mistenker at deg selv eller andre blir forsøkt rekruttert, 
    ta kontakt med din lokale sikkerhetsavdeling.
    
    Vi ber også alvebetjenter om å være forsiktige med å dele intern informasjon. 
    Selv om informasjonen ikke nødvendigvis er gradert temmelig hemmelig 
    så kan den gi et innblikk i hvordan NPST jobber. 
    Dette kan gjøre fremmede makters påvirkningskampanjer mer effektive, 
    og øke sjansene for velykket sabotasje mot nordpolare interesser. 
    Vær derfor ekstra påpasselig når du snakker med andre om arbeidet ditt i NPST 
    og sørg for at du ikke deler informasjon unødig. 
    Husk også at fremmede makter kan forsøke å overhøre samtaler, 
    så vær forsiktig dersom du observerer pingviner i området.
    
    Du snakker. Hvem lytter?
    
    Falske nyheter og julevurdering
    En artikkel om at julen er avlyst har spredd seg på sosiale medier. Med bakgrunn i disse falske nyhetene 
    nedjusteres sannsynligheten for at det blir en god jul. Det er nå mulig at det blir en GOD JUL.
    
Ingen oppgaver denne dagen.

## Luke 17 - Passorddatabase (5)
    Passorddatabase
    I tråd med Jule NISSENS anmodninger om at en bør beskytte informasjon bedre har alvene i kantinen tatt i bruk 
    en hjemmesnekret passorddatabase. Problemet er bare at de dessverre har glemt passordet til denne. 
    Det er kritisk at kantinealvene får tilgang til passordet som er lagret i denne databasen så fort som mulig, 
    ellers så vil de ikke kunne åpne matlageret og lage julegrøt til i morgen! 
    Kan en alvebetjent ta på seg oppgaven med å finne passordet til matlageret?
    
    FIL: https://kalender.npst.no/p2w
    
    Oppdatering av Jule NISSENS sykdomsforløp
    Jule NISSEN er frisk og rask og vil være tilbake i full jobb i morgen. 
    Det vil forståelig nok være litt tregere behandling av listen over snille og slemme barn mens Jule NISSEN 
    tar igjen det utestående arbeidet. Vi ber om at alle tar hensyn til dette.
    
Hm, hva er dette? la oss se hva vi har å jobbe med.
~~~bash
file p2w
p2w: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, 
interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, 
BuildID[sha1]=758ae2fdb3384d82e1c697e0977354612bce94bf, not stripped
$ chmod +x p2w
$ ./p2w
Passord: hei
Feil, pigg av!
~~~
Fantastisk. Pigg av igjen. Oh well, la oss ta en titt på om den gjør noe gøy vha [ltrace](https://en.wikipedia.org/wiki/Ltrace).
Ørrr, tror jeg ser passordet på høyresiden av strcmp(), la meg prøve... yep, it works.
~~~bash
$ ltrace ./p2w
printf("Passord: ")
fgets(Passord: hei 
"hei\n", 33, 0x7fbafec5ea00)
strcmp("hei\n", "625b2055fe2dcda72e418940599b881d"...)
puts("Feil, pigg av!"Feil, pigg av!)
+++ exited (status 0) +++
$ ltrace ./p2w
printf("Passord: ")                
fgets(Passord: 625b2055fe2dcda72e418940599b881d
"625b2055fe2dcda72e418940599b881d"..., 33, 0x7f19021b2a00)
strcmp("625b2055fe2dcda72e418940599b881d"..., "625b2055fe2dcda72e418940599b881d"...)
puts("PST{f3ad88918fd18414cc773271f586"...PST{f3ad88918fd18414cc773271f586f6a9})
+++ exited (status 0) +++

~~~
Så... der var svaret ja. rimelig enkel begynner reverserings oppgave uten at man har behov for noe tungt ala [IDA](https://www.hex-rays.com/products/ida/), 
[Radare2](https://rada.re/n/radare2.html), [Ghidra](https://ghidra-sre.org/) etc.  

Flagg: PST{f3ad88918fd18414cc773271f586f6a9}
{:.note}

## Luke 18
    Mistenkelig ponni
    En ponni ble oppdaget i reinsdyrstallen i formiddag. 
    Den ble oppdaget i etterkant av svært uvanlige lyder blant reinsdyrene. 
    Det er uvisst hvor lenge ponnien har befunnet seg i stallen. 
    Det foretas undersøkelser av våre fremste teknikere for å utelukke at det er en trojansk ponni.
    
Ingen oppgaver denne dagen.

## Luke 19 - PPKv3 (5)
    Ny PPK
    I lys av senere tids passordproblematikk har NISSEN utviklet en ny PPK. 
    Kan noen alvebetjenter undersøke om NISSEN har klart å luke ut alle svakheter fra tidligere designiterasjoner?
    
    Eksempel
    PPKv3("Pompøst og metodisk") → øMSijrt Mc SÅtMZPrU
    
    ømQ UæjEEi4æÅktÅr i4æÅktÅr SZG tWM tPSÅ i4Z i4æÅktÅr rE0tt UæjEEi4æÅktÅr rE0tt

Mere penn og papir kryptering. Med et eksempel. Ser ut til at det er en substitusjon av noe slag, 
la oss ta input fra eksemplet og legge inn det vi har vha substitusjon i [cyberchef](https://gchq.github.io/CyberChef/#recipe=Substitute('ABCDEFGHIJKLMNOPQRSTUVWXYZ%C3%86%C3%98%C3%85abcdefghijklmnopqrstuvwxyz%C3%A6%C3%B8%C3%A5','ABCDEFGHIJKLoNOiQRmTkVWXYd%C3%86%C3%98eabgdefghp%C3%B8klmnopqsstuvwxyz%C3%A6P%C3%A5')&input=%2BE1TaWpydCBNYyBTxXRNWlByVQr4bVEgVeZqRUVpNObFa3TFciBpNObFa3TFciBTWkcgdFdNIHRQU8UgaTRaIGk05sVrdMVyIHJFMHR0IFXmakVFaTTmxWt0xXIgckUwdHQ), for å se hva vi får.
~~~
Pompøst og metodisk
PmQ kæøEEp4æektes p4æektes mdG tWo time p4d p4æektes sE0tt kæøEEp4æektes sE0tt
~~~
Begynner å ligne på noenting, la oss gjekke litt til på krøllparentes, PST og slutt
~~~
Pompøst og metodisk
PST krøllparentes parentes mdG tWo time pad parentes slutt krøllparentes slutt
~~~
Litt mere gjetting, så tipper jeg det står `PST{(md5 two time pad)}`. parentesene er nok feil, så resultatet skal nok være
`PST{md5(two time pad)}`. Litt banning og sverting, men med et hint fra Roy Solberg gjorde at jeg kom i mål her også. 
(husk på -n på echo for å unngå å få med linjeskift)
~~~bash
echo -n 'two time pad' | md5sum
4a0fc5f3c88874cab11c64e965dff58d  -
~~~

Flagg: PST{4a0fc5f3c88874cab11c64e965dff58d}
{:.note}

## Luke 20 - Mystisk kort (5)
    Mystisk kort
    
    NPST har grunn til å tro at SPST har noen systemer kjørende på meget gammel hardware. 
    Fra temmelig hemmelige kilder har vi greie på at SPST har outsourcet utviklingen av disse systemene, 
    og får tilsendt kildekode for oppdateringen og ny funksjonalitet på postkort. 
    NPST har nylig snappet opp et slikt kort markert "360", som anntas å innholde kode av noe slag. 
    Dessverre ser det ut til at teknikerne våre har noen hull i programmeringskunnskapene sine, 
    de kommer rett og slett til kort. Kunne en alvebetjent sett på kortet?
    
    1020 2020 0010 2012 2001 2200 1020 0000 0800 0200 0001 200A 2001 2200 1020 0C00 0300 0008 0800 1012
    
    Om noen finner ut av det hadde NISSEN satt pris på om vedkommende kunne kjørt koden, 
    og lagt outputen de får inn i intranettet omkranset av { og }, samt prefikset med PST. 
    Eksempel: kode: print("Hei, verden!") → output: Hei, verden! → svar: PST{Hei, verden!}.

Haugevis med hint til gammelt system, kort, hull og 360. Så her snakker vi gammelt hullkort system. Finner masse
gøy informasjon på [The virtual keypunch](https://www.masswerk.at/keypunch/), spesielt under "advanced usage",
hvor vi kan finne IBM/360 Column Binary Format. Jeg finner ingen online tools for dette (er jo gammelt),
så jeg skriver en dekoder basert på informasjonen over og får ut `MD5(IBM 029+IBM/S60)`. Tipper S er feil og at det skal stå
`MD5(IBM 029+IBM/360)`, som gir oss korrekt flagg.  

Flagg: PST{8dc9112fecac5e5d50eaf14e0fff2440}
{:.note}


~~~python
# file: "solver.py"
input = '1020 2020 0010 2012 2001 2200 1020 0000 0800 0200 0001 200A 2001 2200 1020 0C00 0300 0008 0800 1012'
# split into blocks
blocks = input.split(' ')
result = []


def getCbf(b, res):
    retVal = ''
    for pos in range(0, 7):
        if b & 1 << pos != 0:
            retVal = res[7 - pos] + retVal
    return retVal


def getCbf1(b):
    return getCbf(b, 'JKYX0123')


def getCbf2(b):
    return getCbf(b, 'LM456789')


def getKeyPunch(cbf):
    cbf1 = cbf[0:1]
    if len(cbf) == 3:
        cbf2 = str(int(cbf[1:2]) + int(cbf[2:3]))
    elif len(cbf) == 1:
        return cbf
    else:
        cbf2 = cbf[1:2]
    cY = '&ABCDEFGHI¢.<(+|'
    cX = '-JKLMNOPQR!$*);¬'
    c0 = '0/STUVWXYZ ,%_>?'
    c1 = '1AJ/'
    c2 = '2BKS:¢! '
    c3 = '3CLT#.$,'
    c4 = '4DMU@<*%'
    c5 = '5ENV\'()_'
    c6 = '6FOW=+;>'
    c7 = '7GPX"|¬?'
    c8 = '8QY:#@\'="¢.<(+|!$*);¬ ,%_>?'
    c9 = '9IRZ'
    if cbf1 == '' and cbf2 == '':
        return ' '
    if cbf2 == '':
        return '?'
    if cbf1 == 'X':
        return cX[int(cbf2)]
    if cbf1 == 'Y':
        return cY[int(cbf2)]
    if cbf1 == '0':
        return c0[int(cbf2)]
    if cbf1 == '1':
        return c1[int(cbf2)]
    if cbf1 == '2':
        return c2[int(cbf2)]
    if cbf1 == '3':
        return c3[int(cbf2)]
    if cbf1 == '4':
        return c4[int(cbf2)]
    if cbf1 == '5':
        return c5[int(cbf2)]
    if cbf1 == '6':
        return c6[int(cbf2)]
    if cbf1 == '7':
        return c7[int(cbf2)]
    if cbf1 == '8':
        return c8[int(cbf2)]
    if cbf1 == '9':
        return c9[int(cbf2)]
    return '?'


for block in blocks:
    b1 = int(block[0:2], 16)
    b2 = int(block[2:4], 16)
    c1 = getCbf1(b1)
    c2 = getCbf2(b2)
    cbf = c1 + c2
    kp = getKeyPunch(cbf)
    result.append([block, b1, b2, c1, c2, cbf, kp])

print('hex\tb1\tb2\tc1\tc2\tcbf\tkp')
for line in result:
    print(
        line[0] + '\t' +
        ("0x%0.2X" % line[1]) + '\t' +
        ("0x%0.2X" % line[2]) + '\t' +
        line[3] + '\t' +
        line[4] + '\t' +
        line[5] + '\t' +
        line[6])
for line in result:
    print(line[6], end='')

~~~
## Luke 21 - Nytt kryptosystem (5)
    Ukens ansatt Jule NISSEN gratulerer Karine som ukens ansatt, og takker for god innsats. 
    NISSENS sekretariat vil ta kontakt via elektronisk post så fort det lar seg gjøre. 
    Beregn noe ekstra tid grunnet julestri.
    
    Nytt kryptosystem
    
    SPST har nylig fått installert et nytt kryptosystem, og vi i NPST har vært tidlig på ballen 
    og fått plassert avlyttingsutstyr på linjene over Antarktis.
    
    Det kan se ut som at leverandøren til SPST har noen innkjøringsproblemer, da nøkkelen ikke ser ut til å virke. 
    Klarer du å finne ut av hva som er galt?
    
    Link til melding
    
    Jule NISSENS sykdomsforløp
    
    Jule NISSEN er frisk og rask, og har vært på jobb innenfor kjernetiden de siste dagene. 
    Han vil for sikkerhets skyld ta det rolig frem til og med lille julaften i påvente av den store dagen, 
    og vil på grunn av dette sannsynligvis trenge noe ekstra hjelp på mandag.
    
    Julevurdering
    
    Ettersom Jule NISSEN er blitt frisk, blir julevurderingen oppgradert. 
    Det er nå meget sannsynlig at det blir en GOD JUL.

Javel, et nytt kryptosystem. Skal vi se hva vi har å jobbe med:
~~~json
{
    "cipher": "aes-256-gcm",
    "ciphertext": "69cf99390e143fbab3ea8326c05b2fde58c964555bd673de10ff4cf2bf49586454b0466afd01c36b0e4dc1e3361d8ff⏎
                  ec8998d88c13b6ff83798a4607b86f3d14f20f63486d256e65d2164ac90a931d7c36fed071321298ce6eb4206bbc31db⏎
                  dbd08d72dca0f5ce486e68979f083e0e4f46d1f0eee0fec2aa48de030cb2f2069eb719563443c324b052a913e5007f11⏎
                  4de4ed7ae44044c03278e2392b46e7815626424d735196f93adc446c4a4a30373e936fa7164112d0867e63e63a4d809d⏎
                  10e90e805130eb7114422ae17fddd3a272cee6100087fb37eb0268ab187721fc7e8dc8b2b79e91a1d9e276a16bcb79c3⏎
                  6a7c91b127ea3b08fe57a33ba7d0767e35508b4ab2127aa0a2948cd2a1aace305a49cf63d03a41ca110e9d04636a8595⏎
                  6aa1b9eac89bc4091ab59bd1ea9d9cd1c225422a7a40ef56d4b65d1e56f138df24dcf74557a72ba055160f82bf39470f⏎
                  785b12633584ae9639a4352ef08fe5c7f788b0f83021ccf13d7a99a0c088abf72ccc36a30e2447ce10157113a56461bf⏎
                  afd68be40f6f79dbdf2c901028cfe06a96e5ab9a1121c7e2d8b91e495c9722b2cd97d378844328bfba9870b814df6028⏎
                  2fec98f70d1639d35205d399d0a858a5cf7210cd9110faecc55c79ae7fe963d908aaa8a34d68b4c0556aeaa4db1ec74c⏎
                  d321b249602c24dabc961b30ae456199fb1deca09f36cdf8a5d9b0b492e42f7f841cb4627880f9fdd4b2e2d94a553c61⏎
                  c5c9a0a253a4616af93310eb7a55a74316d595c017cd2953e505d893f85f324a50fecd459e6bce60294aef9e34b7d57c⏎
                  98f4e9dc5e09a9a6fe620865308d807d767e4c9fec6041b58d003b152d18473e8ce5a9bfe5203b748945a8f4c4683177⏎
                  b09c6b97603d37451a6220c7cf94fa4cb2f8c26331e08f8b8035acebdeaba4cec24bbdacad448b332ba67d355a91363d⏎
                  13a4030a7f4e5b3e0d12a5cca08d6431e356a1bf5050c9bb6373a375a58909cb59d851c0da4905a62389aef2809e354f⏎
                  b2d6b672071ded673f50a3ca475b229999d5a8566e3d7742a271c91f97d8182cb6dcbd3811276cea2386e66b10f047f5⏎
                  cfd0628e3a55ed420412a4fd1b568ef4a62384694dc6b966defe4226afcc545ff10cd61c181d5df9ce6011995df72a21⏎
                  9f45ac44eccc63450989df746edbc7165ae008caa158c9298e6c8f8907be02c89313ec55be8eef3521a251518e394071⏎
                  b9deb5af8d2273cf7fe2ed9ddc96609a8057ca50e5a5c1b0a9b9e4dfa04e58be60feee17632dd713257916416471560c⏎
                  82da72e49962d1bfcda8db174948765bfc51159e98d38cad9a7ba51a7c8658af998534a6b223d25ba4805c4a12bf0807⏎
                  e043abc0b47e4562a800b6f79fca703bbd93b1c617d3a8b9244ebfcaffd7f8e44440a20ce1ed7ab8039d5cf182c7c16e⏎
                  7be51856ac8a516ca25acf79001af3612d7b4d304c2a5b9671620b80db4fedca8df2ec6b57ddbcf4fe9ece3da44bf47e⏎
                  0f3b21b61f0699e6c38a3169b2890f57636757a894caf22879b1515d1aa00ee8131d3cf7b61ab9c747690bd6ef3d93ed⏎
                  002c1f3a8e9b7a6b056f76cad42d67ae3f01c1d8d4d8576b950efe65b6312b700f2000f8c3821a70ba1e4800b5bdb422⏎
                  3f9db27c509495eb638025c4dc898c13ae9f86c944e5428209a111701bc4e2db44a7c890774b0063e20158fd0d9cebf4⏎
                  f05d40c74afd168d0e8968b14a56c95b0a3b657cbe0ad270170e4154d596c112156c2a9a9fc30898be1f362ca2ce1fe4⏎
                  e4399ab8ea3735b8f091ada9d613411b54760b7c39c956f225bb9724bd4fd8174a4703151d1b5acf2c979f446d8a3303⏎
                  6ec652add04478a44f34bab617f15ed40451cf7603ead7f8f58f8e2fc02ca7c75c7d37cbcd9f09811abd881ef16a15e6⏎
                  bb4e1fe37ff96fe03421286fe37f19120fc66a105fb2b1ee0cbd19feb4e9f20b02fc5d06f6b0d8bbf393da408e2f7e7c⏎
                  ae536852b5f81690596e78b66ca1ec10af873a8acd62cb716bdb40107019e7f3acb4d11d3e50590c2ca485c0b4a47f0c⏎
                  28847ee0afaa617284e5ef1fcf9fc5ec00c6cc2404016a565e4154538f93d0adee6cdb72665efb4a319a98dd58dd81e5⏎
                  2263516a006c037a27ad249ce0efd69e4f6a685a1b6404e95938325bb1b2db35984bc37c4e9751825aaa592089511344⏎
                  328c84e18a470b718e3a3fa7cd6f06cf2c2352d5e6895c734bf1c8d22eb9c2037378f0609211298f71fa633ed923f31c⏎
                  934b05ece2375bbf700f4fd30525a20a510b7f0102f24ff2b7eb76ac77d98bc",
    "nonce": "d97c2e3410f37ac7b5dcd8df",
    "recovered_key": "800816b1629bcfa519f57a502a6a841298a9f5c20203d8818fdd18271a3b1682",
    "tag": "76fe172806c0b41816887630ca74f2f8",
    "uncertain_bits_count": 2
}
~~~
AES-256 GCM, med en kryptert melding, tilgjengelig nonce, tilgjengelig tag og tilgjengelig key. Pluss en tag, uncertain_bits_count: 2.
En slurve-tabbe fører her til at jeg bruker veldig mye tid på å bytte ut bytes isteden for bits.
Så snart jeg fikk et hint fra Roy Solberg, så var denne raskt i boks ved å flippe bits i nøkkelen.   

Flagg: PST{7e7343c9cbe6114f8fd312490816387d}
{:.note}


~~~python
# file: "solver.py"
from Crypto.Cipher import AES
from bitarray import bitarray

vars = {
    "cipher": "aes-256-gcm",
    "ciphertext": "69cf99390e143fbab3ea8326c05b2fde58c964555bd673de10ff4cf2bf49586454b0466afd01c36b0e4dc1e3361d8ff⏎
                  ec8998d88c13b6ff83798a4607b86f3d14f20f63486d256e65d2164ac90a931d7c36fed071321298ce6eb4206bbc31db⏎
                  dbd08d72dca0f5ce486e68979f083e0e4f46d1f0eee0fec2aa48de030cb2f2069eb719563443c324b052a913e5007f11⏎
                  4de4ed7ae44044c03278e2392b46e7815626424d735196f93adc446c4a4a30373e936fa7164112d0867e63e63a4d809d⏎
                  10e90e805130eb7114422ae17fddd3a272cee6100087fb37eb0268ab187721fc7e8dc8b2b79e91a1d9e276a16bcb79c3⏎
                  6a7c91b127ea3b08fe57a33ba7d0767e35508b4ab2127aa0a2948cd2a1aace305a49cf63d03a41ca110e9d04636a8595⏎
                  6aa1b9eac89bc4091ab59bd1ea9d9cd1c225422a7a40ef56d4b65d1e56f138df24dcf74557a72ba055160f82bf39470f⏎
                  785b12633584ae9639a4352ef08fe5c7f788b0f83021ccf13d7a99a0c088abf72ccc36a30e2447ce10157113a56461bf⏎
                  afd68be40f6f79dbdf2c901028cfe06a96e5ab9a1121c7e2d8b91e495c9722b2cd97d378844328bfba9870b814df6028⏎
                  2fec98f70d1639d35205d399d0a858a5cf7210cd9110faecc55c79ae7fe963d908aaa8a34d68b4c0556aeaa4db1ec74c⏎
                  d321b249602c24dabc961b30ae456199fb1deca09f36cdf8a5d9b0b492e42f7f841cb4627880f9fdd4b2e2d94a553c61⏎
                  c5c9a0a253a4616af93310eb7a55a74316d595c017cd2953e505d893f85f324a50fecd459e6bce60294aef9e34b7d57c⏎
                  98f4e9dc5e09a9a6fe620865308d807d767e4c9fec6041b58d003b152d18473e8ce5a9bfe5203b748945a8f4c4683177⏎
                  b09c6b97603d37451a6220c7cf94fa4cb2f8c26331e08f8b8035acebdeaba4cec24bbdacad448b332ba67d355a91363d⏎
                  13a4030a7f4e5b3e0d12a5cca08d6431e356a1bf5050c9bb6373a375a58909cb59d851c0da4905a62389aef2809e354f⏎
                  b2d6b672071ded673f50a3ca475b229999d5a8566e3d7742a271c91f97d8182cb6dcbd3811276cea2386e66b10f047f5⏎
                  cfd0628e3a55ed420412a4fd1b568ef4a62384694dc6b966defe4226afcc545ff10cd61c181d5df9ce6011995df72a21⏎
                  9f45ac44eccc63450989df746edbc7165ae008caa158c9298e6c8f8907be02c89313ec55be8eef3521a251518e394071⏎
                  b9deb5af8d2273cf7fe2ed9ddc96609a8057ca50e5a5c1b0a9b9e4dfa04e58be60feee17632dd713257916416471560c⏎
                  82da72e49962d1bfcda8db174948765bfc51159e98d38cad9a7ba51a7c8658af998534a6b223d25ba4805c4a12bf0807⏎
                  e043abc0b47e4562a800b6f79fca703bbd93b1c617d3a8b9244ebfcaffd7f8e44440a20ce1ed7ab8039d5cf182c7c16e⏎
                  7be51856ac8a516ca25acf79001af3612d7b4d304c2a5b9671620b80db4fedca8df2ec6b57ddbcf4fe9ece3da44bf47e⏎
                  0f3b21b61f0699e6c38a3169b2890f57636757a894caf22879b1515d1aa00ee8131d3cf7b61ab9c747690bd6ef3d93ed⏎
                  002c1f3a8e9b7a6b056f76cad42d67ae3f01c1d8d4d8576b950efe65b6312b700f2000f8c3821a70ba1e4800b5bdb422⏎
                  3f9db27c509495eb638025c4dc898c13ae9f86c944e5428209a111701bc4e2db44a7c890774b0063e20158fd0d9cebf4⏎
                  f05d40c74afd168d0e8968b14a56c95b0a3b657cbe0ad270170e4154d596c112156c2a9a9fc30898be1f362ca2ce1fe4⏎
                  e4399ab8ea3735b8f091ada9d613411b54760b7c39c956f225bb9724bd4fd8174a4703151d1b5acf2c979f446d8a3303⏎
                  6ec652add04478a44f34bab617f15ed40451cf7603ead7f8f58f8e2fc02ca7c75c7d37cbcd9f09811abd881ef16a15e6⏎
                  bb4e1fe37ff96fe03421286fe37f19120fc66a105fb2b1ee0cbd19feb4e9f20b02fc5d06f6b0d8bbf393da408e2f7e7c⏎
                  ae536852b5f81690596e78b66ca1ec10af873a8acd62cb716bdb40107019e7f3acb4d11d3e50590c2ca485c0b4a47f0c⏎
                  28847ee0afaa617284e5ef1fcf9fc5ec00c6cc2404016a565e4154538f93d0adee6cdb72665efb4a319a98dd58dd81e5⏎
                  2263516a006c037a27ad249ce0efd69e4f6a685a1b6404e95938325bb1b2db35984bc37c4e9751825aaa592089511344⏎
                  328c84e18a470b718e3a3fa7cd6f06cf2c2352d5e6895c734bf1c8d22eb9c2037378f0609211298f71fa633ed923f31c⏎
                  934b05ece2375bbf700f4fd30525a20a510b7f0102f24ff2b7eb76ac77d98bc", 
    "nonce": "d97c2e3410f37ac7b5dcd8df",
    "recovered_key": "800816b1629bcfa519f57a502a6a841298a9f5c20203d8818fdd18271a3b1682",
    "tag": "76fe172806c0b41816887630ca74f2f8",
    "uncertain_bits_count": 2
}

def reverse_mask(x):
    x = ((x & 0x55555555) << 1) | ((x & 0xAAAAAAAA) >> 1)
    x = ((x & 0x33333333) << 2) | ((x & 0xCCCCCCCC) >> 2)
    x = ((x & 0x0F0F0F0F) << 4) | ((x & 0xF0F0F0F0) >> 4)
    x = ((x & 0x00FF00FF) << 8) | ((x & 0xFF00FF00) >> 8)
    x = ((x & 0x0000FFFF) << 16) | ((x & 0xFFFF0000) >> 16)
    return x

max = (len(bytearray.fromhex(vars["recovered_key"])) * 8)
for c1 in range(0, max):
    print("c1:" + str(c1))
    for c2 in range(c1, max):
        keyBytes = bytearray.fromhex(vars["recovered_key"])
        char1Idx = int(c1 / 8)
        bit1Index = c1 - char1Idx * 8
        keyBytes[char1Idx] = keyBytes[char1Idx] ^ 1 << bit1Index
        char2Idx = int(c2 / 8)
        bit2Index = c2 - char2Idx * 8
        keyBytes[char2Idx] = keyBytes[char2Idx] ^ 1 << bit2Index

        nonceBytes = bytearray.fromhex(vars["nonce"])
        cipherBytes = bytearray.fromhex(vars["ciphertext"])
        tagBytes = bytearray.fromhex(vars["tag"])
        cipher = AES.new(keyBytes, AES.MODE_GCM, nonce=nonceBytes)
        plaintext = cipher.decrypt(cipherBytes)
        # print(plaintext.decode('iso-8859-1'))
        try:
            cipher.verify(tagBytes)
            print("Bits in pos " + str(c1) + " and " + str(c2) + " must be flipped.")
            print("The message is authentic:", plaintext.decode('utf-8'))
            exit()
        except ValueError:
            a = 0
            # print("Key incorrect or message corrupted")
~~~
    Bits in pos 148 and 174 must be flipped.
    The message is authentic: ﻿
    SPST
    21.12.19
    
    Kantinesituasjonen
    Kantinen har nå landet avtale med ny råvareleverandør, og første leveranse blir idag. 
    Det blir altså vintersolverv-torsk som planlagt.
    
    Lekkasje fra SPSTs interne nettverk
    I forbindelse med angrepet mot stordatamaskinen som startet 12.12.19, ble det eksfiltrert en mindre mengde 
    dokumenter gradert TEMMELIG HEMMELIG. 
    Som en konsekvens av dette har vi fra idag tatt i bruk en ny krypteringsalgoritme 
    for data i bevegelse. Vi har opplevd litt problemer med de dedikerte linjene for overføring av nøkkelmateriell, 
    men leverandøren vår jobber på spreng for å fikse dette.
    
    
    Ytterligere operasjoner mot NPST
    - Lørdag 14.12.19 fikk en av våre dyktige kildeføringspingviner informasjon om NPSTs planlagte julebord. 
    Vi har ikke lykkes i å bekrefte informasjonen, men kilden forteller at det skal konkurreres i hammerdistanse
    - Tirsdag 17.12.19 ble det gjennomført en vellykket sabotasjeaksjon mot NPSTs matlager. 
    Her ble passordet skiftet til rudolf2, og hashing-algoritme byttet ut med MD5. 
    Det vurderes som MULIG at dette vil påvirke julen.
    - Onsdag 18.12.19 ble det som en HOAX/avledningsoperasjon plassert 
    en helt vanlig ponni i reinsdyrstallen til Jule NISSEN. 
    Dette vil trolig kunne påvirke julen ytterligere.
    
    
    Ukens ansatt
    Denne ukens ansatt ønsker ikke å opptre med navn, 
    men vi kan avsløre at pingvinen jobber ved SPSTs PINGINT-seksjon, 
    og har gjort en utmerket jobb med kildene sine den siste tiden.
    
    
    Takk til alle for utmerket arbeid uka som har vært.
    Sjef SPST, Keiserpingvinen
    
    FLAGG: PST{7e7343c9cbe6114f8fd312490816387d}

## Luke 22
### 22. desember
    Opptrapping av falske nyheter
    SPST har den siste tiden trappet opp sin pågående falske nyhets-kampanje. som, i følge betrodde kilder, 
    refereres til som "Operasjon Avlys julen" internt hos SPST. Det er kritisk at vi får stoppet denne operasjonen 
    så rask som mulig, men foreløpig har vi ikke helt oversikt over hva dette innebærer.
    
    Analytikerne våre mener det kanskje ligger noe gjemt på SPST sin kampanje-side, 
    men dessverre greier ikke agentene våre å finne noe. Kunne noen alvebetjeneter sett om de finner noe der?
    
    https://spst.no

Her har vi altså en kampanjeside. En rask titt på robots.txt gir oss:
    
    User-agent: *
    Disallow: /temmelig-hemmelig/
    
`https://spst.no/temmelig-hemmelig/` gir oss flagget direkte på siden.   


Flagg: PST{fc35fdc70d5fc69d269883a822c7a53e}
{:.note}

### Kildekode (15)
Joda, source koden til websiden inneholder en skjult tag:
~~~html
<span id="forkongithub"><a href="https://github.com/SydpolarSikkerhetstjeneste/spst.no">🍴 meg på GitHub</a></span>
~~~
Kildekoden er tilgjengelig på github, og ved å gå gjennom historikken, finner vi at nøkkelen var `pingvinerbestingenprotest`,
har blitt endret til morse med regex pattern `[ -\.]+`. Cyberchef er nok en gang mitt goto-tool. Ved å legge
inn korrekt morsekode på denne siden får vi:
    
    Til Pen Gwyn
    Fra Keiserpingvinen
    
    Alt ligger i vedlagt bilde.

Flagget ligger bokstavelig talt i alt-tag til bildet.  


Flagg: PST{f2e0e89f59722af1f388529720b9db03}
{:.note}

Bildet er også av en gul lapp med teksten `CO NG RA TS`. I tillegg ser det ut til at det er noen spesielle kabler i bildet, 
sannsynligvis [ENIGMA](https://en.wikipedia.org/wiki/Enigma_machine) kabler (du har vel sett [The Imitation Game](https://en.wikipedia.org/wiki/The_Imitation_Game)?).

### &npsp; (20)
    Våre analytikere mener SPST kan ha skjult informasjon for hackere som deg.

Denne er litt tricky, men veldig gøy. Jeg har stort sett vært gjennom alt på websidene fram til nå, så det eneste som 
kan inneholde skjult informasjon må jo være bildet. zsteg viser at det kan være noe i `b1,rgb,lsb,xy`, ser iallefall ut til 
å være lange linjer og et linjeskift. En extract viser at det ser ut til å være en [QR kode](https://en.wikipedia.org/wiki/QR_code).
~~~bash
$ zsteg -E b1,rgb,lsb,xy 95728ce2159815f2e2a253c664b2493f.png > qr-code.txt
~~~
For å lese av QR koden erstatter jeg 'S' med '█', åpner opp i firefox, 
lager en skjermdump, åpner denne i [gimp](https://www.gimp.org/), reskalerer slik at qr koden er kvadratisk, 
bruker en QR leser på mobiltelefonen til å lese av koden og tilslutt
kopierer resultatet til meg selv. QR koden representerer en URL, ser det ut til `/8a2a8e12017977d9dbf0ed33e254e94e.txt`.

    These aren't the Penguins you're looking for
    
    
                 a8888b.
                d888888b.
                8P"YP"Y88
                8|o||o|88
                8'    .88
                8`._.' Y8.
               d/      `8b.
             .dP   .     Y8b.
            d8:'   "   `::88b.
           d8"           `Y88b
          :8P     '       :888
           8a.    :      _a88P
         ._/"Yaa_ :    .| 88P|
         \    YP"      `| 8P  `.
        /     \._____.d|    .'
         `--..__)888888P`._.'

Det ser tilsynelatende ut som en standard feilmeldings-side, men skinnet kan bedra. Den gir faktisk ikke 404 Not Found, 
men derimot 200 OK. Ved å laste ned filen og ta en nærmere titt på den, kan man se at den inneholder en rekke bytes som
ikke tilsvarer noen karakterer.
Ved å kjøre litt nummeranalyse på disse bytes, kan jeg se at det er 888 bytes totalt. Jeg kan også se at det er et mønster, 
det ser ut til at tallene opptrer i grupper på 3 og 3. UTF-8/Unicode er det ikke, ei heller RGB. Ved å sjekke antall av unike grupper,
kan jeg se at det er kun 4 unike grupperinger. Jeg forsøker base 4 på de tre første gruppene, og får PST som resultat. 
Løsningen er altså base4 enkodet, erstattet med en gruppering på 3 bytes og printet ut.
Ved å reversere dette finner vi det siste flagget.  

Flagg: PST{67b8601a11e47a9ee3bf08ddfd0b79ba}
{:.note}

~~~python
# file: "solver.py"
import string
from PIL import Image
import numpy as np

with open("8a2a8e12017977d9dbf0ed33e254e94e.txt", "rb") as f:
    bytes = f.read()
    other = []
    for b in bytes:
        if chr(b) in string.printable:
            print(chr(b), end='')
        else:
            other.append(b)

# OK, the first letter should be P.
# the first 24 values represent this somehow.
# lets see if we find a pattern:
print("")
# we have these unique combos (subtracted with the base value):
# 0,0,0 - 0
# 0,0,1 - 1
# 0,0,32 - 2
# 13,59,51 - 3

# what happens if we decode to base4? I think I got it!
# 80 - 83 - 84


i = 0
base1 = 226
base2 = 128
base3 = 140
line = ''
lines = []
while i < 888:
    num1 = other[i]
    num2 = other[i+1]
    num3 = other[i+2]
    if i % 24 == 0 and i!=0:
        lines.append(line)
        line = ''
    num_base1 = num1 - base1
    num_base2 = num2 - base2
    num_base3 = num3 - base3
    actual = 0
    if num_base1 == 0 and num_base2==0 and num_base3 == 0:
        actual = 0
    elif num_base1 == 0 and num_base2 == 0 and num_base3 == 1:
        actual = 1
    elif num_base1 == 0 and num_base2 == 0 and num_base3 == 32:
        actual = 2
    elif num_base1 == 13 and num_base2 == 59 and num_base3 == 51:
        actual = 3
    else:
        print("agh!")
        exit()
    line+=str(actual)
    i = i + 3
lines.append(line)

for line in lines:
    print(chr(int(line, 4)), end='')
print("")
~~~

## Luke 23
### Fragmentert samtale (5)
    Fragmentert samtale
    
    Arbeidet som ble gjort 21. desember rundt avdekking av feil ved SPST sitt nye kryptosystem, 
    har gjort oss i stand til å snappe opp en samtale mellom to høytstående SPST-agenter. 
    Teknikerne våre er dessverre på ferie, og vi greier ikke dekode meldingen helt, kunne noen alvebetjenter hørt på det?
    Vi har misstanke om at det utveksles en kode, og at denne koden kan benyttes for å finn'e en artikkel av noe slag. 
    Analytekerne våre mener det er artikkelens navn som er av interesse. Om noen finner ut av det, 
    legg inn md5-hashen av tittelen på artikkelen dere finn'er.
    
    Last ned PCAP
    
    Eksempel:
    tittel: "Gløgg og pepperkaker" → svar: PST{md5("Gløgg og pepperkaker")} = PST{9612ad6af8b6d78045b840b3c477d004}

Vi får en zip-fil med en [pcap](https://en.wikipedia.org/wiki/Pcap) og en tekstfil.
    
    INVITE sip:24700720@10.143.23.244:5060 SIP/2.0
    Max-Forwards: 59
    Via: SIP/2.0/UDP 10.22.48.6:5060;branch=z9hG4bKg3Zqkv7iqzibm3wfn8td5orxo7vyxe5q9
    To: <tel:24700720;phone-context=+47>
    From: "Anonymous" <sip:anonymous@anonymous.invalid>;tag=h7g4Estg_p63344t1976798986m456675c4958193147s1_3997725510-548775950
    Call-ID: p65599t1576767456m455575f2658294147s2
    CSeq: 1 INVITE
    Contact: <sip:sgc_c@10.22.48.6;transport=udp>;+g.3gpp.icsi-ref="urn%3Aurn-7%3A3gpp-service.ims.icsi.mmtel"
    Record-Route: <sip:10.22.48.6;transport=udp;lr>
    Accept-Contact: *;+g.3gpp.icsi-ref="urn%3Aurn-7%3A3gpp-service.ims.icsi.mmtel"
    Min-Se: 900
    Resource-Priority: q735.4
    Session-Expires: 1800
    Supported: timer
    Supported: 100rel
    Supported: precondition
    Supported: histinfo
    Content-Type: application/sdp
    Content-Length: 421
    Session-ID: 5b8e6d5f89b8ea975b77e0a6bbf782a8
    Allow: REGISTER, REFER, NOTIFY, SUBSCRIBE, PRACK, UPDATE, INVITE, ACK, OPTIONS, CANCEL, BYE
    Accept: application/sdp
    
    v=0
    o=- 2022048203 3887725258 IN IP4 10.22.48.6
    s=-
    c=IN IP4 10.64.65.129
    t=0 0
    a=sendrecv
    m=audio 45248 RTP/AVP 97 100 105
    c=IN IP4 10.64.65.129
    a=curr:qos local none
    a=curr:qos remote none
    a=des:qos mandatory local sendrecv
    a=des:qos optional remote sendrecv
    a=rtpmap:97 PCMA/8000
    a=rtpmap:100 telephone-event/16000
    a=fmtp:100 0-15
    a=rtpmap:105 telephone-event/8000
    a=fmtp:105 0-15
    a=maxptime:40
    
Så det ser ut til at vi har fått en pcap av en lydsamtale med protokollen [RTP](https://en.wikipedia.org/wiki/Real-time_Transport_Protocol)
med [PCMA](https://tools.ietf.org/html/rfc3551#section-4.5.14) payload på 8000 Hz.
Jeg åpner pcap filen i [wireshark](https://www.wireshark.org/), spesifiserer at det er RTP istedenfor UDP, 
så får jeg faktisk opp at dette er en samtale mellom to parter. Jeg får ikke til å spille av lyd i min wireshark, 
så jeg eksporterer i RAW format, og importerer dette til [audacity](https://www.audacityteam.org/).
Her finner jeg egentlig ikke ut av så mye, men etter litt konferering med Roy Solberg, så var det en enklere måte
å løse denne på, det er masse hint til finn, og PST hadde kun tre annonser ute denne dagen, så det var bare å prøve tittelen
på alle tre, så kom man fram til riktig flagg.
Jeg ser i etterkant at jeg var inne på den påtenkte løsningen, [her er en beskrivelse](https://github.com/myrdyr/ctf-writeups/tree/master/npst#fragmentert-samtale).  

`PST{0844d949169d24679a1f0438f89c69e3}`
{:.note}
### Mystisk julekort (5)
    Mystisk julekort
    
    Jule NISSEN fikk i går et meget spesielt julekort levert i den elektroniske postkassen hans. 
    Julekortet har ingen bakside, og derfor heller ingen tekst knyttet til seg.
    Klarer du å finne ut noe mer om avsender e.l.?

Eneste oppgaven jeg ikke løste, og jeg ser i etterkant at jeg var langt ifra å finne korrekt løsning.
Awesome writeup av løsningen [finner du her](https://github.com/myrdyr/ctf-writeups/tree/master/npst#mystisk-julekort).

## 24. desember (24)
    Operasjon Avlys julen
    Situasjonen rundt SPSTs operasjon "Avlys julen" har eskalert, 
    og vi må sette inn umiddelbare tiltak for å få stoppet den. 
    Betrodde kilder sier det hele styres fra SPSTs digitale operasjonssentral. 
    Keiserpingvinens mange reiser ryktes det at operasjonssentralen er åpent tilgjengelig fra hele verden, 
    men at den er beskyttet med avansert kryptografi.
    Kan noen av alvebetjentene se om greier å spore opp SPST sin operasjonssentral, 
    hacke seg inn og avbryte operasjonen deres?

Endelig til slutten! Det har vært mange oppgaver, en fin blanding av enkle, vanskelige og ultra-vanskelige (23 - Mystisk julekort).
Denne oppgaven hadde jeg også problemer med, men klarte den til slutt med fine hint ifra Roy Solberg.
Ikke så mye informasjon å gå på, men la oss gjette på at vi trenger enten å finne et subdomene eller en katalog.
Jeg setter igang directory enumeration med [gobuster](https://github.com/OJ/gobuster) 
og subdomain enumeration med [dnsenum](https://github.com/fwaeytens/dnsenum), 
og finner til slutt et nytt subdomene, `ops.spst.no`
~~~bash
$ dnsenum -f /usr/share/seclists/Discovery/DNS/namelist.txt --threads 50 --noreverse spst.no
~~~
Der får vi presentert en side med en sjenert knapp. litt kildekode inspisering sier oss at URL-en er:
`https://ops.spst.no/api/abort`. Denne siden gir oss tilbake en kryptert melding, tydeligvis:

    SAXVC OIWPT GQOJZ OXEHI ZVCWU NCCOW FIKVP NOENT CETAU IKPCM ZLOYP BJHEC KPEXG RJWDO DJBBI HQDTG FFBQV LJAZC ZOFIC 
    ZAIWJ QEVCL FXAVC PDUWT GBIGM SSWAO OXJHP PLKXH TGQAY COIQL ZSWIL HKMYR YMPZZ PTIEL PSRIP YVRKC DINBR WJZJP HHNXM 
    HGYWN XXIGB UTTOX AEPKZ TUCMC MGFHC WHSAY KFVVS PDBFE KABAB PNBVR IZGTX PERJZ GDHQJ JDUYV FAOYV JWZOU WXXPR HVDLL 
    BQTJI HULQP ACIXG NUPUS PCKHT LOKLN ZCLZO QVWSL HPBWD ATZES JEITM AJIFU SIVVF PHPEN UYHZK AWIZY MNQLH ZVKJJ EEYSZ 
    LLUEM NZAFA OZXYL WBRPX JUKQG KIEXX CDYAT IHVJK HOMGI UVAOQ PBXRN HAAWG XOBAZ UILJB KYSBP IOBKH GYZBD IPQNG VSUTS 
    YXOGY KEIKK TIKKQ RFVWQ NBCEK TIJLC CXRDB TUNXT SBKWR YDBR
    
La oss håpe på at vi nå får bruk for alle kodene på de gule lappene vi har funnet (ser mistenkelig ut som enigma innstillinger):  
`IV PS ⚙️`  
`VII TF ⚙️`  
`VII TW ⚙️`  
`CO NG RA TS`  
Vi gjettet korrekt, ved å punche inn kodene over i [Cyberchef Enigma](https://gchq.github.io/CyberChef/#recipe=Enigma('3-rotor','LEYJVCNIXWPBQMDRTAKZGFUHOS','A','A','ESOVPZJAYQUIRHXLNFTGKDCMWB%3CK','P','S','NZJHGRCXMYSWBOUFAIVLPEKQDT%3CAN','T','F','NZJHGRCXMYSWBOUFAIVLPEKQDT%3CAN','T','W','AY%20BR%20CU%20DH%20EQ%20FS%20GL%20IP%20JX%20KN%20MO%20TZ%20VW','CO%20NG%20RA%20TS',true)&input=U0FYVkMgT0lXUFQgR1FPSlogT1hFSEkgWlZDV1UgTkNDT1cgRklLVlAgTk9FTlQgQ0VUQVUgSUtQQ00gWkxPWVAgQkpIRUMgS1BFWEcgUkpXRE8gREpCQkkgSFFEVEcgRkZCUVYgTEpBWkMgWk9GSUMgWkFJV0ogUUVWQ0wgRlhBVkMgUERVV1QgR0JJR00gU1NXQU8gT1hKSFAgUExLWEggVEdRQVkgQ09JUUwgWlNXSUwgSEtNWVIgWU1QWlogUFRJRUwgUFNSSVAgWVZSS0MgRElOQlIgV0paSlAgSEhOWE0gSEdZV04gWFhJR0IgVVRUT1ggQUVQS1ogVFVDTUMgTUdGSEMgV0hTQVkgS0ZWVlMgUERCRkUgS0FCQUIgUE5CVlIgSVpHVFggUEVSSlogR0RIUUogSkRVWVYgRkFPWVYgSldaT1UgV1hYUFIgSFZETEwgQlFUSkkgSFVMUVAgQUNJWEcgTlVQVVMgUENLSFQgTE9LTE4gWkNMWk8gUVZXU0wgSFBCV0QgQVRaRVMgSkVJVE0gQUpJRlUgU0lWVkYgUEhQRU4gVVlIWksgQVdJWlkgTU5RTEggWlZLSkogRUVZU1ogTExVRU0gTlpBRkEgT1pYWUwgV0JSUFggSlVLUUcgS0lFWFggQ0RZQVQgSUhWSksgSE9NR0kgVVZBT1EgUEJYUk4gSEFBV0cgWE9CQVogVUlMSkIgS1lTQlAgSU9CS0ggR1laQkQgSVBRTkcgVlNVVFMgWVhPR1kgS0VJS0sgVElLS1EgUkZWV1EgTkJDRUsgVElKTEMgQ1hSREIgVFVOWFQgU0JLV1IgWURCUg), får vi ut en rekke med engelske tall.

    FIVEZ EROSI XNINE SIXSE VENSI XSEVE NTWOZ EROSI XONES EVENS IXTWO ONEZE ROAFO URNIN ESIXE SIXSE VENSI XFIVE SIXET 
    WOZER OSIXT WOSIX FSIXB SEVEN THREE SEVEN FOURS IXONE SEVEN SIXSI XBSIX ASIXF IVESI XBSEV ENTHR EETWO EZERO AFOUR 
    CSIXF IVESE VENFO URTWO ZEROS IXNIN ETWOZ EROSI XEIGH TSIXF IVESE VENEI GHTTW OZERO SEVEN ZEROC THREE AFIVE TWOZE 
    ROSIX SIXCT HREEB EIGHT SIXCS IXSEV ENSIX FIVES IXESI XFOUR SIXFI VETWO ZEROS EVENT WOSIX FIVES IXSEV ENSIX FIVES 
    EVENE IGHTT HREEA TWOZE ROFIV EBFIV ECSIX FOURS IXONE TWODS IXSIX FIVED SEVEN BTHRE ETWOS EVEND FIVEC TWOES EVENT 
    HREES EVENZ EROSE VENTH REESE VENFO URFIV ECTWO ESIXE SIXF
Et lite python program som søker og erstatter gir oss:

    50696767206176210A496E67656E20626F6B737461766B6A656B732E0A4C65742069206865782070C3A52066C3B86C67656E64652072656765783A205B5C64612D665D7B327D5C2E737073745C2E6E6F

Som kan enkelt konverteres til tekst vha [Cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('None')&input=NTA2OTY3NjcyMDYxNzYyMTBBNDk2RTY3NjU2RTIwNjI2RjZCNzM3NDYxNzY2QjZBNjU2QjczMkUwQTRDNjU3NDIwNjkyMDY4NjU3ODIwNzBDM0E1MjA2NkMzQjg2QzY3NjU2RTY0NjUyMDcyNjU2NzY1NzgzQTIwNUI1QzY0NjEyRDY2NUQ3QjMyN0Q1QzJFNzM3MDczNzQ1QzJFNkU2Rg) 
(det er rå hex verdier), som gir oss neste steg:
    
    Pigg av!
    Ingen bokstavkjeks.
    Let i hex på følgende regex: [\da-f]{2}\.spst\.no

Masse testing, feiling og banning senere, så kom jeg et steg videre. på domenene fra `00.spst.no` opp til `ff.spst.no`
så finnes det [TXT records og CNAME records](https://en.wikipedia.org/wiki/List_of_DNS_record_types).
Jeg hentet ut TXT og CNAME recordene med [dig](https://en.wikipedia.org/wiki/Dig_(command)).
En TXT record inneholder en bokstav, mens en CNAME record inneholder en annen dns entry, eksempelvis `aa.spst.no`.
En CNAME record inneholder den spesielle DNS entry: `slutt.spst.no`
Enda mere testing feiling og banning senere kom jeg fram til at alle 256 entries i CNAME recordene dannet en "self referencing chain".
Så, ved å splitte opp i hex-fra og hex-til, "slutt" er i hex-til, så har vi den siste hex entry. så finner vi denne i hex-til og 
plukker hex-fra. Slik fortsetter vi inntill vi har plukket alle hex verdier.
ved å rokkere om TXT entriene basert på denne rekkefølgen får vi:
    
    bokstav=edG1y9Dq9ram2hb0mQYNT4wcWeNXRkY22JU7wa6qFJqkWMLRF0nxeFZNr02jxpJ7ZIzVeWnwe60pbSKLXcwvbV23yFOdN4aPXCV6GHN4fYnzswDTAop3O8vTEEDJFOeuKdBVWGcWy7LDcsucwz8nBHbR9UG9CP4zpMZLQPvEl1eu4Tp9Lto4zuA0ijU2eLk0qQBlRQdxZKrajIqiW5P1K1HkKrjgGIj4M7xP7Sg3pSNXLklGm4LBBbhG

Noe kjent er det iallefall, og leser vi litt tilbake, så tyter api/abort over "Ingen bokstavkjeks", som sannsynligvis 
betyr "Du mangler en cookie med navn 'bokstav'".
Klem inn denne verdien som cookie, og vi får et annet resultat.
    
    Arrgh! Vi gir opp for denne gang.
    https://npst.no/_6331fff126233c324c9f5fc49c49a8b6.html

Flagg: PST{82a1f79e6ce39ef16d0ef4ef1c1d2fcc}
{:.note}
![star wars](/assets/img/blog/2020-npst/24-final-scene.png)

## Til slutt
Jeg satt virkelig fast på noen oppgaver, og brukte Roy Solberg ([Twitter](https://twitter.com/roysolberg), [Blog](https://blog.roysolberg.com/)) som en veldig kjekk "rubber duck" og hint master. 
Uten hans hjelp hadde jeg nok ikke kommet videre på en del av oppgavene.

🙌🙌 Takk for at du tok deg tid til å lese alt, trykk på recommend og legg inn en kommentar! 🙌🙌