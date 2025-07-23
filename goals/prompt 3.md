```yaml
# Record of Processing Activities – Woven Dating App
# (GDPR Art.30)

controller:
  name: Woven UG (haftungsbeschränkt)
  address: Goethestraße, 82110 Germering, Germany
  representative: luna@owee.de

processors:
  - name: Hetzner Online GmbH
    service: Managed VPS / hosting
    country: DE
  - name: Amazon SES
    service: Transactional e-mail
    country: EU-West

datasets:
  users:
    collection: users
    data_categories:
      - basic_identifiers
      - contact_details
      - special_categories (religion, politics)
      - profile_preferences
      - credentials
    purpose: provide matchmaking service
    lawful_basis: contract
    retention: until account deletion + 30 days backup
  chats:
    collection: chats
    data_categories:
      - communications_content
      - location
      - voice_data
    purpose: messaging between users
    lawful_basis: contract
    retention: delete 6 months after both users delete chat
  restricted_users:
    collection: restricted_users
    data_categories: same_as_users
    purpose: store account during processing restriction
    lawful_basis: legal_obligation
    retention: until user lifts restriction
  logs:
    location: /var/www/woven/logs
    data_categories: system_logs (IP, token id)
    purpose: security & troubleshooting
    lawful_basis: legitimate_interests
    retention: 14 days rolling
```
1.
|   |
|---|
|**Categories of data subjects**|

|   |
|---|
|`datasets.*.data_subjects`|

|   |
|---|
|e.g. `- app_users`, `- chat_partners`, `- newsletter_subscribers`.|
2.
Already started, but list _all_ special categories separately (religion, political opinion, voice biometrics …).

3.
[find out everything i actually use]

|Mandatory field|Where it goes in YAML|Notes / examples you still need to add|
|---|---|---|
|**Controller & contact**|`controller:`|You already have name / address – add your company Register-No. & telephone.|
|**Data-Protection Officer** (or “Not applicable”)|`dpo:`|Must name or at least give contact e-mail if you appoint one.|
|**Categories of data subjects**|`datasets.*.data_subjects`|e.g. `- app_users`, `- chat_partners`, `- newsletter_subscribers`.|
|**Categories of personal data**|`data_categories:`|Already started, but list _all_ special categories separately (religion, political opinion, voice biometrics …).|
|**Purposes of processing**|`purpose:`|✔ Present. Make sure each purpose is **specific** (“matching algorithm”, “fraud prevention”).|
|**Lawful basis (Art 6 + 9)**|`lawful_basis:`|✔ Present. Include consent **document-ids / mechanism** (e.g. “in-app checkbox v1.2”).|
|**Recipients / categories of recipients**|`recipients:`|e.g. `- Amazon SES (mail)` `- Google FCM (push)` `- Crashlytics (analytics)`|
|**International transfers + safeguards**|`third_country_transfers:`|List “US – SCC 2021/914” for Firebase etc.|
|**Planned erasure deadlines / retention**|`retention:`|Add numbers for _all_ datasets (voice memos, profile images, logs, backups).|
|**General description of TOMs** (Technical & Organisational Measures)|`toms:`|Short bullet list: encryption at rest, 2-factor admin, rate-limit, nightly backups, …|