We now have this flag set like this:

"

woven> db.users.findOne({ "userdata.email" : "luna.vogl.35@gmail.com" })

{

_id: ObjectId('68781ac9fd33ab3ed40abc2c'),

profile: {

topSection: {

firstName: 'luna',

lastName: 'vogl',

name: 'luna vogl',

infoBubbles: {

gender: 'Female',

languages: [ 'German' ],

religion: 'Spiritual',

religion_show: true,

politics: 'Socialist',

career: 'programming'

},

age: 0,

profileImages: [

'https://v32582.1blu.de/new_signup/uploads/68781b36996f83.55116409_image_cropper_1752701739630.jpg',

'https://v32582.1blu.de/new_signup/uploads/68781b36997a82.86771031_image_cropper_1752701745405.jpg',

'https://v32582.1blu.de/new_signup/uploads/68781b36997ba7.11746985_image_cropper_1752701750073.jpg'

]

},

elements: [

{

_id: ObjectId('68781b5367c6808f860bbb10'),

type: 'voicememo',

content: {

prompt: 'Tell me about a book that changed your life.',

link_voicemessage: 'http://v32582.1blu.de/uploads/voicememos/68781b5367c6808f860bbb0f.aac'

}

},

{

_id: ObjectId('68781b59fd33ab3ed40abc2d'),

type: 'question_and_inputfield',

content: {

question: 'Which movie never fails to make you laugh?',

answer: ''

}

},

{

_id: ObjectId('68781b5e67c6808f860bbb11'),

type: 'icebreaker',

content: {

question: 'What’s your go-to comfort food on a bad day?',

answer: 'gwhwbe'

}

},

{

_id: ObjectId('68781b61fd33ab3ed40abc2e'),

type: 'question_and_inputfield',

content: {

question: "What's the best meal you've ever cooked?",

answer: ''

}

}

]

},

search_filter: {

searching_for: {

genders: [ 'Female', 'Transgender', 'Non Binary', 'Genderqueer' ],

ageRange: [ 18, 38 ],

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

location: { type: 'Point', coordinates: [ 11.5635216, 48.1309089 ] },

radius: 60

}

},

userdata: {

email: 'luna.vogl.35@gmail.com',

password_hash: '$2y$10$6bigqAADCPi9Gv9gkaVWheL1VvtxKp4ZCLhJIVS8nduh0bG2diWTC',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: null,

device_auth: {

devices: [ 'df7b5b3f-edc7-412a-b159-3dce90c46c95' ],

enabled: true,

hardware_ids: [ 'TP1A.220624.014' ],

method: 'biometricOrPin',

since: ISODate('2025-07-16T21:34:21.386Z'),

last_login: ISODate('2025-07-17T17:22:46.470Z')

},

consents: {

location: { granted: true, timestamp: ISODate('2025-07-17T16:15:33.358Z') },

microphone: { granted: true, timestamp: ISODate('2025-07-17T16:15:33.358Z') },

newsletter: { granted: true, timestamp: ISODate('2025-07-17T16:15:33.358Z') },

notifications: {

granted: false,

timestamp: ISODate('2025-07-17T16:15:33.358Z')

},

profile_sensitives: { granted: true, timestamp: ISODate('2025-07-17T16:15:33.358Z') }

}

},

chats: {},

processing_restricted_at: ISODate('2025-07-17T17:28:21.083Z')

}"

Now, we need to actually make the other scripts check this flag.

Lets start by making the match.php not return such profiles