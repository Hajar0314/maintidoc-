# MaintiDoc : assistant documentaire multimodal et agentique pour la maintenance

**Livrable 1 sur 6 : Cadrage, corpus et architecture**
Module IA générative multimodale, EMSI 2026 · Enseignante : Mariam CHERRABI

| | |
|---|---|
| **Équipe** | BELHACHMI, ZIDANI, LANOUARI |
| **Dépôt GitHub** | URL sur la ligne unique du fichier `L1_MaintiDoc_BELHACHMI_ZIDANI_LANOUARI_LienGitHub.txt` |
| **Période** | Semaine 1 (constitution du corpus du 2 au 6 novembre 2026) |

---

## 1. Résumé du projet

MaintiDoc est un assistant destiné aux techniciens et aux responsables de maintenance. Il exploite des manuels, des procédures, des schémas et des photographies pour répondre à des questions techniques avec des **références vérifiables** (document, page, figure). Il peut consulter l'historique des interventions d'un équipement et préparer un ticket de maintenance. **La création effective d'un ticket exige une validation humaine.**

Exemple de demande type :

> « Voici une photo du panneau de commande. Que signifie ce voyant orange ? Consulte les dernières interventions et prépare un ticket si nécessaire. »

MaintiDoc est une aide documentaire. Il ne pilote aucun équipement, ne garantit aucun diagnostic et ne remplace ni les procédures de sécurité ni un technicien habilité.

## 2. Contexte, utilisateurs et cas d'usage

**Contexte.** Sur un site disposant d'une alimentation de secours, les techniciens consultent plusieurs documents (manuels, tableaux de codes d'alarme, historiques) pour comprendre un voyant ou une alarme. Les messages se ressemblent d'un équipement à l'autre, les procédures changent de version, et l'information est dispersée. MaintiDoc centralise cette recherche et force la citation des sources.

**Utilisateurs visés.**

| Profil | Besoin | Accès |
|---|---|---|
| Technicien de maintenance | Comprendre un voyant ou un code, trouver la bonne procédure, relever les dernières interventions | Groupe `techniciens_energie` |
| Responsable de maintenance | Mêmes besoins, plus les retours d'expérience et les bulletins techniques, validation des tickets | Groupe `responsables` |

**Cas d'usage concrets.**

1. *Voyant orange sur le groupe électrogène.* Le technicien envoie la photo du panneau ; MaintiDoc identifie GE-01, lit le code (par exemple GE-W04 Low Battery) et cite le manuel avec la page et la figure.
2. *Message ambigu entre deux équipements.* « Battery Low » existe sur GE-01 (code GE-W04) et sur UPS-01 (code A101). Si l'en-tête de la photo n'est pas visible, l'assistant demande l'identifiant de l'équipement avant de répondre.
3. *Transfert trop lent sur l'inverseur de sources.* Un transfert de 12 s au lieu de 7 s : l'assistant relie l'alarme T201 au paramètre TD de l'ATS-01 et cite la procédure en vigueur.
4. *Procédures contradictoires.* Deux versions d'une même procédure donnent des valeurs différentes : l'assistant signale la contradiction et indique la version en vigueur au lieu de choisir au hasard.
5. *Préparation d'un ticket.* Après une alarme récurrente, l'assistant consulte l'historique via l'outil MCP, rédige un brouillon de ticket, le présente à l'utilisateur et ne le crée qu'après approbation explicite.

## 3. Les équipements du projet

Le corpus couvre **deux familles** qui forment une seule chaîne d'alimentation de secours. Tous les équipements, constructeurs, numéros de série, valeurs et tickets sont **fictifs** (constructeur « VoltaSecure » inventé) et destinés à un usage pédagogique uniquement.

```
Réseau 400 V ──► ATS-01 (inverseur de sources) ──► UPS-01 (onduleur) ──► charges critiques
                    │   ▲                      └──► charges non protégées
          ordre de  │   │ 400 V
          démarrage ▼   │
                  GE-01 (groupe électrogène)
```

### 3.1 Famille 1 : groupe électrogène GE-01

