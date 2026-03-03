# Convertisseur de Données Médicales vers FHIR

Application Python pour convertir différents formats de données médicales vers le standard FHIR R4 (Fast Healthcare Interoperability Resources).

## Formats Supportés

### Formats d'entrée
- **JSON** - Données patient au format JSON personnalisé
- **DICOM** (DCM) - Images et métadonnées médicales
- **CDA** - Clinical Document Architecture (documents XML HL7)

### Format de sortie
- **FHIR R4** - Ressources FHIR au format JSON

## Installation

### Prérequis
- Python 3.8 ou supérieur

### Installation des dépendances

```bash
# Pour la conversion DICOM (optionnel)
pip install pydicom pillow --break-system-packages

# Ou installer toutes les dépendances
pip install -r requirements.txt --break-system-packages
```

## Utilisation

### Ligne de commande

```bash
# Conversion JSON vers FHIR
python fhir_converter.py example_patient.json json -o patient_fhir.json

# Conversion CDA vers FHIR
python fhir_converter.py example_cda.xml cda -o cda_fhir.json

# Conversion DICOM vers FHIR (nécessite pydicom)
python fhir_converter.py image.dcm dicom -o imaging_fhir.json
```

### Options de la ligne de commande

```
usage: fhir_converter.py [-h] [-o OUTPUT] input_file {json,dicom,dcm,cda}

Arguments:
  input_file            Fichier d'entrée à convertir
  {json,dicom,dcm,cda}  Format du fichier d'entrée

Options:
  -h, --help           Afficher l'aide
  -o OUTPUT, --output OUTPUT
                       Fichier de sortie FHIR (défaut: output_fhir.json)
```

### Utilisation comme bibliothèque

```python
from fhir_converter import FHIRConverter

# Créer une instance du convertisseur
converter = FHIRConverter()

# Conversion JSON → FHIR
fhir_patient = converter.convert('patient.json', 'json', 'output.json')

# Conversion CDA → FHIR
fhir_bundle = converter.convert('document.xml', 'cda', 'output.json')

# Conversion DICOM → FHIR
fhir_imaging = converter.convert('scan.dcm', 'dicom', 'output.json')
```

## Format JSON d'entrée

Le format JSON attendu pour les données patient :

```json
{
  "id": "pat-001",
  "patient_id": "123456789",
  "first_name": "Jean",
  "last_name": "Dupont",
  "gender": "M",
  "birth_date": "1980-05-15",
  "phone": "+33612345678",
  "email": "jean.dupont@example.fr",
  "address": {
    "street": "123 Rue de la République",
    "city": "Paris",
    "postal_code": "75001",
    "country": "France"
  }
}
```

Champs supportés (français et anglais) :
- `id`, `patient_id` - Identifiant
- `first_name`, `prenom` - Prénom
- `last_name`, `nom` - Nom de famille
- `gender`, `sexe` - Genre (M/F/O/U)
- `birth_date`, `date_naissance` - Date de naissance
- `phone`, `telephone` - Téléphone
- `email` - Email
- `address`, `adresse` - Adresse complète

## Exemples de sortie FHIR

### Patient FHIR (depuis JSON)

```json
{
  "resourceType": "Patient",
  "id": "pat-001",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2024-01-15T10:30:00"
  },
  "identifier": [{
    "use": "official",
    "system": "http://example.org/patient-ids",
    "value": "123456789"
  }],
  "active": true,
  "name": [{
    "use": "official",
    "family": "Dupont",
    "given": ["Jean"]
  }],
  "gender": "male",
  "birthDate": "1980-05-15"
}
```

### ImagingStudy FHIR (depuis DICOM)

```json
{
  "resourceType": "ImagingStudy",
  "id": "study-001",
  "status": "available",
  "subject": {
    "reference": "Patient/pat-001"
  },
  "series": [{
    "uid": "2.16.124.113543.6003.1154777499.30246.19789.3503430045",
    "modality": {
      "system": "http://dicom.nema.org/resources/ontology/DCM",
      "code": "CT"
    }
  }]
}
```

### Bundle FHIR (depuis CDA)

```json
{
  "resourceType": "Bundle",
  "type": "document",
  "entry": [
    {
      "resource": {
        "resourceType": "Composition",
        "status": "final",
        "title": "Document CDA"
      }
    },
    {
      "resource": {
        "resourceType": "Patient",
        "name": [{"family": "Martin", "given": ["Marie"]}]
      }
    },
    {
      "resource": {
        "resourceType": "Observation",
        "code": {"coding": [{"code": "8480-6", "display": "Pression artérielle"}]},
        "valueQuantity": {"value": 120, "unit": "mm[Hg]"}
      }
    }
  ]
}
```

## Structure du code

### Classe FHIRConverter

Méthodes principales :
- `convert(input_file, input_format, output_file)` - Méthode principale de conversion
- `json_to_fhir(json_file)` - Convertit JSON → FHIR Patient
- `dicom_to_fhir(dicom_file)` - Convertit DICOM → FHIR ImagingStudy
- `cda_to_fhir(cda_file)` - Convertit CDA → FHIR Bundle

Méthodes utilitaires :
- `_map_gender(gender)` - Normalise les codes de genre
- `_dicom_date_to_fhir(date, time)` - Convertit dates DICOM
- `_format_cda_date(date_str)` - Convertit dates CDA
- `_extract_patient_from_cda()` - Extrait patient du CDA
- `_extract_observations_from_cda()` - Extrait observations du CDA

## Fonctionnalités

### ✓ Implémenté
- Conversion JSON → FHIR Patient
- Conversion CDA → FHIR Bundle (Patient + Composition + Observations)
- Conversion DICOM → FHIR ImagingStudy
- Support multilingue (français/anglais)
- Mapping automatique des genres
- Conversion des dates
- Interface en ligne de commande
- Utilisation comme bibliothèque

### 📋 Extensions possibles
- Support de plus de ressources FHIR (Practitioner, Organization, etc.)
- Validation FHIR avec fhir.resources
- Support HL7 v2 messages
- Export vers serveur FHIR
- Interface graphique
- Batch processing

## Ressources FHIR créées

Selon le format d'entrée :

| Format | Ressource FHIR principale |
|--------|---------------------------|
| JSON | Patient |
| DICOM | ImagingStudy |
| CDA | Bundle (Composition + Patient + Observations) |

## Normes et standards

- **FHIR R4** (4.0.1) - http://hl7.org/fhir/R4/
- **DICOM** - Digital Imaging and Communications in Medicine
- **CDA R2** - Clinical Document Architecture Release 2
- **HL7** - Health Level Seven International

## Dépannage

### Erreur : pydicom non installé
```bash
pip install pydicom pillow --break-system-packages
```

### Erreur de parsing XML (CDA)
Vérifiez que le fichier CDA est bien formé et contient les namespaces corrects.

### Date invalide
Les dates doivent être au format ISO 8601 pour JSON : `YYYY-MM-DD`

## Licence

Ce code est fourni à titre d'exemple éducatif.

## Support

Pour toute question ou amélioration, n'hésitez pas à créer une issue ou contribuer au projet.

## Exemples de test

Des fichiers d'exemple sont fournis :
- `example_patient.json` - Exemple de patient JSON
- `example_cda.xml` - Exemple de document CDA

Testez avec :
```bash
python fhir_converter.py example_patient.json json -o test_patient.json
python fhir_converter.py example_cda.xml cda -o test_cda.json
```
