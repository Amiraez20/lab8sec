# Notes d'analyse Yaazhini — InsecureBankv2

## Éléments identifiés

### Élément 1: Communication non sécurisée (Insecure communication)
- **Localisation**: Rapport Yaazhini — Findings 1 / sources/com/android/insecurebankv2/
- **Description**: L'application utilise des protocoles de communication non sécurisés (HTTP en clair). La valeur du protocole est définie en dur dans le code source, sans aucun mécanisme de chiffrement.
- **Impact potentiel**: Un attaquant en position MITM (man-in-the-middle) peut intercepter et lire les données échangées entre l'application et le serveur, incluant des credentials et des données bancaires.
- **Remédiation suggérée**: Forcer HTTPS pour toutes les communications réseau. Implémenter TLS 1.2 minimum et activer le certificate pinning pour prévenir les attaques MITM.

### Élément 2: Mode Debug activé (Android debuggable enabled)
- **Localisation**: Rapport Yaazhini — Findings 3 / AndroidManifest.xml — android:debuggable="true"
- **Description**: L'attribut android:debuggable='true' est présent dans le tag application du manifest. L'application peut être déboguée même sur des appareils non rootés via ADB.
- **Impact potentiel**: Un attaquant avec accès physique au device peut extraire des données sensibles, injecter du code ou contourner les contrôles de sécurité via ADB.
- **Remédiation suggérée**: Mettre android:debuggable="false" dans AndroidManifest.xml pour les builds de production. Ne jamais livrer une app avec le mode debug activé.

### Élément 3: Backup non protégé (Android backup vulnerability)
- **Localisation**: Rapport Yaazhini — Findings 4 / AndroidManifest.xml — android:allowBackup="true"
- **Description**: L'attribut android:allowBackup='true' est activé dans le manifest. Les données de l'application peuvent être sauvegardées via ADB backup sans rooter le device.
- **Impact potentiel**: N'importe qui avec accès USB au téléphone peut extraire toutes les données privées de l'application (tokens, préférences, base de données locale).
- **Remédiation suggérée**: Mettre android:allowBackup="false" dans AndroidManifest.xml ou configurer des règles de backup sélectives avec android:fullBackupContent.

### Élément 4: Algorithme de hachage faible — MD5 et SHA-1 (Weak hash)
- **Localisation**: Rapport Yaazhini — Findings 7 & 8 / sources/com/android/insecurebankv2/ChangePassword.java
- **Description**: L'application utilise MD5 et SHA-1 pour hasher des données potentiellement sensibles (mots de passe). Ces algorithmes sont considérés comme cryptographiquement faibles et obsolètes.
- **Impact potentiel**: Un attaquant peut retrouver les valeurs originales via rainbow tables ou attaques par force brute. SHA-1 est officiellement cassé depuis 2017.
- **Remédiation suggérée**: Utiliser SHA-256 minimum. Pour les mots de passe, utiliser PBKDF2, bcrypt ou Argon2 avec un sel unique.

### Élément 5: Providers exportés sans protection (Improper export of providers)
- **Localisation**: Rapport Yaazhini — Findings 5 / AndroidManifest.xml — android:exported='true'
- **Description**: Des content providers sont exportés publiquement dans le manifest (android:exported='true') sans permissions requises. N'importe quelle application peut y accéder.
- **Impact potentiel**: Une application malveillante installée sur le même appareil peut lire, modifier ou supprimer les données exposées par ces providers (données utilisateur, transactions).
- **Remédiation suggérée**: Marquer les providers comme android:exported="false" s'ils ne sont pas destinés à être partagés. Sinon, ajouter des permissions android:readPermission et android:writePermission.

### Élément 6: Valeurs aléatoires insuffisantes (Use of insufficiently random values)
- **Localisation**: Rapport Yaazhini — Findings 2 / code source utilisant java.util.Random
- **Description**: L'application utilise java.util.Random au lieu de java.security.SecureRandom pour la génération de valeurs aléatoires dans un contexte de sécurité.
- **Impact potentiel**: La génération de nombres pseudo-aléatoires non cryptographique peut être prédite, compromettant des tokens de session ou des nonces de sécurité.
- **Remédiation suggérée**: Remplacer java.util.Random par java.security.SecureRandom pour toute génération de valeurs à usage cryptographique ou sécuritaire.

### Élément 7: Signature APK faible (Insecure signature — SHA1withRSA)
- **Localisation**: Rapport Yaazhini — Findings 9 — certificat de signature de l'application
- **Description**: L'APK est signé avec l'algorithme SHA1withRSA, considéré comme obsolète et vulnérable aux attaques de collision.
- **Impact potentiel**: Vulnérabilité théorique à des attaques de collision sur la signature, permettant potentiellement la falsification de l'identité du développeur.
- **Remédiation suggérée**: Signer avec SHA256withRSA — utiliser le schéma de signature Android v2 au minimum.
