omatch.php:"

<?php

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

// --- 1. Get JWT from Authorization header ---

$headers = getallheaders();

if (!isset($headers['Authorization'])) {

http_response_code(401);

echo json_encode(["error" => "Missing Authorization header"]);

exit;

}

$authHeader = $headers['Authorization'];

if (strpos($authHeader, "Bearer ") !== 0) {

http_response_code(401);

echo json_encode(["error" => "Invalid Authorization header format"]);

exit;

}

$jwtToken = trim(str_replace("Bearer", "", $authHeader));

// --- 2. Decode JWT ---

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

$decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

} catch (Exception $e) {

http_response_code(401);

echo json_encode(["error" => "Invalid token: " . $e->getMessage()]);

exit;

}

// Assume the token payload contains the user ID (as "user_id")

$userId = $decodedToken->user_id;

// --- 3. Connect to MongoDB and load current user profile ---

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

$namespace = "swipe_chat_play.users"; // Change to your actual DB and collection name

$query = new MongoDB\Driver\Query(['_id' => new MongoDB\BSON\ObjectId($userId)]);

try {

$cursor = $mongoManager->executeQuery($namespace, $query);

$currentUser = current($cursor->toArray());

if (!$currentUser) {

http_response_code(404);

echo json_encode(["error" => "User not found"]);

exit;

}

} catch (Exception $e) {

http_response_code(500);

echo json_encode(["error" => "Database error: " . $e->getMessage()]);

exit;

}

// Ensure the current user's profile has a "searching_for" field

if (!isset($currentUser->profile->searching_for)) {

http_response_code(400);

echo json_encode(["error" => "Searching criteria not defined"]);

exit;

}

$searchingFor = (array)$currentUser->profile->searching_for;

// --- 4. Build hard filter query using "searching_for" ---

$hardFilter = [];

// Gender: candidate's about_you.gender must be one of the desired genders

if (isset($searchingFor['genders'])) {

$hardFilter['profile.about_you.gender'] = [

'$in' => $searchingFor['genders']

];

}

// Age: candidate's about_you.age should be within the provided ageRange

if (isset($searchingFor['ageRange'])) {

$ageRange = $searchingFor['ageRange'];

$minAge = (int)$ageRange[0];

$maxAge = (int)$ageRange[1];

$hardFilter['profile.about_you.age'] = [

'$gte' => $minAge,

'$lte' => $maxAge

];

}

// Exclude the current user from the candidate list

$hardFilter['_id'] = [

'$ne' => new MongoDB\BSON\ObjectId($userId)

];

// NOTE: The lines below are REMOVED so that religion and politics aren't hard filters.

// if (isset($searchingFor['religion'])) {

// $hardFilter['profile.about_you.religion'] = ['$in' => $searchingFor['religion']];

// }

// if (isset($searchingFor['politics'])) {

// $hardFilter['profile.about_you.politics'] = ['$in' => $searchingFor['politics']];

// }

// --- 5. Handle pagination parameters ---

$page = isset($_GET['page']) ? (int)$_GET['page'] : 1;

$limit = isset($_GET['limit']) ? (int)$_GET['limit'] : 10;

$skip = ($page - 1) * $limit;

// --- 5.1. Get total count of matching candidates for has_more calculation ---

$command = new MongoDB\Driver\Command([

'count' => 'users',

'query' => $hardFilter

]);

try {

$cursor = $mongoManager->executeCommand('swipe_chat_play', $command);

$result = $cursor->toArray();

$totalCount = (isset($result[0]) && isset($result[0]->n)) ? $result[0]->n : 0;

} catch (Exception $e) {

http_response_code(500);

echo json_encode(["error" => "Count command failed: " . $e->getMessage()]);

exit;

}

// --- 5.2. Execute query with pagination ---

$queryOptions = [

'skip' => $skip,

'limit' => $limit

];

$query = new MongoDB\Driver\Query($hardFilter, $queryOptions);

try {

$cursor = $mongoManager->executeQuery($namespace, $query);

$candidates = $cursor->toArray();

} catch (Exception $e) {

http_response_code(500);

echo json_encode(["error" => "Database query error: " . $e->getMessage()]);

exit;

}

