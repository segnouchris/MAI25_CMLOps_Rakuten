MAI25_CMLOPS : project Rakuten 
==============================

Projet pédagogique réalisé dans le cadre de la formation MLOps de DataScientest (Cohorte MAI 2025), axé sur la mise en place d’une architecture MLOps complète pour le traitement et la classification de données produits Rakuten dans le cadre du challenge ens-data : https://challengedata.ens.fr/participants/challenges/35/ .
Les modèles déployés sont dérivés de ceux définis par l'équipe Olivier ISNARD / Julien TREVISAN / Loïc RAMAYE lors de leur formation Data Scientist (cohorte Juin 2025) et qui avaient permis d'obtenir la première place au classement public et privé du challenge.


---
## 🚀 Objectifs réalisés 

- Mettre en place un pipeline complet de Machine Learning avec Airflow.
- Intégrer des étapes de data loading, preprocessing, entraînement, évaluation et déploiement.
- Suivre les expériences via MLflow.
- Conteneuriser l’environnement avec Docker.
- Fournir une API de prédiction REST sécurisée.
- Suivre les versions de données avec DVC
- Tests unitaires

---
## 📁 Structure du dépôt

```bash
.
├── airflow/                   # Composants liés à Airflow
│   ├── dags/                 # DAG principal orchestrant le pipeline
│   │   └── rakuten_dags.py
│
├── data/                     # Données versionnées avec DVC
│   ├── raw/                 # Données brutes (ex : images, CSV initiaux)
│   ├── processed/           # Données traitées (X_train, y_train, etc.)
│   ├── processed.dvc        # Fichier DVC de suivi de `/processed`
│   ├── raw.dvc              # Fichier DVC de suivi de `/raw`
│   └── .gitignore           # Évite de traquer les gros fichiers localement
│
├── docker/                   # Dockerfiles spécifiques à chaque étape
│   ├── Dockerfile.airflow
│   ├── Dockerfile.api
│   ├── Dockerfile.dataloading
│   ├── Dockerfile.evaluate
│   ├── Dockerfile.mlflow
│   ├── Dockerfile.preprocessing
│   ├── Dockerfile.train
│   ├── requirements-airflow.txt     # Dépendances Airflow
│   ├── requirements-api.txt         # Dépendances FastAPI
│   ├── requirements-dataloading.txt
│   ├── requirements-evaluate.txt
│   ├── requirements-mlflow.txt
│   ├── requirements-preprocessing.txt
│   └── requirements-train.txt
├── docker-compose.yml        # Orchestration des services via Docker Compose
│
├── models/                   # Modèles entraînés (.pkl ou autres)
│
├── mlruns/                   # Répertoire d’expérimentation MLflow (tracking local)
│
├── params.yaml               # Paramètres globaux pour le pipeline (modèle, seed, split, etc.)
│
├── src/                      # Code source modulaire pour chaque étape
│   ├── dataloading/         # Scripts pour charger les données brutes
│   ├── preprocessing/       # Feature engineering, normalisation, etc.
│   ├── training/            # Entraînement de modèles
│   ├── evaluation/          # Évaluation de performance
│   └── utils/               # Fonctions utilitaires (log, I/O, etc.)
│
├── tests/                    # Tests unitaires Pytest pour chaque module
│   ├── test_dataloading.py
│   ├── test_preprocessing.py
│   ├── test_training.py
│   ├── test_evaluation.py
│   └── conftest.py
│
├── .dvc/                     # Répertoire interne de configuration DVC
├── .dvcignore                # Équiv. de .gitignore pour DVC
├── .env                      # Variables d’environnement (ex: BASE_DIR)
└── README.md                 # Documentation du projet (ce fichier)
```
🔍 Fichiers de configuration importants
| Fichier              | Rôle                                                           |
| -------------------- | -------------------------------------------------------------- |
| `docker-compose.yml` | Lance tous les services nécessaires (Airflow, API, MLflow…)    |
| `params.yaml`        | Centralise les hyperparamètres, chemins, splits, etc.          |
| `processed.dvc`      | Suit les transformations de données via DVC                    |
| `.dvcignore`         | Exclut certains fichiers des suivis DVC                        |
| `.env`               | Définit les variables d’environnement Docker (base\_dir, etc.) |

