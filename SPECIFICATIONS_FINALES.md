# SPECIFICATIONS FINALES - JARVIS

## Statut du document

Ce document consolide les arbitrages fonctionnels et techniques validés lors de la phase finale de cadrage.

Il constitue la base de reference de la V1/MVP.

Les zones encore ouvertes ne bloquent pas la definition du socle initial. Elles sont explicitees dans la section `Questions differees` pour approfondissement ulterieur.

## Source de cadrage

- Vision source : `docs/architecture_vision.md`
- Mode de cadrage retenu : consolidation iterative des decisions, avec reduction volontaire du nombre de questions en fin de session

## Vision produit retenue

Jarvis est une plateforme d'assistant personnel local-first, gouvernee, multi-canal a terme, construite autour d'un orchestrateur central.

Le MVP ne cherche pas a couvrir toute la vision cible. Il valide d'abord le socle metier : orchestration, gouvernance, routage local/cloud, memoire utile et exposition via un premier client.

## Principes directeurs confirmes

- local-first par defaut
- cloud seulement quand il apporte une vraie valeur
- gouvernance explicite des decisions et des capacites
- cerveau unique, clients interchangeables a terme
- backend API-first, independant du frontend
- architecture simple, auditable et evolutive

## Perimetre du MVP retenu

Le MVP correspond au `socle metier`.

### Inclus dans le MVP

- orchestrateur central en `Python + FastAPI`
- routage via `LiteLLM`
- modele local via `Ollama`
- memoire via `SQLite + sqlite-vec`
- `policy engine`
- premier client via `Open-WebUI`

### Explicitement hors MVP

- `Home Assistant`
- `XTTS`
- `n8n`
- `Telegram`
- `WhatsApp`
- ecriture et actions externes non triviales
- connecteurs externes reels, meme en lecture

## Architecture fonctionnelle du MVP

### 1. Orchestrateur central

L'orchestrateur est le coeur du systeme. Il porte les responsabilites suivantes :

- reception des requetes du client
- gestion du contexte de session
- application des politiques de gouvernance
- decision de routage local ou cloud
- gestion des lectures/ecritures memoire autorisees
- exposition des metadonnees de decision au client

L'orchestrateur reste le proprietaire final des regles. `LiteLLM` n'est pas le cerveau du produit.

### 2. Routage local/cloud

Le routage suit une logique `hybride`.

- des pre-filtres deterministes sont appliques dans l'orchestrateur
- une classification LLM ciblee est utilisee pour les cas ambigus
- la decision finale reste gouvernee par l'orchestrateur

### 3. Priorite de decision

Le critere principal de routage est le `type d'action`.

Toutefois, la specification retient aussi un garde-fou obligatoire :

- le `niveau de risque` peut surclasser la decision lorsque la securite ou la gouvernance l'exigent

### 4. Voie cloud

La voie cloud est disponible des la V1, mais :

- elle est desactivable par configuration
- si elle est desactivee et qu'une requete depasse le perimetre local autorise, Jarvis doit refuser explicitement
- ce refus doit indiquer qu'une capacite cloud existe, sans tentative implicite de contournement local

## Identite, utilisateur et sessions

### Identite V1

Le MVP fonctionne avec une `identite unique fixe`.

- un seul utilisateur logique cote backend
- pas de multi-compte dans la V1
- pas de granularite utilisateur riche transmise par `Open-WebUI`

### Proxy de confiance

`Open-WebUI` est traite comme `proxy de confiance` pour le MVP.

- le backend Jarvis ne gere pas encore une couche d'authentification utilisateur complete
- la securite du MVP repose sur l'environnement d'hebergement et la protection de `Open-WebUI`

### Sessions

Le modele de session retenu est `hybride`.

- Jarvis accepte une session amont si elle est fournie
- sinon Jarvis peut creer sa propre session backend
- le backend garde la maitrise du contexte court

## Memoire

### Modele general

Le modele memoire retenu est `mixte`.

- memoire longue attachee a l'utilisateur logique
- contexte court attache a la session
- metadonnees de canal prevues architecturalement, meme si le MVP reste centre sur `Open-WebUI`

### Donnees persistees dans le MVP

La memoire persistante conserve :