// --- 5.3. Calculate has_more ---

$hasMore = ($totalCount > ($skip + $limit));

// --- Debug: Log raw candidates ---

$debugFile = __DIR__ . '/debug.log';

$rawCandidatesJson = json_encode($candidates, JSON_PRETTY_PRINT);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Raw Candidates:\n" . $rawCandidatesJson . "\n", FILE_APPEND);

// --- 6. Rank the results based on additional matching attributes ---

$weightOpenTo = 5; // Open-to interests are more important

$weightSpokenLanguage = 2;

$weightReligion = 1;

$weightPolitics = 1;

$rankedCandidates = [];

foreach ($candidates as $candidate) {

$score = 0;

// Compare "open_to" arrays

if (isset($currentUser->profile->open_to) && isset($candidate->profile->open_to)) {

$currentOpenTo = $currentUser->profile->open_to;

$candidateOpenTo = $candidate->profile->open_to;

$commonOpenTo = array_intersect($currentOpenTo, $candidateOpenTo);

$score += count($commonOpenTo) * $weightOpenTo;

}

// Compare spokenLanguage

if (isset($currentUser->profile->about_you->spokenLanguage) && isset($candidate->profile->about_you->spokenLanguage)) {

if ($currentUser->profile->about_you->spokenLanguage === $candidate->profile->about_you->spokenLanguage) {

$score += $weightSpokenLanguage;

}

}

// Bonus for matching religion

if (isset($currentUser->profile->about_you->religion) && isset($candidate->profile->about_you->religion)) {

if ($currentUser->profile->about_you->religion === $candidate->profile->about_you->religion) {

$score += $weightReligion;

}

}

// Bonus for matching politics

if (isset($currentUser->profile->about_you->politics) && isset($candidate->profile->about_you->politics)) {

if ($currentUser->profile->about_you->politics === $candidate->profile->about_you->politics) {

$score += $weightPolitics;

}

}

// Convert candidate to array, attach the matching score

$candidateArray = (array)$candidate;

$candidateArray['matching_score'] = $score;

$rankedCandidates[] = $candidateArray;

}

// --- Debug: Log ranked candidates ---

$rankedCandidatesJson = json_encode($rankedCandidates, JSON_PRETTY_PRINT);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Ranked Candidates:\n" . $rankedCandidatesJson . "\n", FILE_APPEND);

// Sort in descending order of matching score

usort($rankedCandidates, function($a, $b) {

return $b['matching_score'] <=> $a['matching_score'];

});

// --- 6.5. Transform each candidate into the desired structure ---

$formattedCandidates = [];

foreach ($rankedCandidates as $candidate) {

$profile = $candidate['profile'];

$topSection = [

'profileImages' => isset($profile->images) ? $profile->images : [],

'name' => (isset($candidate['firstname']) ? $candidate['firstname'] : "") . ' ' . (isset($candidate['lastname']) ? $candidate['lastname'] : ""),

'age' => isset($profile->about_you->age) ? (string)$profile->about_you->age : '',

'infoBubbles' => [

isset($profile->about_you->gender) ? $profile->about_you->gender : '',

isset($profile->about_you->spokenLanguage) ? $profile->about_you->spokenLanguage : '',

isset($profile->about_you->religion) ? $profile->about_you->religion : '',

isset($profile->about_you->politics) ? $profile->about_you->politics : ''

]

];

$elements = [];

// If there's at least one Q&A

if (

isset($profile->questions_and_answers) &&

is_array($profile->questions_and_answers) &&

count($profile->questions_and_answers) > 0

) {

$firstQA = $profile->questions_and_answers[0];

$elements["1"] = [

"type" => "icebreaker",

"content" => [

"question" => isset($firstQA->question) ? $firstQA->question : "",

"answer" => isset($firstQA->answer) ? $firstQA->answer : ""

]

];

}

// Bubbles from open_to with title "open to"

if (isset($profile->open_to)) {

$elements["2"] = [

"type" => "bubbles",

"content" => [

"title" => "open to",

"bubbles" => $profile->open_to

]

];

}

$formattedCandidates[] = [

"topSection" => $topSection,

"elements" => $elements

];

}

