# E6-epreuve-raspberry

Ce projet présente une solution d'authentification multi-facteurs (MFA) déployée sur Raspberry Pi. Il sécurise l'accès à une application métier (KALIEMIE) en combinant une authentification logicielle via API et une validation biométrique par reconnaissance faciale.

Architecture du projet
Le système repose sur une architecture modulaire où chaque script remplit une fonction précise au sein de la chaîne de sécurité.

login.py (Interface Principale)
Il s'agit du point d'entrée du programme. Ce script gère l'interface graphique via Tkinter et assure la première étape de l'authentification :

Interrogation d'une API REST externe pour valider les identifiants saisis.

Ordonnancement des étapes suivantes en fonction de la réponse du serveur.

Gestion des erreurs de saisie et de connectivité réseau.

reco.py (Moteur de reconnaissance)
Ce script contient la logique de traitement d'image pour la phase de biométrie :

Chargement du modèle de reconnaissance LBPH (trainer.yml).

Analyse du flux vidéo en temps réel pour identifier l'utilisateur attendu.

Implémentation d'un délai de sécurité (timeout) de 30 secondes.

Retour de codes d'état ("OK", "TIMEOUT", "ANNULER") vers le script principal.

photo.py (Enrôlement biométrique)
Utilisé lorsqu'un nouvel utilisateur est détecté ou qu'une mise à jour du profil est nécessaire :

Capture d'un échantillon de 30 photographies.

Option d'enrôlement spécifique pour les porteurs de lunettes (60 captures).

Stockage structuré des images dans des répertoires nominatifs pour l'apprentissage.

train.py (Apprentissage automatique)
Ce script assure la maintenance du modèle d'Intelligence Artificielle :

Parcours récursif du répertoire de stockage des photographies.

Extraction des caractéristiques faciales et conversion en données numériques.

Génération et mise à jour du fichier de persistance trainer.yml.

insertTable.py (Journalisation)
Responsable de la traçabilité des événements sur la base de données locale SQLite :

Enregistrement systématique des tentatives de connexion.

Stockage de l'horodatage, de l'identifiant utilisateur et du résultat de l'authentification (succès ou motif d'échec).

camDetection.py (Outil de diagnostic)
Fichier de test permettant de vérifier l'alignement de la caméra et le bon fonctionnement des Haar Cascades (détection de visage) sans activer les fonctions de reconnaissance ou de base de données.

Spécifications techniques
Matériel : Raspberry Pi, Camera Module (libcamera/Picamera2).

Langage : Python 3.

Bibliothèques principales : OpenCV (Traitement d'image), SQLite3 (Base de données), Requests (API REST), Tkinter (GUI).

Algorithme : Local Binary Patterns Histograms (LBPH), choisi pour son efficacité sur des systèmes embarqués à ressources limitées.

Consultation des logs
Les données de connexion sont stockées dans le fichier KALIEMIE.db. Pour auditer les accès via le terminal :