- faits utilisateur
- preferences utilisateur
- resumes conversationnels
- embeddings utiles associes aux elements de memoire longue valides

La memoire persistante ne conserve pas, par defaut :

- historique conversationnel brut complet

### Ecriture memoire

L'ecriture memoire est `automatique` dans le MVP.

Elle doit toutefois rester filtree par regles.

Le critere fonctionnel de memorisation doit reposer sur :

- la durabilite de l'information
- son utilite future probable pour l'assistant

### Donnees exclues de la memoire automatique

Jarvis ne doit pas memoriser automatiquement :

- secrets, cles, tokens, credentials
- donnees personnelles sensibles
- informations temporaires ou sans valeur durable
- brouillons et demandes ponctuelles non structurantes

### Correction et suppression memoire

Le mecanisme retenu est `conversationnel uniquement`, avec comportement `tracable`.

- l'utilisateur peut demander d'oublier, corriger ou remplacer une information
- Jarvis doit viser l'element memoire concerne avec une precision raisonnable
- Jarvis doit confirmer explicitement ce qui a ete modifie, remplace ou supprime

### Contexte de session court

La retention du contexte de session est `moyenne duree`.

- conservation sur plusieurs jours
- objectif : reprise fluide d'une conversation interrompue
- distinction maintenue entre contexte court et memoire longue

### Production des resumes

La production des resumes conversationnels suit un mode `hybride`.

- mise a jour opportuniste quand une information durable emerge
- consolidation periodique
- pas d'ecriture systematique a chaque tour

### Embeddings

Les embeddings sont generes `uniquement pour la memoire longue validee`.

## Policy Engine

Le `policy engine` est `hybride`.

- invariants critiques dans le code de l'orchestrateur
- parametres ajustables dans une configuration versionnee
- pas de gouvernance dynamique en base au MVP

## Regles de capacites dans le MVP

### Lecture locale autorisee

Le MVP autorise la `lecture d'outils bornes` en principe, mais sans connecteur externe reellement branche en V1.

En pratique, cela signifie que le MVP exploite seulement :

- la memoire Jarvis
- le contexte de session

### Definition d'une lecture bornee

Une lecture bornee est definie de facon `hybride` :

- source explicitement autorisee
- volume maximal controle
- intents de lecture autorises et identifiables

### Ecriture et actions externes

Les modifications et actions externes non triviales sont `interdites en MVP`.

## Open-WebUI

### Role dans le MVP

`Open-WebUI` est un `client enrichi`.

- interface conversationnelle principale
- affichage de metadonnees utiles
- pas de role de gouvernance metier majeur

### Metadonnees visibles

Le niveau de visibilite retenu est `standard`.

L'interface doit pouvoir montrer :

- mode `local` ou `cloud`
- motif de routage
- statut memoire pertinent

### Statut memoire visible

Le statut memoire visible doit etre `utile` :

- memoire utilisee
- memoire mise a jour
- memoire ignoree

### Reponses

Le MVP doit supporter un `streaming enrichi`.

- texte genere de maniere incrementale
- metadonnees de routage et de memoire exposees en parallele

### Format fonctionnel du streaming SSE

Le streaming SSE doit utiliser des evenements separes par type.

Exemples de familles d'evenements attendues :

- `token`
- `routing`
- `memory`
- `done`
- `error`

## API backend

La structuration retenue est `hybride`.

- facade externe simple pour la V1
- structuration interne preparee pour extension

### Exposition officielle minimale V1

Le backend doit exposer officiellement :

- un endpoint principal de chat
- des endpoints simples de memoire
- des endpoints simples de diagnostic
- un endpoint de sante minimal

### Endpoints de diagnostic

Les endpoints de diagnostic du MVP doivent permettre au minimum de consulter :

- l'etat de sante du service
- l'etat des composants critiques
- un resume de la configuration active non sensible

### Mode d'appel principal

Le mode d'appel principal entre `Open-WebUI` et Jarvis est :

- `HTTP` synchrone
- `streaming SSE` pour les reponses incrementales et les metadonnees associees

## Modeles LLM

### Cloud

Le parametrage cloud est en `double niveau`.

- des familles ou capacites de modeles cloud recommandes doivent etre specifiees
- ils doivent pouvoir etre remplaces par configuration

### Local

