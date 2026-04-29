## US3 — Extraction de fichiers

L'APK est une archive ZIP standard. L'extraction se fait sans décompilateur.

```bash
# Extraction du manifest et du DEX
unzip app.apk AndroidManifest.xml -d apk_extracted/
unzip app.apk classes.dex -d apk_extracted/

# Vérification des hashs
sha256sum apk_extracted/AndroidManifest.xml
sha256sum apk_extracted/classes.dex
```

**Fichiers extraits et leurs hashs :**

| Fichier | SHA-256 |
|---|---|
| `AndroidManifest.xml` | `80b9b101e3bf02f03eec4f2c9c9ccf4d61403612e1435949b51cec8b85d7a276` |
| `classes.dex` | `57898c5978715da96560ed2692b4a8fa1a8e914b37988dcf369cc59951f6c429` |

**Structure de l'archive (extraits significatifs) :**

```
AndroidManifest.xml          (27 516 octets — format binaire AXML)
classes.dex                  (7 685 916 octets — bytecode Dalvik)
assets/crashlytics-build.properties
assets/fonts/[...]
META-INF/CERT.RSA
META-INF/CERT.SF
META-INF/MANIFEST.MF
```

L'APK contient un seul fichier `classes.dex`, ce qui signifie que le code n'est pas multidex — il tient dans la limite des 65 536 méthodes Dalvik.

---