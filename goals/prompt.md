Lets refocus on what the real mission of this chat really is:
	Making my Dating app GDPR compliant

Here is the overall structure of my Database:
"
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
"

Here is what has been done so far:
- User rights api
	- delete_me endpoint
		- requires auth
		- creates deletion job with cron job deleting it after 30 days
	- request_me
		- user gets this accessable to download via a link which is deleted after a while via cron job:
		[hier noch die antwort rein kopieren]
	- objection
		- user can change the permissions any time and i update it in the server
		- [ ] but im not quite sure if the permissions like location are actually effective in the frontend, (remember this as open task)
	- rectification
		- user can edit their userdata after authentification any time in one screen in the app
	- restriction
		- asks for auth
		- user can restrict the proccessing activity, i set a flag with timestamp and move the user into a seperate collection "db.restricted_users" 
		- verify code file checks both collections and if the user is restricted, frontend will forward to the page where user can unrestrict processing, request their data, request deletion in 30 days
	- All these endpoints still lag
		- [ ] rate limit fehlt noch
		- [ ] nochmal checken ob die download url wirklich nur 24 gültig ist
		- [ ] 2-admin rule für admin löschung noch einführen


diese privacy policy abschnitt nicht vergessen / aber noch angleichen wo es wirklich zu finden ist:
```
### 7. Auskunft, Datenportabilität & Löschung
Sie können unter *Einstellungen → Datenschutz* jederzeit
* eine Kopie aller von uns gespeicherten Daten als JSON-Datei herunterladen (Art. 20 DSGVO) und
* die Löschung Ihres Kontos veranlassen (Art. 17 DSGVO).
Wir führen die endgültige Löschung binnen 30 Tagen durch, soweit keine gesetzlichen Aufbewahrungspflichten entgegenstehen.
```


Was sind jetzt noch alle dokumente die ich brauche?