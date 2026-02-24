# 🔍 Projet : VulnPy-Scanner - Outil de Reconnaissance & Mapping CVE

![Python](https://img.shields.io/badge/Python-3.x-blue.svg)
![Nmap](https://img.shields.io/badge/Nmap-Network_Scanner-red.svg)
![NIST API](https://img.shields.io/badge/API-NIST_NVD-green.svg)

## 📝 Description
VulnPy-Scanner est un outil d'audit de sécurité en ligne de commande (CLI) développé en Python. Il automatise la phase de reconnaissance réseau en scannant les ports ouverts, en effectuant une prise d'empreinte des services (fingerprinting), et croise dynamiquement ces informations avec la base de données gouvernementale du NIST pour identifier les vulnérabilités publiques (CVE) associées.

## ⚠️ Avertissement Légal
Cet outil a été créé à des fins strictement éducatives et pour une utilisation dans des environnements contrôlés (laboratoires locaux, CTF, machines virtuelles avec autorisation). L'auteur décline toute responsabilité quant à l'utilisation malveillante de ce script sur des réseaux tiers sans consentement explicite.

## 🚀 Fonctionnalités
- **Scan Réseau Multi-threadé :** Utilisation de Nmap pour un scan rapide des hôtes actifs et des ports ouverts.
- **Service Fingerprinting :** Détection précise des noms et versions des services en cours d'exécution.
- **Threat Intelligence Automatisée :** Interrogation en temps réel de l'API publique du NIST (National Vulnerability Database v2.0).
- **Génération de Rapports :** Export des résultats au format JSON, incluant les identifiants CVE et les scores de criticité (CVSS).

## 🛠️ Prérequis et Installation
1. Installez le moteur [Nmap](https://nmap.org/download.html) sur votre machine (sur Windows, assurez-vous que Npcap est installé et que Nmap est dans votre `PATH`).
2. Clonez le dépôt et installez les dépendances :
   ```bash
   git clone [https://github.com/Bergson26/VulnPy-Scanner.git](https://github.com/Bergson26/VulnPy-Scanner.git)
   cd VulnPy-Scanner
   pip install -r requirements.txt

💻 Utilisation
Pour lancer un scan (nécessite des privilèges administrateur/root pour le fingerprinting) :

Bash
python main.py -t <IP_CIBLE_OU_RESEAU>
Exemple de cible de test autorisée : scanme.nmap.org ou un conteneur local 127.0.0.1.

# 📚 Documentation Technique - VulnPy-Scanner

## Architecture Globale
Le projet est structuré de manière modulaire pour séparer la logique de scan, l'interrogation d'API et la génération de rapports. 

### 1. Module `scanner/network.py`
Ce module agit comme un wrapper autour de l'outil Nmap via la bibliothèque `python-nmap`.
- **Classe `NetworkScanner` :** Gère l'initialisation et l'exécution.
- **Paramètres Nmap utilisés :** `-sV` (détection de version), `-T4` (vitesse agressive), `-F` (scan rapide des ports les plus courants).
- **Gestion de la concurrence :** Utilisation du module `threading` natif de Python. Chaque hôte détecté est scanné dans un thread séparé pour accélérer le traitement sur des plages IP (ex: `/24`), avec un `Lock` pour éviter les conditions de concurrence lors de l'écriture des résultats.

### 2. Module `scanner/cve_api.py`
Ce module gère la Threat Intelligence en interrogeant la base de données du NIST.
- **API Utilisée :** NIST NVD REST API v2.0.
- **Classe `CVEMapper` :** Formate les requêtes avec la syntaxe `keywordSearch=<produit> <version>`.
- **Gestion du Rate Limiting :** L'API publique sans clé d'authentification étant strictement limitée (environ 5 requêtes par fenêtre de 30 secondes), un délai artificiel (`time.sleep(6)`) est implémenté entre chaque requête HTTP GET pour garantir la stabilité de l'outil et éviter les erreurs HTTP 403.

### 3. Module `utils/report.py`
- **Classe `Reporter` :** Compile les données fusionnées (Services + CVE) et génère un rapport JSON horodaté. La structure JSON a été choisie pour sa facilité d'intégration avec d'autres outils d'analyse de logs ou des tableaux de bord type SIEM.

# 🚧 Défis Techniques & Résolutions

Durant le développement de VulnPy-Scanner, plusieurs défis d'architecture et de configuration ont été rencontrés :

### 1. Limite de caractères sous Windows (MAX_PATH) lors du build
**Problème :** Lors de l'installation de la bibliothèque `python-nmap` via `pip`, une erreur `[Errno 2] No such file or directory` est survenue. Le processus de build créait des chemins de fichiers temporaires dépassant la limite historique de 260 caractères de Windows.
**Résolution :** Exécution de l'installation en contournant le cache profond de pip via la commande `pip install -r requirements.txt --no-cache-dir`, forçant une installation directe et plus courte.

### 2. Accès à l'exécutable Nmap depuis Python
**Problème :** L'erreur `nmap program was not found in path` bloquait l'exécution. Bien que Nmap soit installé, le script Python ne parvenait pas à le localiser via les variables d'environnement système.
**Résolution :** Modification du code d'initialisation de `PortScanner()` pour inclure explicitement le chemin absolu de l'exécutable (`nmap_search_path`), garantissant le fonctionnement même sur des machines dont le `PATH` est mal configuré.

### 3. Fiabilité du Fingerprinting
**Problème :** Les premiers scans locaux renvoyaient des ports ouverts mais aucune version de service n'était détectée, rendant l'interrogation de l'API CVE inutile.
**Résolution :** Identification du besoin strict de forger des paquets réseau bruts (raw sockets) pour la détection de version (`-sV`). L'outil nécessite une exécution avec des privilèges élevés (Administrateur sous Windows / Root sous Linux) pour que le moteur Npcap fonctionne pleinement.


   **Bergson Jean-Michel A.**
