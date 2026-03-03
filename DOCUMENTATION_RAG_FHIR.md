# RAG avec FHIR - Récupération et Génération
## Guide complet sans base de données vectorielle

---

## Table des matières
1. [Qu'est-ce que RAG?](#quest-ce-que-rag)
2. [Pourquoi RAG avec FHIR?](#pourquoi-rag-avec-fhir)
3. [Architecture RAG simple](#architecture-rag-simple)
4. [Méthodes de recherche sans base vectorielle](#méthodes-de-recherche)
5. [Stratégies de fetch FHIR](#stratégies-de-fetch)
6. [Exemples concrets](#exemples-concrets)

---

## Qu'est-ce que RAG?

### Définition

**RAG** = **R**etrieval **A**ugmented **G**eneration

**Traduction** : Génération Augmentée par Récupération

### Les 3 composants de RAG

```
┌─────────────────────────────────────────────────┐
│  1. RETRIEVAL (Récupération)                    │
│  Chercher et trouver les documents pertinents   │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│  2. AUGMENTATION (Enrichissement)               │
│  Combiner les documents avec la question        │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│  3. GENERATION (Génération)                     │
│  Créer une réponse basée sur les documents      │
└─────────────────────────────────────────────────┘
```

### Exemple simple

**Question**: "Quels patients ont une tension artérielle élevée?"

**Étape 1 - Récupération**:
```
Chercher dans les données FHIR:
→ Trouver toutes les Observations
→ Filtrer: code = "8480-6" (PA systolique)
→ Filtrer: valeur > 140 mmHg
→ Résultat: 3 observations trouvées
```

**Étape 2 - Augmentation**:
```
Pour chaque observation:
→ Récupérer le patient associé
→ Récupérer les détails (nom, âge, etc.)
→ Créer un contexte enrichi

Contexte créé:
- Patient 1: Marie Dubois, 45 ans, PA=152/95
- Patient 2: Jean Martin, 62 ans, PA=145/88
- Patient 3: Sophie Bernard, 58 ans, PA=148/92
```

**Étape 3 - Génération**:
```
Utiliser le contexte pour générer une réponse:

"J'ai identifié 3 patients avec une tension artérielle 
élevée (>140 mmHg):

1. Marie Dubois (45 ans) : 152/95 mmHg
2. Jean Martin (62 ans) : 145/88 mmHg
3. Sophie Bernard (58 ans) : 148/92 mmHg

Ces patients nécessitent un suivi médical pour 
l'hypertension."
```

### Différence avec une simple recherche

**Recherche classique**:
- Retourne des documents bruts
- L'utilisateur doit les lire et interpréter
- Pas de synthèse

**RAG**:
- Analyse les documents
- Synthétise l'information
- Répond en langage naturel
- Contextualise la réponse

---

## Pourquoi RAG avec FHIR?

### Les défis des données médicales

**Volume de données**:
- Un patient peut avoir des centaines de ressources FHIR
- Un hôpital a des millions de ressources
- Impossible de tout charger en mémoire

**Complexité**:
- Les ressources FHIR sont interconnectées
- Relations complexes (Patient → Observation → Practitioner)
- Codes médicaux standardisés (LOINC, SNOMED)

**Questions médicales**:
- "Quels patients diabétiques ont une HbA1c > 8%?"
- "Montrez-moi tous les scanners thoraciques du dernier mois"
- "Quels médicaments prend ce patient?"

### Solution RAG

RAG permet de:
1. **Chercher** efficacement dans les ressources FHIR
2. **Récupérer** seulement ce qui est pertinent
3. **Générer** des réponses compréhensibles

---

## Architecture RAG simple

### Vue d'ensemble

```
┌──────────────────────────────────────────────────┐
│         CORPUS DE RESSOURCES FHIR                │
│                                                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐         │
│  │ Patient │  │ Patient │  │ Patient │  ...    │
│  └─────────┘  └─────────┘  └─────────┘         │
│                                                  │
│  ┌──────────────┐  ┌──────────────┐            │
│  │ Observation  │  │ Observation  │  ...       │
│  └──────────────┘  └──────────────┘            │
│                                                  │
│  ┌──────────────┐  ┌──────────────┐            │
│  │ ImagingStudy │  │ ImagingStudy │  ...       │
│  └──────────────┘  └──────────────┘            │
└───────────────────┬──────────────────────────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  QUESTION UTILISATEUR │
        │                       │
        │ "Montrez-moi tous les │
        │  patients avec une PA │
        │  > 140 mmHg"          │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │   MODULE RETRIEVAL    │
        │   (Récupération)      │
        │                       │
        │  1. Parser question   │
        │  2. Identifier        │
        │     critères          │
        │  3. Chercher dans     │
        │     corpus FHIR       │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  RÉSULTATS PERTINENTS │
        │                       │
        │  - 3 Observations     │
        │  - 3 Patients liés    │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  MODULE AUGMENTATION  │
        │                       │
        │  1. Enrichir données  │
        │  2. Créer contexte    │
        │  3. Formatter pour    │
        │     génération        │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  CONTEXTE ENRICHI     │
        │                       │
        │  Patient: Marie...    │
        │  PA: 152/95          │
        │  Date: 15/01/2024    │
        │  ...                 │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  MODULE GENERATION    │
        │  (LLM ou règles)      │
        │                       │
        │  Créer réponse en     │
        │  langage naturel      │
        └───────────┬───────────┘
                    │
                    ▼
            ┌───────────────┐
            │   RÉPONSE     │
            │   FINALE      │
            └───────────────┘
```

### Composants clés

#### 1. Storage (Stockage)

**Organisation des fichiers**:
```
/fhir_data/
├── patients/
│   ├── patient_001.json
│   ├── patient_002.json
│   └── patient_003.json
├── observations/
│   ├── obs_001.json
│   ├── obs_002.json
│   └── obs_003.json
└── imaging/
    ├── study_001.json
    └── study_002.json
```

**Index simple (optionnel)**:
```
index.json:
{
  "patients": {
    "pat-001": "patients/patient_001.json",
    "pat-002": "patients/patient_002.json"
  },
  "observations": {
    "obs-001": "observations/obs_001.json",
    "obs-002": "observations/obs_002.json"
  }
}
```

#### 2. Retrieval Engine (Moteur de recherche)

**Fonction**: Trouver les ressources pertinentes

**Méthodes**:
- Recherche par ID
- Recherche par type de ressource
- Recherche par champs spécifiques
- Recherche par références

#### 3. Context Builder (Constructeur de contexte)

**Fonction**: Assembler les informations pour le LLM

**Actions**:
- Lier les ressources entre elles
- Résoudre les références
- Enrichir avec métadonnées
- Formatter pour la génération

#### 4. Generator (Générateur)

**Fonction**: Créer la réponse finale

**Options**:
- LLM (GPT, Claude, etc.)
- Règles basées sur templates
- Hybride (règles + LLM)

---

## Méthodes de recherche sans base vectorielle

### Pourquoi sans base vectorielle?

**Bases vectorielles** (comme Pinecone, Weaviate):
- Complexes à mettre en place
- Coût supplémentaire
- Nécessitent maintenance

**Alternatives simples**:
- Suffisantes pour beaucoup de cas
- Plus faciles à comprendre et maintenir
- Pas de dépendances externes

### Méthode 1: Recherche par filtres natifs

**Principe**: Utiliser les champs FHIR standard pour filtrer

**Exemple - Trouver patients masculins**:
```
ÉTAPE 1: Charger toutes les ressources Patient
↓
ÉTAPE 2: Filtrer où gender = "male"
↓
RÉSULTAT: Liste de patients masculins
```

**Critères de recherche possibles**:

**Pour Patient**:
- Genre (gender)
- Nom de famille (name.family)
- Date de naissance (birthDate)
- Ville (address.city)
- Statut actif (active)

**Pour Observation**:
- Code LOINC (code.coding.code)
- Patient associé (subject.reference)
- Date (effectiveDateTime)
- Valeur (valueQuantity.value)
- Statut (status)

**Pour ImagingStudy**:
- Patient (subject.reference)
- Modalité (series.modality)
- Date (started)
- Statut (status)

**Workflow**:
```
Question: "Patients nés après 1990"
    ↓
Identifier le critère:
  - Type: Patient
  - Champ: birthDate
  - Condition: > "1990-01-01"
    ↓
Charger tous les Patient
    ↓
Pour chaque patient:
  Si patient.birthDate > "1990-01-01":
    Ajouter à résultats
    ↓
Retourner résultats
```

### Méthode 2: Recherche textuelle simple

**Principe**: Chercher du texte dans les ressources

**Cas d'usage**:
- Chercher un nom
- Chercher un diagnostic
- Chercher une description

**Exemple - Chercher "thorax"**:
```
ÉTAPE 1: Charger ressources ImagingStudy
↓
ÉTAPE 2: Pour chaque ressource
  Regarder dans:
  - series.description
  - series.bodyPartExamined
↓
ÉTAPE 3: Si texte contient "thorax"
  → Ajouter aux résultats
↓
RÉSULTAT: Tous les examens thoraciques
```

**Amélioration: Normalisation**:
```
Texte original: "THORAX SANS INJECTION"
    ↓
Normaliser:
  - Mettre en minuscules: "thorax sans injection"
  - Enlever accents: "thorax sans injection"
  - Tokenizer: ["thorax", "sans", "injection"]
    ↓
Recherche devient plus flexible
```

### Méthode 3: Recherche par TF-IDF

**TF-IDF** = Term Frequency - Inverse Document Frequency

**Principe**: Mesurer l'importance des mots dans les documents

**Comment ça marche**:

**1. Term Frequency (TF)**:
```
"À quelle fréquence ce mot apparaît dans CE document?"

Document 1: "diabète diabète glucose"
  - "diabète" apparaît 2 fois
  - "glucose" apparaît 1 fois
```

**2. Inverse Document Frequency (IDF)**:
```
"Ce mot est-il rare ou commun dans TOUS les documents?"

Si "diabète" apparaît dans 2 documents sur 100:
  → C'est un mot assez spécifique
  → Score IDF élevé

Si "patient" apparaît dans 95 documents sur 100:
  → C'est un mot très commun
  → Score IDF faible
```

**3. Score TF-IDF**:
```
Score = TF × IDF

Mot rare ET fréquent dans le document = Score élevé
Mot commun = Score faible
```

**Utilisation pour RAG**:
```
Question: "diabète insuline"
    ↓
Calculer TF-IDF de tous les documents
    ↓
Trouver documents avec scores élevés pour:
  - "diabète"
  - "insuline"
    ↓
Classer par pertinence
    ↓
Retourner top 5 documents
```

**Avantages**:
- Pas besoin de base vectorielle
- Rapide à calculer
- Fonctionne bien pour texte médical

**Limites**:
- Ne comprend pas le sens (sémantique)
- "diabète" et "diabétique" sont considérés différents
- Nécessite pré-traitement du texte

### Méthode 4: Recherche par références FHIR

**Principe**: Suivre les liens entre ressources

**Structure des références FHIR**:
```
Observation:
{
  "subject": {
    "reference": "Patient/pat-001"
  }
}

Cette observation concerne le patient pat-001
```

**Exemple - Trouver toutes les observations d'un patient**:
```
Patient: "pat-001"
    ↓
ÉTAPE 1: Charger toutes Observations
    ↓
ÉTAPE 2: Pour chaque Observation
  Si subject.reference = "Patient/pat-001":
    → Ajouter aux résultats
    ↓
RÉSULTAT: Toutes observations du patient
```

**Exemple - Chaînage de références**:
```
Question: "Quel médecin a prescrit ce médicament?"

MedicationRequest → référence → Practitioner
    ↓
ÉTAPE 1: Trouver MedicationRequest
    ↓
ÉTAPE 2: Lire requester.reference
  "Practitioner/pract-042"
    ↓
ÉTAPE 3: Charger ressource Practitioner/pract-042
    ↓
RÉSULTAT: Dr. Dupont
```

### Méthode 5: Index inversé

**Principe**: Créer un index pour accélérer les recherches

**Index inversé - Qu'est-ce que c'est?**:
```
Au lieu de:
  Document 1 → contient mots A, B, C
  Document 2 → contient mots B, D, E
  Document 3 → contient mots A, C, F

Créer:
  Mot A → Document 1, Document 3
  Mot B → Document 1, Document 2
  Mot C → Document 1, Document 3
  Mot D → Document 2
  Mot E → Document 2
  Mot F → Document 3
```

**Exemple avec FHIR**:
```
Index par code LOINC:
{
  "8480-6": ["obs-001", "obs-005", "obs-012"],
  "8462-4": ["obs-002", "obs-006", "obs-013"],
  "2339-0": ["obs-003", "obs-007", "obs-014"]
}

Pour trouver observations de PA systolique:
  → Regarder index["8480-6"]
  → Charger obs-001, obs-005, obs-012
  → RAPIDE!
```

**Index par patient**:
```
{
  "pat-001": {
    "observations": ["obs-001", "obs-002", "obs-003"],
    "imaging": ["study-001"],
    "medications": ["med-001", "med-002"]
  },
  "pat-002": {
    "observations": ["obs-004", "obs-005"],
    "imaging": ["study-002", "study-003"],
    "medications": ["med-003"]
  }
}
```

**Avantages**:
- Recherche très rapide
- Facile à maintenir
- Pas de dépendances

**Construction de l'index**:
```
ÉTAPE 1: Charger toutes ressources
    ↓
ÉTAPE 2: Pour chaque ressource
  Extraire champs clés:
    - Patient ID
    - Codes (LOINC, SNOMED)
    - Dates
    - Types
    ↓
ÉTAPE 3: Créer entrées d'index
    ↓
ÉTAPE 4: Sauvegarder index (JSON)
    ↓
Index prêt pour recherches rapides
```

---

## Stratégies de fetch FHIR

### Stratégie 1: Fetch Direct (Par ID)

**Quand l'utiliser**: Vous connaissez l'ID exact

**Processus**:
```
ID connu: "Patient/pat-001"
    ↓
Construire chemin fichier:
  "/fhir_data/patients/patient_001.json"
    ↓
Lire fichier
    ↓
Parser JSON
    ↓
Retourner ressource
```

**Avantages**:
- Très rapide
- Simple
- Pas d'ambiguïté

**Utilisation**:
- Afficher détails d'un patient spécifique
- Résoudre une référence
- Mise à jour d'une ressource

### Stratégie 2: Fetch avec filtre simple

**Quand l'utiliser**: Recherche avec un seul critère

**Processus**:
```
Critère: gender = "female"
    ↓
Charger tous patients
    ↓
Filtrer où patient.gender = "female"
    ↓
Retourner liste filtrée
```

**Optimisation**:
```
Si index existe:
  index["gender"]["female"] → IDs
    ↓
  Charger seulement ces IDs
    ↓
Sinon:
  Charger tous
    ↓
  Filtrer manuellement
```

### Stratégie 3: Fetch avec critères multiples

**Quand l'utiliser**: Recherche complexe

**Processus**:
```
Critères:
  - gender = "male"
  - birthDate > "1980-01-01"
  - address.city = "Paris"
    ↓
ÉTAPE 1: Appliquer filtre le plus sélectif
  (celui qui réduit le plus les résultats)
    ↓
ÉTAPE 2: Sur résultats, appliquer filtre 2
    ↓
ÉTAPE 3: Sur résultats, appliquer filtre 3
    ↓
Résultats finaux
```

**Ordre optimal**:
```
1. Critères d'égalité stricte
   (ID, codes spécifiques)
2. Critères de plage
   (dates, valeurs numériques)
3. Critères textuels
   (noms, descriptions)
```

### Stratégie 4: Fetch avec relations

**Quand l'utiliser**: Données liées entre ressources

**Exemple - Patient et ses observations**:
```
ÉTAPE 1: Fetch Patient
  "Patient/pat-001"
    ↓
ÉTAPE 2: Fetch Observations liées
  Chercher où subject.reference = "Patient/pat-001"
    ↓
ÉTAPE 3: Assembler résultat
  {
    patient: {...},
    observations: [...]
  }
```

**Optimisation avec include**:
```
Si beaucoup de relations:
  Utiliser index de références
    ↓
  index["patient_observations"]["pat-001"]
    = ["obs-001", "obs-002", "obs-003"]
    ↓
  Fetch direct de ces IDs
```

### Stratégie 5: Fetch incrémental

**Quand l'utiliser**: Beaucoup de résultats

**Principe**: Charger par morceaux (pagination)

**Processus**:
```
Requête: Tous les patients
  Total: 10,000 patients
    ↓
Page 1: Patients 1-50
  Afficher à l'utilisateur
    ↓
Si utilisateur continue:
  Page 2: Patients 51-100
    ↓
  Page 3: Patients 101-150
    ↓
  etc.
```

**Avantages**:
- Ne charge pas tout en mémoire
- Interface réactive
- Économie de ressources

**Implémentation**:
```
Paramètres:
  - page_size = 50
  - page_number = 1
    ↓
Calculer:
  start = (page_number - 1) × page_size
  end = start + page_size
    ↓
Retourner résultats[start:end]
```

---

## Exemples concrets

### Exemple 1: Trouver patients hypertendus

**Question utilisateur**:
"Quels patients ont une tension artérielle élevée?"

**Processus RAG complet**:

**1. RETRIEVAL (Récupération)**:
```
Analyser question:
  → Besoin: Observations de PA
  → Code LOINC: 8480-6 (PA systolique)
  → Critère: valeur > 140 mmHg
    ↓
Chercher dans observations:
  Filtrer par code = "8480-6"
  Filtrer par valueQuantity.value > 140
    ↓
Résultats:
  - obs-001: valeur=152, patient=pat-001
  - obs-005: valeur=145, patient=pat-002
  - obs-012: valeur=148, patient=pat-003
```

**2. AUGMENTATION (Enrichissement)**:
```
Pour chaque observation:
  Récupérer patient associé
    ↓
obs-001 → Patient pat-001:
  Nom: Marie Dubois
  Âge: 45 ans (né 1979-03-20)
  Genre: female
    ↓
obs-005 → Patient pat-002:
  Nom: Jean Martin
  Âge: 62 ans (né 1962-08-15)
  Genre: male
    ↓
obs-012 → Patient pat-003:
  Nom: Sophie Bernard
  Âge: 58 ans (né 1966-11-30)
  Genre: female
```

**3. GENERATION (Réponse)**:
```
Contexte assemblé:
  3 patients avec PA > 140
  Détails de chacun

Génération (template ou LLM):

"J'ai identifié 3 patients avec une pression artérielle  
systolique élevée (>140 mmHg) :

1. **Marie Dubois** (45 ans, femme)
   - PA : 152/95 mmHg
   - Date mesure : 15/01/2024

2. **Jean Martin** (62 ans, homme)
   - PA : 145/88 mmHg
   - Date mesure : 14/01/2024

3. **Sophie Bernard** (58 ans, femme)
   - PA : 148/92 mmHg
   - Date mesure : 16/01/2024

Recommandation : Ces patients devraient être suivis pour
hypertension artérielle."
```

### Exemple 2: Historique d'imagerie patient

**Question utilisateur**:
"Montrez-moi tous les scanners thoraciques de M. Dupont"

**Processus**:

**1. RETRIEVAL**:
```
Identifier patient:
  Nom: "Dupont"
    ↓
Chercher Patient:
  Filtrer name.family = "Dupont"
    ↓
Résultat: Patient/pat-001 (Jean Dupont)
    ↓
Chercher ImagingStudy:
  Filtrer subject.reference = "Patient/pat-001"
  Filtrer series.modality.code = "CT"
  Filtrer series.description contient "thorax"
    ↓
Résultats:
  - study-001: CT Thorax 15/01/2024
  - study-007: CT Thorax 10/12/2023
```

**2. AUGMENTATION**:
```
Pour chaque étude:
  Extraire détails:
    ↓
study-001:
  Date: 15/01/2024
  Séries: 2
  Images: 245
  Description: "Thorax haute résolution"
  Modalité: CT
    ↓
study-007:
  Date: 10/12/2023
  Séries: 1
  Images: 120
  Description: "Thorax sans injection"
  Modalité: CT
```

**3. GENERATION**:
```
"Voici l'historique des scanners thoraciques de 
M. Jean Dupont :

**Examen récent - 15/01/2024**
- Type : Scanner thoracique haute résolution
- Nombre d'images : 245 (2 séries)
- Statut : Disponible

**Examen précédent - 10/12/2023**
- Type : Scanner thoracique sans injection
- Nombre d'images : 120 (1 série)
- Statut : Disponible

Évolution : Deux examens à 1 mois d'intervalle.
Accès : Ces examens sont disponibles dans le PACS."
```

### Exemple 3: Statistiques globales

**Question utilisateur**:
"Combien de patients diabétiques avons-nous?"

**Processus**:

**1. RETRIEVAL**:
```
Chercher conditions de diabète:
  Code SNOMED: 44054006 (Diabète Type 2)
  ou
  Code SNOMED: 46635009 (Diabète Type 1)
    ↓
Charger toutes ressources Condition
    ↓
Filtrer par codes diabète
    ↓
Extraire patient IDs uniques
    ↓
Résultats:
  45 patients différents
```

**2. AUGMENTATION**:
```
Analyser les patients:
  - Répartition par type:
    Type 1: 8 patients
    Type 2: 37 patients
  - Répartition par genre:
    Hommes: 23
    Femmes: 22
  - Âge moyen: 58 ans
```

**3. GENERATION**:
```
"Statistiques des patients diabétiques :

**Total : 45 patients**

Répartition par type :
- Diabète Type 2 : 37 patients (82%)
- Diabète Type 1 : 8 patients (18%)

Répartition démographique :
- Hommes : 23 (51%)
- Femmes : 22 (49%)
- Âge moyen : 58 ans

Ces patients représentent 15% de notre file active."
```

---

## Conclusion

### Récapitulatif

**RAG avec FHIR sans base vectorielle** est possible et efficace grâce à:

1. **Filtrage natif**: Utiliser les champs FHIR standard
2. **Index simples**: Créer des index JSON pour accélérer
3. **Recherche textuelle**: TF-IDF pour chercher dans descriptions
4. **Relations**: Suivre les références entre ressources
5. **Templates**: Générer réponses avec ou sans LLM

### Quand utiliser chaque méthode

**Filtres natifs**:
- Critères simples (genre, date, code)
- Données structurées
- Recherches fréquentes

**TF-IDF**:
- Recherche textuelle
- Descriptions, notes cliniques
- Questions en langage naturel

**Index**:
- Beaucoup de données
- Recherches répétées
- Performance critique

**Références**:
- Données liées
- Contexte complet patient
- Analyses relationnelles

### Avantages de cette approche

✓ **Simple**: Pas de dépendances complexes
✓ **Rapide**: Pour volumes petits/moyens
✓ **Maintenable**: Facile à comprendre et déboguer
✓ **Flexible**: Adaptable aux besoins
✓ **Économique**: Pas de coûts additionnels

### Limites et solutions

**Limite**: Performance sur gros volumes
**Solution**: Ajouter index, pagination

**Limite**: Recherche sémantique limitée
**Solution**: Utiliser LLM pour parsing questions

**Limite**: Pas de similarité sémantique
**Solution**: Pour cas avancés, considérer base vectorielle

---

## Pour aller plus loin

### Améliorations possibles

1. **Cache**: Mettre en cache résultats fréquents
2. **Index avancés**: Multi-critères, full-text
3. **Parallel processing**: Traiter plusieurs fichiers en parallèle
4. **Compression**: Réduire taille des fichiers
5. **Validation**: Vérifier conformité FHIR

### Intégration LLM

**Sans base vectorielle, avec LLM**:
```
Question → Parse avec LLM → Critères
    ↓
Recherche dans FHIR
    ↓
Contexte → LLM → Réponse naturelle
```

**Avantage**: Meilleure compréhension questions
**Coût**: Appels API LLM

### Migration future

Si besoin base vectorielle plus tard:
```
Système actuel → Base vectorielle
  ↓
Garder la même interface
  ↓
Changement transparent pour utilisateurs
```

Les données FHIR restent compatibles!
