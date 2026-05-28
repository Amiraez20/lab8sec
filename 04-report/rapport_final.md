# Rapport d'analyse de sécurité mobile

## A. Informations générales
- **Analyste**: [EZBIRI Amira]
- **Cible**: InsecureBankv2 (com.android.insecurebankv2)
- **Version/Hash**: B18AF2A0E44D7634BBCDF93664D9C78A2695E050393FCFBB5E8B91F902D194A4
- **Outils utilisés**: BeVigil , Yaazhini

## B. Résumé exécutif
L'audit d'**InsecureBankv2** a révélé **12 failles**, dont 5 critiques (High) exposant l'application à des risques majeurs : communications en clair, mode debug actif, cryptographie faible et composants Android vulnérables.

Le niveau de risque global est **élevé** et inacceptable pour un environnement de production bancaire.

Une **remédiation immédiate des vulnérabilités "High"** est indispensable avant tout déploiement.

## C. Top 5 constats

### 1. Exposition des flux réseaux au transit en clair — FIND-001
- **Sévérité**: High
- **Preuve**: Corrélations BeVigil (détection de 4 instances d'appels HTTP non chiffrés) et Yaazhini (hardcoding de schémas d'URI non sécurisés au sein des classes sources).
- **Impact**: Risque critique d'interception passive et active de données de session, d'identifiants bancaires et d'informations nominatives par le biais d'attaques de type *Man-In-The-Middle* (MITM) sur des réseaux non fiables.
- **Remédiation**: Bannir le trafic en texte clair en imposant l'usage exclusif du protocole HTTPS sous TLS 1.2/1.3. Il est impératif d'intégrer une politique stricte via le fichier `Network Security Configuration` d'Android et de déployer un mécanisme de *Certificate Pinning* pour valider l'authenticité du certificat serveur.
- **Référence OWASP**: MASVS-NETWORK-1

### 2. Persistance des fonctionnalités de débogage sur le package de production — FIND-002
- **Sévérité**: High
- **Preuve**: Audit du manifeste de l'application via Yaazhini (attribut `android:debuggable` explicitement assigné à `true` dans le nœud `<application>`).
- **Impact**: Permet à un attaquant ou à un outil d'analyse dynamique d'injecter du code, d'extraire des éléments volatils en mémoire ou de contourner les règles métiers de l'application via le pont ADB (*Android Debug Bridge*) sans nécessiter de privilèges d'administration (root).
- **Remédiation**: Purger l'attribut du manifeste ou forcer sa désactivation (`android:debuggable="false"`) au moment de la compilation via les règles de build Gradle (*release*).
- **Référence OWASP**: MASVS-CODE-2

### 3. Permissivité de la politique de sauvegarde locale (Sauvegarde ADB active) — FIND-003
- **Sévérité**: High
- **Preuve**: Analyse statique du fichier `AndroidManifest.xml` (Yaazhini) mettant en évidence la directive `android:allowBackup="true"`.
- **Impact**: Possibilité pour un tiers ayant un accès physique transitoire à l'appareil d'extraire l'intégralité du répertoire de données privées de la banque (bases de données, préférences) via une simple interface de sauvegarde USB, même sur un terminal non déverrouillé par root.
- **Remédiation**: Neutraliser cette fonctionnalité en basculant l'attribut à `false` au sein du manifeste, ou segmenter finement les répertoires éligibles en spécifiant une règle d'exclusion stricte via `android:fullBackupContent`.
- **Référence OWASP**: MASVS-STORAGE-8

### 4. Utilisation de primitives cryptographiques obsolètes et vulnérables — FIND-004
- **Sévérité**: High
- **Preuve**: Extraction de code par Yaazhini au niveau de la routine de gestion des secrets (`ChangePassword.java`), confirmant l'appel aux algorithmes MD5 et SHA-1.
- **Impact**: Risque massif de compromission des mots de passe des utilisateurs par collision ou inversion des empreintes (via l'usage de *Rainbow Tables* précalculées ou d'attaques par force brute hautement parallélisées).
- **Remédiation**: Remplacer immédiatement ces fonctions de hachage par des algorithmes modernes. Pour le stockage ou la vérification des mots de passe, implémenter une fonction de dérivation de clé robuste telle que Argon2id ou bcrypt dotée d'un sel cryptographique unique par utilisateur.
- **Référence OWASP**: MASVS-CRYPTO-4

