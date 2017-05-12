# docker-elk
## Elastic-stack med en knapptryckning

### Snabbkomigång:

```bash
docker-compose up
```
Tre servicar har definierats i [docker-compose-filen](docker-compose.yml), kommandot ovan startar dessa.

### Steg för steg
**OBS! Om inget annat anges är kommandon som nämns nedan _docker-compose_-kommandon**, dvs `docker-compose <kommando>`.

#### 1. Olika compose-kommandon och compose-filen
Docker-compose är ett verktyg för att hålla ihop och definiera hur docker-containrar skall köras. Detta definieras som servicar i [compose-filen](docker-compose.yml), det finns ett antal `docker-compose`-kommandon som interagerar med compose-filen. Vilka dessa är framgår genom att ange `--help` som argument eller genom att titta i [referens-dokumentationen](https://docs.docker.com/compose/reference/). Det finns även [referens-dokumentation](https://docs.docker.com/compose/compose-file/compose-file-v2/) om compose-filens struktur.
- Ägna några minuter att se vilka kommandon som finns tillgängliga och ta en titt på [compose-filen](docker-compose.yml)

#### 2. Miljöspecifika parametrar
Det går bra att köra flera compose-uppsättnignar på samma host, vilket kan vara vettigt för att kunna prova olika inställningar. Men vissa av hostens resurser kan enbart utnyttjas av en uppsättnig i taget, t.ex. kan ett portnummer användas av endast en tjänst i taget och data som skall skrivas från en container till en yta på hosten bör vara unik för att undvika att flera containrar skriver över varandras data.

Det går att ställa in uppsättningsspecifika parametrar i [`.env`-filen](.env):
```
ELASTICPORT1=9200
ELASTICPORT2=9300
ELASTICDATA=./elasticsearch/data
ELASTICLOGS=./elasticsearch/logs
KIBANAPORT=5601
```
- Prova att ändra dessa
- Verifiera ditt resultat med `config`-kommandot

#### 3. Upp och nedtagning av servicar
Många compose-kommandon tar emot servicenamn som argument, detta kan då anges som ett enstaka eller flera stycken åt gången, anges inget servicenamn brukar samtliga servicar behandlas.
- Starta dina servicar med `up`-kommandot
- Prova att stoppa en eller ett par servicar med `stop`
- Verifiera ditt resultat med `ps`
- Finns det nån skillnad på att använda `up` respektive `down` jämfört med `start` respektive `stop`?

#### 4. Verifiera att det kommer loggar till kibana
- Om allt är igång kan du prova om du får nåt svar från kibana via http://localhost:5601 (ersätt med det portnummer du har i din `.env`-fil)
- Första gången du startar kibana behöver ett `index pattern` konfigureras
  - en sida för det bör visas, `logstash-*` föreslås som pattern och det går bra att trycka på `create`-knappen
- Klicka på discover-fliken, exempelloggar bör synas här (om inte, kan du prova att ställa in tidsfiltret uppe i högra hörnet)

#### 5. Skilland på docker-compose och docker-kommandon
Docker-compose kan ses som ett lite mer sammanhållet verktyg för att hålla reda på flera containrar och hur de skall köras. För _compose_-kommandon finns det ganska ofta en direkt motsvarighet till ett _docker_-kommando. Och du är inte begränsad till att bara använda compose-kommandon om du använder compose, _vanliga_ docker-kommandon går också bra.
- Prova att starta en webbserver med `docker run -dit -p 9251:80 httpd:2.4` (ersätt om nödvändigt portnummret `9251` mot något annat)
- Märker du nån skilland på att använda `docker-compose ps` och `docker ps`
- Du kan ta bort webbservercontainer genom att stoppa den med `docker stop`-kommandot och sedan använda `docker rm`-kommandot

#### 6a. Definiera en egen service
Så fort en service finns definierad i [compose-filen](docker-compose.yml) kan du börja göra saker med den med compose-kommandon.
- Gör en kopia av logstash-servicen
- Verifiera med `config`- och/eller `ps`-kommandona
- I servicens [command](https://docs.docker.com/compose/compose-file/compose-file-v2/#command)-del kan du definiera ett eget kommando, prova att använda `ping` (inte `docker-compose ping`) för att se om du kan få kontakt med elasticsearch

#### 6b. Logga in på en startad container
När allt fungerar som det skall körs kommandot som definierats i _command_-delen tills det är klart sedan stoppas containern. När den sedan staras igen har den inget minne av vad som har gjorts vid tidigare körningar, vilket är ett utmärkt sätt att  garantera att allt som servicen är tänkt att göra finns definierat (och kommer att fungera oavsett var någonstans servicen körs ifrån).

Men det kan i felsökningssyfte vara vettigt att kunna logga in på en startad container för att prova att göra saker manuellt.
- Använd `exec`-kommandot för att starta `bash`
- I den inloggade containern, prova att köra något valfritt kommando t.ex. det som den andra logstash-service startar eller `ls` eller `touch fil` (inget `docker-compose` framför här)
- Lista kommando-historiken i containern (med `history` utan `docker-compose` framför) och  avsluta sessionen med `<ctrl>+d`
- Om du nu stannar containern, loggar in (med `exec`) och listar historiken bör du kunna verifiera att containern inte kommer ihåg vad du gjorde sist
- Hur kan du säkerställa att data som containern skriver blir kvar efter containerns nedtagning?

#### 7. Resursutnyttjande
Det kan med vanliga _linux_-kommandon från hosten som kör dina docker-containrar vara svårt att se exakt vilken container som förbrukar mest resurser.
- Prova att uttyda vad t.ex. `top` (utan `docker-compose` före) säger för något
- Det finns även ett `docker stats`-kommando som visar startade containerars resursförbrukning
- Försök att använda `docker stats` för att enbart lista förbrukning för de containrar som är definierade i din compose-fil
 - **Tips** `docker-compose ps|grep -v Name|grep -v ^\-|cut -d ' ' -f1` listar containernamnen i ett compose-context


 #### 8. Loggning
 Om du startar dina containrar med `docker-compose up` utan några argument kommer det som händer i containrarna att ekas ut på skärmen. Detta är sannolikt helt ok om du endast skall ha containrarna startade en begränsad tid. Men i en drift-situation vill du sannolikt dels kunna söka i loggen vid ett senare tillfälle och från en annan session än den som kör docker-compose.
 - Stoppa dina containrar om dom är igång
 - Ta upp dom i _detachat_-läge
 - Använd `logs`-kommandot för att visa samtliga containrars loggar allt eftersom de loggar
 - Prova att enbart visa loggar för nån enstaka container
 - Försök att hitta ett sätt att enbart visa loggar för en viss typ av container, t.ex. samtliga logstash-containrar (som troligen heter _logstash_+nånting)
  - **Tips** `docker-compose config --services` listar namnen på dina servicars


#### 9. Dockerfile och bygga din egen image
En container skapas utifrån en _image_. En image kan ses som en mall-maskin innehållandes det mesta som en viss typ av container behöver för att kunna starta. Det skulle t.ex. bli väldigt tidskrävande att behöva installera en viss typ av mjukvara varje gånge en container startar, det är bättre att ha denna förinstallerad i en image. Hittills har vi använt fördefinierade imagar från [Docker Hub](https://hub.docker.com/), ett så kallat publikt registry för docker imagar.

Det går även bra att bygga egna imagar att ha lokalt på hosten som kör docker eller på ett privat registry (t.ex. [artifactory](https://www.jfrog.com/confluence/display/RTF/Docker+Registry)).

För att bygga en image behövs en så kallad `Dockerfile`. Denna består i sin tur av instruktioner, precis som compose-filen, formatet finns beskrivet i [referens-dokumentation](https://docs.docker.com/engine/reference/builder/). Men det viktigaste är `FROM` som talar om vilken image som skall användas som grund för den nya imagen (det går även skapa helt från grunden). Dvs:

```
$ cat myimage/Dockerfile
FROM httpd:2.4
$
```

- Skapa en katalog för din Dockerfil och fyll filen med innehåll enligt ovan

För att imagen skall skapas behöver den byggas, vilket lämpligen definieras i compose-filen:

```
simpleweb:
        build:
            context: ./myimage
        ports:
            - 9250:80
```

För att kommunicera med containern behöver en portmappning läggas till, ovan konfigsnutt talar om att port `80` till containern går att nå genom att anropa hosten som kör docker på `9250`.

- Lägg till ovan konfigsnutt i compose-filen (ersätt portnummer och katalog med dom värden du vill använda)
- Verifiera med `config`-kommandot
- Starta med `up simpleweb`
- Verifiera att containern fungerar genom att peka din webbläsare mot http://localhost:9250 (ersätt om nödvändigt port till ett lämpligt värde)

#### 10. Lägg till innehåll till din container
En container som bara innehåller data som någon annan har tagit fram är sannolikt inte så värst användbar. Det finns två sätt att lägga till egen data på:
1. Vid bygge av image med `COPY` eller `ADD`
2. Vid start av container med `VOLUME`

##### COPY
- Skapa en egen `index.html`-fil, förslagsvis på samma ställe som din `Dockerfile`
- Använd `COPY` för att lägga filen till containerns `/usr/local/apache2/htdocs/`
- Bygg om imagen, skapa om containern och starta (går att göra med ett kommando eller tre)
- Verifiera att filen finns via webbläsaren

##### VOLUME
- Skapa en katalog, t.ex. `content` som underkatlog till katalogen med Dockerfilen
- Skapa en annan fil i denna katalog
- Dela ut denna katalog som volume i service-specen i compose-filen
- Skapa containern och starta den
- Verifiera att din fil finns via webbläsaren
- Prova nu att lägga till ytterligare filer till katalogen
- Uppdatera webbläsaren och se om de nya filerna är tillgängliga

##### Funderingar
- Finns det för och nackdelar med respektive metod?
- Vad kan vara typiska användningsfall för de olika metoderna?

#### 11. Peka ut plats för loggfiler för logstash
Syftet med logstash är att omvandla text i ett godtyckligt format till en datastruktur som elasticsearch förstår. Logstash består av tre huvuddelar, `input`, `filter` och `output`. Dessa specificeras i en [konfigurationsfil](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html):

```
input {
  ...
}
filter {
  ...
}
output {
  ...
}
```

I inputdelen talar du bland annat om var någonstans logstash hittar loggar. Och för att detta skall fungera behöver loggarna tillgängliggöras för logstashcontainern på något sätt.

Det finns många sätt för logstash att ta emot loggdata, hur detta görs bestäms genom att använda en eller flera [pluginer](https://www.elastic.co/guide/en/logstash/current/input-plugins.html).  

Om en applikation skriver loggar till fil och logstash är installerad i samma container kan `file`-pluginen användas och ingen hänsyn utöver att speca en `PATH` i inputdelen behöver tas.

Om logstash inte kan installeras där applikationen körs kan `file`-pluginen ändå användas om  loggarna skrivas till en delad filarea som hosten som kör logstashcontainern har åtkomst till, i detta fall kan det vara rimligt att använda en _volume_ (vilket görs av logstash-servicen som är definierad i compose-filen).

 - Gör en ny kopia av den ursprungliga logstash-sdervicen i compose-filen
 - Kopiera demologgarna till en lämplig plats på filsystemet, t.ex. `/tmp/minaloggar`
   - Lägg till en egen volume-definition i compose-filen som pekar ut denna plats
