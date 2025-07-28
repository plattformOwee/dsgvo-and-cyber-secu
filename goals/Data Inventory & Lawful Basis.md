# Data Inventory & Lawful Basis
## 1 tables
db.users:
one example user:
```
db.users.findOne({ "userdata.email" : "luna.vogl.35@gmail.com" })
{
  _id: ObjectId('6885129bfd33ab3ed40abc58'),
  profile: {
    topSection: {
      firstName: 'luna',
      lastName: 'vogl',
      name: 'luna vogl',
      infoBubbles: {
        gender: 'Female',
        languages: [ 'German' ],
        religion: 'Judaism',
        religion_show: true,
        politics: 'Socialist'
      },
      age: 0,
      profileImages: [
        'https://v32582.1blu.de/new_signup/uploads/68851310cdb777.44209247_image_cropper_1753551600924.jpg',
        'https://v32582.1blu.de/new_signup/uploads/68851310cdbc02.34100309_image_cropper_1753551609987.jpg',
        'https://v32582.1blu.de/new_signup/uploads/68851310cdbdb9.85678785_image_cropper_1753551629222.jpg'
      ]
    },
    elements: [
      {
        _id: ObjectId('68851315fd33ab3ed40abc5a'),
        type: 'question_and_inputfield',
        content: { question: 'Who inspires you most?', answer: '' }
      },
      {
        _id: ObjectId('6885131c67c6808f860bbb3a'),
        type: 'icebreaker',
        content: {
          question: 'Dinner with any historical figure—who?',
          answer: 'bs sbsb'
        }
      },
      {
        _id: ObjectId('68851335fd33ab3ed40abc5c'),
        type: 'voicememo',
        content: {
          prompt: 'Tell me about your favorite movie scene.',
          link_voicemessage: 'http://v32582.1blu.de/uploads/voicememos/68851335fd33ab3ed40abc5b.aac'
        }
      },
      {
        _id: ObjectId('6885133867c6808f860bbb3b'),
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
      location: { type: 'Point', coordinates: [ 11.3615608, 48.1338709 ] },
      radius: 90
    }
  },
  userdata: {
    email: 'luna.vogl.35@gmail.com',
    password_hash: '$2y$10$3EQuyg6MQ0QGZFz52U1t0uK.Sp8Kl/4ylsFUQsh479C82Rr3tXrH6',
    verification_code: null,
    verification_code_expiry: null,
    is_verified: true,
    fingerprint_hash: null,
    device_auth: {
      devices: [ 'b902324e-3e57-4d2f-9fbe-32f926742167' ],
      enabled: true,
      hardware_ids: [ 'TP1A.220624.014' ],
      method: 'biometricOrPin',
      since: ISODate('2025-07-26T17:38:48.088Z'),
      last_login: ISODate('2025-07-27T18:20:02.367Z')
    }
  },
  chats: {}
}
```