Un moteur diesel entraîne un alternateur et produit une tension triphasée. Le groupe démarre automatiquement lors d'une coupure du réseau, sur ordre de l'ATS-01. Un panneau de commande affiche l'état, les mesures et les alarmes (messages en anglais).

| Caractéristique | Valeur (fictive) |
|---|---|
| Puissance, tension, fréquence | 100 kVA, 400 V triphasé, 50 Hz |
| Cuve de gasoil | 200 L (alerte sous 20 %, soit 40 L) |
| Batterie de démarrage | 24 V (deux batteries 12 V en série), alarme sous 23,5 V |
| Huile moteur | 15W-40, 12 L ; pression normale 3,0 à 5,0 bar, arrêt sous 1,5 bar |
| Refroidissement | 18 L ; température normale 75 à 95 °C, pré-alarme à 95 °C, arrêt à 105 °C |
| Démarrage automatique | 3 tentatives de 10 s, pause de 10 s |

| Code | Message | Nature |
|---|---|---|
| GE-E01 | START FAILURE | Arrêt de protection (bandeau rouge) |
| GE-E02 | LOW OIL PRESSURE | Arrêt de protection |
| GE-W03 / GE-E03 | HIGH COOLANT TEMP | Pré-alarme / arrêt |
| GE-W04 | LOW BATTERY | Avertissement (bandeau orange) |
| GE-W05 | FUEL LEAK | Avertissement |
| GE-W06 | LOW FUEL LEVEL | Avertissement |

### 3.2 Famille 2 : inverseur de sources ATS-01

L'ATS-01 (VoltaSecure TS-160) surveille le réseau et le groupe. Lors d'une coupure, il ordonne le démarrage de GE-01, transfère les charges sur le groupe après un délai réglable (TD), puis revient sur le réseau au retour de celui-ci.

| Caractéristique | Valeur (fictive) |
|---|---|
| Courant, tension, fréquence | 160 A, 400 V triphasé + neutre, 50 Hz, 4 pôles |
| Firmware | 2.4 (mise à jour depuis 2.1) |
| Réglages du site | Mains fail delay 2 s, 3 tentatives de démarrage, TD 5 s, retransfer 120 s, cooldown 180 s |
| Température maximale des contacts | 75 °C (version 1.0 de la procédure) |

| Code | Message | Panne associée |
|---|---|---|
| T101 | Transfer Failed | Ne bascule pas |
| T102 | Gen Start No Signal | Ne bascule pas (groupe absent) |
| T201 | Transfer Delay Out of Range | Bascule trop tard (TD trop long) |
| T301 | Contactor Fault | Contact brûlé |
| T302 | Contact Temp High | Contact brûlé (échauffement) |

### 3.3 Famille 2 : onduleur UPS-01

L'UPS-01 (VoltaSecure VS-40T) est un onduleur à double conversion : il protège les charges critiques contre les coupures et les perturbations du réseau grâce à une batterie, et dispose d'un bypass.

| Caractéristique | Valeur (fictive) |
|---|---|
| Puissance | 40 kVA / 36 kW, triphasé 400 V, 50 Hz, rendement 94 % |
| Batterie | 32 blocs VRLA 12 V 100 Ah, bus DC 384 V |
| Autonomie | 15 min à 80 % de charge (exigence minimale de 10 min dans la v1.0) |
| Température interne maximale | 45 °C |

| Code | Message | Panne associée |
|---|---|---|
| A101 | Battery Low | Batterie faible (bus DC sous 345 V ou autonomie sous 10 min) |
| A201 | Overload | Surcharge |
| A301 / A302 | Fan Fault / Over Temperature | Ventilateur bloqué, surchauffe |
| A401 | On Bypass | Charge passée sur le bypass (non protégée) |
| A501 / A601 | Mains Fail / Battery Test Failed | Réseau absent, test d'autonomie échoué |

**Règle de lecture** : les codes `Txxx` appartiennent à l'ATS, les codes `Axxx` à l'UPS, les codes `GE-` au groupe. Les mots « batterie faible », « surchauffe » et « coupure réseau » désignent des pannes différentes selon l'équipement : c'est la principale source d'ambiguïté prévue pour l'évaluation.

## 4. Périmètre du projet

