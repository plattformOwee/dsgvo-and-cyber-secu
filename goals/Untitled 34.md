I. References:

page where user can view their own profile:

"

<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

header('Content-Type: application/json');

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

// Get JWT token from headers

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

http_response_code(401);

echo json_encode(["error" => "Unauthorized"]);

exit;

}

$jwt = $matches[1];

$decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

$userId = $decoded->user_id ?? null;

if (!$userId) {

http_response_code(401);

echo json_encode(["error" => "Invalid token"]);

exit;

}

// Connect to MongoDB

$mongo = new MongoDB\Driver\Manager($mongoURI);

$filter = ['_id' => new MongoDB\BSON\ObjectId($userId)];

$options = ['projection' => ['profile' => 1]];

$query = new MongoDB\Driver\Query($filter, $options);

$cursor = $mongo->executeQuery('swipe_chat_play.users', $query);

$user = current($cursor->toArray());

if (!$user || !isset($user->profile)) {

http_response_code(404);

echo json_encode(["error" => "User profile not found"]);

exit;

}

$profile = $user->profile;

// Transform MongoDB profile data into the expected JSON structure

// NOTE: The keys are now `topSection` and `elements` so it matches your Dart model.

$response = [

'topSection' => [

'profileImages' => $profile->images ?? [],

'name' => $profile->about_you->name ?? 'Unknown',

'age' => $profile->about_you->age ?? 'Unknown',

'infoBubbles' => [

$profile->about_you->gender ?? 'Unknown',

$profile->about_you->spokenLanguage ?? 'Unknown',

$profile->about_you->religion ?? 'Unknown',

$profile->about_you->politics ?? 'Unknown',

],

],

'elements' => [

'1' => [

'type' => 'icebreaker',

'content' => [

'question' => 'Wenn ich ein Tier wäre, dann wäre ich ein …',

'answer' => $profile->questions_and_answers[0]->answer ?? 'No answer provided',

],

],

'2' => [

'type' => 'icebreaker',

'content' => [

'question' => 'Ein Ort, an den ich immer wieder zurückkehre...',

'answer' => $profile->questions_and_answers[1]->answer ?? 'No answer provided',

],

],

'3' => [

'type' => 'icebreaker',

'content' => [

'question' => 'Etwas, das ich gerne lernen möchte...',

'answer' => $profile->questions_and_answers[2]->answer ?? 'No answer provided',

],

],

'4' => [

'type' => 'bubbles',

'content' => [

'title' => 'Open to',

'bubbles' => $profile->open_to ?? [],

],

],

],

];

echo json_encode($response, JSON_PRETTY_PRINT);

} catch (Exception $e) {

error_log("Profile load error: " . $e->getMessage());

http_response_code(500);

echo json_encode(["error" => "Server error", "message" => $e->getMessage()]);

}

"

profiledata:

"

class ProfileData {

final TopSection topSection;

final Map<String, ProfileElement> elements;

ProfileData({required this.topSection, required this.elements});

factory ProfileData.fromJson(Map<String, dynamic> json) {

// If the JSON is missing 'topSection', fall back to an empty map.

final topSectionJson = (json['topSection'] as Map<String, dynamic>?) ?? {};

final topSection = TopSection.fromJson(topSectionJson);

// If the JSON is missing 'elements', fall back to an empty map.

final elementsJson = (json['elements'] as Map<String, dynamic>?) ?? {};

final elementsMap = elementsJson.map((key, value) {

// Each element should be a Map<String, dynamic>. If it's null, use {}.

final elementMap = (value as Map<String, dynamic>?) ?? {};

return MapEntry(key, ProfileElement.fromJson(elementMap));

});

return ProfileData(

topSection: topSection,

elements: elementsMap,

);

}

}

class TopSection {

final List<String> profileImages;

final String name;

final String age;

final List<String> infoBubbles;

TopSection({

required this.profileImages,

required this.name,

required this.age,

required this.infoBubbles,

});

factory TopSection.fromJson(Map<String, dynamic> json) {

// Provide default empty lists/strings if null or missing.

return TopSection(

profileImages: List<String>.from(json['profileImages'] ?? []),

name: json['name']?.toString() ?? '',

age: json['age']?.toString() ?? '',

infoBubbles: List<String>.from(json['infoBubbles'] ?? []),

);

}

}