Le parametrage local est egalement en `double niveau`.

- des familles ou capacites de modeles locaux recommandes doivent etre specifiees
- il doit pouvoir etre remplace par configuration

## Deploiement et configuration

### Deploiement cible

Le format de deploiement prioritaire du MVP est `local docker-compose`.

### Configuration et secrets

Le modele retenu est :

- configuration non sensible dans des fichiers versionnes
- secrets via variables d'environnement

## Persistance SQLite

La persistance du MVP repose sur une separation `logique` dans une meme base SQLite.

Cela implique :

- une base SQLite unique au MVP
- une separation claire des espaces de donnees entre memoire et traces de decision
- une structure suffisament nette pour permettre une evolution ulterieure sans confusion des responsabilites

## Observabilite et auditabilite

Le niveau retenu pour le MVP est `logs + traces de decision`.

### Logs

- logs applicatifs structures
- traces de routage
- traces des politiques appliquees
- audit des refus, escalades et usages memoire significatifs

### Niveau de detail des traces

Le niveau de trace est `quasi-audit`.

Chaque decision importante doit pouvoir conserver :

- decision finale
- motif lisible
- regle ou politique declenchee
- niveau de confiance utile
- fallback eventuel
- statut des garde-fous appliques

## Gestion des refus

Les refus de capacite doivent etre `guides`.

Ils doivent contenir :

- un refus clair et concis
- une raison fonctionnelle compréhensible
- une suggestion de contournement, d'alternative ou d'activation si pertinent

## Strategie de tests

Le niveau d'exigence retenu est `tests unitaires + integration backend`.

### Couverture cible minimale

- tests unitaires sur `policy engine`, routage, memoire
- tests d'integration sur les principaux flux de l'orchestrateur

## Comportement en panne

La strategie globale est `politique par composant`.

### Memoire indisponible

Si `SQLite/sqlite-vec` est indisponible :

- mode degrade possible pour une conversation simple non dependante de la memoire
- refus propre pour les requetes dependantes de la lecture/ecriture/correction memoire
- fallback trace dans les journaux de decision

### Ollama indisponible

Si `Ollama` est indisponible et que la voie cloud est activee :

- bascule cloud `conditionnelle`
- pas de substitution automatique universelle du local par le cloud

## Arbitrages structurants a retenir

- le MVP valide le coeur orchestral avant les integrations externes
- la gouvernance reste dans l'orchestrateur, pas dans `LiteLLM` ni dans le client
- la memoire apporte une vraie valeur des la V1, mais sans archive brute complete
- l'architecture prepare l'evolution multi-canale sans la livrer des le MVP

## Points de friction assumes

### 1. Type d'action vs niveau de risque

Le `type d'action` est le critere principal de routage, mais un garde-fou de risque doit rester capable de surclasser une decision.

### 2. Identite unique fixe en V1

Ce choix simplifie fortement le socle, mais reporte explicitement la future ouverture multi-utilisateur et multi-canal.

### 3. Proxy de confiance Open-WebUI

Ce choix accelere le MVP, mais differe une vraie couche d'identite backend.

### 4. Memoire automatique

Ce choix maximise la valeur assistant, mais impose des filtres stricts pour eviter sur-memorisation et erreurs durables.

## Questions differees

Ces sujets pourront etre approfondis ulterieurement sans bloquer la definition du socle MVP :

- choix precis des modeles locaux recommandes
- choix precis des modeles cloud recommandes
- schema fin des tables SQLite et des index `sqlite-vec`
- contrat exact des endpoints API
- evolution du modele d'identite pour les futurs canaux
- integration future de `n8n`
- integration future de `Home Assistant`
- integration future de `XTTS`
- extension future a `Telegram` et `WhatsApp`

## Definition finale du MVP

La V1 de Jarvis est un backend orchestral gouverne, local-first, expose via `Open-WebUI`, capable de :

- recevoir une requete conversationnelle
- gerer un contexte de session
- utiliser une memoire longue utile et filtree
- router de facon gouvernee entre local et cloud
- expliquer ses decisions de facon lisible
- produire des traces exploitables pour audit et stabilisation

Ce MVP ne vise pas encore la pleine richesse de la vision cible. Il vise a prouver un socle propre, simple, robuste et extensible.
