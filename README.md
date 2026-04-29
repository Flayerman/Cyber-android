# 🔒 Cyber-android — Rapport d'Analyse de Sécurité Android

Un rapport complet d'audit de sécurité de l'application Android **Nickel v2.12.0**, réalisé dans le cadre du module **Sécurité Android** de l'école ESIEE-IT (E3IN 2025/2026).

---

## 📋 Vue d'ensemble

Ce projet documente une analyse de sécurité approfondie d'une application Android, couvrant :

- ✅ Vérification d'intégrité et métadonnées de l'APK
- ✅ Analyse automatisée avec MobSF (Mobile Security Framework)
- ✅ Extraction et inspection des fichiers de l'application
- ✅ Décompilation et analyse du code source
- ✅ Analyse du manifeste Android
- ✅ Évaluation des permissions
- ✅ Analyse des certificats et signature
- ✅ Inspection des dépendances et vulnérabilités
- ✅ Tests dynamiques (runtime analysis)
- ✅ Analyse des données sensibles
- ✅ Recommendations de sécurité

---

## 📁 Structure du projet

```
Cyber-android/
├── README.md                    # Ce fichier
├── rapport_final.md             # Rapport synthétisé
├── US1/                         # Téléchargement et vérification de l'APK
├── US2/                         # Analyse automatisée avec MobSF
├── US3/                         # Extraction de fichiers
├── US4/                         # Décompilation et inspection du code
├── US5/                         # Analyse du manifeste Android
├── US6/                         # Évaluation des permissions
├── US7/                         # Analyse des certificats et signature
├── US8/                         # Inspection des dépendances
├── US9/                         # Tests dynamiques
├── US10/                        # Analyse des données sensibles
└── US11/                        # Recommendations et remédiation
```

### Détail des User Stories

| US | Titre | Contenu |
|---|---|---|
| **US1** | Téléchargement et vérification | Vérification d'intégrité SHA-256, métadonnées du fichier APK |
| **US2** | Analyse MobSF | Scorecard de sécurité automatisée, findings par sévérité |
| **US3** | Extraction de fichiers | Extraction du manifest, DEX, et structure interne de l'APK |
| **US4** | Décompilation | Analyse du bytecode Dalvik et code source reconstruit |
| **US5** | Manifeste Android | Permissions, composants, filtres d'intent |
| **US6** | Permissions | Audit des permissions déclarées et dangereuses |
| **US7** | Certificats | Analyse de la signature et chaîne de certificats |
| **US8** | Dépendances | Inspection des librairies tierces et vulnérabilités connues |
| **US9** | Tests dynamiques | Monitoring runtime, interception, fuzzing |
| **US10** | Données sensibles | Localisation des données hardcodées, stockage insécurisé |
| **US11** | Recommendations | Plan de remédiation et best practices |

---

## 🔍 Principaux résultats

### Score de sécurité (MobSF)
- **Note globale :** 49 / 100
- **Grade :** B
- **Niveau de risque :** Medium Risk
- **Trackers détectés :** 3

### Vulnérabilités identifiées
| Sévérité | Nombre |
|---|---|
| 🔴 **High** | 3 |
| 🟠 **Medium** | 20 |
| 🔵 **Info** | 2 |
| 🟢 **Secure** | 2 |
| ⚠️ **Hotspot** | 1 |

### Principaux problèmes
- ❌ Trafic en clair activé (`usesCleartextTraffic=true`)
- ❌ Support d'Android 4.4 (APIs obsolètes, `minSdk=19`)
- ❌ Composants non protégés (BroadcastReceiver, Services, ContentProvider)
- ❌ Chiffrement CBC avec padding vulnérable
- ❌ Utilisation de MD5 et générateur de nombres aléatoires non sécurisé

---

## 🛠️ Outils utilisés

- **MobSF v4.5.0** — Analyse statique automatisée
- **Apktool** — Décompilation et extraction d'APK
- **Jadx** — Décompilation Java à partir du bytecode Dalvik
- **APK Analyzer** — Inspection de structure
- **Frida** — Instrumentation et tests dynamiques
- **Network monitoring tools** — Analyse du trafic réseau

---

## 📖 Comment utiliser ce rapport

1. **Lecture rapide :** Consultez `rapport_final.md` pour un résumé exécutif
2. **Analyse détaillée :** Naviguez par User Story (US1-US11) pour chaque aspect
3. **Findings spécifiques :** Consultez US2 pour le scorecard MobSF complet
4. **Recommendations :** Lisez US11 pour le plan d'action et les remediations

---

## 📊 Métadonnées du projet

| Élément | Détail |
|---|---|
| **Application analysée** | Nickel v2.12.0 |
| **Taille APK** | ~23,2 Mo (24 325 620 octets) |
| **SHA-256** | `f01f08c8b6e2fe4612e81bc7e3a3ba9440dad0d7a962f4e67640390dd721d528` |
| **Module** | Sécurité Android — ESIEE-IT E3IN |
| **Responsable pédagogique** | Wael MEGUEBLI |
| **Date** | Avril 2026 |
| **Auteurs** | ENGASSER Antoine, HIBOUX Antonyn |

---

## 📞 Contact

Pour toute question ou clarification sur ce rapport :

- **Antoine ENGASSER** — Analyse de sécurité
- **Antonyn HIBOUX** — Analyse de sécurité

**École :** ESIEE-IT

---

**Généré :** Avril 2026  
**Dernière mise à jour :** Avril 2026