---
### 🧰 Services
| Service     | Port | Description                     |
| ----------- | ---- | ------------------------------- |
| Airflow UI  | 8080 | Orchestration du pipeline       |
| MLflow      | 5000 | Tracking des expériences        |
| API FastAPI | 8000 | Endpoint de prédiction          |
| PostgreSQL  |      | Backend Airflow & MLflow        |
| Redis       |      | Message broker Airflow (Celery) |

---
### ▶️ Lancer l’environnement
1. Prérequis

    docker installé

    Un fichier .env avec les variables suivantes : 
    ```
    # Description: Environment variables for the Rakuten project
    # The account must be subscribed to the challenge https://challengedata.ens.fr/participants/challenges/35/
    ENSDATA_LOGIN=
    ENSDATA_PASSWORD=

    # The path to the directory where the data is stored
    DATA_RAW_DIR="./data/raw"
    # The path to the directory where the the images of train dataset are stored
    DATA_RAW_IMAGES_TRAIN_DIR="./data/raw/image_train"
    # The path to the directory where the the images of test dataset are stored
    DATA_RAW_IMAGES_TEST_DIR="./data/raw/image_test"

    # The path to the directory where the processed data will be stored
    DATA_PROCESSED_DIR="./data/processed"
    # The path to the directory where the model will be stored
    MODEL_DIR="./models"
    # The path to the directory where the logs will be stored
    LOGS_DIR="./logs"
    # The path to the directory where the scores of model evaluation will be stored
    METRICS_DIR="./metrics"

    # Definition of the secret key for signing JWT tokens
    # This key should be kept secret and not shared publicly
    # It is used to ensure the integrity and authenticity of the JWT tokens
    # It is recommended to use a strong, random key for production environments

    JWT_SECRET_KEY = 

    FERNET_KEY=

    # Local directory where is stored the projet
    BASE_DIR = 
    ```
2. Lancement des services
    ```
    docker compose up --build
    ```
    Airflow sera accessible sur localhost:8080, et MLflow sur localhost:5000.
---
### ⚙️ Pipelines Airflow

Le DAG principal (rakuten_dags.py) orchestre les étapes suivantes :
- data_loading_task
- preprocessing_task
- training_task
- evaluation_task
- mlflow_dag_task

---
### 🧪 Suivi des expériences

Les logs et artefacts d'entraînement sont automatiquement tracés dans MLflow :

```http://localhost:5000 ```

Le tracking inclut :
- métriques d’évaluation
- paramètres du modèle
- visualisations

---
### 🌐 API de prédiction

Un service FastAPI expose un endpoint sur le port 8000.

---
### 🧪 Tests et reproductibilité

Les composants sont encapsulés dans des images Docker distinctes pour chaque étape (dataloading, preprocessing, etc.), facilitant l’isolation et les tests.

---
### 🗂️ Gestion des données avec DVC

Les transformations sont suivies avec DVC pour permettre une versioning des datasets transformés.
```
dvc repro
dvc push
``` 

---
### ✅ Tests unitaires

Le répertoire tests/ contient des tests unitaires pour valider les différentes étapes du pipeline ML : chargement des données, prétraitement, entraînement, prédiction, etc.
📦 Structure
```
tests/
├── test_dataloading.py
├── test_preprocessing.py
├── test_training.py
├── test_evaluation.py
└── conftest.py  # Fixtures partagées 
```
▶️ Exécution des tests

Assure-toi d’avoir installé pytest (via pip install pytest ou via un requirements.txt), puis lance les tests avec :
```
python -m pytest
```

---
### 📝 Auteurs

- Olivier ISNARD
- Christian SEGNOU

Encadré dans le cadre de la formation MLOps par Maria de DataScientest.

