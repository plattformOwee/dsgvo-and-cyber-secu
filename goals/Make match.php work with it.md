Edit this match.php:

"

<?php

/* ────────────────────────────────────────────────────────────────────────── */

/* MATCH.PHP · liefert paginierte Kandidaten im NEUEN Profil-Schema und */

/* schließt alle Profile aus, die der aufrufende User bereits bewertet hat */

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

use MongoDB\Driver\Manager;

use MongoDB\Driver\Query;

use MongoDB\Driver\Command;

/* ───────────────────── 1 · JWT → User-ID ───────────────────── */

$hdr = getallheaders();

$auth = $hdr['Authorization'] ?? $hdr['authorization'] ?? null;

if (!$auth || !preg_match('/Bearer\s+(\S+)/', $auth, $m)) {

http_response_code(401);

echo json_encode(['error' => 'Missing / invalid Authorization header']);

exit;

}

$secretKey = 'hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==';

try {

$decoded = JWT::decode($m[1], new Key($secretKey, 'HS256'));

$userId = $decoded->user_id ?? null;

if (!$userId) throw new Exception('No user_id in token');

} catch (Throwable $e) {

http_response_code(401);

echo json_encode(['error' => 'Invalid token']);

exit;

}

/* ───────────────────── 2 · Mongo – aktuelles User-Profil ──── */

$mongo = new Manager($mongoURI);

$userDoc = current(

$mongo->executeQuery(

'swipe_chat_play.users',

new Query(

['_id' => new ObjectId($userId)],

['projection' => ['profile.topSection' => 1, 'search_filter' => 1]]

)

)->toArray()

);

if (!$userDoc || !isset($userDoc->search_filter->searching_for)) {

http_response_code(400);

echo json_encode(['error' => 'User profile incomplete']);

exit;

}

/* ---------- Daten des aktuellen Users für Filter & Scoring ---------- */

$searchingFor = $userDoc->search_filter->searching_for;

$userOpenTo = $userDoc->search_filter->open_to ?? [];

$userBubbles = $userDoc->profile->topSection->infoBubbles ?? [];

$userLang = $userBubbles[1] ?? '';

$userReligion = $userBubbles[2] ?? '';

$userPolitics = $userBubbles[3] ?? '';

/* ───────────────────── 3 · Grundfilter (Gender, Age, ≠ ich) ──────────── */

$filter = ['_id' => ['$ne' => new ObjectId($userId)]];

if (!empty($searchingFor->genders)) {

/* Gender steckt in infoBubbles[0] */

$filter['profile.topSection.infoBubbles.0'] = ['$in' => $searchingFor->genders];

}

if (!empty($searchingFor->ageRange) && is_array($searchingFor->ageRange)) {

[$minAge, $maxAge] = array_map('intval', $searchingFor->ageRange);

$filter['profile.topSection.age'] = ['$gte' => $minAge, '$lte' => $maxAge];

}

/* ───────────────────── 4 · IDs der bereits geswipten Profile ──────────── */

$skipIds = [];

$swiped = $mongo->executeQuery(

'swipe_chat_play.matches',

new Query(["users.$userId" => ['$in' => ['yes', 'no']]], ['projection' => ['users' => 1]])

);

foreach ($swiped as $doc) {

foreach ((array)$doc->users as $uid => $vote) {

if ($uid !== $userId) $skipIds[$uid] = true;

}

}

if ($skipIds) {

$filter['_id']['$nin'] = array_map(fn($id) => new ObjectId($id), array_keys($skipIds));

}

/* ───────────────────── 5 · Pagination ──────────────────────────────── */

$page = max(1, (int)($_GET['page'] ?? 1));

$limit = max(1, (int)($_GET['limit'] ?? 10));

$skip = ($page - 1) * $limit;

/* total count */

$total = current(

$mongo->executeCommand(

'swipe_chat_play',

new Command(['count' => 'users', 'query' => $filter])

)->toArray()

)->n ?? 0;

/* Kandidaten holen */

$candidates = iterator_to_array(

$mongo->executeQuery(

'swipe_chat_play.users',

new Query($filter, ['skip' => $skip, 'limit' => $limit])

)

);

/* ───────────────────── 6 · Scoring ─────────────────────────── */

$wOpen = 5; $wLang = 2; $wRel = 1; $wPol = 1;

$ranked = [];

foreach ($candidates as $cand) {

$score = 0;

/* open_to */

$candOpenTo = $cand->search_filter->open_to ?? [];

$score += count(array_intersect($userOpenTo, $candOpenTo)) * $wOpen;

/* info bubbles */

$bubbles = $cand->profile->topSection->infoBubbles ?? [];

$candLang = $bubbles[1] ?? '';

$candReligion = $bubbles[2] ?? '';

$candPolitics = $bubbles[3] ?? '';

if ($candLang === $userLang) $score += $wLang;

if ($candReligion === $userReligion) $score += $wRel;

if ($candPolitics === $userPolitics) $score += $wPol;

$tmp = (array)$cand;

$tmp['matching_score'] = $score;

$ranked[] = $tmp;

}

usort($ranked, fn($a, $b) => $b['matching_score'] <=> $a['matching_score']);

/* ───────────────────── 7 · Format fürs Frontend ─────────────── */

$out = [];

foreach ($ranked as $c) {

$topObj = $c['profile']->topSection ?? new stdClass();

$eleArr = $c['profile']->elements ?? [];

/* elements als "1","2",... */

$elements = [];

foreach ($eleArr as $i => $e) {

$elements[(string)($i + 1)] = [

'type' => $e->type ?? '',

'content' => (array)($e->content ?? [])

];

}

$out[] = [

'topSection' => [

'id' => (string)$c['_id'],

'profileImages' => $topObj->profileImages ?? [],

'name' => $topObj->name ?? '',

'age' => $topObj->age ?? '',

'infoBubbles' => $topObj->infoBubbles ?? []

],

'elements' => $elements

];

}

/* ───────────────────── 8 · Response ─────────────────────────── */

echo json_encode([

'status' => 'success',

'data' => $out,

'has_more' => $total > ($skip + $limit)

], JSON_UNESCAPED_SLASHES);

?>

"

make it work with new db structure:

"