Okay lets start with task number one.

  

* • Mini-workshop: list every table/collection, field, API, log*:

  

tables:

"

woven> db.chats.find().pretty()

[

  {

    _id: ObjectId('686411d223e0a7fe430c20fd'),

    participants: [ '68640b2343bb3b04de049f83', '6864042223e0a7fe430c2004' ],

    messages: {

      '4d318014add1bbab': {

        sender_id: '68640b2343bb3b04de049f83',

        type: 'game_invite',

        timestamp: '2025-07-01T18:50:39.533802',

        status: 'delivered',

        content: { game: 'tictactoe', game_id: '686411df23e0a7fe430c20fe' }

      },

      '686411e70f93f': {

        sender_id: '68640b2343bb3b04de049f83',

        type: 'text',

        timestamp: '2025-07-01T18:50:47.026322',

        status: 'delivered',

        message: 'gevba'

      },

      '686411eac6f79': {

        sender_id: '68640b2343bb3b04de049f83',

        type: 'voicemessage',

        timestamp: '2025-07-01T18:50:50.552670',

        status: 'delivered',

        audio_url: 'http://v32582.1blu.de/chat/uploads/voicememos/voice_686411eac6efb3.46973572_1751388648165.aac'

      },

      '686411f2055bb': {

        sender_id: '68640b2343bb3b04de049f83',

        type: 'location',

        timestamp: '2025-07-01T18:50:57.950394',

        status: 'delivered',

        content: { lat: 48.123576, lng: 11.5803026 }

      }

    }

  },

  {

    _id: ObjectId('68641f2223e0a7fe430c20ff'),

    participants: [ '68640b2343bb3b04de049f83', '6864042323e0a7fe430c2036' ],

    messages: {}

  },

  {

    _id: ObjectId('68641f2d43bb3b04de049f89'),

    participants: [ '68640b2343bb3b04de049f83', '6864042323e0a7fe430c2031' ],

    messages: {}

  },

  {

    _id: ObjectId('6864272c23e0a7fe430c2100'),

    participants: [ '68640b2343bb3b04de049f83', '6864042323e0a7fe430c202c' ],

    messages: {}

  },

  {

    _id: ObjectId('68642d7023e0a7fe430c2101'),

    participants: [ '68640b2343bb3b04de049f83', '6864042323e0a7fe430c2027' ],

    messages: {}

  },

  {

    _id: ObjectId('6864554d23e0a7fe430c2102'),

    participants: [ '68640b2343bb3b04de049f83', '6864042323e0a7fe430c2018' ],

    messages: {

      ce8f6c82fe08727a: {

        sender_id: '68640b2343bb3b04de049f83',

        type: 'game_invite',

        timestamp: '2025-07-01T23:38:29.692920',

        status: 'delivered',

        content: { game: 'tictactoe', game_id: '6864555343bb3b04de049f8a' }

      }

    }

  },woven> db.restricted_users.findOne({ "userdata.email" : "luna.vogl.35@gmail.com" })

{

  _id: ObjectId('687e6db0fd33ab3ed40abc4e'),

  profile: {

    topSection: {

      firstName: 'luna',

      lastName: 'vogl',

      name: 'luna vogl',

      infoBubbles: {

        gender: 'Female',

        languages: [ 'German' ],

        religion: 'Sikhism',

        religion_show: true,

        politics: 'Conservative'

      },

      age: 0,

      profileImages: [

        'https://v32582.1blu.de/new_signup/uploads/687e6de0523012.42522771_image_cropper_1753116126551.jpg',

        'https://v32582.1blu.de/new_signup/uploads/687e6de0523667.43247463_image_cropper_1753116121374.jpg',

        'https://v32582.1blu.de/new_signup/uploads/687e6de05237d0.56330977_image_cropper_1753116123768.jpg'

      ]

    },

    elements: [

      {

        _id: ObjectId('687e6de3fd33ab3ed40abc4f'),

        type: 'question_and_inputfield',

        content: {

          question: 'Which movie never fails to make you laugh?',

          answer: ''

        }

      },

      {

        _id: ObjectId('687e6de567c6808f860bbb32'),

        type: 'question_and_inputfield',

        content: { question: 'Do you prefer sunrise or sunset?', answer: '' }

      },

      {

        _id: ObjectId('687e6de7fd33ab3ed40abc50'),

        type: 'question_and_inputfield',

        content: { question: "What's one skill you wish you had?", answer: '' }

      },

      {

        _id: ObjectId('687e6deb67c6808f860bbb33'),

        type: 'question_and_inputfield',

        content: { question: 'Who inspires you most?', answer: '' }

      }

    ]

  },

  search_filter: {

    searching_for: {

      genders: [ 'Male', 'Female', 'Non Binary', 'Transgender', 'Genderqueer' ],

      ageRange: [ 18, 93 ],

      religion: [ 'Christianity', 'Islam', 'Hinduism', 'Buddhism', 'Judaism' ],

      politics: [

        'Liberal',

        'Conservative',

        'Centrist',

        'Libertarian',

        'Socialist'

      ]

    },

    location_radius: {

      location: { type: 'Point', coordinates: [ 11.5802829, 48.1235568 ] },

      radius: 79

    }

  },

  userdata: {

    email: 'luna.vogl.35@gmail.com',

    password_hash: '$2y$10$6QEbe1XZ2XaHxqHJUtpgQegY1bzpqcKbbAqtB/94m6YNO93.WA7Eq',

    verification_code: '6214',

    verification_code_expiry: ISODate('2025-07-21T18:57:22.000Z'),

    is_verified: true,

    fingerprint_hash: null,

    device_auth: {

      devices: [ '3292c8db-d7ce-4fd5-8747-411014cdec6a' ],

      enabled: true,

      hardware_ids: [ 'TP1A.220624.014' ],

      method: 'biometricOrPin',

      since: ISODate('2025-07-21T16:41:27.978Z')

    },

    consents: {

      location: { granted: true, timestamp: ISODate('2025-07-21T16:41:46.429Z') },

      microphone: { granted: true, timestamp: ISODate('2025-07-21T16:41:46.429Z') },

      newsletter: { granted: true, timestamp: ISODate('2025-07-21T16:41:46.429Z') },

      notifications: {

        granted: false,

        timestamp: ISODate('2025-07-21T16:41:46.429Z')

      },

      profile_sensitives: { granted: true, timestamp: ISODate('2025-07-21T16:41:46.429Z') }

    }

  },

  chats: {},

  restricted_at: ISODate('2025-07-21T16:47:50.436Z')

}

woven>"

  

woven> db.users.findOne({ "userdata.email" : "luna.vogl.35@gmail.com" })

{

  _id: ObjectId('6880f8c9fd33ab3ed40abc51'),

  profile: {

    topSection: {

      firstName: 'luna',

      lastName: 'vogl',

      name: 'luna vogl',

      infoBubbles: {

        gender: 'Female',

        languages: [ 'German' ],

        religion: 'Sikhism',

        religion_show: true,

        politics: 'Libertarian'

      },

      age: 0,

      profileImages: [

        'https://v32582.1blu.de/new_signup/uploads/6880fae2e42aa8.21630370_image_cropper_1753283279536.jpg',

        'https://v32582.1blu.de/new_signup/uploads/6880fae2e43c33.30868607_image_cropper_1753283287999.jpg',

        'https://v32582.1blu.de/new_signup/uploads/6880fae2e44842.49611433_image_cropper_1753283294924.jpg'

      ]

    },

    elements: [

      {

        _id: ObjectId('6880fae7fd33ab3ed40abc52'),

        type: 'icebreaker',

        content: {

          question: 'If you could teleport anywhere tomorrow, where would you go?',

          answer: 'hshshsh'

        }

      },

      {

        _id: ObjectId('6880faea67c6808f860bbb34'),

        type: 'question_and_inputfield',

        content: { question: 'Do you prefer sunrise or sunset?', answer: '' }

      },

      {

        _id: ObjectId('6880faf0fd33ab3ed40abc54'),

        type: 'voicememo',

        content: {

          prompt: 'Share a childhood tradition you still love.',

          link_voicemessage: 'http://v32582.1blu.de/uploads/voicememos/6880faf0fd33ab3ed40abc53.aac'

        }

      },

      {

        _id: ObjectId('6880faf267c6808f860bbb35'),

        type: 'question_and_inputfield',

        content: { question: 'Do you prefer sunrise or sunset?', answer: '' }

      }

    ]

  },

  search_filter: {

    searching_for: {

      genders: [ 'Male', 'Female', 'Non Binary', 'Transgender', 'Genderqueer' ],

      ageRange: [ 18, 120 ],

      religion: [ 'Christianity', 'Islam', 'Hinduism', 'Buddhism', 'Judaism' ],

      politics: [

        'Liberal',

        'Conservative',

        'Centrist',

        'Libertarian',

        'Socialist'

      ]

    },

    location_radius: {

      location: { type: 'Point', coordinates: [ 11.5803278, 48.1235498 ] },

      radius: 79

    }

  },

  userdata: {

    email: 'luna.vogl.35@gmail.com',

    password_hash: '$2y$10$xfbfPrz/uahJAVWhNGbAe./xyiBUUnivM6vAXnuWlypB0d4oh96ae',

    verification_code: null,

    verification_code_expiry: null,

    is_verified: true,

    fingerprint_hash: null,

    device_auth: {

      devices: [ '3292c8db-d7ce-4fd5-8747-411014cdec6a' ],

      enabled: true,

      hardware_ids: [ 'TP1A.220624.014' ],

      method: 'biometricOrPin',

      since: ISODate('2025-07-23T14:59:34.165Z')

    },

    consents: {

      location: { granted: true, timestamp: ISODate('2025-07-23T15:07:31.153Z') },

      microphone: { granted: true, timestamp: ISODate('2025-07-23T15:07:31.153Z') },

      newsletter: { granted: true, timestamp: ISODate('2025-07-23T15:07:31.153Z') },

      notifications: {

        granted: false,

        timestamp: ISODate('2025-07-23T15:07:31.153Z')

      },

      profile_sensitives: { granted: true, timestamp: ISODate('2025-07-23T15:07:31.153Z') }

    }

  },

  chats: {}

}

woven>"

  

My api endpoints:
"
wov
root@v32582:/var/www/woven/public
|-- agb.html
|-- chat
|   |-- create_chat.php
|   |-- fetchChat.php
|   |-- get_profile_from_chatid.php
|   |-- get_profile_id.php
|   |-- myChats.php
|   |-- my_chats_log.log
|   |-- search_user.php
|   |-- sendInviteMessage.php
|   |-- sendMessage.php
|   |-- sendReaction.php
|   |-- sendVoiceMessage.php
|   `-- uploads
|       `-- voicememos
|-- config.php -> ../config.php
|-- create_message.php
|-- create_profile
|   |-- add_answer.php
|   |-- add_languages.php
|   |-- create_testusers.php
|   |-- create_testusers_structured.php
|   |-- elements_update.php
|   |-- get_elements.php
|   |-- get_profile_backup.php
|   |-- get_userdata.php
|   |-- load_profile.php
|   |-- migrate_profiles.php
|   |-- retrieve_profile.php
|   |-- saveProfile_backup.php
|   |-- update_captionpic.php
|   |-- update_icebreaker.php
|   |-- update_loc_backup.php
|   |-- update_open_to.php
|   |-- update_open_to_backup.php
|   |-- update_question.php
|   |-- update_searching_for_backup.php
|   |-- update_topic_suggestion.php
|   |-- update_userdata_backup.php
|   |-- update_userdate_backup.php
|   |-- update_voicememo.php
|   `-- uploads
|-- dsgvo
|   |-- consent.php
|   |-- delete_me.php
|   |-- get_consents.php
|   |-- method.php
|   |-- request_me.php
|   |-- restore_me.php
|   |-- restrict_me.php
|   |-- scripts
|   |-- status.php
|   |-- unrestrict_me.php
|   |-- verify.php
|   `-- verify_password.php
|-- exports
|-- fahrschul_api
|   `-- generate_route.php
|-- games
|   |-- createGame.php
|   |-- invite.php
|   `-- joinGame.php
|-- impressum.html
|-- login_signup
|   |-- addFingerprint.php
|   |-- fingerprintLogin.php
|   |-- login.php
|   |-- search_users.php
|   |-- signup.php
|   |-- test_jwt.php
|   |-- verify_code.php
|   `-- verifycode_login.php
|-- logs
|-- match_algorithm
|   |-- block_user.php
|   |-- create_matches.php
|   |-- curl_activities_area.php
|   |-- generate.php
|   |-- match.php
|   |-- matchBackup.php
|   |-- my_custom_errors.log
|   |-- react_profile.php
|   |-- sendReaction.php
|   |-- submit_questionaire.php
|   |-- swipe.php
|   |-- swipe_backup.php
|   `-- swipe_errors.log
|-- new_signup
|   |-- add_icebreaker.php
|   |-- add_languages.php
|   |-- add_politics.php
|   |-- add_voicememo.php
|   |-- age_set.php
|   |-- choose_gender.php
|   |-- delete_element.php
|   |-- device_auth.php
|   |-- get_profile.php
|   |-- login_deviceauth.php
|   |-- login_email.php
|   |-- name_lastname.php
|   |-- saveProfile.php
|   |-- signup_email.php
|   |-- update_convostarter.php
|   |-- update_loc.php
|   |-- update_religion.php
|   |-- update_searching_for.php
|   |-- update_userdata.php
|   |-- upload_profile_images.php
|   |-- uploads
|   |   |-- 
|   |-- verify_code.php
|   `-- verify_code_login.php
|-- profile
|   |-- create_example_users.php
|   |-- retrieve_profile.php
|   |-- save_profile.php
|   |-- suggest_5_profiles.php
|   |-- suggest_profiles.php
|   |-- upload_image.php
|   `-- uploads
|-- read_message.php
|-- scripts
|-- test.php
`-- uploads
    `-- voicememos

21 directories, 254 files
root@v32582:/var/www/woven#

root@v32582:/var/www/woven/logs# tree
.
|-- add_icebreaker.log
|-- add_voicememo.log
|-- cleanup.log
|-- cleanup_exports.log
|-- consent.log
|-- convostarter.log
|-- create_message.log
|-- delete_element.log
|-- delete_me.log
|-- device_auth.log
|-- generate_route.log
|-- get_consents.log
|-- get_elements.log
|-- login_deviceauth.log
|-- login_email.log
|-- match.log
|-- method.log
|-- request_me.log
|-- request_me_php_errors.log
|-- restrict_me.log
|-- status.log
|-- unrestrict_me.log
|-- update_userdata.log
|-- upload_images.log
|-- verify.log
`-- verify_code_login.log
"

## answer

Below is a concise GDPR-starter kit focused **only on #1 “Data-inventory & Lawful-basis”**.  
I’ve split it into three parts:

|Part|What you get|Where to put it|
|---|---|---|
|A|**Mapping table** – Mongo collections ⇢ personal-data categories ⇢ lawful basis|Keep in your internal compliance wiki or `/docs/ropa/`|
|B|**RoPA skeleton (YAML)** – ready to fill & version-control|`/docs/ropa/woven_ropa.yaml`|
|C|**next-info we’ll still need** to finalise everything|——|

---

## A. Mapping table (collections / fields → data category → Art 6 basis)

| #   | Collection / resource                                                                | Example fields                                       | GDPR data-type                          | Purpose                                                                                      | Lawful basis (Art 6) |
| --- | ------------------------------------------------------------------------------------ | ---------------------------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------- |
| 1   | `users.profile.topSection.*` • `firstName`, `lastName`, `age`, `gender`, `languages` | Standard PII                                         | Provide dating profile                  | **Contract** 6 (1)(b)                                                                        |                      |
| 2   | `users.profile.topSection.infoBubbles.religion`, `politics`                          | **Special category** Art 9(1)                        | Optional self-disclosure to matches     | **Explicit consent** 6 (1)(a) + 9 (2)(a)                                                     |                      |
| 3   | `users.profile.profileImages` (URLs ⇒ stored on own server)                          | Biometric/facial image                               | Show pictures                           | Consent 6 (1)(a)                                                                             |                      |
| 4   | `users.search_filter.*`                                                              | Preferences (age range, religion filter, etc.)       | Personal data (not special cat.)        | Matching algorithm                                                                           | Contract 6 (1)(b)    |
| 5   | `users.userdata.email`, `password_hash`                                              | Identifiers & credentials                            | Account auth, security                  | Contract 6 (1)(b) & Legitimate interests 6 (1)(f) (security)                                 |                      |
| 6   | `users.userdata.consents.*`                                                          | Boolean + timestamp                                  | Proof of consent                        | Legal obligation 6 (1)(c)                                                                    |                      |
| 7   | `users.userdata.device_auth.*`                                                       | Device IDs / biometrics flag                         | Strong auth                             | Legitimate interests 6 (1)(f)                                                                |                      |
| 8   | `chats.*.messages` (text, voice, location, game_invite)                              | Communication content – may include location & voice | Messaging service                       | Contract 6 (1)(b) Special cat. (voice) ⇒ explicit consent 9 (2)(a) via microphone permission |                      |
| 9   | `restricted_users` (mirror of `users` + `restricted_at`)                             | Same as above but processing paused                  | Demonstrate compliance with restriction | Legal obligation 6 (1)(c)                                                                    |                      |
| 10  | Log files in `/var/www/woven/logs/*.log`                                             | IP + timestamps (in Apache / PHP error logs)         | Security & audit                        | Legitimate interests 6 (1)(f)                                                                |                      |

**Why these files are “associated”**  
_Any PHP endpoint that reads/writes those collections or log files is automatically in-scope for the inventory._  
Examples:

- `public/new_signup/*` – creates & updates collection **users**
    
- `public/dsgvo/*` – uses both **users** and **restricted_users**
    
- `public/chat/*` – reads/writes **chats**
    
- Logs in `/logs/*` – all endpoints append here
    

---

## B. RoPA skeleton (drop-in YAML)

Save this as `/docs/ropa/woven_ropa.yaml` in your repo and keep it updated whenever the DB or purposes change.

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

_(Feel free to extend with more endpoints, analytics vendors, etc.)_

---

## C. What’s **still missing** before calling #1 “done”

|Still needed|Why|Where to get / action|
|---|---|---|
|**Full list of external sub-processors** (e.g. push-notification provider, crashlytics)|Art 28 & RoPA|Ask dev-ops / billing records|
|**Exact data retention intervals** for _each_ data category (profile images, voice memos, logs, backups)|Required in RoPA + privacy notice|Decide with product & legal|
|**Copy of standard contractual clauses** if you use any non-EU cloud|Accountability|Keep PDF in `/docs/dpa/`|
|**Link to privacy notice** in every mobile client|Transparency|Already have `impressum.html` – add _Rights_ section|
|**Data Protection Impact Assessment (DPIA)** (if large-scale special-cat. data)|Maybe mandatory|Legal team|

---

### What to do next

1. **Create the YAML file** shown above and commit:
    
    ```bash
    mkdir -p docs/ropa
    nano docs/ropa/woven_ropa.yaml   # paste content
    git add docs/ropa/woven_ropa.yaml
    git commit -m "Add initial RoPA skeleton"
    git push origin master
    ```
    
2. **Fill the blanks**: processors, exact retention, SCC status.
    
3. **Update privacy-notice** to reference the lawful bases table and user rights.
    
4. Ping me when you’re ready for the next checklist item (rate-limiting, 2-man-rule, etc.) or if you need help filling the missing info.
    

_(If you’d rather I expand any section—e.g. produce a ready-to-sign processor contract template—just let me know.)_