// --- Debug: Log formatted candidates ---

$formattedCandidatesJson = json_encode($formattedCandidates, JSON_PRETTY_PRINT);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Formatted Candidates:\n" . $formattedCandidatesJson . "\n", FILE_APPEND);

// --- 7. Return the formatted list as JSON ---

$finalJson = json_encode([

"status" => "success",

"data" => $formattedCandidates,

"has_more" => $hasMore

]);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Final JSON:\n" . $finalJson . "\n", FILE_APPEND);

echo $finalJson;

?>

" // rewrite to also give the mongo_uinique id for each profile in the stack

swipe.php:

"

<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

// Only allow POST requests.

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

http_response_code(405);

echo json_encode(["status" => "error", "message" => "Method not allowed"]);

exit;

}

// Retrieve JWT from the Authorization header.

$headers = getallheaders();

if (!isset($headers['Authorization'])) {

http_response_code(401);

echo json_encode(["status" => "error", "message" => "Missing Authorization header"]);

exit;

}

$authHeader = $headers['Authorization'];

if (strpos($authHeader, "Bearer ") !== 0) {

http_response_code(401);

echo json_encode(["status" => "error", "message" => "Invalid Authorization header"]);

exit;

}

$jwtToken = trim(str_replace("Bearer ", "", $authHeader));

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

$decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

$senderId = $decodedToken->user_id ?? null;

} catch(Exception $e) {

http_response_code(401);

echo json_encode(["status" => "error", "message" => "Invalid token"]);

exit;

}

if (!$senderId) {

http_response_code(401);

echo json_encode(["status" => "error", "message" => "Invalid token"]);

exit;

}

// Connect to MongoDB and get sender (swiper) information.

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

$senderFilter = ['_id' => new ObjectId($senderId)];

$senderQuery = new MongoDB\Driver\Query($senderFilter);

$senderCursor = $mongoManager->executeQuery("swipe_chat_play.users", $senderQuery);

$sender = current($senderCursor->toArray());

if (!$sender || !isset($sender->username)) {

http_response_code(404);

echo json_encode(["status" => "error", "message" => "Sender not found"]);

exit;

}

$senderUsername = $sender->username;

// Get JSON input.

$input = json_decode(file_get_contents('php://input'), true);

$inputReceiverName = $input['receiver_username'] ?? null; // e.g. "Sara Doe_17"

$swipeValue = $input['swipe'] ?? null;

if (!$inputReceiverName || !$swipeValue) {

http_response_code(400);

echo json_encode(["status" => "error", "message" => "receiver_username and swipe are required"]);

exit;

}

// Use only the first part of the name (before the space) for lookup.

$parts = explode(' ', $inputReceiverName);

$firstName = $parts[0];

$receiverFilter = ['profile.about_you.name' => $firstName];

// Look up the receiver by "profile.about_you.name".

$receiverQuery = new MongoDB\Driver\Query($receiverFilter);

$receiverCursor = $mongoManager->executeQuery("swipe_chat_play.users", $receiverQuery);

$receiver = current($receiverCursor->toArray());

if (!$receiver) {

http_response_code(404);

echo json_encode(["status" => "error", "message" => "Receiver not found"]);

exit;

}

// Override the receiver identifier with the actual username (e.g. "sara_17")

$receiverUsername = $receiver->username;

// Look for an existing match document that already includes both usernames.

$queryFilter = [

"users.$senderUsername" => ['$exists' => true],

"users.$receiverUsername" => ['$exists' => true]

];

$query = new MongoDB\Driver\Query($queryFilter);

$cursor = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

$matchDoc = current($cursor->toArray());

$bulk = new MongoDB\Driver\BulkWrite();

