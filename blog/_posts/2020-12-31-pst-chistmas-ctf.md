---
layout: post
title: PST Julekalender 2020
description: >
  PST (Politiets Sikkerhetstjeneste) hadde i 2020 også en julekalender. Den ble annonsert via en fiktiv annonse for
  NPST (Norpolar Sikkerhetstjeneste), som hadde en jobbannonse etter midlertidig stilling som snikende alvebetjent.
  Dette er en gjennomgang av julekalenderen som var en full CTF. 
---
# NPST utlyser en midlertidig stilling som snikende alvebetjent
* this unordered seed list will be replaced by toc as unordered list
{:toc}
Denne gangen hadde de introdusert DASS - Digitale Arkiv- og SaksbehandlingsSystem. 🚽
## Luke 1 🏴‍☠️ Flagg
    Hei,
    Kan du bekrefte at du har fått tilgang til systemet?
    Det gjør du ved å svare på denne meldingen med verifiseringskoden RUV{JgkJqPåGtFgvLwnKilgp}.
    OBS: Jeg mistet verifiseringskoden din i salaten, så mulig du må rette opp i den før du svarer.
    Vennlig hilsen din nærmeste leder

Ser ut som en rot -2
~~~python
# file: "solver.py"
def rot(rchar, num):
    chars = 'abcdefghijklmnopqrstuvwxyz'
    idx = chars.find(rchar.lower())
    return chars[idx + num] if idx != -1 else rchar

def rotall(msg, num):
    rv = ''
    for ch in msg:
        rotated = rot(ch, num)
        rv += rotated.upper() if ch.isupper() else rotated
    return rv
print(rotall('RUV{JgkJqPåGtFgvLwnKilgp}', -2))
~~~

🏴‍☠️ `PST{HeiHoNåErDetJulIgjen}`
{:.note}

    Strålende!
    Velkommen til NPST! Som din nærmeste leder håper jeg du er klar for større utfordringer i dagene som kommer.
    Hver dag vil jeg ha nye arbeidsoppgaver til deg.
    Bruk gjerne tiden mellom oppgavene til å sette deg godt til rette i DASS og dets funksjoner.
    For hver arbeidsoppgave du gjennomfører får du poeng som vises i poengoversikten.
    Hvordan du gjør det der kan muligens innvirke på neste års lønnsforhandlinger.

## Luke 2 🏴‍☠️ Flagg
    Vi får beskjed om å se på en mid fil som er i en zip fil.

Mid filen gir ikke så mye mening hvis man hører på den. [pen_gwyn_greatest_hits.mid](assets/files/npst-2020/pen_gwyn_greatest_hits.mid)
Ved å se på notene, og gjøre om notene til bokstaver, så får man ut en tekststreng, som inneholder flagget.

~~~python
# file: "solver.py"
from mido import MidiFile
def decrypt():
    mid = MidiFile('pen_gwyn_greatest_hits.mid')
    notes = []
    for i, track in enumerate(mid.tracks):
        for msg in track:
            if msg.type != 'note_on':
                continue
            note = int(str(msg).split(' ')[2].split('=')[1])
            notes.append(note)
    res = ''
    for note in notes:
        res += chr(note)
    start = res.find('PST{')
    end = res.find('}', start)
    print(res[start:end+1])
decrypt()
~~~
~~~bash
$ pip3 install mido
$ python3 solver.py 
PST{BabyPenGwynDuhDuhDuhDuhDuhDuh}
~~~

🏴‍☠️ `PST{BabyPenGwynDuhDuhDuhDuhDuhDuh}`
{:.note}

## Luke 3
    Vi får her passordet til 7z arkivet som vi fikk tilgang til i luke 2, `til zip-fila,` med en beskjed om at det er sikkert informasjon i dette bildet.

