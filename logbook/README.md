Forneden er en række datoer. Som beskriver en given arbejdsuge i praktikperioden.
Alle datoer skrevet forneden skal aflæses som værende i året 2024.
Formatet skal aflæses som: Fra dag / måned til og med dag / måned i året 2024.

## 02/01 - 05/01
- Introduceret til kollegaer. Inkluderende alle afdelinger (chefer, salg, marketing, design, support & udvikling.). Jeg vil ikke komme til at have direkte professionel dialog med, nødvendigvis, alle afdelinger. Men grundet det åbne kontorlandskab og den lille størrelse på firmaet, omgås jeg alle kollegaer til dagligt.
- Installation af nødvendige teknologier til at udføre mig arbejde som fullstack udvikler.
	- .NET Framework 4.8 SDK
	- .NET 8.0 SDK
	- IDE: Jetbrains Rider
	- Node.js
	- GIT
	- Docker Desktop
		- MongoDB Image
	- MongoDB DBMS: Studio 3T
	- Google Chrome
		- Vue.js devtools extension
	- Slack
	- MS Teams
	- MS Outlook
	- Azure Service Bus Explorer
	- Azure CLI
	- Azurite
	- Self-Hosted Azure Agent
- Sat ind i arbejdsflowet som fullstack udvikler i udviklingsteamet. Vi bruger en variation af et Kanban flow, som udviklingsstrategi: Resultatet er en kontinuerlig proces fra at *tasks* bliver identificeret, tidsestimeret og prioriteret, til at de bliver implementeret, testet og udrullet. Hvilket betyder at konceptet om *sprints* er fuldstændig ud af vinduet.

## 08/01 - 12/01
- Første uge hvor jeg for alvor har prøvet at arbejde selvstændigt, som en fullstack udvikler i udviklingsteamet.
- Jeg har taget tasks fra vores projektstyrings board på Azure DevOps. Identificeret hvad implementeringskravene er. Gennemført & valideret de krav. Hvorefter jeg har initialiseret en pull request med de ændringer jeg har skabt op mod vores "staging" miljø, som giver lejlighed til at en anden udvikler verificerer mit arbejde.

## 15/01 - 19/01

- Virtualisering af frontend projektet (vue.js)
	- Udfordringer med et *gammelt* projekt og alle dets afhængigheder og transitive afhængigheder.
	- Jeg prøvede at udføre de samme kommandoer som vores build-pipeline på Azure.
		- Men fik overhovedet ikke samme resultat. Jeg tænker også at det har en delvis grund i at vi bruger *self-hosted agents* til at køre vores azure pipelines direkte på vores arbejdscomputere. Hvilket betyder at det baserer sig på det arbejdsmiljø vores computere har, samt at det er et windows operativ system vi altid bruger til at genererer de *artifacts* som bliver distribueret.
		- Det har været enormt svært at genskabe frontenden som et distribuerklart *artifact* på en virtuel maskine i form af Alpine Linux eller Ubuntu.
- Første task der krævede ændringer i frontend, backend såvel som database.
	- Nyt område: 
		- Querying i en mongo database.
			- Mongoshell
		- Ændring af datamodel
	- Få feature til at være bagud-kompatibel med eksisterende dokumenter i databasen

## 22/01 - 26/01

- Første par dage på kontoret uden CTO eller senior udvikler til stede. Mig og en kollega måtte påtage ansvar vi ikke har prøvet før, for at få dagligdagen til at fungere som normalt (fedt!).
	- Kvalitetssikre kodebase gennem pull request reviews - Jeg måtte gennemse andres pull requests for første gang.