| Élément | Périmètre |
|---|---|
| Entrées | Questions en français, photographies, documents PDF |
| Sorties | Réponses sourcées, demandes de clarification, brouillons de tickets |
| Outil externe | Service de maintenance de démonstration accessible via MCP |
| Action sensible | Création d'un ticket après approbation explicite |
| Hors périmètre | Pilotage de machines, diagnostic garanti, interventions autonomes |

## 5. Architecture cible

```
Interface web : question + photographie
        │
API FastAPI : authentification et autorisation
        │
Workflow LangGraph
   │              │               │
RAG multimodal   Modèle VLM      Client MCP
(LangChain)      à poids ouverts    │
   │                            Serveur MCP maintenance
Recherche texte et images        ┌────────┴────────┐
   │                         Historique        Tickets (avec accord)
Reranking adapté
   │
Vérification des références et du format
   │
Réponse sourcée  /  clarification  /  validation humaine
```

| Bloc | Rôle | Outil envisagé (à préciser) |
|---|---|---|
| Interface et API | Recevoir la question et la photo, authentifier l'utilisateur, appliquer les droits | Streamlit, FastAPI |
| Orchestration | Gérer l'état, les transitions, les limites de boucle et la pause pour validation humaine | LangChain, LangGraph |
| Ingestion | Extraire le texte, appliquer l'OCR aux scans, découper en passages et éléments visuels | PyMuPDF, OCR local |
| Recherche | Recherche textuelle et visuelle, fusion des résultats, reclassement | Qdrant, BGE-M3, encodeur type CLIP, Reciprocal Rank Fusion |
| Modèle vision-langage | Décrire la photo, identifier l'équipement, rédiger la réponse à partir des extraits | Qwen2.5-VL-7B-Instruct |
| Outil MCP | Historique, tickets ouverts, création de ticket | Serveur MCP de démonstration |
| Stockage | Métadonnées et fichiers | PostgreSQL, MinIO |
| Suivi et déploiement | Expériences, métriques, conteneurs | MLflow, Docker Compose |

Les versions exactes, les licences des outils et la mémoire GPU mesurée seront figées dans le dépôt à partir de la semaine 2.

**Points de contrôle.** (1) Droits d'accès : avant la recherche, puis à l'ouverture des documents et des images. (2) Validation humaine : avant toute création de ticket. (3) Vérification des citations : avant l'envoi de la réponse, pour s'assurer que chaque référence existe et soutient l'affirmation.

**Parcours d'une requête.**

1. Vérification de l'identité et des groupes de l'utilisateur.
2. Analyse de la question et de la photo par le modèle vision-langage.
3. Recherche textuelle et visuelle, uniquement dans les documents autorisés.
4. Consultation de l'historique via MCP si nécessaire.
5. Rédaction de la réponse avec ses sources, puis vérification des citations.
6. Réponse, ou demande de clarification si la photo est illisible ou les sources insuffisantes.
7. Si un ticket est demandé : brouillon, approbation humaine, création via MCP avec clé d'idempotence.

## 6. Flux de données et références

**Ingestion.** Les documents sont découpés en passages textuels et en éléments visuels (paragraphes, tableaux, pages, figures, schémas). L'OCR complète les documents scannés ; les images originales restent disponibles pour le modèle vision-langage. Chaque élément conserve : identifiant et version du document, page, identifiant de l'élément, type d'équipement, groupes d'accès autorisés.

**Citations.** Elles pointent vers les pages et figures réellement récupérées. Une référence valide doit aussi soutenir l'affirmation associée. Les sorties sont validées par un schéma structuré (statut, réponse, citations, incertitudes, question de clarification, action proposée).

**Création de ticket.** Préparation du brouillon, affichage du contenu exact, approbation liée à l'utilisateur et au contenu, appel MCP, clé d'idempotence contre les doublons. Toute modification du ticket impose une nouvelle approbation.

## 7. Gestion de l'incertitude et sécurité

L'assistant distingue les informations établies par les sources, les observations visuelles et les hypothèses non confirmées. Il demande une clarification ou s'abstient quand la photo est illisible, quand aucune source pertinente n'est accessible, quand les documents se contredisent ou quand la demande dépasse son rôle documentaire. Il n'affiche pas de pourcentage de confiance inventé.

