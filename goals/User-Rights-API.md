### Ja – zwei sichtbare Self-Service-Schaltflächen genügen, dahinter je ein Endpoint

Die DSGVO verlangt, dass Betroffene **ohne Hürden**

- **ihre Daten erhalten** (Art. 15 u. 20) und
    
- **sie löschen lassen** können (Art. 17).
    

Ein Button in _Einstellungen → Datenschutz_ ist die bequemste Lösung und wird von Aufsichtsbehörden ausdrücklich empfohlen („One-click export / delete“).

---

## 1 Front-End-UX (Flutter)

```dart
ListTile(
  leading: const Icon(Icons.download),
  title: const Text('Daten exportieren'),
  onTap: () => api.exportMyData().then(showDownloadDialog),
),
ListTile(
  leading: const Icon(Icons.delete_forever, color: Colors.red),
  title: const Text('Account löschen'),
  subtitle: const Text('Entfernt dein Profil nach 30 Tagen unwiderruflich'),
  onTap: () => showDeleteConfirmDialog(),
),
```

_Nach Bestätigung_ ruft die App:

- `POST /v1/me/export` → liefert eine einmalige **download-URL**
    
- `POST /v1/me/delete` → startet den Lösch-Workflow
    

---

## 2 Back-End-Skeleton (PHP + MongoDB)

```php
// 1) EXPORT
$app->post('/v1/me/export', function() use ($db, $userId) {
    // Pull everything you hold about this user
    $user   = $db->users->findOne(['_id' => $userId]);
    $chats  = $db->users->aggregate([
        ['$match' => ['_id' => $userId]],
        ['$project' => ['chats' => 1, '_id' => 0]]
    ]);
    $package = [
        'profile' => $user['user_data'],
        'chats'   => $chats->toArray(),
    ];

    // Store temp file (e.g., S3 signed URL, expires 24 h)
    $file = tempnam('/tmp', 'export_');
    file_put_contents($file, json_encode($package, JSON_PRETTY_PRINT));
    $signedUrl = $storage->putAndSign($file, "exports/$userId.json", 86400);

    return ['url' => $signedUrl];          // 200 OK
});

// 2) DELETE
$app->post('/v1/me/delete', function() use ($db, $userId) {
    // Soft-flag for grace period / fraud charge-back window
    $db->users->updateOne(
        ['_id' => $userId],
        ['$set' => ['deletion_requested_at' => new UTCDateTime()]]
    );
    // Immediate effects
    $auth->revokeTokens($userId);
    $mailer->sendGoodbyeMail($userId);

    return ['status' => 'scheduled'];      // 202 Accepted
});
```

### Asynchrone Hard-Wipe-Worker (Cron)

- **T+30 Tage**:
    
    - Löschen aus `users`, `sessions`, `consents`, Neben-Collections
        
    - `chats`: _Entweder_ komplette Nachricht löschen, _oder_ Nachrichtentext ersetzen durch „Nachricht vom Benutzer gelöscht“ (wenn du aus rechtlichen Gründen die Historie des Empfängers bewahren musst).
        
- **Backups**: Maskiere Benutzer-ID / E-Mail oder halte ein separates „tombstone“-Verzeichnis, das nach weiteren 30 Tagen rotiert wird.
    

---

## 3 Was muss ins **Export-Paket**?

| Kategorie                     | Beispielinhalt                                                 |
| ----------------------------- | -------------------------------------------------------------- |
| Profildaten                   | username, firstname, lastname, e-mail, created_at, is_verified |
| Einstellungen / Consents      | consents.analytics, consents.newsletter                        |
| Gerätedaten (falls vorhanden) | fingerprint_hash, letzte Login-IP (gekürzt)                    |
| Inhalte                       | alle Chatnachrichten **des Nutzers** (eigene Messages)         |
| Metadaten                     | JSON-Schema-Version, Exportzeitstempel                         |

> **Fremde Daten** (Nachrichten deiner Gesprächspartner) gehören **nicht** zwingend ins Export-Zip; gib nur das zurück, was „sich auf die betroffene Person bezieht“. Praktisch übermittelst du meist die **ganzen Konversationen**, weil der Text des Anderen mitgespeichert ist. Das wird allgemein akzeptiert, sofern die Daten ausschließlich dem Anforderer bereitgestellt werden.

---

## 4 Sicherheit & Missbrauchsschutz

|Maßnahme|Warum|
|---|---|
|**Re-auth-Prompt** vor Delete/Export|Verhindert, dass ein entwendetes Handy den Account löscht|
|**Rate-Limit** (z. B. max 1 Export/Tag)|DoS verhindern|
|**Download-URL nur 24 h gültig**|Minimiert Leck-Risiko|
|**2-Man-Rule für Admin-Löschung**|Wenn Ops manuell eingreifen muss|

---

## 5 Privacy-Policy-Abschnitt (Muster)

```md
### 7. Auskunft, Datenportabilität & Löschung
Sie können unter *Einstellungen → Datenschutz* jederzeit
* eine Kopie aller von uns gespeicherten Daten als JSON-Datei herunterladen (Art. 20 DSGVO) und
* die Löschung Ihres Kontos veranlassen (Art. 17 DSGVO).
Wir führen die endgültige Löschung binnen 30 Tagen durch, soweit keine gesetzlichen Aufbewahrungspflichten entgegenstehen.
```

---

### Kurz-Fazit

- **Ja**, ein _Export-_ und ein _Delete-Button_ erfüllen die DSGVO-Benutzerrechte elegant.
    
- Halte das Back-End simpel: Soft-Flag → Worker → Hard-Wipe.
    
- Vergiss nicht die **Re-auth-Sicherung**, eine **Grace-Period** und die **Dokumentation** im RoPA/Verarbeitungsverzeichnis.
    

Wenn du Beispiel-Tests, einen Bash-Cron-Job oder eine fertige Worker-Queue-Implementation (e.g. Laravel + Horizon) brauchst – sag Bescheid!