class ProfileElement {

final String type;

final Map<String, dynamic> content;

ProfileElement({required this.type, required this.content});

factory ProfileElement.fromJson(Map<String, dynamic> json) {

// Provide defaults for 'type' and 'content'.

return ProfileElement(

type: json['type']?.toString() ?? '',

content: (json['content'] as Map<String, dynamic>?) ?? {},

);

}

}

"

a php script returning a profile

"

<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

header('Content-Type: application/json');

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

// Get JWT token from headers

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

http_response_code(401);

echo json_encode(["error" => "Unauthorized"]);

exit;

}

$jwt = $matches[1];

$decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

$userId = $decoded->user_id ?? null;

if (!$userId) {

http_response_code(401);

echo json_encode(["error" => "Invalid token"]);

exit;

}

// Connect to MongoDB

$mongo = new MongoDB\Driver\Manager($mongoURI);

$filter = ['_id' => new MongoDB\BSON\ObjectId($userId)];

$options = ['projection' => ['profile' => 1]];

$query = new MongoDB\Driver\Query($filter, $options);

$cursor = $mongo->executeQuery('swipe_chat_play.users', $query);

$user = current($cursor->toArray());

if (!$user || !isset($user->profile)) {

http_response_code(404);

echo json_encode(["error" => "User profile not found"]);

exit;

}

$profile = $user->profile;

// Transform MongoDB profile data into the expected JSON structure

// NOTE: The keys are now `topSection` and `elements` so it matches your Dart model.

$response = [

'topSection' => [

'profileImages' => $profile->images ?? [],

'name' => $profile->about_you->name ?? 'Unknown',

'age' => $profile->about_you->age ?? 'Unknown',

'infoBubbles' => [

$profile->about_you->gender ?? 'Unknown',

$profile->about_you->spokenLanguage ?? 'Unknown',

$profile->about_you->religion ?? 'Unknown',

$profile->about_you->politics ?? 'Unknown',

],

],

'elements' => [

'1' => [

'type' => 'icebreaker',

'content' => [

'question' => 'Wenn ich ein Tier wäre, dann wäre ich ein …',

'answer' => $profile->questions_and_answers[0]->answer ?? 'No answer provided',

],

],

'2' => [

'type' => 'icebreaker',

'content' => [

'question' => 'Ein Ort, an den ich immer wieder zurückkehre...',

'answer' => $profile->questions_and_answers[1]->answer ?? 'No answer provided',

],

],

'3' => [

'type' => 'icebreaker',

'content' => [

'question' => 'Etwas, das ich gerne lernen möchte...',

'answer' => $profile->questions_and_answers[2]->answer ?? 'No answer provided',

],

],

'4' => [

'type' => 'bubbles',

'content' => [

'title' => 'Open to',

'bubbles' => $profile->open_to ?? [],

],

],

],

];

echo json_encode($response, JSON_PRETTY_PRINT);

} catch (Exception $e) {

error_log("Profile load error: " . $e->getMessage());

http_response_code(500);

echo json_encode(["error" => "Server error", "message" => $e->getMessage()]);

}

"

a stack of cards (but with local json):

"

import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:google_fonts/google_fonts.dart';

// Import your model & widgets (prefix the model to avoid name conflicts if needed).

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/image_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

class CardStack extends StatefulWidget {

const CardStack({Key? key}) : super(key: key);

@override

_CardStackState createState() => _CardStackState();

}

class _CardStackState extends State<CardStack>