| Exigence | Mesure prévue |
|---|---|
| Secrets | Injectés à l'exécution, exclus de Git, des prompts et des journaux |
| Données personnelles | Minimisation, anonymisation, durée de conservation définie |
| Contrôle documentaire | Filtrage par utilisateur et groupe avant récupération, puis à l'affichage |
| Outils | Autorisations vérifiées côté serveur MCP à chaque appel |
| Documents non fiables | Documents et résultats d'outils traités comme des données, jamais comme des instructions |
| Injection de prompt | Tests dans les textes, les images OCR et les réponses MCP |
| Budgets | 3 appels MCP, 6 appels au modèle et 60 s de traitement automatique au maximum par demande |

## 8. Ce que le projet doit produire : tâches générales et planning

| Semaine | Travaux | Livrable associé |
|---|---|---|
| 1 | Cadrage, constitution du corpus, architecture, flux de données, analyse des risques | **Livrable 1** (ce dossier) |
| 2 | Ingestion (PyMuPDF, OCR), recherche textuelle et visuelle, métadonnées et gestion des accès | Dépôt documenté |
| 3 | Intégration du modèle vision-langage, citations vérifiables, sorties structurées | Prototype |
| 4 | Workflow LangGraph, serveur MCP, validation humaine du ticket | Prototype |
| 5 | Adaptation d'un petit reranker multilingue, évaluation (environ 150 scénarios), tests d'injection | Évaluation |
| 6 | Déploiement Docker Compose, optimisations, documentation, soutenance | Fiche de limites, démonstration |

Les six livrables du projet : cadrage et architecture, dépôt documenté, prototype, évaluation, fiche de limites, démonstration (scénario nominal, ambiguïté, injection, accès refusé, approbation de ticket).

**Évaluation prévue** : questions textuelles, questions avec image, consultation MCP, sources absentes ou contradictoires, contrôle d'accès, injections. Indicateurs : Recall@5, nDCG@5, MRR@10, validité des citations, abstentions justifiées, fuites entre groupes, actions non approuvées, latences médiane et p95. Les objectifs sont des cibles à vérifier, pas des résultats acquis.

**Fine-tuning ciblé** : un petit reranker multilingue, entraîné sur 800 à 1 500 exemples vérifiés (question, passage pertinent, passage négatif difficile), comparé à l'absence de reranker et au reranker d'origine. Exemple de cas difficile : distinguer une alarme de température du groupe d'un voyant similaire de l'onduleur.

## 9. Le corpus documentaire (livrable de la semaine 1)

### 9.1 Chiffres clés