if ($matchDoc) {

// Update the current user’s (swiper’s) swipe.

$updateField = "users.$senderUsername";

$bulk->update(

['_id' => $matchDoc->_id],

['$set' => [$updateField => $swipeValue]],

['multi' => false, 'upsert' => false]

);

$mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

// Re-read the document to check the swipe status.

$query = new MongoDB\Driver\Query(['_id' => $matchDoc->_id]);

$cursor = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

$updatedDoc = current($cursor->toArray());

$users = $updatedDoc->users;

$swiperSwipe = $users->$senderUsername ?? null;

$swipedSwipe = $users->$receiverUsername ?? null;

if ($swiperSwipe === "yes" && $swipedSwipe === "yes") {

// OPTIONAL: Check if a chat was already created to avoid duplicates.

if (isset($updatedDoc->chat_id)) {

// Return the already created chat details.

echo json_encode([

"status" => "match",

"chat_id" => $updatedDoc->chat_id,

"profileURL" => $updatedDoc->receiver_profileURL ?? "",

"receiver" => $receiverUsername

]);

exit;

}

// Fetch receiver information (already obtained above) and extract profile image.

$receiverId = (string)$receiver->_id;

$receiverProfileImage = "";

if (isset($receiver->profile)) {

// If profile is a string, decode it; otherwise, cast to array.

if (is_string($receiver->profile)) {

$receiverProfile = json_decode($receiver->profile, true);

} else {

$receiverProfile = (array)$receiver->profile;

}

$receiverProfileImage = $receiverProfile['images'][0] ?? ""; // use the first image

}

$senderProfileImage = "";

if (isset($sender->profile)) {

if (is_string($sender->profile)) {

$senderProfile = json_decode($sender->profile, true);

} else {

$senderProfile = (array)$sender->profile;

}

$senderProfileImage = $senderProfile['images'][0] ?? "";

}

// Create chat document.

$chatData = [

"messages" => new \stdClass(),

"participants" => [$senderId, $receiverId]

];

$bulkChat = new MongoDB\Driver\BulkWrite();

$chatId = $bulkChat->insert($chatData);

$mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkChat);

$chatIdString = (string)$chatId;

// Prepare chat metadata.

$chatMetaSender = [

"profileURL" => $receiverProfileImage,

"receiver" => $receiverUsername,

"last_message" => "",

"unread" => 0

];

$chatMetaReceiver = [

"profileURL" => $senderProfileImage,

"receiver" => $senderUsername,

"last_message" => "",

"unread" => 0

];

// Update sender's and receiver's chat lists.

$bulkUpdate = new MongoDB\Driver\BulkWrite();

$bulkUpdate->update(

["_id" => new ObjectId($senderId)],

['$set' => ["chats.$chatIdString" => $chatMetaSender]]

);

$bulkUpdate->update(

["_id" => new ObjectId($receiverId)],

['$set' => ["chats.$chatIdString" => $chatMetaReceiver]]

);

$mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUpdate);

// OPTIONAL: Update the match document with chat details to avoid duplicate creation.

$bulkMatchUpdate = new MongoDB\Driver\BulkWrite();

$bulkMatchUpdate->update(

['_id' => $updatedDoc->_id],

['$set' => [

"chat_id" => $chatIdString,

"receiver_profileURL" => $receiverProfileImage

]]

);

$mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatchUpdate);

echo json_encode([

"status" => "match",

"chat_id" => $chatIdString,

"profileURL" => $receiverProfileImage,

"receiver" => $receiverUsername

]);

} elseif ($swiperSwipe === "no" || $swipedSwipe === "no") {

echo json_encode(["status" => "no match"]);

} else {

// One swipe is still pending.

echo json_encode(["status" => "no match"]);

}

} else {

// No document exists yet; create one with both usernames.

$newDoc = [

"users" => [

$senderUsername => $swipeValue,

$receiverUsername => null

]

];

$bulk->insert($newDoc);

$mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

echo json_encode(["status" => "no match"]);

}

?>

" // rewrite to work with the id as input instead of the username


cardstach.dart:
"

// cardstack.dart

import 'dart:async';

import 'dart:convert';

import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

// Dummy placeholders for your actual imports:

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

// ^---- The file where ProfileCard is defined

import 'package:swipe_chat_play/tabs/chat/layouts/chat_popup.dart';

