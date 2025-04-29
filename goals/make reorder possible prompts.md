There is a fundamental flaw with how i wanted to make the profile reorderable.

On the server the profile looks something like this:

"

profile: {

about_you: {

name: 'luna',

age: 23,

gender: 'Female',

spokenLanguage: 'German, English',

religion: 'Other',

career: 'bla',

politics: 'Socialist'

},

images: [

'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/680bbb63263f1_image_cropper_1745599317194.jpg'

],

questions_and_answers: [

{

question: 'Wenn ich ein Tier wäre, dann wäre ich ein …',

answer: 'vsvs'

}

],

location_radius: {

location: { type: 'Point', coordinates: [ 11.5801304, 48.1235853 ] },

radius: 46.36363636363636

},

open_to: [ 'jogging', 'camping', 'cooking', 'Arts & Crafts' ],

searching_for: {

genders: [

'Female',

'Transgender',

'Genderqueer',

'Non Binary',

'Male',

'Other'

],

ageRange: [ 18, 75 ],

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

}"

But this structure doesnt actually drive the layout:

"

import 'dart:convert';

import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:google_fonts/google_fonts.dart';

// Import your model & widgets.

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

/// View the current user's profile.

/// Top section is fixed; all subsequent elements are reorderable via drag-and-drop

/// and persisted via *saveProfile.php*.

class ViewMyProfile extends StatefulWidget {

const ViewMyProfile({super.key});

@override

State<ViewMyProfile> createState() => _ViewMyProfileState();

}

class _ViewMyProfileState extends State<ViewMyProfile> {

static const storage = FlutterSecureStorage();

late Future<profile.ProfileData> _futureProfile;

/// Each MapEntry stores the backend *key* plus its element payload.

List<MapEntry<String, profile.ProfileElement>>? _entries;

profile.ProfileData? _fullProfile; // keep a reference so we can upload

@override

void initState() {

super.initState();

_futureProfile = _fetchProfile();

}

//────────────────────────────────────────────────── API

Future<profile.ProfileData> _fetchProfile() async {

final url = '${Config.backendBaseUrl}/create_profile/get_profile.php';

final token = await storage.read(key: 'jwt_token');

if (token == null) throw Exception('No JWT token found');

final response = await http.get(Uri.parse(url), headers: {

'Authorization': 'Bearer $token',

});

if (response.statusCode == 200) {

return profile.ProfileData.fromJson(

json.decode(response.body) as Map<String, dynamic>,

);

} else {

throw Exception('Failed to load profile. Status ${response.statusCode}');

}

}

/// Build a JSON object **identical to the one used to create the layout**, but

/// with element keys in their new order, and POST it to *saveProfile.php*.

Future<void> uploadChange() async {

final url = '${Config.backendBaseUrl}/create_profile/saveProfile.php';

final token = await storage.read(key: 'jwt_token');

if (token == null) {

debugPrint('[uploadChange] ⚠️ No JWT token – abort');

return;

}

// 1️⃣ Build a LinkedHashMap with the new order **and** full objects

final orderedElements = <String, dynamic>{};

for (final e in _entries!) {

orderedElements[e.key] = e.value.toJson(); // keep type + content

}

// 2️⃣ Copy the cached profile and replace its elements

final profileMap = _fullProfile!.toJson();

profileMap['elements'] = orderedElements;

final payload = jsonEncode(profileMap);

// 3️⃣ POST

debugPrint('[uploadChange] Payload → $payload');

final res = await http.post(

Uri.parse(url),

headers: {

'Authorization': 'Bearer $token',

'Content-Type': 'application/json',

},

body: payload,

);

debugPrint('[uploadChange] ⇣ ${res.statusCode} ${res.body}');

}

//────────────────────────────────────────────────── UI

@override

Widget build(BuildContext context) {

return Scaffold(

backgroundColor: AppColors.background,

body: FutureBuilder<profile.ProfileData>(

future: _futureProfile,

builder: (context, snapshot) {

if (snapshot.connectionState == ConnectionState.waiting) {

return const Center(child: CircularProgressIndicator());

} else if (snapshot.hasError) {

return _buildError(snapshot.error);

} else if (!snapshot.hasData) {

return _buildError('No profile data found.');

}

final userProfile = snapshot.data!;

_fullProfile ??= userProfile; // cache

// Lazily initialise the reorderable list.

_entries ??= userProfile.elements.entries.toList(growable: true);

return Padding(

padding: const EdgeInsets.all(8),

child: Card(

color: AppColors.cardBackground,

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(16),

),

child: SingleChildScrollView(

physics: const BouncingScrollPhysics(),

child: Column(

crossAxisAlignment: CrossAxisAlignment.stretch,

children: [

// --- Fixed Top Section --------

TopSectionWidget(

topSection: userProfile.topSection,

swipedUserUsername: userProfile.topSection.username,

),

const Divider(

height: 2,

thickness: 2,

color: AppColors.dividerLine,

),

// --- Re-orderable Elements ----

ReorderableListView.builder(

shrinkWrap: true,

physics: const NeverScrollableScrollPhysics(),

padding: const EdgeInsets.symmetric(vertical: 8),

itemCount: _entries!.length,

buildDefaultDragHandles: false,

onReorder: (oldIndex, newIndex) async {

setState(() {

if (newIndex > oldIndex) newIndex -= 1;

final moved = _entries!.removeAt(oldIndex);

_entries!.insert(newIndex, moved);

});

await uploadChange();

},

itemBuilder: (context, index) {

final entry = _entries![index];

final element = entry.value;

return Column(

key: ValueKey('${entry.key}-$index'),

children: [

_buildReorderableRow(index, entry.key, element,

userProfile.topSection.username),

if (index != _entries!.length - 1)

const Divider(

height: 2,

thickness: 2,

color: AppColors.dividerLine,

),

],

);

},

),

],

),

),

),

);

},

),

);

}

/// Row with drag-handle + actual element widget.

Widget _buildReorderableRow(int index, String elementKey,

profile.ProfileElement element, String swipedUserUsername) {

return Row(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

ReorderableDragStartListener(

index: index,

child: const Padding(

padding: EdgeInsets.only(right: 8, top: 4),

child: Icon(Icons.drag_indicator),

),

),

Expanded(child: _buildElement(element, swipedUserUsername)),

],

);

}