| Indicateur | Valeur |
|---|---|
| Fichiers inventoriés | 149 (53 documents PDF et 96 images) |
| Documents PDF | 47 PDF texte et 6 PDF scannés (pour tester l'OCR) |
| Éléments visuels exploitables | 51 documents avec tableaux, figures ou schémas, plus 96 images |
| Types | manuel 17, procédure 16, historique 9, codes d'alarme 4, schéma 4, fiche de sécurité 3 ; images : 94 photos et 2 schémas |
| Documents publics de fabricants | 8 (APC, Socomec, GELEC ENERGY, Korman, EMSA) |
| Documents synthétiques | 141 (documents rédigés avec Claude, images générées avec Gemini) |
| Langues | français (100), anglais (48, surtout les écrans des images), néerlandais (1, signalé dans l'inventaire) |

### 9.2 Provenance et usage de l'IA

- **Synthétiques** : provenance et licence `synthetique` ; prompts numérotés P01 à P45 (documents) et I01 à I97 (images).
- **Publics** : provenance indiquée dans l'inventaire (URL ou site du fabricant), licence d'usage pédagogique non commercial.
- **Traçabilité** : `prompts_corpus.md` regroupe les prompts avec outil et date, la provenance, la licence, la relecture et l'analyse critique des erreurs et incohérences repérées. La relecture croisée est répartie à parts égales entre les trois membres.
- L'IA est un outil, pas une source : les faits techniques sont recoupés entre documents (identifiants, codes, pages).

### 9.3 Inventaire (`inventaire.csv`)

UTF-8, séparateur virgule, une ligne par fichier ; chaque fichier de `documents/` et `images/` y figure, et inversement. Colonnes : `document_id`, `fichier`, `titre`, `famille_equipement`, `equipement_id`, `type_document`, `version`, `nb_pages`, `langue`, `format`, `elements_visuels`, `provenance`, `licence`, `genere_par_ia`, `outil_ia`, `verifie_par`, `corrections`, `groupe_acces`, `usage_test`, `remarques`.

- `document_id` : minuscules et `_`, unique, stable, réutilisé dans les citations.
- `equipement_id` : GE-01, ATS-01, UPS-01 ; plusieurs valeurs séparées par `;` ; `GENERIQUE` pour les manuels publics.
- `type_document` : valeurs de la liste officielle ; quand aucune ne correspond exactement (FAQ, checklist, nomenclature, fiche technique), le type le plus proche est utilisé et précisé dans `remarques`.

### 9.4 Modèle d'accès

| Groupe | Documents accessibles |
|---|---|
| `techniciens_energie` | tous les fichiers sauf `bulletin_technique_ge01` |
| `responsables` | tous les fichiers, dont `bulletin_technique_ge01`, réservé à ce groupe pour tester la fuite entre groupes |

### 9.5 Cas prévus pour les tests de la semaine 5

| Rôle (`usage_test`) | Nombre | Exemples |
|---|---|---|
| `contradictoire` | 10 | vidange `v1` contre `v2` du groupe (huile, volume, couple) ; démarrage `v1.0` contre scan `v0.8` ; `f2_pro_ats_01` contre `_02` ; `f2_pro_ups_01` contre `_02` ; FAQ qui citent encore les valeurs de la v1.0 |
| `injection` | 2 | `note_service_ge01` et `f2_not_ups_01` : l'annexe technique contient une instruction malveillante, à traiter comme donnée |
| `ambigu` | 9 | voyant WARN orange à 4 codes possibles, « Battery Low » commun à GE-01 et UPS-01, photos floues ou sans en-tête |
| `normal` | 128 | questions à réponse unique et sourçable |

## 10. Contenu du ZIP

```
L1_MaintiDoc_BELHACHMI_ZIDANI_LANOUARI.zip
└── L1_MaintiDoc_BELHACHMI_ZIDANI_LANOUARI/
    ├── L1_MaintiDoc_BELHACHMI_ZIDANI_LANOUARI_Rapport.pdf     rapport complet (parties 2.1 à 2.6)
    ├── L1_MaintiDoc_BELHACHMI_ZIDANI_LANOUARI_LienGitHub.txt  URL du dépôt GitHub
    ├── README.md                                               ce fichier
    └── L1_MaintiDoc_BELHACHMI_ZIDANI_LANOUARI_Corpus/
        ├── inventaire.csv
        ├── prompts_corpus.md
        ├── documents/
        │   ├── famille_1/        groupes électrogènes (19 PDF)
        │   └── famille_2/        ATS et UPS (34 PDF, sous-dossiers ATS/ et UPS/)
        └── images/
            ├── famille_1/        36 images
            └── famille_2/        60 images (sous-dossiers ATS/ et UPS/)
```

## 11. Structure du dépôt GitHub

```
maintidoc/
├── README.md
├── .gitignore                exclut secrets et fichiers .env
├── docs/
│   └── diagrammes/           sources des diagrammes (draw.io, Mermaid)
└── corpus/
    ├── inventaire.csv
    ├── prompts_corpus.md
    ├── documents/{famille_1, famille_2}/
    └── images/
```

Aucun secret (clé d'API, mot de passe) n'est présent dans le ZIP ni dans le dépôt.

## 12. Liste de vérification avant dépôt

- [x] Chaque fichier du corpus figure dans `inventaire.csv`, et inversement
- [x] `prompts_corpus.md` présent (prompts, outil, date, provenance, licence, analyse critique)
- [x] Au moins 2 documents contradictoires et 1 document piégé, signalés dans l'inventaire
- [x] Aucun secret dans le ZIP
- [x] Rapport PDF ajouté et ouvert pour contrôle
- [x] Lien GitHub testé depuis un compte extérieur à l'équipe
