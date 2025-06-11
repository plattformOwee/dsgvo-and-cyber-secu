### „Datenminimierung & Security“ am praktischen Beispiel deiner `users`-Collection

|DSGVO-Prinzip|Was es bedeutet|Konkrete Umsetzung in **deinem** JSON-Modell|
|---|---|---|
|**Nur erheben, was wirklich nötig ist** (Art. 5 Abs. 1 c)|_Data minimization_ – Felder weglassen oder optional machen, wenn sie nicht zwingend für die Kern-Funktion gebraucht werden.|**Pflicht:**• `email` (Login/Kontakt)• `password_hash` (Auth)• `username` (Anzeige)**Optional/weg-optimierbar:**• `firstname`, `lastname` → Falls keine Rechnungs­adresse nötig, erlaub `null`.• `profile_picture_url` → nur speichern, wenn User hochlädt.• `fingerprint_hash` → nur, wenn du wirklich Geräteduplikate/SPAM erkennen musst; andernfalls streichen.|
|**Speichern so kurz wie möglich**|Daten löschen oder anonymisieren, sobald Zweck erfüllt.|• `verification_code(_expiry)` nach erfolgreicher Verifizierung sofort löschen.• Chat-Nachrichten: biete Auto-Delete (z. B. 90 Tage) oder Nutzer-gesteuertes Löschen.• Soft-gelöschte Accounts nach 30 Tage Hard-wipe.|
|**Pseudonymisierung & Hashing**|Persönliche Daten unkenntlich machen, wo 1:1-Bezug nicht nötig ist.|• `password_hash`: Argon2id mit _unique Salt_ + ggf. _Pepper_.• `fingerprint_hash`: SHA-256(fingerprint + per-app-secret).• In Logfiles nur gekürzte IP (z. B. 192.168.xxx.xxx).|
|**Transport & Storage Security** (Art. 32)|Verschlüsselung, Zugriffsbeschränkung, Härtung der DB.|**MongoDB**1. Lauscht **nur** auf `127.0.0.1` oder Private-Subnet2. Auth aktiviert (`SCRAM-SHA-256`), eigene DB-User mit Least-Privilege3. TLS zwischen App-Server ↔ Mongo (`net.tls.mode: requireTLS`)4. IP-Whitelisting in Firewall/VPC5. Optional: **Encrypted Storage** (`--enableEncryption`)|
|**Backups & Dumps**|Backups tragen dieselbe Schutzpflicht.|• Voll-Backups AES-256 verschlüsselt.• Rotation/Retention < 30/60/365 Tage je nach Zweck.• Zugriff nur Key-Vault-geschützt.|
|**API-Härtung**|Verhindert unbefugten Bulk-Zugriff oder Injection.|• Rate-Limiting `POST /login` / `chats/send`• Input-Validation mit JSON-Schema• JWT/OAuth2-Scopes pro Route|

---

#### 1 · Minimaler Datensatz – Beispielstruktur

```jsonc
{
  "_id": "ObjectId",
  "email": "user@example.com",
  "password_hash": "argon2id$...",
  "username": "theusername",
  "created_at": "2025-06-11T14:00:00Z",
  "is_verified": true,

  // Optional blocks (nur angelegt, wenn gebraucht)
  "profile_picture_url": "...",
  "consents": {
    "analytics": { "granted": true,  "timestamp": "..." },
    "newsletter": { "granted": false, "timestamp": "..." }
  }
}
```

- Alle flüchtigen Felder (`verification_code`, `fingerprint_hash`) liegen in einer **separaten** Sub-Collection `security_tokens` und werden nach Erfüllung gelöscht.*
    

---

#### 2 · Passwort-Hashing-Snippet (PHP, Argon2id)

```php
$options = [
    'memory_cost' => PASSWORD_ARGON2_DEFAULT_MEMORY_COST, // >= 1024 KiB
    'time_cost'   => 4,
    'threads'     => 2,
];
$hash = password_hash($plainPw, PASSWORD_ARGON2ID, $options);
```

---

#### 3 · MongoDB-Hardening-Kurzübersicht

```yaml
# mongod.conf (Ausschnitt)
net:
  bindIp: 127.0.0.1            # nur lokal / privates Subnetz
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/ssl/myapp.pem
security:
  authorization: enabled       # Rollen erzwingen
  enableEncryption: true       # WiredTiger-Encryption at rest
```

> **Firewall/VPC-Regel:** Nur der App-Server‐IP-Range darf Port 27017 erreichen (kein Public-Ingress).

---

#### 4 · Datenminimierung bei Nachrichten

_Speichere_ nur: `sender_id`, `timestamp`, `text`.  
_Kein_: IP, Gerät, Geo, Lesestatus-Spam, sofern nicht geschäftskritisch.  
_Option_: Ende-zu-Ende-Verschlüsselung (NaCl/Libsodium) → dann liegt nur Ciphertext in Mongo.

---

### Next steps für dein Projekt

1. **Reduziere das Schema** auf den Minimal-Block + optionale Sub-Docs.
    
2. **Erstelle TTL-Indexe** (`expireAfterSeconds`) für `verification_code_expiry` & Soft-Delete-Flags.
    
3. **Aktiviere TLS + Auth** in MongoDB; stelle sicher, dass der Cluster **nicht** aus dem Internet erreichbar ist.
    
4. **Dokumentiere** die getroffenen Maßnahmen in RoPA-Spalte „TOMs“.
    

Wenn du konkrete Konfig-Dateien oder Sample-Scripts für die TTL-Wipes brauchst, sag Bescheid!