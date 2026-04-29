## US5 — Analyse du manifest décompilé et recherche de secrets

L'analyse s'appuie sur le `AndroidManifest.xml` produit par Apktool, ainsi que sur les chaînes extraites du bytecode DEX.

**Commandes utilisées :**
```bash
cat app_src/AndroidManifest.xml
grep -i "api\|key\|secret\|url\|http" app_src/res/values/strings.xml
```

### 5.1 Métadonnées de l'application

```xml
package="com.fpe.comptenickel"
android:compileSdkVersion="28"
android:compileSdkVersionCodename="9"
```

L'app est compilée contre Android 9 (API 28) mais le `minSdk=19` (Android 4.4) confirme la vulnérabilité HIGH remontée par MobSF.

### 5.2 Flags critiques de l'élément `<application>`

```xml
android:allowBackup="false"
android:usesCleartextTraffic="true"
```

**`allowBackup="false"` :** C'est un point positif. Le backup ADB est désactivé — les données de l'app ne peuvent pas être extraites via `adb backup` sur un terminal non rooté.

**`usesCleartextTraffic="true"` :** En revanche, le trafic HTTP non chiffré est autorisé pour l'ensemble de l'application. Pour une app bancaire, c'est inacceptable. Il n'y a aucune `network_security_config.xml` visible qui restreindrait ce flag à des domaines spécifiques.

### 5.3 `DevSettingsActivity` — activité de debug en production

```xml
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity"/>
```

**Risque élevé.** Aucun attribut `exported="false"` n'est défini sur cette activité. Par défaut en Android, une activity sans `exported` explicite et sans intent-filter est non-exportée depuis API 31 — mais sur les versions ciblées ici (compileSdk=28), les règles sont différentes. Cette activité permet d'activer le rechargement JS à chaud, d'ouvrir le menu debug React Native, ou de pointer l'app vers un bundle JS arbitraire sur un serveur distant. Sa présence en build de production est une erreur de configuration caractérisée.

### 5.4 Composants Pushwoosh non protégés

Le manifest décompilé confirme exactement ce que MobSF a remonté :

```xml
<!-- Service sans protection -->
<service android:name="com.pushwoosh.FcmRegistrationService">
    <intent-filter>
        <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
    </intent-filter>
</service>

<service android:name="com.pushwoosh.PushFcmIntentService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>

<!-- Content Provider exporté sans permission -->
<provider android:authorities="com.fpe.comptenickel.PushwooshSharedDataProvider"
    android:exported="true"
    android:name="com.pushwoosh.PushwooshSharedDataProvider"/>

<!-- Receiver exporté sans protection -->
<receiver android:exported="true"
    android:name="com.learnium.RNDeviceInfo.RNDeviceReceiver">
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER"/>
    </intent-filter>
</receiver>
```

**Risque :** Ces composants exportés sans permission peuvent être invoqués par n'importe quelle application installée sur l'appareil. Le `PushwooshSharedDataProvider` expose en lecture les données partagées du SDK push.

### 5.5 Permissions déclarées

| Permission | Catégorie | Justification dans une app bancaire |
|---|---|---|
| `INTERNET` | Normale | Légitime |
| `ACCESS_NETWORK_STATE` | Normale | Légitime |
| `ACCESS_WIFI_STATE` | Normale | Légitime |
| `VIBRATE` | Normale | Légitime (notifications) |
| `WAKE_LOCK` | Normale | Légitime |
| `RECEIVE_BOOT_COMPLETED` | Normale | Push au démarrage — à justifier |
| `READ_CONTACTS` | **Dangereuse** | **Non justifiée** |
| `WRITE_EXTERNAL_STORAGE` | **Dangereuse** | **Risquée** — stockage accessible par toutes les apps |
| `USE_FINGERPRINT` | **Dangereuse** | Légitime (authentification biométrique) |
| `com.amazon.device.messaging.permission.RECEIVE` | Normale | ADM (Amazon Device Messaging) |
| `com.android.vending.CHECK_LICENSE` | Normale | Vérification Play Store |

En plus des permissions fonctionnelles, l'app déclare un ensemble de permissions constructeur (Samsung, HTC, Sony, Huawei, Oppo) pour la gestion des badges d'icône. Ces permissions sont de niveau normal mais élargissent inutilement la surface d'exposition.

**`READ_CONTACTS` :** Aucun cas d'usage bancaire évident. À supprimer ou à documenter explicitement.

**`WRITE_EXTERNAL_STORAGE` :** Tout fichier écrit sur le stockage externe est lisible par toutes les apps ayant `READ_EXTERNAL_STORAGE`. Si l'app y écrit des relevés, des tokens ou des données de session, ceux-ci sont compromis.

### 5.6 Secrets critiques dans `res/values/strings.xml`

C'est le finding le plus grave de l'analyse. Le fichier `strings.xml`, reconstruit par Apktool depuis les ressources compilées de l'APK, contient en clair des identifiants et clés d'accès à des services tiers.

