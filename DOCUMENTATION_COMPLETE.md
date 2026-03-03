# Documentation Complète - Convertisseur FHIR et RAG
## Guide de compréhension des formats, workflows et récupération de données

---

## Table des matières
1. [Introduction](#introduction)
2. [Formats d'entrée expliqués](#formats-dentrée-expliqués)
3. [Format de sortie FHIR](#format-de-sortie-fhir)
4. [Workflow de conversion](#workflow-de-conversion)
5. [RAG avec FHIR sans base vectorielle](#rag-avec-fhir-sans-base-vectorielle)
6. [Méthodes de fetch et recherche](#méthodes-de-fetch-et-recherche)

---

## Introduction

Ce document explique comment convertir différents formats de données médicales vers FHIR (Fast Healthcare Interoperability Resources) et comment récupérer et utiliser ces données avec des techniques RAG (Retrieval Augmented Generation) sans avoir besoin d'une base de données vectorielle.

### Objectif
Permettre l'interopérabilité des systèmes de santé en convertissant les données existantes vers un format standardisé (FHIR), puis les exploiter intelligemment pour répondre à des questions médicales.

---

## Formats d'entrée expliqués

### 1. Format JSON - Données Patient Personnalisées

#### Qu'est-ce que c'est?
JSON (JavaScript Object Notation) est un format de texte simple pour échanger des données. Dans notre contexte, il représente les informations d'un patient.

#### Structure typique

**Informations de base:**
- **Identifiant unique** : Un numéro qui identifie le patient de manière unique
- **Nom et prénom** : L'identité civile du patient
- **Genre** : Masculin, Féminin, Autre ou Inconnu
- **Date de naissance** : Au format année-mois-jour (ex: 1985-03-20)

**Coordonnées:**
- **Téléphone** : Numéro de contact
- **Email** : Adresse électronique
- **Adresse postale** : Rue, ville, code postal, pays

#### Avantages
- Simple à lire et à écrire
- Léger (peu de données)
- Universel (tous les systèmes peuvent le lire)
- Flexible (peut contenir plus ou moins d'informations)

#### Exemple d'utilisation
Un cabinet médical qui gère ses patients dans Excel peut exporter les données en JSON pour les convertir en FHIR.

---

### 2. Format DICOM - Imagerie Médicale

#### Qu'est-ce que c'est?
DICOM (Digital Imaging and Communications in Medicine) est LE standard mondial pour stocker et transmettre des images médicales. Chaque scanner, IRM, échographie produit des fichiers DICOM.

#### Que contient un fichier DICOM?

**Les images:**
- Les pixels de l'image médicale (scanner, IRM, radio, etc.)
- Plusieurs images peuvent former une série (ex: 120 coupes d'un scanner thoracique)

**Les métadonnées (informations sur l'image):**

**Identification de l'examen:**
- **StudyInstanceUID** : Identifiant unique de l'étude (examen complet)
- **SeriesInstanceUID** : Identifiant de la série d'images
- **SOPInstanceUID** : Identifiant de chaque image individuelle

**Information patient:**
- **Patient ID** : Numéro du patient
- **Patient Name** : Nom du patient
- **Patient Birth Date** : Date de naissance
- **Patient Sex** : Genre

**Information médicale:**
- **Modality** : Type d'examen
  - CT = Scanner (Tomographie par rayons X)
  - MR = IRM (Imagerie par Résonance Magnétique)
  - US = Échographie (Ultrasons)
  - CR/DX = Radiographie numérique
  - MG = Mammographie
  - PT = TEP (Tomographie par Émission de Positons)
  - NM = Médecine nucléaire

**Détails de l'acquisition:**
- **Study Date** : Date de l'examen
- **Study Time** : Heure de l'examen
- **Series Description** : Description (ex: "Thorax sans injection")
- **Body Part Examined** : Partie du corps examinée

#### Pourquoi c'est important?
Tous les hôpitaux du monde utilisent DICOM. Convertir DICOM vers FHIR permet d'intégrer l'imagerie dans un dossier patient moderne.

#### Structure d'un fichier DICOM

```
┌─────────────────────────────────────┐
│  En-tête DICOM (128 bytes)          │
├─────────────────────────────────────┤
│  Préfixe "DICM" (4 bytes)           │
├─────────────────────────────────────┤
│  MÉTADONNÉES (Tags)                 │
│                                     │
│  (0010,0010) Nom patient            │
│  (0010,0020) ID patient             │
│  (0010,0030) Date naissance         │
│  (0010,0040) Sexe                   │
│  (0020,000D) ID étude               │
│  (0020,000E) ID série               │
│  (0008,0060) Modalité               │
│  (0008,103E) Description            │
│  ... (des centaines d'autres tags) │
├─────────────────────────────────────┤
│  DONNÉES IMAGE (Pixels)             │
│                                     │
│  [Matrice de pixels]                │
│  Chaque pixel = niveau de gris      │
│  ou couleur                         │
└─────────────────────────────────────┘
```

---

### 3. Format CDA - Documents Cliniques

#### Qu'est-ce que c'est?
CDA (Clinical Document Architecture) est un standard HL7 pour représenter des documents médicaux en XML. C'est comme un "PDF intelligent" qui contient des données structurées.

#### Types de documents CDA
- Compte-rendu de consultation
- Lettre de sortie d'hospitalisation
- Compte-rendu opératoire
- Résultats de laboratoire
- Compte-rendu d'imagerie
- Ordonnance médicale
- Certificat médical

#### Structure d'un document CDA

**1. En-tête du document**
```
┌─────────────────────────────────────┐
│  IDENTITÉ DU DOCUMENT               │
│  - Type de document                 │
│  - ID unique du document            │
│  - Titre                            │
│  - Date de création                 │
│  - Niveau de confidentialité        │
└─────────────────────────────────────┘
```

**2. Informations sur le patient**
```
┌─────────────────────────────────────┐
│  PATIENT                            │
│  - Identifiant(s)                   │
│  - Nom, prénom                      │
│  - Genre                            │
│  - Date de naissance                │
│  - Adresse                          │
│  - Téléphone                        │
└─────────────────────────────────────┘
```

**3. Informations sur l'auteur**
```
┌─────────────────────────────────────┐
│  AUTEUR (Médecin)                   │
│  - Nom du médecin                   │
│  - Spécialité                       │
│  - Établissement                    │
│  - Date de rédaction                │
└─────────────────────────────────────┘
```

**4. Corps du document (sections)**
```
┌─────────────────────────────────────┐
│  SECTION: Signes vitaux             │
│  ├─ Pression artérielle: 120/80     │
│  ├─ Fréquence cardiaque: 72 bpm     │
│  ├─ Température: 37.2°C             │
│  └─ Poids: 70 kg                    │
├─────────────────────────────────────┤
│  SECTION: Antécédents médicaux      │
│  ├─ Diabète type 2 depuis 2015      │
│  └─ Hypertension artérielle         │
├─────────────────────────────────────┤
│  SECTION: Allergies                 │
│  ├─ Pénicilline (réaction cutanée)  │
│  └─ Pollen (rhinite)                │
├─────────────────────────────────────┤
│  SECTION: Médicaments actuels       │
│  ├─ Metformine 500mg x2/jour        │
│  └─ Ramipril 5mg x1/jour            │
├─────────────────────────────────────┤
│  SECTION: Diagnostics               │
│  └─ Diabète de type 2 équilibré     │
└─────────────────────────────────────┘
```

#### Codes utilisés dans CDA

**Codes LOINC (pour les observations):**
- `8480-6` : Pression artérielle systolique
- `8462-4` : Pression artérielle diastolique
- `8867-4` : Fréquence cardiaque
- `8310-5` : Température corporelle
- `29463-7` : Poids corporel
- `8302-2` : Taille

**Codes de genre:**
- `M` : Masculin (Male)
- `F` : Féminin (Female)
- `UN` : Inconnu (Unknown)
- `OTH` : Autre (Other)

#### Avantages du CDA
- Standard international (utilisé mondialement)
- Structuré (facile à analyser par ordinateur)
- Sémantique (utilise des codes standardisés)
- Signable numériquement (authentification)
- Interopérable (échangeable entre systèmes)

---

## Format de sortie FHIR

### Qu'est-ce que FHIR?

**FHIR** = Fast Healthcare Interoperability Resources

C'est le **nouveau standard mondial** pour l'échange de données de santé, développé par HL7.

### Pourquoi FHIR?

**Les problèmes qu'il résout:**

1. **Ancien monde (avant FHIR):**
   - Chaque hôpital a son propre format
   - Difficile de partager les données entre systèmes
   - Standards anciens compliqués (HL7 v2, v3)
   - Pas adapté au web et aux applications mobiles

2. **FHIR apporte:**
   - Format moderne (JSON, facile à lire)
   - API web standard (REST)
   - Extensible et flexible
   - Adoption massive mondiale
   - Compatible mobiles et web

### Concepts clés FHIR

#### 1. Les Ressources

Une **ressource** est un bloc de données qui représente un concept médical.

**Exemples de ressources:**
- **Patient** : Un patient
- **Practitioner** : Un professionnel de santé
- **Observation** : Une mesure (pression artérielle, glycémie, etc.)
- **Medication** : Un médicament
- **ImagingStudy** : Un examen d'imagerie
- **Condition** : Un diagnostic
- **Procedure** : Une intervention
- **Encounter** : Une consultation
- **Organization** : Un établissement de santé

#### 2. Structure d'une ressource FHIR

Toute ressource FHIR a:
```
┌─────────────────────────────────────┐
│  resourceType                       │
│  (Type de la ressource)             │
├─────────────────────────────────────┤
│  id                                 │
│  (Identifiant unique)               │
├─────────────────────────────────────┤
│  meta                               │
│  (Métadonnées: version, date MAJ)   │
├─────────────────────────────────────┤
│  DONNÉES SPÉCIFIQUES                │
│  (Selon le type de ressource)       │
└─────────────────────────────────────┘
```

### Ressources FHIR générées par notre convertisseur

#### 1. Ressource Patient (depuis JSON)

**Contenu:**
```
┌─────────────────────────────────────┐
│  PATIENT                            │
├─────────────────────────────────────┤
│  resourceType: "Patient"            │
│  id: "pat-001"                      │
├─────────────────────────────────────┤
│  IDENTIFIANTS                       │
│  - Numéro sécurité sociale          │
│  - Numéro dossier hôpital           │
├─────────────────────────────────────┤
│  NOM                                │
│  - Nom de famille: Dupont           │
│  - Prénom: Jean                     │
│  - Usage: officiel                  │
├─────────────────────────────────────┤
│  DÉMOGRAPHIE                        │
│  - Genre: male                      │
│  - Date naissance: 1980-05-15       │
│  - Statut: actif                    │
├─────────────────────────────────────┤
│  CONTACT                            │
│  Téléphone:                         │
│    - Système: phone                 │
│    - Valeur: +33612345678           │
│    - Usage: domicile                │
│  Email:                             │
│    - Système: email                 │
│    - Valeur: jean@email.fr          │
├─────────────────────────────────────┤
│  ADRESSE                            │
│  - Rue: 123 Rue de la République    │
│  - Ville: Paris                     │
│  - Code postal: 75001               │
│  - Pays: France                     │
│  - Usage: domicile                  │
└─────────────────────────────────────┘
```

**Champs obligatoires:**
- `resourceType` : Toujours "Patient"
- `id` : Identifiant unique

**Champs recommandés:**
- `identifier` : Au moins un identifiant système
- `name` : Au moins un nom
- `gender` : Genre du patient
- `birthDate` : Date de naissance

**Valeurs possibles pour gender:**
- `male` : Masculin
- `female` : Féminin
- `other` : Autre
- `unknown` : Inconnu

#### 2. Ressource ImagingStudy (depuis DICOM)

**Contenu:**
```
┌─────────────────────────────────────┐
│  IMAGINGSTUDY (Étude d'imagerie)    │
├─────────────────────────────────────┤
│  resourceType: "ImagingStudy"       │
│  id: "study-001"                    │
│  status: "available"                │
├─────────────────────────────────────┤
│  PATIENT                            │
│  - Référence: Patient/pat-001       │
│  - Nom affiché: DUPONT Jean         │
├─────────────────────────────────────┤
│  INFORMATIONS EXAMEN                │
│  - Date début: 2024-01-15 14:30     │
│  - Nombre de séries: 2              │
│  - Nombre total d'images: 245       │
├─────────────────────────────────────┤
│  SÉRIE 1                            │
│  ├─ UID: 1.2.840.113543...          │
│  ├─ Numéro: 1                       │
│  ├─ Modalité: CT (Scanner)          │
│  ├─ Description: "Thorax sans       │
│  │   injection"                     │
│  ├─ Nombre d'images: 120            │
│  └─ INSTANCES                       │
│      ├─ Image 1                     │
│      ├─ Image 2                     │
│      └─ ... (120 images)            │
├─────────────────────────────────────┤
│  SÉRIE 2                            │
│  ├─ UID: 1.2.840.113543...          │
│  ├─ Numéro: 2                       │
│  ├─ Modalité: CT                    │
│  ├─ Description: "Abdomen avec      │
│  │   injection"                     │
│  ├─ Nombre d'images: 125            │
│  └─ INSTANCES                       │
│      ├─ Image 1                     │
│      ├─ Image 2                     │
│      └─ ... (125 images)            │
└─────────────────────────────────────┘
```

**Hiérarchie:**
```
ImagingStudy (Examen complet)
    └─ Series (Série d'images)
        └─ Instance (Image individuelle)
```

**Codes de modalité courants:**
- `CT` : Scanner (rayons X)
- `MR` : IRM (résonance magnétique)
- `US` : Échographie
- `CR` : Radiographie numérique
- `DX` : Radiographie digitale
- `MG` : Mammographie
- `PT` : TEP Scan
- `NM` : Médecine nucléaire

#### 3. Ressource Bundle (depuis CDA)

**Qu'est-ce qu'un Bundle?**
Un Bundle est un **conteneur** qui regroupe plusieurs ressources FHIR ensemble. C'est comme une enveloppe qui contient plusieurs documents.

**Contenu:**
```
┌─────────────────────────────────────┐
│  BUNDLE                             │
├─────────────────────────────────────┤
│  resourceType: "Bundle"             │
│  type: "document"                   │
├─────────────────────────────────────┤
│  ENTRÉE 1: Composition              │
│  └─ Document maître                 │
│      ├─ Titre: "Consultation"       │
│      ├─ Date: 2024-01-15            │
│      ├─ Patient: réf vers entrée 2  │
│      └─ Sections du document        │
├─────────────────────────────────────┤
│  ENTRÉE 2: Patient                  │
│  └─ Informations patient            │
│      ├─ Nom: MARTIN Marie           │
│      ├─ Genre: female               │
│      └─ Naissance: 1975-06-20       │
├─────────────────────────────────────┤
│  ENTRÉE 3: Observation              │
│  └─ Pression artérielle systolique  │
│      ├─ Code: 8480-6 (LOINC)        │
│      ├─ Valeur: 120 mm[Hg]          │
│      └─ Patient: réf vers entrée 2  │
├─────────────────────────────────────┤
│  ENTRÉE 4: Observation              │
│  └─ Pression artérielle diastolique │
│      ├─ Code: 8462-4 (LOINC)        │
│      ├─ Valeur: 80 mm[Hg]           │
│      └─ Patient: réf vers entrée 2  │
├─────────────────────────────────────┤
│  ENTRÉE 5: Observation              │
│  └─ Fréquence cardiaque             │
│      ├─ Code: 8867-4 (LOINC)        │
│      ├─ Valeur: 72 /min             │
│      └─ Patient: réf vers entrée 2  │
├─────────────────────────────────────┤
│  ENTRÉE 6: Observation              │
│  └─ Température                     │
│      ├─ Code: 8310-5 (LOINC)        │
│      ├─ Valeur: 37.2 °C             │
│      └─ Patient: réf vers entrée 2  │
└─────────────────────────────────────┘
```

**Types de Bundle:**
- `document` : Document clinique (notre cas)
- `message` : Message HL7
- `transaction` : Transaction (créer/modifier plusieurs ressources)
- `collection` : Simple collection de ressources
- `searchset` : Résultats de recherche

---

## Workflow de conversion

### Vue d'ensemble du processus

```
ENTRÉE                  TRAITEMENT                 SORTIE
═════                   ══════════                 ══════

┌─────────┐
│  JSON   │────┐
│ Patient │    │
└─────────┘    │
               │         ┌──────────────┐         ┌─────────┐
┌─────────┐    ├────────>│ Convertisseur│────────>│  FHIR   │
│ DICOM   │────┤         │     FHIR     │         │  JSON   │
│ Images  │    │         └──────────────┘         └─────────┘
└─────────┘    │
               │
┌─────────┐    │
│   CDA   │────┘
│Document │
└─────────┘
```

### Étapes communes à toutes les conversions

```
┌────────────────────────────────────────────────┐
│  ÉTAPE 1: RÉCEPTION DU FICHIER                 │
│  - Identifier le type (JSON/DICOM/CDA)         │
│  - Vérifier que le fichier est valide          │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────┐
│  ÉTAPE 2: LECTURE ET PARSING                   │
│  - Lire le contenu du fichier                  │
│  - Parser selon le format                      │
│    • JSON → dictionnaire                       │
│    • DICOM → métadonnées                       │
│    • CDA → arbre XML                           │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────┐
│  ÉTAPE 3: EXTRACTION DES DONNÉES               │
│  - Identifier les champs importants            │
│  - Extraire les valeurs                        │
│  - Nettoyer et normaliser                      │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────┐
│  ÉTAPE 4: MAPPING VERS FHIR                    │
│  - Créer la structure FHIR appropriée          │
│  - Mapper chaque champ source → FHIR           │
│  - Convertir les codes et formats              │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────┐
│  ÉTAPE 5: ENRICHISSEMENT                       │
│  - Ajouter métadonnées (date, version)         │
│  - Ajouter identifiants uniques                │
│  - Compléter les champs manquants              │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────┐
│  ÉTAPE 6: VALIDATION                           │
│  - Vérifier les champs obligatoires            │
│  - Valider les types de données                │
│  - Vérifier la conformité FHIR                 │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────┐
│  ÉTAPE 7: GÉNÉRATION FHIR JSON                 │
│  - Formatter en JSON                           │
│  - Sauvegarder le fichier                      │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
           [Ressource FHIR]
```

### Workflow détaillé: JSON → FHIR Patient

```
FICHIER JSON
    │
    │  {
    │    "id": "pat-001",
    │    "first_name": "Marie",
    │    "last_name": "Dubois",
    │    "gender": "F",
    │    "birth_date": "1985-03-20",
    │    "phone": "+33612345678"
    │  }
    │
    ▼
┌─────────────────────────────────────┐
│  1. LECTURE JSON                    │
│  Parser le fichier en mémoire       │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  2. EXTRACTION DES CHAMPS           │
│  id        → "pat-001"              │
│  first_name → "Marie"               │
│  last_name  → "Dubois"              │
│  gender     → "F"                   │
│  birth_date → "1985-03-20"          │
│  phone      → "+33612345678"        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  3. MAPPING VERS FHIR               │
│                                     │
│  id → Patient.id                    │
│  "pat-001"                          │
│                                     │
│  first_name + last_name →           │
│  Patient.name[0].given[0] = "Marie" │
│  Patient.name[0].family = "Dubois"  │
│                                     │
│  gender → Patient.gender            │
│  "F" transformé en "female"         │
│                                     │
│  birth_date → Patient.birthDate     │
│  "1985-03-20" (déjà format FHIR)    │
│                                     │
│  phone → Patient.telecom[0]         │
│  system: "phone"                    │
│  value: "+33612345678"              │
│  use: "home"                        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  4. CRÉATION STRUCTURE FHIR         │
│                                     │
│  {                                  │
│    "resourceType": "Patient",       │
│    "id": "pat-001",                 │
│    "meta": {                        │
│      "versionId": "1",              │
│      "lastUpdated": "2024-01-15..." │
│    },                               │
│    "name": [{                       │
│      "family": "Dubois",            │
│      "given": ["Marie"]             │
│    }],                              │
│    "gender": "female",              │
│    "birthDate": "1985-03-20",       │
│    "telecom": [{                    │
│      "system": "phone",             │
│      "value": "+33612345678"        │
│    }]                               │
│  }                                  │
└──────────────┬──────────────────────┘
               │
               ▼
     [RESSOURCE PATIENT FHIR]
```

### Workflow détaillé: DICOM → FHIR ImagingStudy

```
FICHIER DICOM (.dcm)
    │
    │  En-tête + Métadonnées DICOM
    │  + Données image (pixels)
    │
    ▼
┌─────────────────────────────────────┐
│  1. LECTURE DICOM                   │
│  Utilise bibliothèque spécialisée   │
│  (pydicom)                          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  2. EXTRACTION MÉTADONNÉES          │
│                                     │
│  Patient:                           │
│    PatientID: "123456"              │
│    PatientName: "DUBOIS^MARIE"      │
│    PatientSex: "F"                  │
│    PatientBirthDate: "19850320"     │
│                                     │
│  Étude:                             │
│    StudyInstanceUID: "1.2.840..."   │
│    StudyDate: "20240115"            │
│    StudyTime: "143052"              │
│                                     │
│  Série:                             │
│    SeriesInstanceUID: "1.2.840..."  │
│    SeriesNumber: "1"                │
│    Modality: "CT"                   │
│    SeriesDescription: "Thorax"      │
│                                     │
│  Instance:                          │
│    SOPInstanceUID: "1.2.840..."     │
│    InstanceNumber: "1"              │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  3. CONVERSION DES FORMATS          │
│                                     │
│  Date DICOM → Date FHIR:            │
│  "20240115" → "2024-01-15"          │
│                                     │
│  Heure DICOM → Heure FHIR:          │
│  "143052" → "14:30:52"              │
│                                     │
│  Datetime complet:                  │
│  "2024-01-15T14:30:52"              │
│                                     │
│  Nom DICOM → Nom FHIR:              │
│  "DUBOIS^MARIE" → "DUBOIS Marie"    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  4. MAPPING VERS IMAGING