- Knoklet på at få skub i en masse tasks der havde overrumplet vores Kanban board. Vi har et "maksimum" antal tasks der må være i en given status; og den var overskredet. TLDR: Mange tasks der var sat i gang under implementering og aktive pull requests, men ikke færdiggjort.
- Sat på nogle opgaver som ingen andre i firmaet har ekspert viden inden for (fedt!)
	- Virtualisér self-hosted agents til vores pipeline jobs.
	- Skriv *Infrastructure as Code* til nogle azure resourcer, så vi kan genskabe miljøer, og vi har dokumentation på resource kravene (versioner, service konfigurationer, etc.).
		- Bicep template filer (Microsoft's Azure Resource Manager Infrastructure as Code Powershell filtyper)
	- Få deployed vores IaC resourcer gennem et azure pipeline job. Som skal køres på en dockerized self-hosted agent.
- Der er også denne uge jeg er blevet introduceret til en masse ting vedrørende Azure. Jeg har aldrig rigtig prøvet at arbejde med cloud providers før. Værktøjer i punktform jeg har vedrørt denne uge:
	- Services
		- Container Registry
		- Key Vault
		- Resource Groups
	- Azure Resource Manager (ARM)
		- Templates
			- Bicep files
	- Access Control
		- Service Principles
		- Service Connections
		- Subscriptions
		- Users
	- Azure DevOps
		- Pipelines
			- Agents
			- Agent Pools
			- Yaml files
		- Repositories
		- Projekter
		- Organisationer
		- Users
		- Personal Access Tokens
	- Azure CLI

## 29/01 - 02/02

- Mere dybdegående arbejde med *self-hosted agents*.
	- *Dockerized self-hosted agent* baseret på *alpine linux* (så lille et image som muligt, da vi betaler for pladsen på azure container registry).
	- Installering af python pakker på alpine linux system-wide og virtuelle miljøer
	- Installering af azure cli på alpine linux
	- Azure pipelines (alt efter yaml'en) har en række krav til den agent der skal gennemføre den. Pipeline krav op mod agent *capabilities*.
- Dybdegående arbejde med *azure pipelines*
	- yaml syntaksen ligger fundament på azp *tasks*. Såsom `AzureCLI@2` eller `DotNetCoreCLI@2`.
		- Nogle *tasks* kræver pre-installed software (*agent capabilities*), og andre gør ikke - i stedet installerer de den påkrævede software midlertidigt. Én af de *task* typer er `UseDotnet@2` i kombination med `DotNetCoreCLI@2`.
	- Authentication af agenten der eksekverer pipelinen, på en sikker måde (ingen blottet credentials).
		- Oftest gjort igennem den pågældende `service connection` som er knyttet til pipelinen.
			- Som giver en implicit authentication til alle *tasks* i pipelinen
		- Hentning af `azure key vault` værdier, instantieret som variabler (kun med levetid for pipelinen), til brug til hhv. `az login --service-principal <...>`.
- Arbejde med *azure container registry*
	- Kobling mellem `Docker@2` azp task og acr.
	- Multi-tagging af bygget images før de skubbes op til acr. Således man kan have et dynamisk tag (e.g. `latest`) der altid refererer til den nyeste version.
		- F.eks.: `tags: | latest $(Build.BuildId)` i azp
	- Grundlæggende forståelse for container image registries.
		- Images bliver komprimmeret på cloud registriet - for at spare plads. Det er derfor, når man *puller* et image lokalt, skal det *extractes*.
		- Images gemmes som multiple *manifester*. Som beskriver stadier på systemet i imaget.
			- Dette betyder at nye images der bliver skubbet til acr, sammenlignes med eksisterende manifester. Dermed bliver der undgået redundans, og kun nye *manifester* gemmes. Dette giver høj hukommelses effektivitet.

## 05/02 - 09/02

- Nyt Azure Functions projekt i .Net 8
	- Forskellige typer function triggers. Primært HTTP (genkendeligt til en klassisk Asp .Net Web Api).
	- Authorization level: Function-niveau eller ingen.
		- I et staging & prod miljø køres med function-niveau authorization. Som typisk er en *nøgle* der bliver sendt med *http requesten* mod den givne *route*. Ses ofte som en http header: `x-function-key: <secret>`.
- Virtualisering af en eksisterende Azure Functions applikation.
	- Konfigurering af applikationen til et `staging` miljø lokalt. Da det hidtil kun har kørt i en distribueret facon.
- Ydermere arbejde med Azure Pipelines. Til formål for at lave et CI / CD flow der: *tester*, *bygger for release*, *laver et artifakt ud af byg resultat*, *skubber artifakt op på artifact store*, *laver et docker image ud fra artifact* og *skubber docker image op på et container registry*.
	- Hertil arbejdede jeg også med at følge gode retningslinjer for en ordentlig pipeline: Identificere hvilke `jobs` er uafhængige af hinanden, og kan køre parallelt af den årsag. Samt separere `jobs` ud i forskellige `stages`, således at vi lettere kan drifte og identificere mulige fejl i pipelinen.

## 12/02 - 16/02

- Arbejdet med authentication på diverse resourcer i monolitten.
- Fået sat et stort *builder pattern* op for objekter i vores domæne i monolitten.
	- Til formål for hurtigere at få instantieret relevante komplekse objekter til unit tests i monolitten.
	- Op til nu, kan vi nogle have haft hundredevis af linjer med mocking i *hver* unit test klasse.
		- Det vil kunne optimeres med et builder pattern.
		- Samt er det langt bedre *maintainable* hvis domænet ændrer sig eller ekspanderer.

## 19/02 - 23/02

- Afholdt møde vedrørende bachelor projekt
- Fået lavet første udkast til et bachelor projekt, indeholdende følgende punkter:
	- Problemformulering
	- En indledning: Eksisterende workflow / problem der sigtes til forbedring
	- Løsningen: Softwaremæssigt hvad jeg kan udrette som kan forbedre ovenstående
- Dykket ned i Selenium.
	- Distribueret med Docker
- Lavet PoC med selenium, til at påvise at vi kan lave E2E tests på portalen.

## 26/02 - 01/03

- Lavet systemdokumentation til E2E test systemet.
	- Stort system diagram der viser afhængigheder, arkitektur, teknologier, eksekvering, protokoller.
- Skrevet IaC i form af Bicep op mod Azure.
	- Herunder med bl.a. azure container instances, virtuelle netværker, storage accounts & storage types.

## 4/03 - 08/03

- Vi ville sætte en nextcloud op for vores udviklingsafdeling
- Undersøgelse af nextcloud (bag kulisserne)
- Undersøgelse af distribueringsmuligheder for vores nextcloud
	- En fuldt ud distribueret stil der gjorde brug af Azure's (out-of-the-box) services, ville være enormt overkill og dyrt. Da det krævede enormt meget overhead, for bare at nævne nogle få ting:
		- Web Service med alt der hører til (backup, storage, sidecars, etc.)
		- Database
		- Cache
		- Load balancing
	- Distribueret (med egen konfiguration) på Azure Container Instances og Azure Container Apps
		- Jeg fik noget op at køre på ACI, men det var suboptimalt. Og efter nærmere undersøgelse, er ACA det mere moderne og rette at gå med.
- Indtil vi får noget fedt op at køre på ACA, satte jeg en nextcloud op på en Azure VM.