### 🏴‍☠ Flagg
Zsteg gir oss en link til en youtube video, [CSI Zoom Enhance](https://youtu.be/I_8ZH1Ggjk0).
~~~bash
$ zsteg cupcake.png 
b1,rgb,lsb,xy       .. text: "youtu.be/I_8ZH1Ggjk0"
~~~
Et hint til DASS 🚽 som inneholder nettop en bildeforbedrings modul. Hvis vi bruker denne på cupcake, kommer flagget i siste bilde.

🏴‍☠️ `PST{HuskMeteren}`
{:.note}

### 🥚 Egg
Hvis vi laster ned alle mellombildene som forbedringsmodulen lager, så er det også noe informasjon i ett av bildene, som gir oss vårt første egg 🥚, noe som tydeligvis er ekstrapoeng for å komme helt til topps i konkurransen.
~~~python
# file: "egg-3.py"
import subprocess

import requests
import os
# egg is on one of the intermediate images: 9bab0c0ce96dd35b67aea468624852fb.png
# b1,rgb,lsb,xy
img = requests.get('https://dass.npst.no/images/9bab0c0ce96dd35b67aea468624852fb.png').content
with open('9bab0c0ce96dd35b67aea468624852fb.png', 'wb') as file:
    file.write(img)
res = subprocess.check_output(['zsteg', '9bab0c0ce96dd35b67aea468624852fb.png']).decode('utf-8')
start = res.find('EGG{')
end = res.find('}', start)
print(res[start:end+1])
~~~
~~~bash
$ python3 egg-3.py 
EGG{MeasureOnceCutTwice}
~~~

🥚 `EGG{MeasureOnceCutTwice}`
{:.note}

## Luke 4 🏴‍☠️ Flagg
    Luke 4
    Hei,
    Som alle vet, så varer jula helt til påske, og her starter problemene...
    Vi i mellomledergruppa har begynt på et forprosjekt for utredning av bemanningsstrategi for påsken i årene fremover.
    Systemet vi benytter for å finne ut når det er påske oppfører seg rart,
    slik at dette viktige arbeidet nå har blitt satt på vent. Klarer du å finne ut hva som er feil?
    Vi i mellomledergruppa er svært interessert i måltall,
    og ledelsen ønsker en rapport snarest på summen av kolonnen Maaltall fra og med 2020 til og med 2040.
    Kan du svare meg med denne summen, omkranset av PST{ og } når du finner ut av det?
    📎 filer.zip


Etter å ha kikket igjennom innholdet i [filer.zip](assets/files/npst-2020/filer.zip), kom jeg fram til at det var fint å reimplementere denne utregningen i python.

~~~python
# file: "solution-4.py"
import datetime
import math
from dateutil.relativedelta import *

def getPaaskeAften(aar):
    a = aar % 19
    b = math.floor(1.0 * aar / 100)
    c = aar % 100
    d = math.floor(1.0 * b / 4)
    e = b % 4
    f = math.floor((8.0 + b) / 25)
    g = math.floor((1.0 + b - f) / 3)
    h = (19 * a + b - d - g + 15) % 30
    i = math.floor(1.0 * c / 4)
    k = aar % 4
    l = (32.0 + 2 * e + 2 * i - h - k) % 7
    m = math.floor((1.0 * a + 11 * h + 22 * l) / 451)
    # year = DATEADD(yy, aar - 2000, '2000/01/01')
    year = datetime.datetime(aar, 1, 1)
    monthadd = math.floor((1.0 * h + l - 7 * m + 114) / 31) - 1
    month = year + relativedelta(months=+monthadd)
    dayadd = (h + l - 7 * m + 114) % 31
    easterday = month + relativedelta(days=+dayadd)
    paaskeaften = easterday + relativedelta(days=-1)
    return paaskeaften

def days_since_jan_1_1900_to_datetime(d):
    return (d - datetime.datetime(1900, 1, 1)).days

def loadCorrect():
    rv = 0
    for aar in range(2020, 2041):
        paaskeaften = getPaaskeAften(aar)
        rv += days_since_jan_1_1900_to_datetime(paaskeaften)
    return rv

if __name__ == '__main__':
    correct = loadCorrect()
    print('PST{{' + correct + '}}')
~~~

~~~bash
$ python3 solution-4.py 
PST{999159}
~~~

🏴‍☠️ `PST{999159}`
{:.note}

## Luke 5 🏴‍☠️ Flagg
    Vi får en indikasjon på at det er noe galt, og får en csv log for inspeksjon.

[log.csv](assets/files/npst-2020/log.csv) har haugevis med linjer, så jeg brukte python for å explore og finne ut særegenheter. Navnet på Julenissen var feilstavet på en av logglinjene.

~~~python
# file: "solution-5.py"
import csv
from urllib.parse import unquote

def parseRow(row):
    rv = dict()
    rv['date'] = row[0]
    rv['name'] = unquote(row[1]).replace('+', ' ')
    rv['subj'] = unquote(row[2]).replace('+', ' ')
    rv['body'] = unquote(row[3]).replace('+', ' ')
    return rv

def getShadyName(res):
    for line in res:
        name = line['name']
        lname = name.split(' ')[0]
        rname = name.split(' ')[1][1:-1]
        if lname != rname and lname != 'Nissen':
            return line
    return None

with open('log.csv', encoding='utf-16') as file:
    res = []
    reader = csv.reader(file, delimiter=';')
    for row in reader:
        res.append(parseRow(row))
    line = getShadyName(res)
    body = line['body']
    start = body.find('PST{')
    end = body.find('}', start)
    print(body[start:end + 1])
~~~

~~~bash
$ python3 solution-5.py 
PST{879502f267ce7b9913c1d1cf0acaf045}
~~~

🏴‍☠️ `PST{879502f267ce7b9913c1d1cf0acaf045}`
{:.note}


## Luke template
    desc

forklaring

~~~python
kode
~~~

~~~bash
run
~~~

🏴‍☠️ `flagg`
{:.note}

🙌🙌 Takk for at du tok deg tid til å lese alt, trykk på recommend og legg inn en kommentar! 🙌🙌