import 'package:swipe_chat_play/tabs/profile/layouts/profile_layout.dart';

// ^---- The new popup for sending "react_..." messages

class CardStack extends StatefulWidget {

const CardStack({Key? key}) : super(key: key);

@override

_CardStackState createState() => _CardStackState();

}

class _CardStackState extends State<CardStack> {

List<profile.ProfileData> cardData = [];

bool isLoading = true;

bool isFetchingMore = false;

bool hasMore = true;

int currentPage = 1;

final Debouncer _debouncer = Debouncer(milliseconds: 500);

// Show at most 2 cards at once (back + top).

final int maxVisibleCards = 2;

final int prefetchThreshold = 2;

// Tracks the drag offset

final ValueNotifier<Offset> _dragPositionNotifier =

ValueNotifier<Offset>(Offset.zero);

// Controller for the top card’s vertical scrolling

ScrollController _topCardScrollController = ScrollController();

// Swipe thresholds

double get swipeThreshold => MediaQuery.of(context).size.width * 0.07;

double get directionChangeThreshold =>

MediaQuery.of(context).size.width * 0.02;

// Gesture detection

Offset? _gestureStart;

bool _isHorizontalSwipe = false;

bool _decisionMade = false;

final double _slopDistance = 10;

String? _currentDirection;

double _furthestRightSoFar = 0.0;

double _furthestLeftSoFar = 0.0;

@override

void initState() {

super.initState();

_fetchProfiles(reset: true);

}

Future<void> _fetchProfiles({bool reset = false}) async {

if (isFetchingMore || (reset == false && !hasMore)) return;

setState(() => reset ? isLoading = true : isFetchingMore = true);

const storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) {

setState(() => isLoading = false);

return;

}

final int limit = reset ? 50 : 10;

try {

final response = await http.get(

Uri.parse(

'${Config.backendBaseUrl}/match_algorithm/match.php?page=$currentPage&limit=$limit'),

headers: {"Authorization": "Bearer $token"},

);

if (response.statusCode == 200) {

final data = jsonDecode(response.body);

if (data["status"] == "success") {

final profilesList = (data["data"] as List<dynamic>)

.map((json) => profile.ProfileData.fromJson(json))

.toList();

setState(() {

if (reset) {

cardData = profilesList;

currentPage = 1;

} else {

cardData.addAll(profilesList);

}

hasMore = data["has_more"] ?? false;

currentPage++;

});

}

}

} catch (e) {

debugPrint("Error fetching profiles: $e");

} finally {

setState(() => reset ? isLoading = false : isFetchingMore = false);

}

}

List<profile.ProfileData> get visibleCards =>

cardData.reversed.take(maxVisibleCards).toList().reversed.toList();

void _dismissCard(String swipeValue) {

if (cardData.isEmpty) return;

final topProfile = cardData.last;

setState(() {

cardData.removeLast();

});

// Send the swipe to server

_sendSwipe(topProfile, swipeValue);

// Reset scroll controller for new top card

_topCardScrollController.dispose();

_topCardScrollController = ScrollController();

// Prefetch more if needed

_debouncer.run(() {

if (mounted && cardData.length <= prefetchThreshold && hasMore) {

_fetchProfiles();

}

});

}

Future<void> _sendSwipe(

profile.ProfileData? profileData, String swipeValue) async {

if (profileData == null) return;

const storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) return;

final url = Uri.parse(

'${Config.backendBaseUrl}/match_algorithm/swipe.php',

);

final body = jsonEncode({

"receiver_username": profileData.topSection.name,

"swipe": swipeValue,

});

try {

final response = await http.post(

url,

headers: {

"Content-Type": "application/json",

"Authorization": "Bearer $token",

},

body: body,

);

if (response.statusCode == 200) {

final cleanedResponse =

response.body.replaceAll(RegExp(r'<br\s*/?>'), '').trim();

final responseData = jsonDecode(cleanedResponse);

final status = responseData["status"];

if (status == "match") {

debugPrint("It's a match!");

// If it's a match, do something...

} else if (status == "no match") {

debugPrint("No match");

}

} else {

debugPrint(

"Swipe endpoint error: ${response.statusCode} ${response.body}");

}

} catch (e) {

debugPrint("Error sending swipe: $e");

}

}

