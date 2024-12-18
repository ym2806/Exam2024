# CineGuard

**CineGuard** er en webapplikasjon som gir informasjon om eksplisitt innhold i filmer og lar brukere vurdere filmer basert på personlige og religiøse verdier. Prosjektet er utviklet som en del av eksamen i PG3402 Mikrotjenester.

## Innholdsfortegnelse

1. [Prosjektoversikt](#prosjektoversikt)
2. [Arkitektur](#arkitektur)
3. [Tjenester](#tjenester)
4. [Kommunikasjonsmønstre](#kommunikasjonsmønstre)
5. [Installasjon og kjøring](#installasjon-og-kjøring)
6. [Brukerhistorier](#brukerhistorier)
7. [Systemarkitekturdiagram](#systemarkitekturdiagram)
8. [Refleksjonsrapport](#refleksjonsrapport)

---

## Prosjektoversikt

**CineGuard** består av flere mikrotjenester som samarbeider for å tilby funksjonalitet for å finne filmer, vise detaljer om eksplisitt innhold, registrere brukerfeedback og sende varsler. Applikasjonen benytter synkron kommunikasjon (REST API) og asynkron kommunikasjon (RabbitMQ) for å sikre en skalerbar og responsiv arkitektur.

---

## Arkitektur

Prosjektet er bygget med en mikrotjenestearkitektur som inkluderer følgende komponenter:

- **Backend**: Java, Spring Boot, Spring Cloud Gateway
- **Asynkron meldingskø**: RabbitMQ
- **Konteinerisering**: Docker, Docker Compose
- **Byggverktøy**: Maven
- **Databaser**: H2 (in-memory)

---

## Tjenester

### 1. Movie Search Service
- **Port**: 8081
- **Funksjonalitet**: Søker etter filmer og gir generell informasjon.
- **Endepunkter**:
    - `GET /api/movies`: Hent alle filmer
    - `POST /api/movies`: Legg til en ny film

### 2. Content Service
- **Port**: 8082
- **Funksjonalitet**: Viser detaljer om eksplisitt innhold i filmer.
- **Endepunkter**:
    - `GET /api/content/title/{movieTitle}`: Hent innholdsinformasjon for en spesifikk film
    - `POST /api/content`: Legg til innholdsinformasjon

### 3. Rating Service
- **Port**: 8083
- **Funksjonalitet**: Registrerer brukerfeedback for filmer.
- **Endepunkter**:
    - `GET /api/ratings`: Hent alle vurderinger
    - `POST /api/ratings`: Legg til en ny vurdering

### 4. Notification Service
- **Port**: 8080
- **Funksjonalitet**: Sender varsler når en ny vurdering blir registrert eller når innholdsinformasjon blir hentet.
- **Kommunikasjon**: Asynkron kommunikasjon via RabbitMQ.

### 5. Gateway Service
- **Port**: 8087
- **Funksjonalitet**: Sentral tilgangspunkt for alle tjenester.
- **Ruter**:
    - `/api/movies/**` -> **Movie Search Service** (http://localhost:8081)
    - `/api/content/**` -> **Content Service** (http://localhost:8082)
    - `/api/ratings/**` -> **Rating Service** (http://localhost:8083)

---

## Kommunikasjonsmønstre

### Synkron kommunikasjon
- **Gateway Service** kommuniserer med **Movie Search Service**, **Content Service** og **Rating Service** via REST API.

### Asynkron kommunikasjon
- **Rating Service** sender meldinger til **Notification Service** via RabbitMQ når en ny vurdering blir registrert.
- **Content Service** sender meldinger til **Notification Service** via RabbitMQ når innholdsinformasjon blir hentet.

---

## Installasjon og kjøring

ør du kjører prosjektet, sørg for at følgende verktøy er installert og konfigurert på maskinen din:

- **Java 21**: [Last ned Java](https://adoptium.net/)
- **Maven**: [Last ned Maven](https://maven.apache.org/download.cgi)
- **Docker** og **Docker Compose**: [Installer Docker](https://docs.docker.com/get-docker/)
- **RabbitMQ**: Kan kjøre via Docker

### **Sjekk installasjoner**

```bash
java -version
mvn -version
docker --version
docker-compose --version
```

---

## **Start RabbitMQ**

Start RabbitMQ ved hjelp av Docker:

```bash
docker run -it --rm -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management
```

RabbitMQ vil nå kjøre på følgende adresser:
- **AMQP-port**: `localhost:5672`
- **Admin GUI**: [http://localhost:15672](http://localhost:15672) (Brukernavn: `guest`, Passord: `guest`)

---

### 1. Klone prosjektet

```bash
git clone https://github.com/ym2806/MicroservicesExam.git
cd CineGuard
```

##  **Kjør alle mikrotjenestene**

Åpne **fem separate PowerShell-vinduer** og kjør hver tjeneste som beskrevet nedenfor.

### **1. Movie Search Service**

```bash
cd backend/movie-search-service
mvn spring-boot:run
```
- **Port**: `8081`

### **2. Content Service**

```bash
cd backend/content-service
mvn spring-boot:run
```
- **Port**: `8082`

### **3. Rating Service**

```bash
cd backend/rating-service
mvn spring-boot:run
```
- **Port**: `8083`

### **4. Notification Service**

```bash
cd backend/notification-service
mvn spring-boot:run
```
- **Port**: `8080`

### **5. Gateway Service**

```bash
cd backend/gateway-service
mvn spring-boot:run
```
- **Port**: `8087`

---

##  **Testing av tjenestene**

Når alle tjenestene kjører, kan du teste funksjonaliteten via **Gateway Service** på `http://localhost:8087`.

### **1. Legg til filmer**

```bash
Invoke-RestMethod -Uri "http://localhost:8087/api/movies" -Method Post -Headers @{ "Content-Type" = "application/json" } -Body '{"title": "Inception", "releaseYear": "2010", "synopsis": "A mind-bending thriller about dreams within dreams."}'
```

### **2. Legg til eksplisitt innhold**

```bash
Invoke-RestMethod -Uri "http://localhost:8087/api/content" -Method Post -Headers @{ "Content-Type" = "application/json" } -Body '{"movieTitle": "Inception", "contentDescription": "Violence and intense sequences", "severity": "Moderate"}'
```

### **3. Legg til vurderinger**

```bash
Invoke-RestMethod -Uri "http://localhost:8087/api/ratings" -Method Post -Headers @{ "Content-Type" = "application/json" } -Body '{"movieTitle": "Inception", "userFeedback": "like"}'
```

### **4. Hent alle filmer**

```bash
Invoke-RestMethod -Uri "http://localhost:8087/api/movies" -Method Get
```

### **5. Hent eksplisitt innhold for en film**

```bash
Invoke-RestMethod -Uri "http://localhost:8087/api/content/title/Inception" -Method Get
```

### **6. Hent alle vurderinger**

```bash
Invoke-RestMethod -Uri "http://localhost:8087/api/ratings" -Method Get
```

---

## **Sjekk databaser**

Hver tjeneste bruker en **H2-database** som kan nås via følgende konsolladresser:

- **Movie Search Service** (Port 8081):  [http://localhost:8081/h2-console](http://localhost:8081/h2-console)
- **Content Service** (Port 8082):       [http://localhost:8082/h2-console](http://localhost:8082/h2-console)
- **Rating Service** (Port 8083):        [http://localhost:8083/h2-console](http://localhost:8083/h2-console)

**H2-databaseinnstillinger:**
- **JDBC URL**: `jdbc:h2:mem:moviedb` (for Movie Search Service) eller tilsvarende for de andre tjenestene
- **Brukernavn**: `sa`
- **Passord**: (tomt)

---

##  **Sjekk RabbitMQ**

Åpne RabbitMQ Admin GUI for å sjekke køer og meldinger:

[http://localhost:15672](http://localhost:15672)

- **Brukernavn**: `guest`
- **Passord**: `guest`

Sjekk følgende køer:
- `notificationQueue`: For meldinger fra **Rating Service**.
- `contentNotificationQueue`: For meldinger fra **Content Service**.

---

---

## Brukerhistorier

1. **Søke etter filmer**
    - Som en bruker vil jeg søke etter filmer for å finne generell informasjon.

2. **Se eksplisitt innhold**
    - Som en bruker vil jeg se detaljer om eksplisitt innhold i filmer for å vurdere om filmen passer for mine verdier.

3. **Gi vurderinger**
    - Som en bruker vil jeg kunne gi vurderinger ("like" eller "dislike") for filmer.

4. **Motta varsler**
    - Som en bruker vil jeg motta en bekreftelse når min vurdering er registrert.

---

## Systemarkitekturdiagram

```plaintext
             +-----------------------+       
             |    Gateway Service    |       
             |    (Port: 8087)       |       
             +----------+------------+       
                        |                    
    +-------------------+------------------+
    |          |            |              |
+---v---+  +---v---+    +---v---+     +---v---+
| Movies|  |Content|    |Rating |     | Notif- |
|Service|  |Service|    |Service|     |Service |
|(8081) |  |(8082) |    |(8083) |     |(8080)  |
+-------+  +-------+    +-------+     +--------+
    |            |            |              ^
    |            |            +--------------+
    |            |             RabbitMQ (Async)
    v            v
H2 Databaser  H2 Databaser
```

---

## Refleksjonsrapport

**Arkitekturvalg:**
- Jeg valgte en mikrotjenestearkitektur for å oppnå løs kobling, skalerbarhet og enklere vedlikehold.

**Teknologivalg:**
- **Spring Boot** for rask utvikling av REST API-er.
- **RabbitMQ** for asynkron kommunikasjon mellom tjenester.
- **Spring Cloud Gateway** som API Gateway for å sentralisere tilgang til backend-tjenestene.

**Trade-offs:**
- Bruken av H2-database er enkel for utvikling, men ville blitt byttet ut med en mer robust database eller API key fra IMDB i produksjon.
- Asynkron kommunikasjon gir bedre ytelse, men legger til kompleksitet.

**Erfaringer:**
- Å implementere mikrotjenester ga god forståelse av hvordan tjenester kan kommunisere effektivt.
- Å konfigurere RabbitMQ, Docker og Spring Cloud Gateway var utfordrende, men lærerikt.
- Å undersøke og eksperimentere hvordan tjenester kan kommunisere med hverandre var som et puslespill.
---
**Framtidige implementeringer:**
- React-basert UI: Utvikle et brukervennlig grensesnitt med React for å gjøre det enklere for brukere å søke etter filmer, se innholdsdetaljer og gi vurderinger.
- Interaktive Komponenter: Implementere søkefelt, filtre og dynamiske visninger for å forbedre brukeropplevelsen.
---
**Ekstra Funksjonalitet**
- Anbefalingssystem: Legge til et system for å anbefale filmer basert på brukerens tidligere vurderinger.
- Varslinger: Implementere e-post- eller SMS-varslinger for brukere som ønsker oppdateringer om bestemte filmer.
- Internasjonalisering (i18n): Støtte flere språk for å gjøre applikasjonen tilgjengelig for et bredere publikum.
---