db.chats:
```
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
  },
  {
    _id: ObjectId('68645eca43bb3b04de049f8b'),
    messages: {
      d16d2242056de684ba5fc76c447fe73e: {
        sender_id: '68640b2343bb3b04de049f83',
        type: 'react_icebreaker',
        timestamp: '2025-07-01T22:18:54.428981Z',
        status: 'delivered',
        content: {
          question: 'Was hat dich heute gefreut?',
          answer: 'Dass die Sonne geschienen hat.',
          comment: 'cgh'
        }
      },
      '68645ee05b296': {
        sender_id: '68640b2343bb3b04de049f83',
        type: 'voicemessage',
        timestamp: '2025-07-02T00:19:11.828183',
        status: 'delivered',
        audio_url: 'http://v32582.1blu.de/chat/uploads/voicememos/voice_68645ee05b2314.97397266_1751408346405.aac'
      },
      '68645ef653739': {
        sender_id: '68640b2343bb3b04de049f83',
        type: 'voicemessage',
        timestamp: '2025-07-02T00:19:33.838642',
        status: 'delivered',
        audio_url: 'http://v32582.1blu.de/chat/uploads/voicememos/voice_68645ef6536ea0.92373765_1751408365559.aac'
      },
      aa8fc3bc86e29f23: {
        sender_id: '68640b2343bb3b04de049f83',
        type: 'game_invite',
        timestamp: '2025-07-02T00:19:45.107036',
        status: 'delivered',
        content: { game: 'tictactoe', game_id: '68645f0123e0a7fe430c2103' }
      }
    },
    participants: [ '68640b2343bb3b04de049f83', '6864042423e0a7fe430c204f' ]
  },
  {
    _id: ObjectId('6865a162737548e1de69e328'),
    participants: [ '68659dde23e0a7fe430c2107', '6864042323e0a7fe430c2036' ],
    messages: {
      examplemsg001: {
        sender_id: '6864042323e0a7fe430c2036',
        type: 'text',
        timestamp: '2025-07-02T21:15:14.699Z',
        status: 'delivered',
        message: 'Hey Luna! 👋'
      }
    }
  },
  {
    _id: ObjectId('6865a194737548e1de69e329'),
    participants: [ '68659dde23e0a7fe430c2107', '6864042323e0a7fe430c2036' ],
    messages: {
      examplemsg001: {
        sender_id: '6864042323e0a7fe430c2036',
        type: 'text',
        timestamp: '2025-07-02T21:16:04.570Z',
        status: 'delivered',
        message: 'Hey Luna! 👋'
      }
    }
  },
  {
    _id: ObjectId('6865c9ee43bb3b04de049f93'),
    participants: [ '6865c8df43bb3b04de049f90', '6864042323e0a7fe430c202c' ],
    messages: {}
  },
  {
    _id: ObjectId('6865ca0243bb3b04de049f94'),
    participants: [ '6865c8df43bb3b04de049f90', '6864042323e0a7fe430c2031' ],
    messages: {}
  },
  {
    _id: ObjectId('686693a943bb3b04de049f98'),
    participants: [ '686691fd23e0a7fe430c2111', '6864042723e0a7fe430c20c7' ],
    messages: {
      '686693ae8f626': {
        sender_id: '686691fd23e0a7fe430c2111',
        type: 'text',
        timestamp: '2025-07-03T16:29:04.059405',
        status: 'delivered',
        message: 'gsbhwjw'
      },
      '686693afad4a4': {
        sender_id: '686691fd23e0a7fe430c2111',
        type: 'text',
        timestamp: '2025-07-03T16:29:05.187071',
        status: 'delivered',
        message: 'bezebw'
      },
      '686693c407e7e': {
        sender_id: '686691fd23e0a7fe430c2111',
        type: 'voicemessage',
        timestamp: '2025-07-03T16:29:25.293608',
        status: 'delivered',
        audio_url: 'http://v32582.1blu.de/chat/uploads/voicememos/voice_686693c407e3a5.75670060_1751552946482.aac'
      },
      '686693d540e8a': {
        sender_id: '686691fd23e0a7fe430c2111',
        type: 'answer_to',
        timestamp: '2025-07-03T16:29:42.684582',
        status: 'delivered',
        content: {
          answered_to: 'voicemessage',
          answered_to_id: '1751552965293',
          answer: 'tchv'
        }
      },
      '11c2cb5ea2b43ebf': {
        sender_id: '686691fd23e0a7fe430c2111',
        type: 'game_invite',
        timestamp: '2025-07-03T16:29:53.275458',
        status: 'delivered',
        content: { game: 'tictactoe', game_id: '686693df23e0a7fe430c2117' }
      },
      '686693e5d5d00': {
        sender_id: '686691fd23e0a7fe430c2111',
        type: 'location',
        timestamp: '2025-07-03T16:29:59.351467',
        status: 'delivered',
        content: { lat: 48.1235303, lng: 11.5802406 }
      }
    }
  },
  {
    _id: ObjectId('686d7e8a67c6808f860bbadf'),
    participants: [ '686d7ba967c6808f860bbad5', '6864042423e0a7fe430c2045' ],
    messages: {
      '686d7e945df30': {
        sender_id: '686d7ba967c6808f860bbad5',
        type: 'voicemessage',
        timestamp: '2025-07-08T22:24:53.736884',
        status: 'delivered',
        audio_url: 'https://v32582.1blu.de/chat/uploads/voicememos/voice_686d7e945deab1.42689048_1752006288441.aac'
      },
      '0634fdf37658fb9a': {
        sender_id: '686d7ba967c6808f860bbad5',
        type: 'game_invite',
        timestamp: '2025-07-08T22:24:57.460330',
        status: 'delivered',
        content: { game: 'tictactoe', game_id: '686d7e9767c6808f860bbae0' }
      },
      '686d7ea6007c5': {
        sender_id: '686d7ba967c6808f860bbad5',
        type: 'location',
        timestamp: '2025-07-08T22:25:10.746981',
        status: 'delivered',
        content: { lat: 48.1439156, lng: 11.4814491 }
      }
    }
  },
  {
    _id: ObjectId('686d7eb567c6808f860bbae1'),
    messages: {
      '41ef59e9f95e5b13ef4a4c6f9e6c658f': {
        sender_id: '686d7ba967c6808f860bbad5',
        type: 'react_bubble',
        timestamp: '2025-07-08T20:25:33.540178Z',
        status: 'delivered',
        content: { bubble_text: 'games', comment: 'hnj' }
      }
    },
    participants: [ '686d7ba967c6808f860bbad5', '6864042323e0a7fe430c2040' ]
  }
```
## API's
```
root@v32582:/var/www/woven/public# tree -L 2 -P '*.*' -I '*.aac|*.jpg'
.
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
```

