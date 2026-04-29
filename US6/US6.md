## US6 — Recompilation de l'APK

Après décompilation avec Apktool, la recompilation vérifie que le cycle décompilation → recompilation est fonctionnel, ce qui est un prérequis pour toute modification de l'APK.

```bash
# Recompilation
apktool b app_src/ -o rebuilt.apk

# Vérification du fichier produit
file rebuilt.apk
ls -lh rebuilt.apk
```

**Log de build (Apktool 2.9.3) :**
```
I: Using Apktool 2.9.3
I: Checking whether sources has changed...
I: Checking whether resources has changed...
I: Building resources...
I: Copying libs... (/lib)
I: Copying libs... (/kotlin)
I: Building apk file...
I: Copying unknown files/dir...
I: Built apk into: rebuilt.apk
```

**Vérification du fichier produit :**
```
$ file rebuilt.apk
rebuilt.apk: Zip archive data, at least v1.0 to extract, compression method=store
$ ls -lh rebuilt.apk
-rw-r--r-- 1 ubuntu ubuntu 23M Apr 29 11:21 rebuilt.apk
```

> **Livrable :** `rebuilt.apk` + `build.log` déposés dans `US6/` sur GitHub.

