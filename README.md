# E6-Épreuve — Authentification MFA sur Raspberry Pi

> Solution d'authentification multi-facteurs déployée sur Raspberry Pi, sécurisant l'accès à l'application **KALIEMIE** via une double validation : identifiants API + reconnaissance faciale biométrique.

---

## Sommaire

- [Aperçu](#aperçu)
- [Architecture](#architecture)
- [Modules](#modules)
- [Spécifications techniques](#spécifications-techniques)
- [Installation](#installation)
- [Consultation des logs](#consultation-des-logs)

---

## Aperçu

Ce projet implémente un système MFA (Multi-Factor Authentication) embarqué sur Raspberry Pi. L'accès à l'application métier KALIEMIE est conditionné par deux couches de sécurité successives :

1. **Validation des identifiants** via une API REST externe
2. **Vérification biométrique** par reconnaissance faciale en temps réel (algorithme LBPH)

---

## Architecture

Le système repose sur une architecture modulaire : chaque script remplit une fonction précise et indépendante au sein de la chaîne de sécurité.

```
E6-epreuve-raspberry/
│
├── login.py          # Point d'entrée — Interface graphique & orchestration
├── reco.py           # Moteur de reconnaissance faciale (LBPH)
├── photo.py          # Enrôlement biométrique — Capture de photos
├── train.py          # Entraînement du modèle IA → trainer.yml
├── insertTable.py    # Journalisation des événements → KALIEMIE.db
├── camDetection.py   # Outil de diagnostic caméra
│
├── trainer.yml       # Modèle LBPH persisté (généré par train.py)
├── KALIEMIE.db       # Base de données SQLite des logs
└── dataset/          # Répertoires nominatifs des photos d'entraînement
    └── <nom_utilisateur>/
```

---

## Modules

### `login.py` — Interface Principale

Point d'entrée du programme. Gère l'interface graphique (Tkinter) et orchestre l'ensemble du flux d'authentification.

- Interrogation d'une API REST externe pour valider les identifiants saisis
- Ordonnancement des étapes suivantes selon la réponse du serveur
- Gestion des erreurs de saisie et de connectivité réseau

## Se connecter

```bash
lancer le login.py et utiliser l'utilisateur lwald
```

---

### `reco.py` — Moteur de Reconnaissance

Cœur du système biométrique. Traite le flux vidéo en temps réel pour identifier l'utilisateur.

- Chargement du modèle LBPH depuis `trainer.yml`
- Analyse du flux vidéo en temps réel
- Délai de sécurité (**timeout de 30 secondes**)
- Retourne un code d'état vers le script principal :

| Code | Signification |
|------|---------------|
| `OK` | Utilisateur reconnu avec succès |
| `TIMEOUT` | Délai de 30 s dépassé sans reconnaissance |
| `ANNULER` | Interruption manuelle de l'utilisateur |

---

### `photo.py` — Enrôlement Biométrique

Utilisé lors de l'ajout d'un nouvel utilisateur ou d'une mise à jour de profil.

- Capture d'un échantillon de **30 photographies**
- Option dédiée aux porteurs de lunettes (**60 captures**)
- Stockage structuré dans des répertoires nominatifs pour l'apprentissage

---

### `train.py` — Entraînement du Modèle

Assure la maintenance et la mise à jour du modèle d'intelligence artificielle.

- Parcours récursif du répertoire de photos
- Extraction des caractéristiques faciales et conversion en données numériques
- Génération et mise à jour du fichier `trainer.yml`

---

### `insertTable.py` — Journalisation

Garantit la traçabilité complète des événements d'authentification dans la base SQLite.

Chaque tentative enregistre :
- Horodatage
- Identifiant utilisateur
- Résultat (succès ou motif d'échec)

---

### `camDetection.py` — Diagnostic Caméra

Outil de test permettant de vérifier l'alignement de la caméra et le bon fonctionnement des **Haar Cascades** (détection de visage), sans activer les fonctions de reconnaissance ni d'écriture en base de données.

---

## Spécifications Techniques

| Composant | Détail |
|-----------|--------|
| **Matériel** | Raspberry Pi + Camera Module (libcamera / Picamera2) |
| **Langage** | Python 3 |
| **Interface graphique** | Tkinter |
| **Traitement d'image** | OpenCV |
| **Base de données** | SQLite3 |
| **Communication API** | Requests (REST) |
| **Algorithme IA** | LBPH — Local Binary Patterns Histograms |

> **Pourquoi LBPH ?**
> Cet algorithme a été retenu pour son efficacité sur des systèmes embarqués à ressources limitées. Il offre un bon compromis entre précision de reconnaissance et faible empreinte mémoire/CPU.

---

## Consultation des Logs

Les données de connexion sont stockées dans `KALIEMIE.db`. Pour auditer les accès via le terminal :

```bash
sqlite3 KALIEMIE.db
```

```sql
.mode column
.headers on
SELECT * FROM logs;
```