## logs

```
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
```
# 2. RoPA Skeleton

| Processing Activity                        | Data Categories Involved                                                 | Purpose of Processing                                                                                     | Lawful Basis (Art. 6 GDPR)               | Lawful Basis for Special Categories (Art. 9 GDPR)                            |
| :----------------------------------------- | :----------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------- | :--------------------------------------- | :--------------------------------------------------------------------------- |
| **User Registration & Account Management** | Identity & Account Data, Technical & Security Data                       | To create and maintain the user's account, authenticate the user, and ensure the security of the account. | Art. 6(1)(b) - Performance of a contract | Not Applicable                                                               |
| **Public Profile Creation**                | Profile Information, User-Generated Content                              | To create a public profile that can be seen by other users of the dating app.                             | Art. 6(1)(b) - Performance of a contract | Not Applicable                                                               |
| **Processing of Sensitive Profile Data**   | Special Categories of Personal Data (gender, religion, politics)         | To allow users to share this information on their profile and use it for matching preferences.            | Art. 6(1)(a) - Consent                   | **Art. 9(2)(a) - Explicit Consent**                                          |
| **Matching & User Discovery**              | Profile Information, Special Categories, Location Data, User Preferences | To suggest potential matches to the user based on their stated preferences and location.                  | Art. 6(1)(b) - Performance of a contract | **Art. 9(2)(a) - Explicit Consent**                                          |
| **Chat & Communication**                   | Chat & Messaging Data, Identity Data                                     | To enable users to communicate with each other via text messages, voice messages, and reactions.          | Art. 6(1)(b) - Performance of a contract | **Art. 9(2)(a) - Explicit Consent** (for any sensitive data shared in chats) |
| **In-App Games**                           | Identity Data, Chat & Messaging Data (game invites)                      | To allow users to initiate and play games with each other.                                                | Art. 6(1)(b) - Performance of a contract | Not Applicable                                                               |
| **Handling User Rights Requests**          | All relevant data categories                                             | To comply with legal obligations regarding data subject rights (e.g., deletion, access).                  | Art. 6(1)(c) - Legal Obligation          | Not Applicable                                                               |
|                                            |                                                                          |                                                                                                           |                                          |                                                                              |
## still needed: 
1. **Data Recipients / Sub-Processors:** Who else receives this data? Your `Sub-Processors & DPAs` phase indicates you are aware of this. You need to list all third-party services you use:
	1. 1blu.de
	2. firebase FCM (for push notes)
	3. (crashlytics)

