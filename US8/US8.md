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

> **Livrable :** `firebase_check.md` + output `curl` dans `US8/`.

---