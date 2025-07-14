import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:http/http.dart' as http;
import 'package:swipe_chat_play/config.dart'; // contains backendBaseUrl

class SecurityPrivacyPage extends StatelessWidget {
  const SecurityPrivacyPage({Key? key}) : super(key: key);

  Future<String?> _token() =>
      const FlutterSecureStorage().read(key: 'jwt_token');

  void _snack(BuildContext ctx, String msg) =>
      ScaffoldMessenger.of(ctx).showSnackBar(SnackBar(content: Text(msg)));

  /// 1) Request export of all user data (Art. 15/20 DSGVO)
  Future<void> _exportData(BuildContext context) async {
    final jwt = await _token();
    if (jwt == null) return _snack(context, 'Please login first');

    final res = await http.post(
      Uri.parse('${Config.backendBaseUrl}/dsgvo/request_me.php'),
      headers: {'Authorization': 'Bearer $jwt'},
    );

    if (res.statusCode == 200) {
      final body = json.decode(res.body) as Map<String, dynamic>;
      final url = body['url'] as String?;
      if (url != null) {
        _snack(context, 'Download ready');
        showDialog(
          context: context,
          builder: (_) => AlertDialog(
            title: const Text('Your data export'),
            content: SelectableText('Download link (24 h valid):\n$url'),
            actions: [
              TextButton(
                onPressed: () {
                  Clipboard.setData(ClipboardData(text: url));
                  Navigator.pop(context);
                  _snack(context, 'Link copied to clipboard');
                },
                child: const Text('Copy'),
              ),
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('Close'),
              ),
            ],
          ),
        );
      } else {
        _snack(context, 'Unexpected response format');
      }
    } else {
      _snack(context, 'Export failed: HTTP ${res.statusCode}');
    }
  }

  /// 2) Initiate account deletion (Art. 17 DSGVO)
  Future<void> _deleteMe(BuildContext context) async {
    final jwt = await _token();
    if (jwt == null) return _snack(context, 'Please login first');

    final confirm = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Account löschen?'),
        content: const Text(
            'Dein Profil wird nach einer 30-tägigen Frist endgültig entfernt. Fortfahren?'),
        actions: [
          TextButton(
              onPressed: () => Navigator.pop(context, false),
              child: const Text('Abbrechen')),
          TextButton(
              onPressed: () => Navigator.pop(context, true),
              child: const Text('Löschen')),
        ],
      ),
    );
    if (confirm != true) return;

    final res = await http.post(
      Uri.parse('${Config.backendBaseUrl}/dsgvo/delete_me.php'),
      headers: {'Authorization': 'Bearer $jwt'},
    );

    // ← print status and body for debugging
    print('DELETE /dsgvo/delete_me.php → ${res.statusCode}');
    print('Response body: ${res.body}');

    if (res.statusCode == 202) {
      _snack(context, 'Löschung geplant – du wirst abgemeldet.');
      // e.g. Navigator.pushReplacementNamed(context, kRouteHome);
    } else {
      _snack(context, 'Delete failed: HTTP ${res.statusCode}');
    }
  }

  @override
  Widget build(BuildContext context) {
    // … your existing UI …
    return Scaffold(
      appBar: AppBar(title: const Text('Sicherheit & Datenschutz')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            ElevatedButton(
              onPressed: () => _deleteMe(context),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.redAccent,
                minimumSize: const Size.fromHeight(48),
              ),
              child: const Text('Delete Me',
                  style: TextStyle(color: Colors.white)),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => _exportData(context),
              style: ElevatedButton.styleFrom(
                minimumSize: const Size.fromHeight(48),
              ),
              child: const Text('Request My Data',
                  style: TextStyle(color: Colors.black)),
            ),
          ],
        ),
      ),
    );
  }
}"

request me:
<?php

declare(strict_types=1);

  

// ── 0) PHP error/reporting ─────────────────────────────────────

ini_set('display_errors',       '1');

ini_set('display_startup_errors','1');