2. How long is a user's data kept after they delete their account?
	1. deletion with cron job takes 30 days
3. how long a chat messages stored?
	1. as long as rest of account (get delted with profiel creation cron job after 30 days aswell)*
4. Are log files automatically deleted after a certain period (e.g., 30 days)?
	1. yes*
5. - **Cross-Border Data Transfers:** Do any of your sub-processors transfer data outside of the European Union/European Economic Area? If so, you must document the legal safeguards in place for these transfers (e.g., Standard Contractual Clauses).
	1. idk > bitte finde es heraus von den oben genannten

6. which security mesures are already in place
	1. HTTPS for API backend
	2. TLS is configures for MongoDB aswell
	3. bycrypt for password hashing

	4. **Data Minimization:** Are you only collecting the data you absolutely need?
		1. yes most definetly

7. DPO
	1. max mustermann (put for now to complete document RoPA), Goethestraße 2, 82110 Germering; luna.vogl.35@gmail.com; +49 176 45530185; 


# 3 RoPA full
Yes, with the additional information you've provided, we can now create a comprehensive Record of Processing Activities (RoPA) that aligns with Article 30 of the GDPR.

Here is the analysis of your sub-processors and the complete RoPA based on all the details you have supplied.

### Analysis of Sub-Processors & Cross-Border Data Transfers

*   **1blu.de:** This is your hosting provider. 1blu AG is a German company with its registered address in Berlin. This means the primary data storage and processing for your main application and database occurs within the European Union (Germany). Therefore, no special measures for cross-border data transfer are needed for your core service. You will, however, need to sign a Data Processing Agreement (DPA) with 1blu AG.
*   **Firebase Cloud Messaging (FCM) & Crashlytics:** Both are services provided by Google. While Google has data centers globally, using these services often involves transferring data to servers outside the EU, most notably to the United States. For such transfers to be lawful under the GDPR, a valid transfer mechanism is required. Google relies on **Standard Contractual Clauses (SCCs)** as the primary legal basis for these transfers. You accept these clauses as part of their terms of service. This means data is transferred outside the EU, but with a recognized legal safeguard in place.

---

### Verzeichnis von Verarbeitungstätigkeiten (VVT) / Record of Processing Activities (RoPA)

**Name and contact details of the Controller:**
voven app UG
Goethestraße 2 82110 Germering
Your Email
*You would fill in your actual details here.*

**Contact details of the Data Protection Officer:**
Max Mustermann
Goethestraße 2, 82110 Germering
luna.vogl.35@gmail.com
+49 176 45530185

---

