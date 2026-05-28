# Notes d'analyse BeVigil — InsecureBankv2

## Ce qui est certain
- Score de sécurité global : 7.4/10 (AVERAGE) selon BeVigil
- Package identifié : com.android.insecurebankv2 (version 1.0)
- 23 vulnérabilités de niveau LOW détectées dans le code source
- 4 activités exportées sans mécanisme de protection dans le fichier AndroidManifest.xml
- 1 secret potentiel repéré dans le fichier res/values/strings.xml
- 144 assets LOW répertoriés (fichiers, URLs, hostnames)
- Présence de 3 trackers : Google AdMob, Google Analytics, Google Tag Manager
- 15 bibliothèques tierces identifiées dans l'application
- 5 permissions classées comme risquées

## Ce qui est hypothèse
- Les secrets trouvés dans strings.xml pourraient correspondre à des clés API ou des credentials hardcodés
- Les activités exportées sans protection pourraient permettre un accès non autorisé depuis d'autres applications installées
- L'utilisation du protocole HTTP non sécurisé (Insecure HTTP Client) expose potentiellement les données en transit à des interceptions

## Points d'intérêt
- Attaque CBC Padding Oracle possible (3 occurrences) → vulnérabilité cryptographique sérieuse
- Stockage d'informations sensibles dans les Shared Preferences (2 occurrences)
- Informations sensibles écrites dans les Logs (2 occurrences)
- Requête SQL non paramétrée (1 occurrence) → risque d'injection SQL
- Vérification du root device présente mais implémentation SafetyNet absente

## Domaines et sous-domaines
- www.googleapis.com (4 occurrences)
- pagead2.googlesyndication.com (3 occurrences)
- www.google-analytics.com (2 occurrences)
- www.google.com
- googleads.g.doubleclick.net
- csi.gstatic.com
- schema.org
- www.googletagmanager.com
- ssl.google-analytics.com
- plus.google.com
- www.facebook.com
- accounts.google.com
- www.linkedin.com
- login.live.com
- www.paypal.com
- Total : 23 hostnames détectés

## Endpoints et APIs
- 2 endpoints REST API identifiés dans le code
- Points d'accès liés aux services Google (Analytics, AdMob, Tag Manager)

## URLs HTTP/HTTPS
- 61 URLs détectées au total
- Présence de connexions HTTP en clair (non chiffrées)
- URLs de services tiers (tracking, publicité)

## Emails et identifiants
- 6 adresses email de développeurs retrouvées dans le code source
- Adresses potentiellement liées aux comptes de développement

## Technologies détectées
- Android SDK 15 (minimum) / SDK 22 (cible)
- Google Play Services
- Google AdMob (monétisation publicitaire)
- Google Analytics (suivi comportemental)
- Google Tag Manager (gestion de tags)
- 15 bibliothèques tierces intégrées