with SingleTickerProviderStateMixin {

// Sample JSON list holding multiple profile objects.

final List<Map<String, dynamic>> profileJsonList = [

{

"topSection": {

"profileImages": [

"https://pix4free.org/assets/library/2021-06-16/originals/example.jpg",

"https://pix4free.org/assets/library/2021-06-16/originals/example.jpg"

],

"name": "John Doe",

"age": "28",

"infoBubbles": ["Male", "English", "Non-religious", "Moderate"]

},

"elements": {

"1": {

"type": "icebreaker",

"content": {

"question": "What's your favorite travel destination?",

"answer": "I love exploring hidden gems in Southeast Asia"

}

},

"2": {

"type": "bubbles",

"content": {

"title": "Open To",

"bubbles": ["Hiking", "Photography", "Coffee", "Startups"]

}

}

}

},

{

"topSection": {

"profileImages": [

"https://pix4free.org/assets/library/2021-06-16/originals/example.jpg",

"https://pix4free.org/assets/library/2021-06-16/originals/example.jpg"

],

"name": "Jane Smith",

"age": "30",

"infoBubbles": ["Female", "Spanish", "Atheist", "Liberal"]

},

"elements": {

"1": {

"type": "icebreaker",

"content": {

"question": "What's your idea of a perfect weekend?",

"answer": "A cozy weekend with books and coffee."

}

},

"2": {

"type": "bubbles",

"content": {

"bubbles": ["Reading", "Travel", "Cooking"]

}

}

}

},

];

late List<profile.ProfileData> cardData;

double dragPositionX = 0.0;

AnimationController? swipeController;

final double swipeThreshold = 100;

@override

void initState() {

super.initState();

// Convert each JSON map into a ProfileData object.

cardData = profileJsonList

.map((json) => profile.ProfileData.fromJson(json))

.toList();

swipeController = AnimationController(

vsync: this,

duration: const Duration(milliseconds: 100),

);

}

@override

void dispose() {

swipeController?.dispose();

super.dispose();

}

void dismissCardAnimated(int index) {

swipeController?.forward().then((_) {

swipeController?.reset();

dismissCard(index);

});

}

void dismissCard(int index) {

setState(() {

cardData.removeAt(index);

dragPositionX = 0.0;

});

}

void resetDrag() {

setState(() {

dragPositionX = 0.0;

});

}

@override

Widget build(BuildContext context) {

return SafeArea(

child: Stack(

children: cardData

.asMap()

.entries

.map((entry) => buildCard(entry.key, entry.value))

.toList(),

),

);

}

Widget buildCard(int index, profile.ProfileData data) {

final isTopCard = index == cardData.length - 1;

return Positioned.fill(

child: isTopCard

? GestureDetector(

onPanUpdate: (details) {

setState(() {

dragPositionX += details.delta.dx;

});

},

onPanEnd: (_) {

if (dragPositionX.abs() > swipeThreshold) {

dismissCardAnimated(index);

} else {

resetDrag();

}

},

child: Stack(

children: [

Transform.translate(

offset: Offset(dragPositionX, 0),

child: Transform.rotate(

angle: dragPositionX / 200,

child: ProfileCard(data: data),

),

),

buildSwipeIndicator(),

],

),

)

: ProfileCard(data: data),

);

}

Widget buildSwipeIndicator() {

return Positioned(

top: MediaQuery.of(context).size.height / 2 - 30,

left: dragPositionX > 0 ? null : 20,

right: dragPositionX > 0 ? 20 : null,

child: Opacity(

opacity: (dragPositionX.abs() / swipeThreshold).clamp(0.0, 1.0),

child: CircleAvatar(

radius: 35, // Outer circle (white outline)

backgroundColor: Colors.white,

child: CircleAvatar(

radius: 30, // Inner circle (actual indicator)

backgroundColor: dragPositionX > 0 ? Colors.green : Colors.red,

child: Icon(

dragPositionX > 0 ? Icons.check : Icons.close,

color: Colors.white,

size: 30,

),

),

),

),

);

}

}

class ProfileCard extends StatelessWidget {

final profile.ProfileData data;

const ProfileCard({Key? key, required this.data}) : super(key: key);

@override

Widget build(BuildContext context) {

// Reuse the layout widgets from your profiles directory.

return Padding(

padding: const EdgeInsets.all(8.0),

child: Card(

shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),

child: SingleChildScrollView(

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

// Use the TopSectionWidget from profiles/widgets/top_section.dart.

TopSectionWidget(topSection: data.topSection),

// For each element, use the corresponding layout widget.

...data.elements.entries.map((entry) => Column(

children: [

const Divider(height: 1, thickness: 1),

_buildElement(entry.value),

],

))

],

),

),

),

);

}

Widget _buildElement(profile.ProfileElement element) {

switch (element.type) {

case 'icebreaker':

return IceBreakerLayout(content: element.content);

case 'image':

return ImageLayout(content: element.content);

case 'bubbles':

return BubbleGridLayout(content: element.content);

default:

return const SizedBox();

}

}

}

"

II. Task and Goal

The Goal is to make swiping through profiles based on matching criteria possible in this app.

1 task:
write a script which generates 100 users and writes them to mongo db