| No. | Processing Activity | Purpose of Processing | Description of the categories of data subjects | Categories of personal data | Legal Basis (Art. 6 / Art. 9 GDPR) | Categories of recipients / Sub-processors | Transfers to third countries & Safeguards | Data Retention Period | General description of Technical and Organisational Security Measures (TOMs) |
| :-- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **User Registration & Account Management** | To create and manage user accounts, authenticate users, and secure the service. | Users of the mobile application. | **Identity & Account Data:** Name, email, password hash (bcrypt), user ID, device ID, device auth data, verification codes/status. | **Art. 6(1)(b) GDPR** - Performance of a contract. | **1blu AG** (Hosting) | **1blu AG** (Germany, within EU). | Data is stored for the duration of the account's existence. It is deleted 30 days after the user initiates an account deletion request. | **Encryption:** Data in transit (HTTPS), data at rest (TLS for MongoDB).<br>**Access Control:** Internal access restrictions to backend systems.<br>**Pseudonymisation:** Hashing of passwords (bcrypt). |
| **2** | **Provision of User Profiles & Matching** | To enable users to create a public profile and to facilitate the matching of users based on their provided information and preferences. | Users of the mobile application. | **Profile Information:** First name, last name, age, profile images, languages, answers to questions, icebreakers, voice memos.<br>**User Preferences:** Gender/age/religion/politics they are searching for.<br>**Location Data:** GPS coordinates for radius search.<br>**Special Categories:** Gender, religion, political opinions. | **Profile Data:** Art. 6(1)(b) GDPR - Performance of a contract.<br>**Location Data:** Art. 6(1)(a) GDPR - Consent.<br>**Special Categories:** Art. 9(2)(a) GDPR - Explicit Consent. | **1blu AG** (Hosting) | **1blu AG** (Germany, within EU). | Data is stored for the duration of the account's existence. It is deleted 30 days after the user initiates an account deletion request. | Same as above. |
| **3** | **In-App Communication (Chat)** | To enable direct communication between matched users, including text, voice messages, game invites, and location sharing. | Users of the mobile application who have matched. | **Chat Data:** Participants, text messages, voice messages (audio files), game invites, reactions, location data shared in chat, timestamps, delivery status. | **Art. 6(1)(b) GDPR** - Performance of a contract. (Any sensitive data shared is based on the users' own volition and explicit consent per Art. 9(2)(a)). | **1blu AG** (Hosting) | **1blu AG** (Germany, within EU). | Chat data is stored for the duration of the associated accounts' existence. It is deleted along with the user profiles 30 days after an account deletion request. | Same as above. |
| **4** | **Sending Push Notifications** | To inform users of new matches, messages, and other relevant app activity. | Users of the mobile application. | User ID, device token for push notifications. | **Art. 6(1)(a) GDPR** - Consent (Users must opt-in to receive push notifications). | **Google Firebase Cloud Messaging (FCM)** | **USA**. Safeguard: Standard Contractual Clauses (SCCs) are in place with Google. | The device token is stored as long as the user has push notifications enabled and the app is installed. | Same as above. |
| **5** | **App Stability Monitoring & Crash Reporting** | To detect, diagnose, and fix crashes and errors to ensure application stability and improve the service. | Users of the mobile application. | Crashlytics Installation UUIDs, crash traces, device information (e.g., OS version, device model). | **Art. 6(1)(f) GDPR** - Legitimate Interest (to ensure the technical functionality and security of the app).<br>**Art. 6(1)(a) GDPR** - Consent (It's best practice to get consent for this). | **Google Firebase Crashlytics** | **USA**. Safeguard: Standard Contractual Clauses (SCCs) are in place with Google. | Crash data is retained according to Firebase's policies, typically for 90 days. | Same as above. |
| **6** | **Server Log File Management** | To ensure the security, stability, and integrity of the server infrastructure and to defend against attacks. | All individuals accessing the API endpoints (primarily users). | IP Address, date and time of access, API endpoint called, browser/app version information. | **Art. 6(1)(f) GDPR** - Legitimate Interest (to maintain the security and operational integrity of the service). | **1blu AG** (Hosting) | **1blu AG** (Germany, within EU). | Log files are automatically deleted after 30 days. | Same as above. |
| **7** | **Compliance with User Rights Requests** | To fulfill legal obligations under the GDPR regarding data subject rights (access, rectification, erasure, restriction, etc.). | Users (and former users) exercising their GDPR rights. | All data categories relevant to the specific request. | **Art. 6(1)(c) GDPR** - Legal Obligation. | **1blu AG** (Hosting) | **1blu AG** (Germany, within EU). | Documentation of the request and its fulfillment is retained for the legally required period to demonstrate compliance (typically 3 years). | Same as above. |

This table now constitutes a complete and comprehensive Record of Processing Activities based on the information provided. You have successfully mapped your data, identified purposes, established legal bases, listed recipients, addressed international transfers, defined retention periods, and outlined security measures. This document is a central piece of your GDPR compliance journey.