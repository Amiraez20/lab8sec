# Mapping OWASP — InsecureBankv2

## FIND-001: Communication non sécurisée HTTP
- **Catégorie OWASP**: MASVS-NETWORK
- **Référence spécifique**: MASVS-NETWORK-1
- **Justification**: Toute communication réseau doit être chiffrée. L'utilisation de HTTP en clair expose les données en transit, en violation directe du standard MASVS sur la sécurité réseau.

## FIND-002: Mode Debug activé
- **Catégorie OWASP**: MASVS-CODE
- **Référence spécifique**: MASVS-CODE-2
- **Justification**: Le mode debug activé en production facilite le reverse engineering et l'extraction de données. Le standard MASVS exige la désactivation du debugging pour les releases.

## FIND-003: Backup ADB non protégé
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: MASVS-STORAGE-8
- **Justification**: Les sauvegardes automatiques peuvent exposer des données sensibles stockées localement si elles ne sont pas correctement protégées ou désactivées.

## FIND-004: Algorithmes de hachage faibles MD5/SHA-1
- **Catégorie OWASP**: MASVS-CRYPTO
- **Référence spécifique**: MASVS-CRYPTO-4
- **Justification**: L'utilisation d'algorithmes cryptographiques obsolètes compromet la confidentialité des données sensibles comme les mots de passe. Le MASVS impose des algorithmes à jour.

## FIND-005: Content Providers exportés sans permission
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-2
- **Justification**: Les composants Android exportés sans permission permettent à des applications tierces d'accéder aux données internes de l'application, violant le principe de moindre privilège.

## FIND-006: Générateur aléatoire non cryptographique
- **Catégorie OWASP**: MASVS-CRYPTO
- **Référence spécifique**: MASVS-CRYPTO-6
- **Justification**: java.util.Random est prévisible. Pour des usages de sécurité, un générateur cryptographiquement fort est obligatoire selon le MASVS.

## FIND-007: Signature APK faible
- **Catégorie OWASP**: MASVS-CODE
- **Référence spécifique**: MASVS-CODE-1
- **Justification**: La signature garantit l'intégrité de l'application. Un algorithme faible réduit cette garantie et peut permettre du tampering.

## FIND-008: Trackers tiers
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-1
- **Justification**: Les bibliothèques tierces peuvent accéder à des données sensibles et les transmettre sans consentement explicite de l'utilisateur.

## FIND-010: Receivers exportés sans protection
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-2
- **Justification**: Les receivers exportés sans permission peuvent être déclenchés par n'importe quelle application tierce, permettant des actions non autorisées.

## FIND-012: JavaScript activé dans WebView
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-5
- **Justification**: JavaScript dans les WebViews peut permettre des attaques XSS si du contenu non fiable est chargé, exposant les données de l'application.
