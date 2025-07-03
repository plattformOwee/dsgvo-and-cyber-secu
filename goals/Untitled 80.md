Task1: write a age_set.php endpoint in same schema as other endpoints in this folder which will take the date of birth from the frontend and calculate the age of the person and safe that in the mongo profile structure

references for task1:

- other php endpoint (copy how config and bootstrap are used aswell as how mongodb is used):

"

<?php

declare(strict_types=1);

use MongoDB\Driver\Manager;

use MongoDB\Driver\BulkWrite;

use MongoDB\Driver\Exception\Exception as MongoException;

use MongoDB\BSON\ObjectId;

// 1) includes

require_once __DIR__ . '/../../bootstrap.php'; // dbg(), auth(), Composer

require_once __DIR__ . '/../../config.php'; // sets $mongoURI and $mongoDb

header('Content-Type: application/json; charset=utf-8');

// 2) only POST

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

http_response_code(405);

echo json_encode(['error'=>'Method not allowed']);

exit;

}

// 3) authenticate

$claims = auth(); // will 401/exit if invalid

$userId = $claims->sub;

dbg("name_lastname for user={$userId}");

// 4) input validation

$first = trim($_POST['first_name'] ?? '');

$last = trim($_POST['last_name'] ?? '');

dbg("Input first='{$first}' last='{$last}'");

if ($first === '' || $last === '') {

http_response_code(400);

echo json_encode(['error'=>'Missing first or last name']);

exit;

}

// 5) update

try {

$mgr = new Manager($mongoURI);

$bulk = new BulkWrite();

$bulk->update(

['_id' => new ObjectId($userId)],

['$set'=>[

'profile.topSection.firstName' => $first,

'profile.topSection.lastName' => $last,

'profile.topSection.name' => "{$first} {$last}",

]]

);

$ns = "{$mongoDb}.users"; // e.g. "woven.users"

$result = $mgr->executeBulkWrite($ns, $bulk);

$modified = $result->getModifiedCount();

dbg("Mongo modified={$modified}");

if ($modified === 0) {

throw new \RuntimeException('No documents updated');

}

http_response_code(200);

echo json_encode(['message'=>'Success']);

} catch (MongoException|\RuntimeException $e) {

dbg('Mongo/update error: '.$e->getMessage());

http_response_code(500);

echo json_encode(['error'=>'Internal server error']);

}

"

- the mongo structure:

"

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

"link_voicemessage":"http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f877edb34a88.05803339_1744336870367.aac" // the link generated after saving the thing i will provide test link for now

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

}"

Task2: connect this age_page to the php endpoint using the jwt from secure storage like in other example file i will include

// lib/profile_creation/age_page.dart

import 'package:flutter/material.dart';

import 'package:swipe_chat_play/main.dart'; // for kRouteFilterSelection

import 'package:swipe_chat_play/usables/rounded_input_field.dart';

import 'package:swipe_chat_play/usables/primary_button.dart';

import 'package:swipe_chat_play/usables/signup_progress_bar.dart';

class AgePage extends StatefulWidget {

const AgePage({Key? key}) : super(key: key);

@override

State<AgePage> createState() => _AgePageState();

}

class _AgePageState extends State<AgePage> {

static const _stripeColor = Color(0xFFFFEDE6);

static const _highlightColor = Color(0xFFFA938E);

final _dateCtrl = TextEditingController();

bool _loading = false;

Future<void> _pickDate() async {

final now = DateTime.now();

final picked = await showDatePicker(

context: context,

initialDate: now,

firstDate: DateTime(now.year - 100),

lastDate: now,

);

if (picked != null) {

setState(() {

_dateCtrl.text =

'${picked.year}-${picked.month.toString().padLeft(2, '0')}-${picked.day.toString().padLeft(2, '0')}';

});

}

}

Future<void> _onContinue() async {

if (_dateCtrl.text.isEmpty) {

ScaffoldMessenger.of(context).showSnackBar(

const SnackBar(content: Text('Please select your birthday')),

);

return;

}

setState(() => _loading = true);

// TODO: POST to your choose_age.php if needed.

// Navigate straight to SearchingForPage:

Navigator.of(context).pushReplacementNamed(kRouteSearchinFor);

}

@override

void dispose() {

_dateCtrl.dispose();

super.dispose();

}

@override

Widget build(BuildContext context) {

return Scaffold(

backgroundColor: Colors.grey[100],

body: SafeArea(

top: true,

bottom: false,

child: Column(

crossAxisAlignment: CrossAxisAlignment.stretch,

children: [

// full-width progress bar (11%) directly under stripe

SignupProgressBar(

percentage: 16,

height: 8.0,

fillColor: _highlightColor,

backgroundColor: _stripeColor,

),

const SizedBox(height: 20),

// title

Center(

child: Text(

'birthday',

style: TextStyle(

fontSize: 20,

fontWeight: FontWeight.w500,

color: Colors.grey[800],

),

),

),

const Spacer(),

// date picker & continue button

Padding(

padding: const EdgeInsets.symmetric(horizontal: 24),

child: Column(

children: [

GestureDetector(

onTap: _pickDate,

child: AbsorbPointer(

child: RoundedInputField(

controller: _dateCtrl,

hintText: 'select date',

highlightColor: _highlightColor,

centerText: true,

),

),

),

const SizedBox(height: 24),

PrimaryButton(

onPressed: _onContinue,

text: 'continue',

isLoading: _loading,

),

const SizedBox(height: 32),

],

),

),

],

),

),

);

}

}"