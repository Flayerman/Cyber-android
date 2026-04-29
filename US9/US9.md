## US9 — Rapport technique consolidé

Ce document constitue le rapport final (`rapport_final.pdf`). Il agrège les findings de toutes les user stories.

### Score global

| Outil | Score | Grade | Risque |
|---|---|---|---|
| MobSF v4.5.0 | **49/100** | **B** | Medium Risk |

### Synthèse des vulnérabilités

| ID | Source | Vulnérabilité | Sévérité |
|---|---|---|---|
| V01 | MANIFEST | `minSdk=19` — installation possible sur Android 4.4 non patché | **Haute** |
| V02 | MANIFEST | `usesCleartextTraffic=true` — trafic HTTP non chiffré autorisé | **Haute** |
| V03 | CODE | Chiffrement CBC/PKCS5-PKCS7 — vulnérable aux attaques padding oracle | **Haute** |
| V04 | CERTIFICATE | Vulnérabilité Janus (CVE-2017-13156) | Moyenne |
| V05 | CERTIFICATE | Algorithme de certificat vulnérable aux collisions de hash | Moyenne |
| V06 | MANIFEST | BroadcastReceiver/Services Pushwoosh non protégés (`exported=true`) | Moyenne |
| V07 | CODE | Stockage externe accessible en lecture/écriture par toutes les apps | Moyenne |
| V08 | CODE | Utilisation de MD5 (algorithme de hash obsolète) | Moyenne |
| V09 | CODE | Requêtes SQL brutes — risque d'injection SQL | Moyenne |
| V10 | CODE | Secrets potentiellement hardcodés dans le binaire | Moyenne |
| V11 | CODE | Générateur de nombres aléatoires non sécurisé (`java.util.Random`) | Moyenne |
| V12 | CODE | Fichiers temporaires — données sensibles potentiellement exposées | Moyenne |
| V13 | FIREBASE | Firebase Remote Config activé | Moyenne |
| V14 | TRACKERS | 3 trackers vie privée (Firebase Analytics, Crashlytics, Pushwoosh) | Moyenne |
| V15 | STRINGS.XML | Clé API Google `AIzaSyBTg...` exposée en clair | **Critique** |
| V16 | STRINGS.XML | URL Firebase Realtime DB `application-client-nickel.firebaseio.com` exposée | **Critique** |
| V17 | STRINGS.XML | OAuth Client ID Google exposé (`default_web_client_id`) | Haute |
| V18 | STRINGS.XML | Pushwoosh App ID `DDF11-D1BF9` exposé | Haute |
| V19 | STRINGS.XML | Endpoints API bancaires production en clair (`ENVIRONMENT=production`) | Haute |
| V20 | MANIFEST | `DevSettingsActivity` sans `exported="false"` en production | Haute |
| V21 | PERMISSIONS | `READ_CONTACTS` non justifié fonctionnellement | Moyenne |
| V22 | STRINGS.XML | Strings de debug React Native présentes en build production | Moyenne |
| V23 | CODE | Logs applicatifs potentiellement sensibles | Info |

**Points positifs identifiés par MobSF :**
- Certificate pinning SSL implémenté (protection contre les attaques MITM)
- Détection de root présente dans le code

### Recommandations

**V01 — minSdk=19 :** Relever le `minSdkVersion` à 26 minimum (Android 8.0, Oreo). Android 4.4 date de 2013 et ne reçoit plus aucun patch de sécurité. La base utilisateurs sur ces versions est négligeable pour une app bancaire.

**V02 — Cleartext traffic :** Ajouter une `network_security_config.xml` qui désactive le cleartext sur tous les domaines. Aucune communication bancaire ne devrait transiter en HTTP.

**V03 — Padding oracle (CBC/PKCS7) :** Migrer vers AES-GCM (mode AEAD) qui garantit à la fois la confidentialité et l'intégrité sans être vulnérable aux attaques oracle.

**V04 — Janus :** Vérifier que l'APK est signé avec le schéma v2 ou v3 minimum. La faille Janus n'affecte que les APK signés uniquement en v1 (JAR signing).

**V06 — Composants Pushwoosh exportés :** Ajouter `android:exported="false"` sur tous les receivers et services qui n'ont pas besoin d'être accessibles depuis l'extérieur. Évaluer le remplacement de Pushwoosh par FCM seul — le SDK a fait l'objet d'une alerte DHS en 2022.

**V08/V09 — MD5 et SQL brut :** Remplacer MD5 par SHA-256 minimum. Utiliser des requêtes paramétrées (PreparedStatement) pour toutes les interactions SQLite.

**V10 — Secrets hardcodés :** Auditer le code source pour supprimer toute clé, token ou secret en dur. Utiliser Android Keystore ou un mécanisme de configuration serveur.

**V11 — RNG insécurisé :** Remplacer `java.util.Random` par `java.security.SecureRandom` pour toute génération de valeurs à caractère sécuritaire.

---