// -----------------------------------------------------------

// GESTURE DETECTION

// -----------------------------------------------------------

void _onPanStart(DragStartDetails details) {

_gestureStart = details.globalPosition;

_decisionMade = false;

_isHorizontalSwipe = false;

}

void _onPanUpdate(DragUpdateDetails details) {

if (_gestureStart == null) return;

final dx = details.globalPosition.dx - _gestureStart!.dx;

final dy = details.globalPosition.dy - _gestureStart!.dy;

if (!_decisionMade) {

final distance = math.sqrt(dx * dx + dy * dy);

if (distance < _slopDistance) return;

_decisionMade = true;

final angleDeg = math.atan(dy.abs() / dx.abs()) * (180 / math.pi);

_isHorizontalSwipe = (angleDeg <= 45);

}

if (_isHorizontalSwipe) {

final current = _dragPositionNotifier.value;

final newDx = current.dx + details.delta.dx;

final newDy = current.dy + details.delta.dy * 0.7;

_dragPositionNotifier.value = Offset(newDx, newDy);

_updateSwipeDirection(newDx);

} else {

if (_topCardScrollController.hasClients) {

final offsetChange = -details.delta.dy;

final currentOffset = _topCardScrollController.offset;

final maxExtent = _topCardScrollController.position.maxScrollExtent;

final newOffset = math.min(

math.max(currentOffset + offsetChange, 0.0),

maxExtent,

);

_topCardScrollController.jumpTo(newOffset);

}

}

}

void _updateSwipeDirection(double currentDx) {

if (_currentDirection == null) {

if (currentDx > 0) {

_currentDirection = "right";

_furthestRightSoFar = currentDx;

_furthestLeftSoFar = 0.0;

} else {

_currentDirection = "left";

_furthestLeftSoFar = currentDx;

_furthestRightSoFar = 0.0;

}

return;

}

if (_currentDirection == "right") {

if (currentDx > _furthestRightSoFar) {

_furthestRightSoFar = currentDx;

}

if (currentDx < 0 &&

currentDx < (_furthestRightSoFar - directionChangeThreshold)) {

_currentDirection = "left";

_furthestLeftSoFar = currentDx;

}

} else if (_currentDirection == "left") {

if (currentDx < _furthestLeftSoFar) {

_furthestLeftSoFar = currentDx;

}

if (currentDx > 0 &&

currentDx > (_furthestLeftSoFar + directionChangeThreshold)) {

_currentDirection = "right";

_furthestRightSoFar = currentDx;

}

}

}

void _onPanEnd(DragEndDetails details) {

if (_isHorizontalSwipe && cardData.isNotEmpty) {

final offset = _dragPositionNotifier.value;

final dx = offset.dx;

if (dx.abs() > swipeThreshold) {

final direction = (dx > 0) ? "yes" : "no";

_dismissCard(direction);

}

} else {

if (_topCardScrollController.hasClients) {

final velocityY = details.velocity.pixelsPerSecond.dy;

final currentOffset = _topCardScrollController.offset;

final maxExtent = _topCardScrollController.position.maxScrollExtent;

double targetOffset = currentOffset - velocityY * 0.2;

targetOffset = math.max(0.0, math.min(targetOffset, maxExtent));

_topCardScrollController.animateTo(

targetOffset,

duration: const Duration(milliseconds: 500),

curve: Curves.easeOut,

);

}

}

_dragPositionNotifier.value = Offset.zero;

_gestureStart = null;

_isHorizontalSwipe = false;

_decisionMade = false;

_currentDirection = null;

}

@override

void dispose() {

_dragPositionNotifier.dispose();

_topCardScrollController.dispose();

_debouncer.cancel();

super.dispose();

}

// This method is called whenever a child element is tapped and wants to open reaction chat

