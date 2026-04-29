## US8 — Vérification Firebase

L'URL Firebase a été identifiée directement dans `strings.xml` — pas besoin de la chercher dans le DEX.

```xml
<string name="firebase_database_url">https://application-client-nickel.firebaseio.com</string>
<string name="project_id">application-client-nickel</string>
<string name="google_storage_bucket">application-client-nickel.appspot.com</string>
```

### 8.1 Test d'accès public à la base Realtime Database

```bash
curl -s "https://application-client-nickel.firebaseio.com/.json"
```

**Résultat :**
```json
{
  "error" : "Permission denied"
}
```

### 8.2 Analyse

La base Firebase Realtime Database existe (`HTTP 200` avec corps JSON) mais les règles de sécurité refusent tout accès non authentifié. C'est le comportement attendu et correct.

| Critère | Résultat |
|---|---|
| Base Firebase accessible publiquement | ❌ Non — Permission denied |
| Règles de sécurité configurées | ✅ Oui |
| Fuite de données via accès anonyme | ❌ Non |

### 8.3 Risque résiduel
 
Bien que la base soit protégée, **deux informations critiques restent exposées** dans `strings.xml` :
 
- L'URL exacte de la base Firebase (`firebase_database_url`)
- La clé API Google (`google_api_key` : `AIzaSy***************************`)
Un attaquant disposant de ces deux éléments peut :
 
- Tenter une authentification Firebase avec des credentials compromis (credential stuffing)
- Utiliser la clé API pour accéder à d'autres services Google du projet (Cloud Storage, Maps, Firebase Auth)
- Cibler l'endpoint Firebase Auth (`/v1/accounts:signInWithPassword`) avec la clé API exposée

**Recommandation :** supprimer `firebase_database_url` et `google_api_key` des ressources compilées dans l'APK. Ces valeurs doivent être récupérées dynamiquement depuis un backend ou protégées via Android Keystore.

> **Livrable :** output `curl` dans `US8/`.

---