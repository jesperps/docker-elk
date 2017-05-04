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

#### 4. Skilland på docker-compose och docker-kommandon
Docker-compose kan ses som ett lite mer sammanhållet verktyg för att hålla reda på flera containrar och hur de skall köras. För _compose_-kommandon finns det ganska ofta en direkt motsvarighet till ett _docker_-kommando. Och du är inte begränsad till att bara använda compose-kommandon om du använder compose, _vanliga_ docker-kommandon går också bra.
- Märker du nån skilland på att använda `docker-compose ps` och `docker ps`

#### 5a. Definiera en egen service
Så fort en service finns definierad i [compose-filen](docker-compose.yml) kan du börja göra saker med den med compose-kommandon.
- Gör en kopia av logstash-servicen
- Verifiera med `config`- och/eller `ps`-kommandona
- I servicens [command](https://docs.docker.com/compose/compose-file/compose-file-v2/#command)-del kan du definiera ett eget kommando, prova att använda `ping` (inte `docker-compose ping`) för att se om du kan få kontakt med elasticsearch

#### 5b. Logga in på en startad container
När allt fungerar som det skall körs kommandot som definierats i _command_-delen tills det är klart sedan stoppas containern. När den sedan staras igen har den inget minne av vad som har gjorts vid tidigare körningar, vilket är ett utmärkt sätt att  garantera att allt som servicen är tänkt att göra finns definierat (och kommer att fungera oavsett var någonstans servicen körs ifrån).

Men det kan i felsökningssyfte vara vettigt att kunna logga in på en startad container för att prova att göra saker manuellt.
- Använd `exec`-kommandot för att starta `bash`
- I den inloggade containern, prova att köra något valfritt kommando t.ex. det som den andra logstash-service startar eller `ls` eller `touch fil` (inget `docker-compose` framför här)
- Lista kommando-historiken i containern (med `history` utan `docker-compose` framför) och  avsluta sessionen med `<ctrl>+d`
- Om du nu stannar containern, loggar in (med `exec`) och listar historiken bör du kunna verifiera att containern inte kommer ihåg vad du gjorde sist
- Hur kan du säkerställa att data som containern skriver blir kvar efter containerns nedtagning?

#### 6. Resursutnyttjande
Det kan med vanliga _linux_-kommandon från hosten som kör dina docker-containrar vara svårt att se exakt vilken container som förbrukar mest resurser.
- Prova att uttyda vad t.ex. `top` (utan `docker-compose` före) säger för något
- Det finns även ett `docker stats`-kommando som visar startade containerars resursförbrukning
- Försök att använda `docker stats` för att enbart lista förbrukning för de containrar som är definierade i din compose-fil
 - **Tips** `docker-compose ps|grep -v Name|grep -v ^\-|cut -d ' ' -f1` listar containernamnen i ett compose-context


 #### 7. Loggning
 Om du startar dina containrar med `docker-compose up` utan några argument kommer det som händer i containrarna att ekas ut på skärmen. Detta är sannolikt helt ok om du endast skall ha containrarna startade en begränsad tid. Men i en drift-situation vill du sannolikt dels kunna söka i loggen vid ett senare tillfälle och från en annan session än den som kör docker-compose.
 - Stoppa dina containrar om dom är igång
 - Ta upp dom i _detachat_-läge
 - Använd `logs`-kommandot för att visa samtliga containrars loggar allt eftersom de loggar
 - Prova att enbart visa loggar för nån enstaka container
 - Försök att hitta ett sätt att enbart visa loggar för en viss typ av container, t.ex. samtliga logstash-containrar (som troligen heter _logstash_+nånting)
  - **Tips** `docker-compose config --services` listar namnen på dina servicars
