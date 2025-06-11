
- [x] lets find out what match.php accpects as input json and what it gives as output when it works
	- [x] revert to old version on github / from telegram
	- [x] revert to old version in server from chat
- [x] let viewprofile print what json it gets to show profile
- [ ] lets refactor profile creation with one json as goal and debug each step such that this json is met and such that frontend can remain the same
	- [ ] lets just nest the previous "profile" into structured profile as "hidden" and then put whats doubble ony into structured profile. Elements key becomes elements "type" and match algortithm searches by "type" instead of key and everything should be easily refactorable.
	- [ ] write the perfect profile structure
		- [ ] 
- [x] rewrite the matching algorithm to work with the new structure 


- [x] find out what structure each fronend wants (start doc) by printing it
- [x] This is the end goal profile structure (model after what frontend wants + hidden section with "type" for everything)
- [x] write a mapping for match.php for each variable that needs defining, write in a list the exact path on server in new profile structure.
- [x] make new fetchProfile.php get data from this new strcuture aswell
- [x] make match.php work with new structure
	- [x] genereate other users with new strcutrure
	- [x] change match.php
- [ ] make scripx save jsony into db.path (for all scripts)
	- [ ] AboutYouLayout > add_answer.php > 
		    name > profile.topSection.name
		    age > profile.topSection.name
		    gender > profile.topSection.infoBubbles.gender
		    spokenLanguage > profile.topSection.infoBubbles.languages
		    religion > profile.topSection.infoBubbles.religion
		    politics > profile.topSection.infoBubbles.politics
		    career >  profile.topSection.infoBubbles.career
	- [ ] ProfilbilderLayout
- [x] rewrite get_profile.php to work with the new structure 
- [x] match.php should get each value by "type" and map onto expected structure and then respond as usual.


#### prompt structure

This is the end goal profile structure (model after what frontend wants + hidden section with):

===goal_profile_json structure:===
"id":ObjectId('681ccfa0ed13220a4c0b6243'),
"profile":{
	"topSection": {
	    "profileImages": ["http:\/\/panel.krasserserver.com:8002\/swipe_chatt_play_api\/create_profile\/uploads\/681cb62220a04_image_cropper_1746712091332.jpg"],
	    "name": "luna2",
	    "age": 23,
	    "infoBubbles": {
			"gender":"Genderqueer"
			"languages":[language1, language2],
			"religion":"judaism",
			"politics":"socialist",
		}
	},
	"elements": {
	    ObjectId('681ccfa0ed13220a4c0b6243'): {
	        "type": "icebreaker",
	        "content": {
	            "question": "Ein Ort, an den ich immer wieder zur\u00fcckkehre...",
	            "answer": "susi"
	        }
	    },
	    ObjectId('681ccfa0ed13220a4c0b6243'): {
	        "type": "bubbles",
	        "content": {
	            "title": "Open to",
	            "bubbles": [
	                "biking",
	                "puzzles",
	                "games"
	            ]
	        }
	    },
	    ObjectId('681ccfa0ed13220a4c0b6243'):{
		    "type":"question and inputfield",
		    "content":{
				      "question":"was hat dich heute gefreut?",
				      "answer":"what user typed into inputfield"
			      }
	    },
	    ObjectId('681ccfa0ed13220a4c0b6243'):{
		    "type":"voicememo",
			"content":{
			      "prompt":"the prompt", // can be null
			      "link_voicemessage":"http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f877edb34a88.05803339_1744336870367.aac"  // the link generated after saving the thing i will provide test link for now
			  }
	    },
	    ObjectId('681ccfa0ed13220a4c0b6243'):{
		    "type":"topic_suggestions",
			"content":{
			      "prompt":"the prompt" // example: "ask me about..."
				  "choice":"the chosen topic" // example: "musik"
				  "answer":"the answer from the swiper" // example: "welche interpreten magst du?"
			  }
	    },
	    ObjectId('681ccfa0ed13220a4c0b6243'):{
		    "type":"image_and_prompt"
			"content":{
				      "prompt":"the prompt", // can be null and then just image is shown
				      "image_linke":"http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg", // will come from menu later but now use this 
			      }
	    },
	}
},
"search_filter":{
	location_radius: {
        location: { type: 'Point', coordinates: [ 11.5801204, 48.1235802 ] },
        radius: 87
      },
      searching_for: {
        genders: [
          'Male',
          'Female',
          'Non Binary',
          'Transgender',
          'Genderqueer',
          'Other'
        ],
        ageRange: [ 18, 120 ],
        religion: [
          'Christianity',
          'Islam',
          'Hinduism',
          'Buddhism',
          'Judaism',
          'Other'
        ],
        politics: [
          'Liberal',
          'Conservative',
          'Centrist',
          'Libertarian',
          'Socialist'
        ]
      },
      open_to: [ 'biking', 'puzzles', 'games' ],
},
"userdata":{
	 email: 'luna.vogl.35@gmail.com',
	  password_hash: '$2y$10$YYyEA8ocGk9iU9dZUjLBaeEj9iToWfgcy0oECE5UrA3NIhik2Cm5e',
	  verification_code: null,
	  verification_code_expiry: null,
	  is_verified: true,
	  fingerprint_hash: '$2y$10$eoEopqwq3ZtyePlBmpP8e.yHN2QQrsl2VGqqt9TrztElBT6aFXS5.',
}