Widget _buildElement(

profile.ProfileElement element, String swipedUserUsername) {

switch (element.type) {

case 'icebreaker':

return IceBreakerLayout(

content: element.content,

swipedUserUsername: swipedUserUsername,

);

case 'bubbles':

return BubbleGridLayout(

content: element.content,

swipedUserUsername: swipedUserUsername,

);

default:

return const SizedBox.shrink();

}

}

Widget _buildError(Object? error) => Center(

child: Text(

'Error: $error',

style: GoogleFonts.montserrat(),

),

);

}

"

To fix this we want to do several things.

1. i want to keep "profile" as is becouse some of the other code like the match algorithm depends on it.

2. i want to add a new "profile_structured" to the db which will drive the actual layout.

profile_structured:{

"

}

3. change "get_profile.php" such that it will first look for "profile_structured" and if it doesnt exist, use the normal "profile"

get_profile.php:"

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

4. let saveProfile.php save the reordered profile under "profile_structured" instead of "profile"

saveProfile.php

"

<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

header('Content-Type: application/json');

// 1. HTTP-method check

if (\$_SERVER['REQUEST_METHOD'] !== 'POST') {

http_response_code(405);

echo json_encode(['error' => 'Method not allowed']);

exit;

}

// 2. JWT Authentication

\$hdr = getallheaders();

\$authHeader = \$hdr['Authorization'] ?? \$hdr['authorization'] ?? null;

if (!\$authHeader || !preg_match('/Bearer\\s(\\S+)/', \$authHeader, \$m)) {

http_response_code(401);

echo json_encode(['error' => 'Unauthorized']);

exit;

}

\$jwtToken = \$m[1];

try {

\$decoded = JWT::decode(\$jwtToken, new Key(\$secretKey, 'HS256'));

\$userId = \$decoded->user_id ?? null;

if (!\$userId) throw new Exception('Invalid token');

} catch (Exception \$e) {

http_response_code(401);

echo json_encode(['error' => \$e->getMessage()]);

exit;

}

// 3. Read + validate JSON body

\$bodyRaw = file_get_contents('php://input');

\$body = json_decode(\$bodyRaw, true);

if (\$body === null || !is_array(\$body)) {

http_response_code(400);

echo json_encode(['error' => 'Request body must be valid JSON']);

exit;

}

if (!isset(\$body['elements']) || !is_array(\$body['elements'])) {

http_response_code(400);

echo json_encode(['error' => "Missing 'elements' array in payload"]);

exit;

}

// 4. Build ordered profile object (key => content)

\$profileObj = new stdClass();

foreach (\$body['elements'] as \$key => \$elem) {

if (isset(\$elem['content'])) {

\$profileObj->\$key = \$elem['content'];

}

}

// 5. Persist to MongoDB

try {

\$manager = new MongoDB\Driver\Manager(\$mongoURI);

\$bulk = new MongoDB\Driver\BulkWrite();

\$bulk->update(

['_id' => new MongoDB\BSON\ObjectId(\$userId)],

['$set' => ['profile' => \$profileObj]],

['multi' => false, 'upsert' => false]

);

\$result = \$manager->executeBulkWrite('swipe_chat_play.users', \$bulk);

if (\$result->getModifiedCount() === 0) {

throw new Exception('No document updated (user not found?)');

}

echo json_encode(['success' => true, 'message' => 'Profile order saved']);

} catch (Exception \$e) {

http_response_code(500);

echo json_encode(['error' => \$e->getMessage()]);

}

"

5. rewrite ViewMyProfile such that it directly depends on the structure of this json when deciding the order of the elements below top section.