```bash
cat app_src/res/values/strings.xml
```

**Secrets identifiés :**

| Clé | Valeur | Sévérité |
|---|---|---|
| `google_api_key` | `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg` | **Critique** |
| `google_crash_reporting_api_key` | `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg` | **Critique** |
| `firebase_database_url` | `https://application-client-nickel.firebaseio.com` | **Critique** |
| `default_web_client_id` | `717748501407-v621tmfvqd7etdouc3df5vuv31scle0g.apps.googleusercontent.com` | Haute |
| `google_app_id` | `1:717748501407:android:e0cdeb712e27275d` | Haute |
| `gcm_defaultSenderId` | `717748501407` | Haute |
| `GCM_PROJECT_NUMBER` | `246766419677` | Haute |
| `PUSHWOOSH_APPID` | `DDF11-D1BF9` | Haute |
| `google_storage_bucket` | `application-client-nickel.appspot.com` | Moyenne |
| `project_id` | `application-client-nickel` | Moyenne |
| `com.crashlytics.android.build_id` | `9ad6d741-4e10-4abd-9f12-163a0e3fb0b2` | Faible |

**Analyse des risques :**

**`google_api_key` / `google_crash_reporting_api_key` — Critique**  
La clé API Google `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg` est exposée en clair. Une clé API Google sans restriction d'usage ou de domaine permet à un attaquant de l'utiliser pour consommer des services Google au nom du projet (Maps, Firebase, etc.), générant des coûts ou accédant à des données. Elle devrait être restreinte aux adresses IP/applications autorisées via la Google Cloud Console, et idéalement ne jamais figurer dans un fichier compilé dans l'APK.

**`firebase_database_url` — Critique**  
L'URL `https://application-client-nickel.firebaseio.com` est l'adresse exacte de la base de données Firebase Realtime du projet Nickel. Avec cette URL et la clé API, un attaquant peut tenter d'accéder directement à la base. L'US8 testera si les règles de sécurité Firebase sont correctement configurées.

**`default_web_client_id` — Haute**  
L'OAuth client ID Google est exposé. Il peut être utilisé pour des attaques de type redirect URI ou pour du phishing ciblé usurpant l'identité de l'application.

**`PUSHWOOSH_APPID` — Haute**  
L'identifiant de l'application Pushwoosh `DDF11-D1BF9` permet potentiellement d'envoyer des notifications push au nom de l'application si l'API Pushwoosh n'est pas correctement sécurisée côté serveur.

### 5.7 Endpoints API dans `strings.xml`

Les URLs sont également stockées en ressources, pas seulement dans le DEX :

```xml
<string name="ACCOUNT_ENDPOINT">https://api.nickel.eu/customer-banking-api</string>
<string name="CUSTOMER_AUTHENTICATION_ENDPOINT">https://api.nickel.eu/customer-authentication-api</string>
<string name="PERSONAL_SPACE_API_ENDPOINT">https://api.nickel.eu/personal-space-api</string>
<string name="ENVIRONMENT">production</string>
```

La présence de `ENVIRONMENT=production` confirme qu'il s'agit d'un build de production réel, pas d'un environnement de test. Toutes les clés et URLs exposées sont donc actives.

### 5.8 Strings React Native de debug en production

```xml
<string name="catalyst_debugjs">Debug JS Remotely</string>
<string name="catalyst_hot_module_replacement">Enable Hot Reloading</string>
<string name="catalyst_live_reload">Enable Live Reload</string>
<string name="catalyst_settings_title">Catalyst Dev Settings</string>
```

Ces chaînes confirment que le bundle React Native de développement (ou ses artefacts) est présent dans le build de production. Elles sont liées à la `DevSettingsActivity` identifiée dans le manifest.

### 5.9 Fichier `assets/crashlytics-build.properties`

Présent en clair dans l'APK, lisible sans décompilation :

```properties
version_name=2.12.0
package_name=com.fpe.comptenickel
build_id=9ad6d741-4e10-4abd-9f12-163a0e3fb0b2
version_code=150
app_name=Nickel
```

Le `build_id` est cohérent avec la valeur retrouvée dans `strings.xml` (`com.crashlytics.android.build_id`).

### 5.10 SDKs tiers identifiés

| SDK | Origine | Usage déclaré | Risque |
|---|---|---|---|
| Firebase Analytics | Google (US) | Tracking comportemental | Transfert de données hors EU |
| Firebase Crashlytics | Google (US) | Reporting de crashs | Logs potentiellement sensibles |
| Firebase Messaging | Google (US) | Notifications push | — |
| **Pushwoosh** | **Russie** | Notifications push | **Alerte DHS 2022 — souveraineté des données** |
| React Native | Meta (US) | Framework UI | Surface d'attaque bundle JS |
| Amazon Device Messaging | Amazon (US) | Push sur devices Amazon | — |

---