ini_set('log_errors',           '1');

error_reporting(E_ALL);

  

// ── 1) Logging setup ──────────────────────────────────────────

$logDir  = dirname(__DIR__, 2) . '/logs';  // /var/www/woven/logs

if (!is_dir($logDir) && false===mkdir($logDir,0755,true)) {

    error_log("Could not create log dir: {$logDir}");

}

$logFile = $logDir . '/request_me.log';

function logDebug(array $lines): void {

    global $logFile;

    $ts = date('c');

    $out= '';

    foreach($lines as $l) {

      $out .= "[{$ts}] {$l}\n";

    }

    file_put_contents($logFile, $out, FILE_APPEND|LOCK_EX);

}

logDebug(['--- request_me.php start ---']);

  

// ── 2) Boot & headers ─────────────────────────────────────────

require_once dirname(__DIR__,2).'/bootstrap.php';  // auth(), etc.

require_once dirname(__DIR__,2).'/config.php';     // $mongoURI, $mongoDb

header('Content-Type: application/json; charset=utf-8');

header('Cache-Control: no-store');

  

// ── 3) Authenticate ──────────────────────────────────────────

try {

    logDebug(['Calling auth()']);

    $claims = auth();

    $userId = $claims->sub;

    logDebug(["Authenticated userId={$userId}"]);

} catch (Throwable $e) {

    logDebug(['Auth failed: '.$e->getMessage()]);

    http_response_code(401);

    echo json_encode(['error'=>'Unauthorized']);

    logDebug(['--- request_me.php end (401) ---']);

    exit;

}

  