### 5. Surexposition systémique de Content Providers non authentifiés — FIND-005
- **Sévérité**: High
- **Preuve**: Cartographie des composants par Yaazhini révélant des nœuds `<provider>` configurés avec la directive `android:exported="true"` sans restriction d'accès associée.
- **Impact**: Fuite d'informations massives et altération de données arbitraire ; n'importe quelle application malveillante cohabitant sur le même terminal Android peut interroger ou modifier le contenu géré par ces composants.
- **Remédiation**: Restreindre la visibilité des composants internes en positionnant l'attribut `android:exported` à `false`. Si le partage de données inter-applicatif est requis, appliquer impérativement des règles de contrôle rigoureuses via les filtres `readPermission` et `writePermission`.
- **Référence OWASP**: MASVS-PLATFORM-2

## D. Faux positifs notables

* **FIND-008 (Intégration de SDK publicitaires et analytiques - Google AdMob/Analytics/Tag Manager)** : 
  Ces librairies tierces ont été levées par les scanners automatisés comme potentiellement suspectes en raison de la collecte de télémétrie. Cependant, dans le cadre de cette application d'évaluation pédagogique, l'inclusion de ces modules officiels est structurelle et ne constitue pas une faille de sécurité exploitable. Le risque associé a donc été déclassé de *High* à un statut de non-pertinence pour la sécurité bancaire directe.
  
* **FIND-009 (Divulgation d'adresses de messagerie électronique au sein des sources)** : 
  La présence de 6 chaînes de caractères identifiées comme des adresses e-mail par BeVigil correspond en réalité aux coordonnées des auteurs du projet open source d'origine. Aucun secret de production ni donnée sensible liée à l'infrastructure cible n'étant exposé à travers ces adresses, ce constat est requalifié en niveau *Low* sans impact opérationnel.

## E. Recommandations prioritaires

1. **Durcissement immédiat de la configuration du Manifeste Android** : Rectifier de toute urgence les trois attributs de sécurité critiques dans `AndroidManifest.xml` en imposant `android:debuggable="false"`, `android:allowBackup="false"`, et en appliquant l'isolation sur les composants exposés (`android:exported="false"`). Cette action offre le meilleur ratio effort/réduction des risques.
2. **Refactorisation de la couche de chiffrement et de hachage** : Mener une refonte du fichier de gestion des mots de passe `ChangePassword.java` pour éliminer les fonctions MD5/SHA-1 au profit de mécanismes robustes (Argon2/SHA-256). Il conviendra également de s'assurer de l'usage exclusif de générateurs de nombres pseudo-aléatoires sécurisés (`SecureRandom`) et de basculer la signature des futurs APK vers un schéma moderne (V2/V3) utilisant SHA256withRSA.
3. **Mise en conformité des flux et de la politique réseau** : Interdire définitivement tout échange de données via le protocole HTTP en clair. Configurer l'application pour n'autoriser que les suites de chiffrement TLS 1.2/1.3 au moyen d'un profil de sécurité réseau standardisé et déployer un mécanisme de vérification d'empreinte de certificat (*pinning*).

## F. Annexes

- `01-bevigil/bevigil_export_summary.txt` — Rapport condensé de l'analyse automatique BeVigil
- `01-bevigil/bevigil_notes.md` — Observations et annotations relatives aux résultats BeVigil
- `02-yaazhini/yaazhini_report.html` — Restitution visuelle native issue de l'outil Yaazhini
- `02-yaazhini/yaazhini_notes.md` — Analyse détaillée et interprétation des alertes Yaazhini
- `03-triage/triage.csv` — Matrice consolidée de qualification et de suivi des vulnérabilités
- `03-triage/owasp_mapping.md` — Table de correspondance technique avec le référentiel OWASP MASVS
