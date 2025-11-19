# 🤖 Application d'Analyse de Sentiment Client

## **Microservice FastAPI Sécurisé avec Next.js et Hugging Face**

Ce projet est la réalisation du **Brief N°6** visant à créer un micro-service d'analyse de sentiment. Il permet de traiter automatiquement les avis clients (provenant de réseaux sociaux, formulaires ou plateformes e-commerce) en utilisant le modèle de pointe `nlptown/bert-base-multilingual-uncased-sentiment` hébergé sur l'API Inference de **Hugging Face**.

L'application est entièrement conteneurisée avec **Docker** et propose une interface utilisateur front-end développée avec **Next.js** pour des tests rapides et une démonstration.

-----

## 🎯 Objectifs du Projet

L'objectif principal est de construire une **API sécurisée, performante et facilement déployable** capable d'intégrer un service d'Intelligence Artificielle externe pour le marketing digital.

  * **Sécurité :** Mise en place d'un système d'authentification **JWT (JSON Web Token)**.
  * **Fonctionnalité :** Appeler l'API Hugging Face et retourner un score de sentiment agrégé (Positif, Négatif, Neutre).
  * **Conteneurisation :** Utilisation de **Docker** pour le déploiement du Backend (FastAPI) et du Frontend (Next.js).
  * **Interface :** Création d'un Front-end **Next.js** pour tester l'authentification et la prédiction en temps réel.

-----

## 🛠️ Stack Technique

| Composant | Technologie | Rôle |
| :--- | :--- | :--- |
| **Backend API** | **FastAPI (Python)** | Gestion des endpoints, Sécurité JWT, Appel à l'API HF. |
| **IA Externe** | **Hugging Face Inference API** | Modèle `nlptown/bert-base-multilingual-uncased-sentiment`. |
| **Authentification** | **PyJWT** | Création et vérification des Tokens d'accès. |
| **Frontend** | **Next.js (React)** | Interface utilisateur (Login, Formulaire de prédiction, Affichage). |
| **Conteneurisation** | **Docker** | Isolation et déploiement facile des deux services. |
| **Tests** | **Pytest & Postman** | Tests unitaires pour le Backend et tests fonctionnels de l'API. |

-----

## 🚀 Fonctionnalités Implémentées

### 1\. Backend (FastAPI)

L'API expose deux points d'accès majeurs :

| Endpoint | Méthode | Description | Sécurité |
| :--- | :--- | :--- | :--- |
| **/login** | `POST` | Authentifie l'utilisateur (`username`/`password`) et retourne un **JWT** en cas de succès. | 🔓 Public |
| **/predict** | `POST` | Reçoit un texte et l'envoie à l'API Hugging Face. | 🔒 Protégé par JWT |

#### Logique de Sentiment

Le modèle Hugging Face retourne un score entre 1 et 5. Le backend FastAPI traduit ce score en catégories lisibles :

  * **Négatif :** Score de 1 ou 2.
  * **Neutre :** Score de 3.
  * **Positif :** Score de 4 ou 5.

### 2\. Frontend (Next.js)

L'interface est structurée en deux pages principales :

  * **`/login` :** Contient un formulaire d'identification. Le JWT reçu est stocké dans le **`localStorage`** du navigateur pour les requêtes futures.
  * **`/sentiment` :** Interface de test où l'utilisateur saisit un texte. La requête est envoyée à `/predict` en incluant le JWT dans l'en-tête `Authorization`. Affiche l'état (`loading`, `error`, `success`), le **sentiment (Positif/Négatif/Neutre)** et le **score (1 à 5)**.

-----

## 🔑 Installation et Démarrage avec Docker

L'application est conçue pour être lancée via `docker-compose`.

### Prérequis

  * Docker et Docker Compose installés.
  * Une clé **Hugging Face API Token** (`HF_API_KEY`).

### Configuration

Créez un fichier `.env` à la racine du dossier **backend** avec les variables suivantes :

```env
# Clé d'API Hugging Face
HF_API_KEY="votre_clé_hugging_face_ici" 

# Clé secrète pour l'encodage et le décodage des JWT (changer pour la production !)
SECRET_KEY="votre_clé_secrète_JWT_ici"
ALGORITHM="HS256"
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

### Commandes de Démarrage

1.  **Construire les images Docker :**

    ```bash
    docker-compose build
    ```

2.  **Démarrer l'application (Backend + Frontend) :**

    ```bash
    docker-compose up
    ```

### Accès aux services

Une fois démarré, les services sont accessibles aux adresses suivantes :

  * **API Backend (FastAPI) :** `http://localhost:8000/docs` (Documentation interactive/Swagger UI)
  * **Frontend (Next.js) :** `http://localhost:3000`

-----

## 🧪 Tests et Validation

### 1\. Tests Fonctionnels avec Postman

Les tests Postman sont essentiels pour valider le flux complet :

1.  **Test /login :** Requête POST avec un `username` et `password` valide pour obtenir un JWT.
2.  **Test /predict (Succès) :** Requête POST avec un texte et le JWT dans l'en-tête `Authorization: Bearer <token>`.
3.  **Test /predict (JWT Manquant/Invalide) :** Requête POST sans JWT ou avec un token expiré pour vérifier le rejet d'accès (statut 401/403).
4.  **Test des cas sentiments :** Envoyer des textes clairement Positifs, Négatifs et Neutres pour valider la logique de conversion 1-5 → Positif/Négatif/Neutre.
5.  **Test d'erreurs HF :** Simuler ou gérer un cas où l'API Hugging Face renvoie une erreur (clé invalide, service indisponible, etc.).

### 2\. Tests Unitaires avec Pytest

Des tests unitaires sont inclus pour valider les fonctions critiques du Backend :

  * Test de la fonction de **création/vérification du JWT**.
  * Test de la **logique de conversion** du score (1-5 vers Positif/Négatif/Neutre).
  * Test des **dépendances de sécurité** (`verify_token`).

-----

## 📄 Documentation et Limites

### Workflow Sécurité (Login → Prédiction Protégée)

Le flux de sécurité garantit que seul le personnel autorisé peut utiliser le service de prédiction, réduisant ainsi le risque d'abus et de dépassement de quota sur l'API Hugging Face.

1.  **Demande de Connexion :** L'utilisateur envoie ses identifiants à `/login`.
2.  **Création du JWT :** Le serveur vérifie les identifiants et génère un JWT contenant l'identité de l'utilisateur et une date d'expiration.
3.  **Stockage du Token :** Le Frontend stocke le JWT dans le `localStorage`.
4.  **Accès Protégé :** Pour toute requête à `/predict`, le Frontend ajoute le JWT dans l'en-tête `Authorization` (format `Bearer Token`).
5.  **Vérification :** L'endpoint `/predict` utilise une dépendance `Depends(verify_token)` qui décode et valide la signature et l'expiration du JWT. Si le token est valide, l'accès est accordé.

### Limites du Service IA Externe

L'utilisation d'un service IA externe introduit des dépendances à considérer :

  * **Latence :** Le temps de réponse dépend de la performance et de la charge des serveurs Hugging Face.
  * **Quota/Coût :** L'utilisation de l'API Inference a souvent des limites ou des coûts associés. La sécurité JWT est essentielle pour contrôler l'accès et gérer les dépenses.
  * **Modèle fixe :** L'application est liée aux performances du modèle `nlptown/bert-base-multilingual-uncased-sentiment`. Toute mise à jour des performances ou bilinguisme est gérée par l'équipe Hugging Face et non par l'agence.
