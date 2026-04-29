## US4 — Décompilation avec Apktool

Apktool permet de reconstituer les ressources lisibles (smali, manifest XML, res/).

```bash
# Installation d'Apktool (WSL2/Ubuntu)
sudo apt update && sudo apt install -y apktool

# Décompilation
apktool d app.apk -o app_src/

```

> **Livrable :** Le dossier `app_src/` est archivé en `app_src.zip` et versionné sur GitHub dans `US4/`.

---