- [x] test to make new "get_profile.php" which gets this profile and echos in correct way for ViewMyProfile
	- [x] the json that the frontend wants:
		- [ ] [[[expected json viemyprofile]]]
	- [x] make a mongosh prompt to insert goal_profile_json as the profile json of my existing profile  
	- [x] debug to make ViewMyProfile & get_profile.php work together
	- [ ] add the new profile element types
		- [ ] add simplest layout files which just accepts the data and displays in string
		- [ ] make work with get_profile.php

- [x] adapt match.php to it

- [ ] Now make each script in creation flow work to achieve this json
	- [ ] adabt existing ones
	- [ ] write new ones where needed

- [ ] Make translation from what profilejson now looks like to what match.php expects
	- [ ] find out by debug printing, what "it" expects
		- [ ] since script itself acpects only id we need ot make it assemble a json before matching to have this json as buffer and goal 
	- [ ] what it puts out



lets now change each of the .php scripts involved in creating this profile structure to make each add their part correctly.

- [ ] AboutYouLayout > add_answer.php > "userdata"


#### working match.php
swipe_chat_play> db.users.findOne({ username: "luna" });
{
  _id: ObjectId('681ccfa0ed13220a4c0b6243'),
  username: 'luna',
  firstname: 'example',
  lastname: 'mample',
  email: 'luna.vogl.35@gmail.com',
  password_hash: '$2y$10$YYyEA8ocGk9iU9dZUjLBaeEj9iToWfgcy0oECE5UrA3NIhik2Cm5e',
  verification_code: null,
  verification_code_expiry: null,
  is_verified: true,
  fingerprint_hash: '$2y$10$eoEopqwq3ZtyePlBmpP8e.yHN2QQrsl2VGqqt9TrztElBT6aFXS5.',
  profile: {
    about_you: {
      name: 'lunaneu',
      age: 23,
      gender: 'Transgender',
      spokenLanguage: 'German, English',
      religion: 'Buddhism',
      career: 'ubsne',
      politics: 'Libertarian'
    },
    images: [
      'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/681ccfc92dbb4_image_cropper_1746718662195.jpg'
    ],
    questions_and_answers: [
      {
        question: 'Was ich erst vor kurzem herausgefunden habe...',
        answer: 'uhwk'
      }
    ],
    location_radius: {
      location: { type: 'Point', coordinates: [ 11.5801486, 48.1235825 ] },
      radius: 100
    },
    open_to: [
      'jogging',
      'birdwatching',
      'cooking',
      'games',
      'Arts & Crafts',
      'puzzles'
    ],
    searching_for: {
      genders: [ 'Other', 'Genderqueer', 'Transgender', 'Female', 'Non Binary' ],
      ageRange: [ 18, 120 ],
      religion: [
        'Christianity',
        'Islam',
        'Hinduism',
        'Buddhism',
        'Judaism',
        'Other'
      ],
      politics: [
        'Liberal',
        'Conservative',
        'Centrist',
        'Libertarian',
        'Socialist'
      ]
    }
  }
}



