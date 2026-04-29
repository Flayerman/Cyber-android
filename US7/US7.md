## US7 — Signature de l'APK

Un APK recompilé sans signature est refusé à l'installation par Android. La signature se fait avec une clé de test générée localement.

```bash
# Génération d'un keystore de test
keytool -genkey -v \
  -keystore test.keystore \
  -alias testkey \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000

# Signature avec apksigner (Android Build Tools)
apksigner sign \
  --ks test.keystore \
  --ks-key-alias testkey \
  --out rebuilt_signed.apk \
  rebuilt.apk

# Vérification
apksigner verify --verbose rebuilt_signed.apk
```

**Résultat de la signature :**
```
$ apksigner sign \
  --ks test.keystore \
  --ks-key-alias testkey \
  --out rebuilt_signed.apk \
  rebuilt.apk
Keystore password for signer #1: [mot de passe saisi]
```

**Vérification :**
```
$ apksigner verify --verbose rebuilt_signed.apk
Verifies
Verified using v1 scheme (JAR signing):                 true
Verified using v2 scheme (APK Signature Scheme v2):     true
Verified using v3 scheme (APK Signature Scheme v3):     true
Verified using v4 scheme (APK Signature Scheme v4):     false
Verified for SourceStamp:                               false
Number of signers: 1
```

**Analyse du résultat :**

| Schéma | Statut | Signification |
|---|---|---|
| v1 (JAR signing) | ✅ true | Compatible Android < 7. Seul, il serait vulnérable à Janus (CVE-2017-13156) |
| v2 (APK Signature Scheme v2) | ✅ true | Obligatoire depuis Android 7 (API 24). Couvre l'APK entier |
| v3 (APK Signature Scheme v3) | ✅ true | Ajoute la rotation de clé — recommandé depuis Android 9 |
| v4 (APK Signature Scheme v4) | ❌ false | Requis uniquement pour ADB incremental install — non critique |
| SourceStamp | ❌ false | Métadonnée optionnelle de distribution — non requis |

La signature v1+v2+v3 confirme que l'APK recompilé est installable sur Android 7+ et que la vulnérabilité Janus est neutralisée (v2 et v3 couvrent l'intégralité du binaire, pas seulement le manifest). C'est également mieux que l'APK original qui, d'après MobSF, présentait un risque lié à Janus.

> **Livrable :** `rebuilt_signed.apk` + output `apksigner verify` dans `US7/`.

---