try {

  // ── 4) Fetch user doc ───────────────────────────────────────

  logDebug(["Connecting to MongoDB: {$mongoURI}"]);

  $mgr = new MongoDB\Driver\Manager($mongoURI);

  $qry = new MongoDB\Driver\Query(

    ['_id'=>new MongoDB\BSON\ObjectId($userId)],

    ['projection'=>[

      'profile'=>1,'search_filter'=>1,'userdata'=>1,'chats'=>1

    ]]

  );

  $rows = $mgr->executeQuery("{$mongoDb}.users",$qry)->toArray();

  if (count($rows)===0) {

    logDebug(['User not found']);

    http_response_code(404);

    echo json_encode(['error'=>'User not found']);

    logDebug(['--- request_me.php end (404) ---']);

    exit;

  }

  $user = $rows[0];

  logDebug(['Fetched user document']);

  

  // ── 5) Build package ────────────────────────────────────────

  $package = [

    'profile'        => $user->profile        ?? new stdClass,

    'search_filter'  => $user->search_filter  ?? new stdClass,

    'userdata'       => $user->userdata       ?? new stdClass,

    'chats'          => $user->chats          ?? new stdClass,

    'exported_at'    => gmdate('c'),

    'schema_version' => 'v1',

  ];

  logDebug(['Built export package']);

  

  // ── 6) Write to public/exports ──────────────────────────────

  $exportDir = dirname(__DIR__,2).'/public/exports';

  if (!is_dir($exportDir) && false===mkdir($exportDir,0755,true)) {

    throw new \RuntimeException("Cannot create exportDir {$exportDir}");

  }

  $filename = "{$userId}_" . time() . ".json";

  $fullpath = "{$exportDir}/{$filename}";

  if (false===file_put_contents(

       $fullpath,

       json_encode($package,JSON_PRETTY_PRINT)

     )) {

    throw new \RuntimeException("Failed writing export to {$fullpath}");

  }

  logDebug(["Wrote export to: {$fullpath}"]);

  

  // ── 7) Return full HTTP(S) URL ─────────────────────────────

  $scheme = (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS']!=='off')

            ? 'https' : 'http';

  $host   = $_SERVER['HTTP_HOST'] ?? 'your-domain.tld';

  $url    = "{$scheme}://{$host}/exports/{$filename}";

  echo json_encode(['url'=> $url]);

  logDebug(["Responded with URL: {$url}"]);

  logDebug(['--- request_me.php end (200) ---']);

  exit;

  

} catch (Throwable $e) {

  logDebug(['Exception: '.$e->getMessage(), $e->getTraceAsString()]);

  http_response_code(500);

  echo json_encode(['error'=>'Server error','message'=>$e->getMessage()]);

  logDebug(['--- request_me.php end (500) ---']);

  exit;

}

delete_me.php:
"
<?php

declare(strict_types=1);

/*───────────────────────────────────────────────────────────────────────────*/

/*  DELETE_ME.PHP – soft‐flag user for deletion (Art. 17 DSGVO)              */

/*───────────────────────────────────────────────────────────────────────────*/

  

// 0 · prepare logging

$logDir  = dirname(__DIR__, 2) . '/logs';

if (!is_dir($logDir)) {

    mkdir($logDir, 0775, true);

}

$logFile = $logDir . '/delete_me.log';

function logDebug(array $lines): void {

    global $logFile;

    $ts = date('c');

    $out = '';

    foreach ($lines as $l) {

        $out .= "[$ts] $l\n";

    }

    file_put_contents($logFile, $out, FILE_APPEND | LOCK_EX);

}

logDebug(['--- delete_me.php start ---']);

  

// 1 · includes & headers

require_once __DIR__ . '/../../bootstrap.php';  // auth(), $auth, $mailer

require_once __DIR__ . '/../../config.php';     // $mongoURI, $mongoDb

header('Content-Type: application/json; charset=utf-8');

ini_set('display_errors', '1');

error_reporting(E_ALL);

  

// 2 · auth

try {

    logDebug(['Calling auth()']);

    $claims = auth();

    $userId = $claims->sub;

    logDebug(["Authenticated userId={$userId}"]);

} catch (Throwable $e) {

    logDebug(['Auth failed: ' . $e->getMessage()]);

    http_response_code(401);

    echo json_encode(['error' => 'Unauthorized']);

    logDebug(['--- delete_me.php end (401) ---']);

    exit;

}

  

// 3 · perform soft‐delete

try {

    logDebug(["Connecting to MongoDB: {$mongoURI}"]);

    $mgr = new MongoDB\Driver\Manager($mongoURI);

  

    logDebug(["Preparing BulkWrite for deletion_requested_at"]);

    $bulk = new MongoDB\Driver\BulkWrite;

    $bulk->update(

        ['_id' => new MongoDB\BSON\ObjectId($userId)],

        ['$set' => ['deletion_requested_at' => new MongoDB\BSON\UTCDateTime()]],

        ['multi' => false, 'upsert' => false]

    );

  

    $result = $mgr->executeBulkWrite("{$mongoDb}.users", $bulk);

    logDebug([

        "BulkWrite done: matched={$result->getMatchedCount()}",

        "modified={$result->getModifiedCount()}"

    ]);

  

    // revoke tokens

    try {

        logDebug(['Revoking tokens']);

        $auth->revokeTokens($userId);

        logDebug(['Tokens revoked']);

    } catch (Throwable $e) {

        logDebug(['Token revoke error: ' . $e->getMessage()]);

    }

  

    // goodbye mail

    try {

        logDebug(['Sending goodbye mail']);

        $mailer->sendGoodbyeMail($userId);

        logDebug(['Goodbye mail sent']);

    } catch (Throwable $e) {

        logDebug(['Goodbye mail error: ' . $e->getMessage()]);

    }

  

    http_response_code(202);

    echo json_encode(['status' => 'scheduled']);

    logDebug(['Response: 202 scheduled']);

} catch (Throwable $e) {

    logDebug(['Exception during delete: ' . $e->getMessage(), $e->getTraceAsString()]);

    http_response_code(500);

    echo json_encode(['error' => 'Server error']);

}

  

logDebug(['--- delete_me.php end ---']);
"


"
{
  _id: ObjectId('6871538a67c6808f860bbb05'),
  profile: {
    topSection: {
      firstName: 'luna',
      lastName: 'vogl',
      name: 'luna vogl',
      infoBubbles: {
        gender: 'Female',
        languages: [ 'German' ],
        religion: 'Atheist',
        religion_show: true,
        politics: 'Socialist'
      },
      age: 0,
      profileImages: [
        'https://v32582.1blu.de/new_signup/uploads/68715405f2e735.13298564_image_cropper_1752257530184.jpg',
        'https://v32582.1blu.de/new_signup/uploads/68715405f2ec79.92166007_image_cropper_1752257533455.jpg',
        'https://v32582.1blu.de/new_signup/uploads/68715405f2ed95.61534512_image_cropper_1752257537642.jpg'
      ]
    },
    elements: [
      {
        _id: ObjectId('6871540afd33ab3ed40abc1f'),
        type: 'question_and_inputfield',
        content: {
          question: "What's a hobby you could do all day?",
          answer: ''
        }
      },
      {
        _id: ObjectId('6871541167c6808f860bbb06'),
        type: 'icebreaker',
        content: {
          question: 'What’s your go-to comfort food on a bad day?',
          answer: 'bsbwb'
        }
      },
      {
        _id: ObjectId('68715417fd33ab3ed40abc21'),
        type: 'voicememo',
        content: {
          prompt: 'Tell me about your favorite movie scene.',
          link_voicemessage: 'http://v32582.1blu.de/uploads/voicememos/68715417fd33ab3ed40abc20.aac'
        }
      },
      {
        _id: ObjectId('6871541a67c6808f860bbb07'),
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
      location: { type: 'Point', coordinates: [ 11.580298, 48.1235543 ] },
      radius: 72
    }
  },
  userdata: {
    email: 'luna.vogl.35@gmail.com',
    password_hash: '$2y$10$UFotiorktiZRtaKAJKkoXu6qjvb1OlJ8XSJhYtOcp40EA0IyHMKeC',
    verification_code: null,
    verification_code_expiry: null,
    is_verified: true,
    fingerprint_hash: null,
    device_auth: {
      devices: [ '69870507-8da9-42c3-a2bd-053a825fa214' ],
      enabled: true,
      hardware_ids: [ 'TP1A.220624.014' ],
      method: 'biometricOrPin',
      since: ISODate('2025-07-11T18:10:26.095Z'),
      last_login: ISODate('2025-07-14T14:32:50.570Z')
    },
    consents: {
      location: { granted: true, timestamp: ISODate('2025-07-11T18:11:10.106Z') },
      microphone: { granted: true, timestamp: ISODate('2025-07-11T18:11:10.106Z') },
      notifications: {
        granted: false,
        timestamp: ISODate('2025-07-11T18:11:10.106Z')
      },
      newsletter: { granted: true, timestamp: ISODate('2025-07-11T18:11:10.106Z') },
      profile_sensitives: { granted: true, timestamp: ISODate('2025-07-11T18:11:10.106Z') }
    }
  },
  chats: {}
}"
"

"

before each of the two actions (delete Me) and (request my data) please ask for device- auth/fingerprint or password and integrate this request into the two backend scripts provided above.
So first the frontend script should check what methods the user provided in backend with one "method.php" script (which you write with code consistency with other backend files of my woven project (one of which you see above) ) which checks, which methods user has given in his backend and then echos that > the frontend checks if this method is currently available on the phone, and if yes, asks for it and with given fingerprint or pattern/code or if both are not given or not available on the device, the password > call the request me / delete me script > the scripts respectivly check the password/device auth/ fingerprint and if correct, echo the link to the data or statement of succesfull deletion:

- delete me was called with correct credentials (device auth / fingerprint / password from email and password in userdata) > echo success message + date of deletion > frontend should save this date in flutter secure storage and start displaying a "you profile will be delted in x day" text with the x being the number of days, calculated from a date in secure storage.

- request me script was called with correct credentials > echo the link to the userdata as is already done