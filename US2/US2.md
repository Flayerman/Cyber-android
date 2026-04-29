## US2 — Analyse automatisée avec MobSF

L'APK a été soumis à MobSF v4.5.0 via la plateforme en ligne [mobsf.live](https://mobsf.live).

**Procédure :**
1. Accéder à https://mobsf.live
2. Uploader `app.apk` via l'interface
3. Attendre la fin du scan statique
4. Exporter le scorecard PDF via le dashboard

**Pour une installation locale :**
```bash
docker pull opensecurity/mobile-security-framework-mobsf:latest
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
# Interface disponible sur http://localhost:8000
```

> **Livrable :** `mobsf_report.pdf` joint en annexe (`US2/mobsf_report.pdf`).

### Résultats MobSF — Nickel 2.12.0

| Métrique | Valeur |
|---|---|
| Score de sécurité | **49 / 100** |
| Grade | **B** |
| Niveau de risque global | **Medium Risk** |
| Trackers vie privée | **3** User/Device Trackers |

**Distribution des findings :**

| Sévérité | Nombre |
|---|---|
| High | 3 |
| Medium | 20 |
| Info | 2 |
| Secure | 2 |
| Hotspot | 1 |

### Détail des findings

**HIGH**

| Source | Description |
|---|---|
| MANIFEST | L'app peut être installée sur Android 4.4–4.4.4 non patché (`minSdk=19`) |
| MANIFEST | Cleartext traffic activé (`android:usesCleartextTraffic=true`) |
| CODE | Chiffrement CBC avec padding PKCS5/PKCS7 — vulnérable aux attaques padding oracle |

**MEDIUM (sélection significative)**

| Source | Description |
|---|---|
| CERTIFICATE | Application vulnérable à la faille Janus |
| CERTIFICATE | Algorithme de certificat potentiellement vulnérable aux collisions de hash |
| MANIFEST | `BroadcastReceiver` RNDeviceInfo non protégé (`exported=true`) |
| MANIFEST | Services Pushwoosh non protégés avec intent-filter exposé |
| MANIFEST | Content Provider Pushwoosh non protégé (`exported=true`) |
| MANIFEST | FirebaseInstanceIdReceiver — niveau de protection de la permission à vérifier |
| CODE | Lecture/écriture sur stockage externe accessible par toutes les apps |
| CODE | Utilisation de MD5 (hash faible, collisions connues) |
| CODE | Requêtes SQL brutes — risque d'injection SQL |
| CODE | Informations sensibles potentiellement hardcodées |
| CODE | Générateur de nombres aléatoires non sécurisé |
| CODE | Création de fichiers temporaires (données sensibles potentiellement exposées) |
| FIREBASE | Firebase Remote Config activé |
| TRACKERS | Présence de trackers de vie privée |
| SECRETS | Secrets potentiellement hardcodés dans le code |

**INFO**

| Source | Description |
|---|---|
| CODE | L'app journalise des informations (logs potentiellement sensibles) |
| FIREBASE | L'app communique avec une base Firebase |

**SECURE** (points positifs)

| Source | Description |
|---|---|
| CODE | Détection de root présente |
| CODE | SSL certificate pinning implémenté (protection MITM) |

**HOTSPOT**

| Source | Description |
|---|---|
| PERMISSIONS | 2 permissions critiques identifiées |

---