<?php

/* ────────────────────────────────────────────────────────────────────────── */

/*  MATCH.PHP  ·  liefert paginierte Kandidaten und schließt alle Profile    */

/*  aus, die der aufrufende User bereits mit "yes" oder "no" geswiped hat    */

/* ────────────────────────────────────────────────────────────────────────── */

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

/* ───────────────────── 1 · JWT → User-ID ───────────────────── */

$headers = getallheaders();

if (!isset($headers['Authorization']) || strpos($headers['Authorization'], 'Bearer ') !== 0) {

    http_response_code(401);

    echo json_encode(["error" => "Missing / invalid Authorization header"]);

    exit;

}

$jwt = trim(substr($headers['Authorization'], 7));

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    $decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

    $userId  = $decoded->user_id;

} catch (Throwable $e) {

    http_response_code(401);

    echo json_encode(["error" => "Invalid token"]);

    exit;

}

  

/* ───────────────────── 2 · Mongo – aktuelles User-Profil ───────────────── */

$mongo = new MongoDB\Driver\Manager($mongoURI);

  

$currentUserDoc = current(

    $mongo->executeQuery(

        "swipe_chat_play.users",

        new MongoDB\Driver\Query(['_id' => new ObjectId($userId)])

    )->toArray()

);

  

/* komplettes User-Dokument in ein Array konvertieren und als zentrales JSON-Objekt verwenden */

if (!$currentUserDoc || !isset($currentUserDoc->profile->searching_for)) {

    http_response_code(400);

    echo json_encode(["error" => "User profile incomplete"]);

    exit;

}

$currentUser = json_decode(json_encode($currentUserDoc), true);   // zentrales JSON-Objekt

$searchingFor = $currentUser['profile']['searching_for'] ?? [];

  

/* ───────────────────── 3 · Grundfilter (Gender, Age, ≠ ich) ────────────── */

$hardFilter = [];

  

if (!empty($searchingFor['genders'])) {

    $hardFilter['profile.about_you.gender'] = ['$in' => $searchingFor['genders']];

}

if (!empty($searchingFor['ageRange'])) {

    [$minAge,$maxAge] = array_map('intval', $searchingFor['ageRange']);

    $hardFilter['profile.about_you.age'] = ['$gte' => $minAge, '$lte' => $maxAge];

}

/* mich selbst niemals zurückgeben */

$hardFilter['_id'] = ['$ne' => new ObjectId($userId)];

  

/* ───────────────────── 4 · IDs der bereits geswipten Profile ───────────── */

$excludeIds = [];

$cursor = $mongo->executeQuery(

    "swipe_chat_play.matches",

    new MongoDB\Driver\Query(

        [ "users.$userId" => ['$in' => ['yes','no']] ],

        [ 'projection' => ['users' => 1] ]

    )

);

foreach ($cursor as $doc) {

    foreach ((array)$doc->users as $uid => $vote) {

        if ($uid !== $userId) {              // das Gegenüber

            $excludeIds[$uid] = true;

        }

    }

}

if ($excludeIds) {

    $hardFilter['_id']['$nin'] = array_map(

        fn($id) => new ObjectId($id),

        array_keys($excludeIds)

    );

}

  

