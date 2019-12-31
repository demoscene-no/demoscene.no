---
layout: article
title:  "Catmull-Rom splines"
date:   2001-11-21 22:00:00
author: Sesse
---
Catmull-Rom splines
===================

Det å lage en rett (lineær) kurve er ikke spesielt vanskelig, men det er
ikke alltid at man ønsker en slik rett overgang. Spesielt naturlig
bevegelse vil neppe ha den typen "hakk" som man får når man bytter
mellom segmentene i en kurve med flere linjestykker (dvs. alt som er mer
komplisert enn en enkelt rett linje). Derfor trenger vi noe litt mer
avansert, nemlig jevne kurver, eller splines.

Denne artikkelen kommer til å gi en kort innføring i den typen splines
som er kjent som Catmull-Rom-splines. De er relativt raske å beregne, og
er enkle nok matematisk til at selv de mindre erfarne av oss kan følge
med. (Skal du lage demoer, vil du dog aldri komme rundt at du trenger
litt matte -- jeg vil anta mattekunnskaper omtrent på linje med fullført
1. gym på videregående, men mer er alltid en fordel :-) )

Grunnprinsipper
---------------

Akkurat som når du lager en rett kurve, vil du ha behov for enkelte
punkter du på forhånd har satt hvor skal være, altså *kontrollpunktene*
dine. I tilfellet for en lineær kurve vil det kun være y-verdien for
kontrollpunktene som må oppgis, men for Catmull-Rom-spline er man også
avhengig av at man vet *den deriverte* i hvert av kontrollpunktene.
Dette er imidlertid noe de færreste designere liker å sette manuelt
(selv om det gir økt kontroll). Derfor vil jeg mot slutten av artikkelen
presentere en metode for å regne ut den deriverte automatisk basert på
punktene i kurven, som også gir at den andrederiverte er konstant og
altså f'''(x) = 0. (Hvis du ikke har matematiske kunnskaper nok til å
vite hva den andre- og tredjederiverte er, er det ikke nødvendig for å
forstå hvordan man bruker splines; kort sagt vil det si at kurven som
helhet ikke brått endrer krumning i noe punkt.)

En kurve kan teoretisk sett ha flere hundre kontrollpunkter, og vi
forenkler derfor problemet med å bare behandle området mellom to punkter
(et segment) av gangen (så lenge ikke endrer på kontrollpunktene mens
man tegner kurven byr ikke dette på noen større problemer).

Grunnfunksjon
-------------

*(Det er ikke nødvendig å lese dette avsnittet for å kunne bruke
funksjonen seinere. De som ikke er så sterke i matematikk, eller rett og
slett ikke har noen interesse av å følge med på utregningen, kan [hoppe
til den ferdig utregnede formelen](#ferdig).*

Vi ønsker å finne en tredjegradsfunksjon y = ax<sup>3</sup> + bx<sup>2</sup> + cx + d som
tilfredsstiller de kravene vi har satt. Det som er gitt, er punktene
(x<sub>1</sub>,y<sub>1</sub>) (det første punktet) og (x<sub>2</sub>,y<sub>2</sub>) (det andre punktet),
dvs. at vi kjenner f(x<sub>1</sub>) og f(x<sub>2</sub>). I tillegg har vi altså tidligere
regnet ut den deriverte i begge punktene (hvordan dette skjer kommer som
sagt seinere); vi kjenner altså f'(x<sub>1</sub>) og f'(x<sub>2</sub>).

Med andre ord får vi disse to ligningene når vi setter inn hhv. x<sub>1</sub> og
x<sub>2</sub> for x:

1.  ax<sub>1</sub><sup>3</sup> + bx<sub>1</sub><sup>2</sup> + cx<sub>1</sub> + d = f(x<sub>1</sub>)
2.  ax<sub>2</sub><sup>3</sup> + bx<sub>2</sub><sup>2</sup> + cx<sub>2</sub> + d = f(x<sub>2</sub>)

I tillegg ønsker vi som sagt også å spesifisere den deriverte (som
uttrykker stigningstallet, eller hvor fort en kurve stiger eller synker
i et gitt punkt) i begge punktene. Deriverer vi funksjonen ax<sup>3</sup> + bx<sup>2</sup>
+ cx + d får vi funksjonen 3ax<sup>2</sup> + 2bx + c, og vi har dermed disse to
ligningene i tillegg til de to ovenfor (de mindre matematisk kyndige kan
legge merke til at f'(x) betyr den deriverte av f(x) i punktet x):

3.  3ax<sub>1</sub><sup>2</sup> + 2bx<sub>1</sub> + c = f'(x<sub>1</sub>)
4.  3ax<sub>2</sub><sup>2</sup> + 2bx<sub>2</sub> + c = f'(x<sub>2</sub>)

Vi ser at siden vi kjenner x<sub>1</sub>, x<sub>2</sub>, f(x<sub>1</sub>), f(x<sub>2</sub>), f'(x<sub>1</sub>) og
f'(x<sub>2</sub>), er det bare a, b, c og d som ikke er kjent, og hvis vi løser
dette ligningssettet (fire ligninger med fire ukjente) vil vi komme fram
til verdier for disse som vi senere kan legge inn i formelen.

En forenkling
-------------

Hvis man forsøker å løse disse ligningssettene direkte for enhver
situasjon, kommer man fort opp i ekstremt kompliserte og kronglete
formler. Det er derfor til stor nytte å *normalisere* x, dvs. å sette at
den normaliserte variabelen t alltid er mellom 0 og 1 (der x = x<sub>1</sub> vil
gi at t = 0, og x = x<sub>2</sub> vil gi at t = 1 -- t er med andre ord 0 helt på
begynnelsen av kurven, og 1 helt på slutten av kurven). Dette gir en
*annen* kurve (da f.eks. t<sup>2</sup> ikke vil gi det samme resultatet som om
man regnet med x<sub>1</sub><sup>2</sup>), men ikke nødvendigvis noen *dårligere* kurve.
Vi har to hovedfordeler med å regne med variabelen t i stedet for å
bruke x-verdiene direkte:

1.  Punktene lenger til venstre på en kurve vil ikke kunne påvirke
    hvordan kurven ser ut lenger til høyre -- bruker man to punkter med
    gitte verdier, vil den resulterende kurven se identisk ut uansett
    hvor på grafen den befinner seg.
2.  Utregningene under blir *enormt* mye enklere -- om jeg ikke skulle
    brukt de normaliserte verdiene, ville dette i stedet vært en
    tutorial på hvordan man kunne løst ligningssett effektivt med
    matriser og determinanter!

Altså kan vi bruke t i stedet for x i de fire ligningene vi allerede har
definert, og sette inn at t<sub>1</sub> = 0 og t<sub>2</sub> = 1. (Legg merke til hvor
mange av leddene som forsvinner etterhvert når vi kan gjøre det slik.)
Vi starter med den første ligningen og setter inn at t<sub>1</sub> = 0:

a \* 0<sup>3</sup> + b \* 0<sup>2</sup> + c \* 0 + d = f(0)\
d = f(0)

På akkurat samme måte tar vi tak i ligning 2 og setter inn at t<sub>2</sub> = 1:

a \* 1<sup>3</sup> + b \* 1<sup>2</sup> + c \* 1 + d = f(1)\
a + b + c + d = f(1)

I motsetning til ligning 1 gir ikke dette oss noen konstant umiddelbart,
men vi lar den ligge inntil videre og setter opp ligning 3 med t<sub>1</sub> = 0:

3a \* 0<sup>2</sup> + 2b \* 0<sup>2</sup> + c = f'(0)\
c = f'(0)

Vi fortsetter med ligning 4, og setter inn t<sub>2</sub> = 1 som vi har gjort
før:

3a \* 1<sup>2</sup> + 2b \* 1<sup>2</sup> + c = f'(1)\
3a + 2b + c = f'(1)

Denne kan vi jobbe videre med, og vi løser den for b:

2b = f'(1) - 3a - c\
b = (f'(1) - 3a - f'(0))/2

Nå ser vi at vi har verdier for både b, c og d. Vi setter disse inn i
ligningen vi fant ut fra ligning 2, og løser for a:

a + b + c + d = f(1)\
a + (f'(1) - 3a - f'(0))/2 + f'(0) + f(0) = f(1)\
2a + f'(1) - 3a - f'(0) + 2f'(0) + 2f(0) = 2f(1)\
-a + f'(1) + f'(0) + 2f(0) = 2f(1)\
-a = 2f(1) - f'(1) - f'(0) - 2f(0)\
a = -2f(1) + f'(1) + f'(0) + 2f(0)

Det eneste vi mangler nå, er å sette inn denne verdien for a i uttrykket
vi tidligere fant for b:

b = (f'(1) - 3a - f'(0))/2\
b = (f'(1) - 3(-2f(1) + f'(1) + f'(0) + 2f(0)) - f'(0))/2\
b = (f'(1) + 6f(1) - 3f'(1) - 3f'(0) - 6f(0) - f'(0)) / 2\
b = (-2f'(1) + 6f(1) - 4f'(0) - 6f(0)) / 2\
b = -f'(1) + 3f(1) - 2f'(0) - 3f(0)

<p id="ferdig">
Nå har vi alle verdiene vi trenger for formelen vår. Vi setter inn og
får den ferdige funksjonen for splinen vår (jeg har fargelagt de enkelte
konstantene for å gjøre funksjonen lettere å lese/forstå):
</p>

f(x) = *a*t<sup>3</sup> + *b*t<sup>2</sup> + *c*t + *d*\
f(x) = *(-2f(1) + f'(1) + f'(0) + 2f(0))*t<sup>3</sup> + *(-f'(1) + 3f(1) -
2f'(0) - 3f(0))*t<sup>2</sup> + *f'(0)*t + *f(0)*

En del andre splines-tutorials på nettet snur litt om på denne formelen,
slik at man ikke grupperer etter t, men etter de enkelte input-verdiene
man har. Det er uansett i bunn og grunn samme formel, og det er egentlig
ikke noe stort poeng å gjøre formelen vanskeligere å forstå på denne
måten med mindre man kanskje vil spare noen ytterst få
flyttallsoperasjoner (compileren vil i de fleste tilfeller bety vel så
mye her).

Praktisk bruk
-------------

Det å bruke denne formelen i praksis gir seg omtrent nesten selv: Man
finner ut hvilke to kontrollpunkter som er nærmest (på hver sin side)
den x-verdien man har (hvordan man ønsker å håndtere x-verdier som ikke
ligger innenfor kontrollpunktene er en sak i seg selv -- personlig synes
jeg det er helt okay å gi en feil i dette tilfellet, og så passe på at
verdiene aldri går utenfor kontrollpunktene). Deretter normaliserer man
x, ved å si at t = (x - x<sub>min</sub>) / (x<sub>max</sub> - x<sub>min</sub>), der x<sub>min</sub> og
x<sub>max</sub> er henholdsvis venstre og høyre kontrollpunkt. Deretter setter
man enkelt og greit inn verdien for t samt de kjente verdiene inn i
formelen, og man har den ferdige verdien fra splinefunksjonen, som man
da kan bruke til å generere et objekt, animere bevegelse, tegne på
skjermen etc..

Utregning av den deriverte
--------------------------

Som nevnt holder det ikke bare å spesifisere y-verdien i hvert av
punktene, man må også spesifisere en verdi for den deriverte (som altså
sier noe om hvor fort grafen stiger eller synker i akkurat det punktet).
Derfor vil jeg her forklare en enkel metode for å regne ut verdier for
den deriverte i punktene automatisk, slik at man slipper å gjøre det
selv. Endepunktene må behandles for seg (vi skal ta dem for oss
seinere), men for alle andre punkter er framgangsmåten denne:

*Finn stigningen mellom kontrollpunktene til venstre og høyre for
kontrollpunktet du behandler, og bruk dette stigningstallet som den
deriverte.*

En enkel figur skulle forklare dette litt bedre:

![Figur som viser tangenten til en spline-funksjon i krysningspunktet
mellom to subsplines](/img/articles/kode/splineexample.png){width="544"
height="388"}

Her har vi tre punkter, og dermed to segmenter (markert med rød og grønn
farge). Den øverste blå linjen markerer tangenten (husk at
stigningstallet til tangenten er lik den deriverte i dette punktet).
Legg spesielt merke til at denne tangenten er parallell med den nederste
blå linjen, som går fra det første til det tredje kontrollpunktet (det
andre kontrollpunktet er toppunktet der de to segmentene møtes, og det
er i dette punktet vi ønsker å regne ut den deriverte). To parallelle
linjer vil alltid ha samme stigningstall, så vi kan bruke
stigningstallet til den nederste linjen til å regne ut den deriverte i
det øverste kontrollpunktet. (Vi kunne selvfølgelig valgt den deriverte
på en helt annen måte, men som nevnt er dette en veldig grei måte å
gjøre det på, som i tillegg har fordeler mht. jevnhet i kurven o.l.)

Det å regne ut den stigningstallet for den nederste blå linjen er ingen
avansert oppgave. Vi sier at punkt n har koordinatene (x<sub>n</sub>,y<sub>n</sub>), og
skal altså regne ut stigningstallet mellom punkt 1 og punkt 3 (altså
f'(x<sub>2</sub>)):

f'(x<sub>2</sub>) = (y<sub>3</sub> - y<sub>1</sub>) / (x<sub>3</sub> - x<sub>1</sub>)

Randpunkter
-----------

Metoden over fungerer naturlig nok rimelig dårlig i endepunktene, dvs.
det første og det siste punktet, da disse punktene ikke ligger mellom to
andre punkter man kan regne ut fra. Hvordan man skal behandle dette ser
ut til å gjøres litt forskjellig forskjellige steder, og dette har nok
vel så mye med smak og behag å gjøre som noe annet. (Enkelte anbefaler
til og med å velge pseudo-random verdier i endepunktene!)

En nogenlunde grei metode er rett å slett å \`loope' splinen, dvs. å
tenke seg at man setter første og siste punkt sammen, og dermed bruker
nest siste punkt som \`venstre' og første punkt som \`høyre' når man
skal regne ut den deriverte for det første og det siste punktet (disse
vil dermed få den samme deriverte, hvilket absolutt ikke er en dum ting
om du har tenkt å la noe følge den samme splinen flere ganger etter
hverandre). Pass bare på at hvis ikke det første og det siste punktet
ender i samme y-verdi, kan du få litt uventede resultater om du ikke
kompenserer for dette når du regner ut stigningstallet. Med denne
korreksjonen vil formelen for den deriverte i det første og det siste
punktet bli:

f'(x<sub>0</sub>) = f'(x<sub>n</sub>) = ((y<sub>1</sub> - y<sub>0</sub>) + (y<sub>n</sub> - y<sub>n-1</sub>)) / ((x<sub>1</sub> - x<sub>0</sub>) + (x<sub>n</sub> - x<sub>n-1</sub>))

(Det å skrive formler i HTML er ikke akkurat leselig -- hvis alle hadde
støttet [MathML](http://www.w3.org/Math/) ville ting vært litt greiere
;-)

Man regner altså i dette tilfellet ikke direkte fra det ene punktet til
det andre, men går via kontrollpunktet i midten (dette kan også gjøres
for alle de andre punktene, men man vil få nøyaktig samme verdier
uansett, så det er ikke noe stort poeng).