void _openReactionChat(Map<String, dynamic> reactionPayload) {

if (!reactionPayload.containsKey("profile_username")) {

debugPrint("Missing profile_username in reactionPayload");

} else {

debugPrint("Profile username: ${reactionPayload["profile_username"]}");

}

showDialog(

context: context,

barrierDismissible: true,

builder: (context) {

return ChatPopupDialog(

reactionPayload: reactionPayload,

);

},

);

}

@override

Widget build(BuildContext context) {

if (!isLoading && cardData.isEmpty) {

return Container(

color: AppColors.background,

child: Center(

child: Text(

"No profiles left, change filters",

style: AppFonts.secondaryFont,

),

),

);

}

if (isLoading) {

return Container(

color: AppColors.background,

child: Center(

child: Column(

mainAxisSize: MainAxisSize.min,

children: [

SizedBox(width: 200, child: LinearProgressIndicator()),

const SizedBox(height: 8),

Text("Loading profiles", style: AppFonts.tertiaryFont),

],

),

),

);

}

return Container(

color: AppColors.background,

child: SafeArea(

child: Stack(

children: [

// BACK CARD

if (visibleCards.length > 1)

Positioned.fill(

child: AnimatedScale(

duration: const Duration(milliseconds: 200),

scale: 0.95,

child: ProfileCard(

data: visibleCards.first,

scrollController: ScrollController(),

onElementTap: _openReactionChat, // <--- Key line

),

),

),

// TOP CARD

if (visibleCards.isNotEmpty)

Positioned.fill(

child: GestureDetector(

behavior: HitTestBehavior.translucent,

onPanStart: _onPanStart,

onPanUpdate: _onPanUpdate,

onPanEnd: _onPanEnd,

child: ValueListenableBuilder<Offset>(

valueListenable: _dragPositionNotifier,

builder: (context, dragOffset, child) {

final dx = dragOffset.dx;

final factor =

(dx.abs() / swipeThreshold).clamp(0.0, 1.0);

final opacity = 1.0 - 0.2 * factor;

return Opacity(

opacity: opacity,

child: Transform.translate(

offset: Offset(dx, dragOffset.dy),

child: AnimatedSwitcher(

duration: const Duration(milliseconds: 300),

transitionBuilder: (child, animation) {

return ScaleTransition(

scale: Tween<double>(begin: 0.95, end: 1.0)

.animate(

CurvedAnimation(

parent: animation,

curve: Curves.easeOutBack,

),

),

child: child,

);

},

child: RepaintBoundary(

key: ValueKey(visibleCards.last),

child: ProfileCard(

data: visibleCards.last,

scrollController: _topCardScrollController,

onElementTap:

_openReactionChat, // <--- Key line

),

),

),

),

);

},

),

),

),

// If fetching more

if (isFetchingMore)

const Positioned(

bottom: 20,

left: 0,

right: 0,

child: Center(

child: Column(

mainAxisSize: MainAxisSize.min,

children: [

CircularProgressIndicator(),

SizedBox(height: 8),

Text("New profiles loading"),

],

),

),

),

],

),

),

);

}

}

/// Simple Debouncer

class Debouncer {

final int milliseconds;

Timer? _timer;

Debouncer({required this.milliseconds});

void run(VoidCallback action) {

_timer?.cancel();

_timer = Timer(Duration(milliseconds: milliseconds), action);

}

void cancel() {

_timer?.cancel();

}

}

"

profile_data.dart:
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

final String username; // new field

final String age;

final List<String> infoBubbles;

TopSection({

required this.profileImages,

required this.name,

required this.username,

required this.age,

required this.infoBubbles,

});

factory TopSection.fromJson(Map<String, dynamic> json) {

return TopSection(

profileImages: List<String>.from(json['profileImages'] ?? []),

name: json['name']?.toString() ?? '',

username: json['username']?.toString() ?? '',

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

Currently the match.php script gives the name and everything but not the mongo uinque id of the profiles and the swipe.php then has to get the id by using the username. Lets streamline this:
- Rewrite match.php to give unique id aswel
- rewrite swipe.php to take the id as input instead of the username
- rewrtie CardStach and ProfileData to correctly work with these new versions of the php files correctly