/* ───────────────────── 5 · Pagination ──────────────────────── */

$page  = max(1, (int)($_GET['page']  ?? 1));

$limit = max(1, (int)($_GET['limit'] ?? 10));

$skip  = ($page - 1) * $limit;

  

/* total count */

$total = current(

    $mongo->executeCommand('swipe_chat_play',

        new MongoDB\Driver\Command([

            'count' => 'users',

            'query' => $hardFilter

        ])

    )->toArray()

)->n ?? 0;

  

/* Kandidaten holen */

$candidates = iterator_to_array(

    $mongo->executeQuery(

        "swipe_chat_play.users",

        new MongoDB\Driver\Query($hardFilter, ['skip'=>$skip,'limit'=>$limit])

    )

);

  

/* ───────────────────── 6 · Scoring ─────────────────────────── */

$wOpen=5; $wLang=2; $wRel=1; $wPol=1;

$ranked=[];

  

/* zentrale Daten des aktuellen Users für schnelleren Zugriff vorbereiten */

$currentOpenTo         = isset($currentUser['profile']['open_to'])

    ? (array)$currentUser['profile']['open_to'] : [];

$currentSpokenLanguage = $currentUser['profile']['about_you']['spokenLanguage'] ?? '';

$currentReligion       = $currentUser['profile']['about_you']['religion']       ?? '';

$currentPolitics       = $currentUser['profile']['about_you']['politics']       ?? '';

  

foreach ($candidates as $cand) {

    $s = 0;

  

    /* open_to */

    $candOpenTo = isset($cand->profile->open_to) ? (array)$cand->profile->open_to : [];

    $s += count(array_intersect($currentOpenTo, $candOpenTo)) * $wOpen;

  

    /* spoken language */

    $candSpokenLanguage = $cand->profile->about_you->spokenLanguage ?? '';

    if ($candSpokenLanguage === $currentSpokenLanguage) { $s += $wLang; }

  

    /* religion */

    $candReligion = $cand->profile->about_you->religion ?? '';

    if ($candReligion === $currentReligion) { $s += $wRel; }

  

    /* politics */

    $candPolitics = $cand->profile->about_you->politics ?? '';

    if ($candPolitics === $currentPolitics) { $s += $wPol; }

  

    $a = (array)$cand;

    $a['matching_score'] = $s;

    $ranked[] = $a;

}

usort($ranked, fn($a, $b) => $b['matching_score'] <=> $a['matching_score']);

  

/* ───────────────────── 7 · Format fürs Frontend ─────────────── */

$out = [];

foreach ($ranked as $c) {

    $p = $c['profile'];

    $out[] = [

        "topSection" => [

            "id"            => (string)$c['_id'],

            "profileImages" => $p->images ?? [],

            "name"          => trim(($c['firstname'] ?? '') . ' ' . ($c['lastname'] ?? '')),

            "age"           => (string)($p->about_you->age ?? ''),

            "infoBubbles"   => [

                $p->about_you->gender          ?? '',

                $p->about_you->spokenLanguage  ?? '',

                $p->about_you->religion        ?? '',

                $p->about_you->politics        ?? ''

            ]

        ],

        "elements" => [

            "1" => [

                "type"    => "icebreaker",

                "content" => (isset($p->questions_and_answers[0]))

                    ? [

                        "question" => $p->questions_and_answers[0]->question ?? '',

                        "answer"   => $p->questions_and_answers[0]->answer   ?? ''

                    ]

                    : ["question" => "", "answer" => ""]

            ],

            "2" => [

                "type"    => "bubbles",

                "content" => [

                    "title"   => "open to",

                    "bubbles" => $p->open_to ?? []

                ]

            ]

        ]

    ];

}

  

/* ───────────────────── 8 · Response ─────────────────────────── */

echo json_encode([

    "status"   => "success",

    "data"     => $out,

    "has_more" => $total > ($skip + $limit)

]);
?>

