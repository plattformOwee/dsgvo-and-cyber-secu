"
import 'package:flutter/material.dart';


void main() => runApp(const MyApp());

  

class MyApp extends StatelessWidget {

  const MyApp({super.key});

  

  @override

  Widget build(BuildContext context) {

    return MaterialApp(

      debugShowCheckedModeBanner: false,

      title: 'Voicememo',

      theme: ThemeData(

        useMaterial3: true,

        colorScheme: ColorScheme.fromSeed(seedColor: Colors.redAccent),

        textTheme: const TextTheme(

          headlineLarge: TextStyle(

            fontSize: 28,

            fontWeight: FontWeight.w600,

            color: Colors.black87,

          ),

          bodyMedium: TextStyle(

            fontSize: 14,

            color: Colors.black54,

          ),

        ),

      ),

      home: const VoiceMemoPage(),

    );

  }

}

  

class VoiceMemoPage extends StatelessWidget {

  const VoiceMemoPage({super.key});

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      body: SafeArea(

        minimum: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),

        child: Column(

          crossAxisAlignment: CrossAxisAlignment.start,

          children: [

            Text('Voicememo', style: Theme.of(context).textTheme.headlineLarge),

            const SizedBox(height: 4),

            const Text(

              'suggest some Topics to get\nthe conversation started',

            ),

            const SizedBox(height: 24),

  

            // Caption input

            Container(

              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),

              decoration: BoxDecoration(

                borderRadius: BorderRadius.circular(12),

                border: Border.all(color: Colors.grey.shade300, width: 1.2),

              ),

              child: Row(

                children: [

                  const Expanded(

                    child: Text(

                      'gewählte caption',

                      style: TextStyle(fontSize: 16),

                    ),

                  ),

                  IconButton(

                    icon: const Icon(Icons.edit, size: 20),

                    splashRadius: 22,

                    onPressed: () {},

                  ),

                ],

              ),

            ),

            const SizedBox(height: 32),

  

            // Waveform & playhead

            Center(

              child: SizedBox(

                height: 100,

                width: double.infinity,

                child: _Waveform(

                  barWidths: const [4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4],

                  barHeights: const [

                    24,

                    36,

                    20,

                    48,

                    32,

                    24,

                    18,

                    40,

                    34,

                    26,

                    20,

                    30

                  ],

                  spacing: 8,

                  playheadColor: Colors.grey.shade700,

                ),

              ),

            ),

            const Spacer(),

  

            // Trash + Mic buttons

            Row(

              mainAxisAlignment: MainAxisAlignment.center,

              children: [

                IconButton(

                  icon: const Icon(Icons.delete_outline),

                  iconSize: 28,

                  splashRadius: 28,

                  onPressed: () {},

                ),

                const SizedBox(width: 32),

                GestureDetector(

                  onTap: () {},

                  child: Container(

                    width: 64,

                    height: 64,

                    decoration: const BoxDecoration(

                      shape: BoxShape.circle,

                      color: Colors.redAccent,

                    ),

                    child: const Icon(Icons.mic, color: Colors.white, size: 32),

                  ),

                ),

              ],

            ),

            const SizedBox(height: 32),

  

            // Skip & Next

            Row(

              children: [

                TextButton(

                  onPressed: () {},

                  style: TextButton.styleFrom(

                    tapTargetSize: MaterialTapTargetSize.shrinkWrap,

                    minimumSize: Size.zero,

                    padding: EdgeInsets.zero,

                  ),

                  child: const Text(

                    'überspringen',

                    style: TextStyle(decoration: TextDecoration.underline),

                  ),

                ),

                const Spacer(),

                GestureDetector(

                  onTap: () {},

                  child: Container(

                    width: 48,

                    height: 48,

                    decoration: BoxDecoration(

                      shape: BoxShape.circle,

                      color: Colors.grey.shade400,

                    ),

                    child:

                        const Icon(Icons.arrow_forward_ios_rounded, size: 22),

                  ),

                ),

              ],

            ),

          ],

        ),

      ),

    );

  }

}

  

/// Simple static waveform with a central playhead

class _Waveform extends StatelessWidget {

  const _Waveform({

    required this.barWidths,

    required this.barHeights,

    required this.spacing,

    required this.playheadColor,

  });

  

  final List<double> barWidths;

  final List<double> barHeights;

  final double spacing;

  final Color playheadColor;

  

  @override

  Widget build(BuildContext context) {

    assert(barWidths.length == barHeights.length,

        'barWidths and barHeights must have the same length');

  

    return LayoutBuilder(

      builder: (context, constraints) {

        // Compute total width of bars + spacing to center waveform

        final totalBarsWidth = barWidths.reduce((a, b) => a + b) +

            spacing * (barWidths.length - 1);

  

        return Stack(

          alignment: Alignment.center,

          children: [

            Positioned(

              left: (constraints.maxWidth - totalBarsWidth) / 2,

              child: Row(

                children: List.generate(barWidths.length, (index) {

                  return Padding(

                    padding: EdgeInsets.only(

                        right: index == barWidths.length - 1 ? 0 : spacing),

                    child: Container(

                      width: barWidths[index],

                      height: barHeights[index],

                      decoration: BoxDecoration(

                        color: Colors.grey.shade600,

                        borderRadius: BorderRadius.circular(2),

                      ),

                    ),

                  );

                }),

              ),

            ),

            Container(

              width: 2,

              height: constraints.maxHeight,

              color: playheadColor,

            ),

          ],

        );

      },

    );

  }

}
"  
task1: edit the above layout such that:
1. "überspringen" button should navigate to DatingRadiusPage. 
2. next button (bottom right) should 
	1. call "http://node02.krasserserver.com:8002/swipe_chatt_play_api/update_voicememo.php" with jwt as auth bearer and as payload:
	   content: {
            prompt: 'the prompt',
            link_voicemessage: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f17f96138772.67969131_1743880080766.aac'
          }
      2. if response is "success" > navigate to DatingRadiusPage 

Task2: create update_voicememo.php:
- get id from jwt
- saves the new profile element of type "voicememo" like such into mongodb:
{
          _id: ObjectId('681ccfa0ed13220a4c0b6246'),
          type: 'voicememo',
          content: {
            prompt: 'the prompt',
            link_voicemessage: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f17f96138772.67969131_1743880080766.aac'
          }
        },"
into this structure:
"
[
  {
    _id: ObjectId('681e16a5e8126a01fc04ff56'),
    profile: {
      topSection: {
        profileImages: [
          'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/681e16dd46f02_image_cropper_1746802394567.jpg'
        ],
        name: 'luna',
        age: 23,
        infoBubbles: {
          career: 'bsbw',
          gender: 'Female',
          languages: [ 'German', 'English' ],
          politics: 'Socialist',
          religion: 'Other'
        }
      },
      elements: [
        {
          _id: ObjectId('681e16e02be764913f0e0d4f'),
          type: 'icebreaker',
          content: {
            question: 'Das verrückteste Abenteuer, das ich je erlebt habe...',
            answer: 'j'
          }
        },
        {
          _id: ObjectId('681ccfa0ed13220a4c0b6243'),
          type: 'icebreaker',
          content: {
            question: 'Ein Ort, an den ich immer wieder zurückkehre...',
            answer: 'susi'
          }
        },
        {
          _id: ObjectId('681ccfa0ed13220a4c0b6244'),
          type: 'bubbles',
          content: {
            title: 'Open to',
            bubbles: [ 'biking', 'puzzles', 'games' ]
          }
        },
        {
          _id: ObjectId('681ccfa0ed13220a4c0b6245'),
          type: 'question_and_inputfield',
          content: { question: 'was hat dich heute gefreut?' }
        },
        {
          _id: ObjectId('681ccfa0ed13220a4c0b6246'),
          type: 'voicememo',
          content: {
            prompt: 'the prompt',
            link_voicemessage: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f17f96138772.67969131_1743880080766.aac'
          }
        },
        {
          _id: ObjectId('681ccfa0ed13220a4c0b6247'),
          type: 'topic_suggestions',
          content: {
            prompt: 'the prompt',
            options: [ 'musik', 'kunst', 'geschichte' ]
          }
        },
        {
          _id: ObjectId('681ccfa0ed13220a4c0b6248'),
          type: 'image_and_prompt',
          content: {
            prompt: 'the prompt',
            image_link: 'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg'
          }
        }
      ]
    },
    search_filter: {
      location_radius: {
        location: { type: 'Point', coordinates: [ 11.5801653, 48.1235981 ] },
        radius: 90
      },
      open_to: [
        'kayaking',
        'yoga',
        'birdwatching',
        'rock climbing',
        'movies',
        'games',
        'puzzles'
      ],
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
      }
    },
    userdata: {
      email: 'luna.vogl.35@gmail.com',
      password_hash: '$2y$10$E/rpAq21ragiL7RyLxduV.1X8.7ly/otk1grNMzWO1rpRZQ2TrX7a',
      verification_code: null,
      verification_code_expiry: null,
      is_verified: true,
      fingerprint_hash: '$2y$10$38adZ8gTcN.CTzRNCbw7S.Ys0nI9W1H1ZKCmnvuMJsPqce4VP/uJO'
    },
    chats: {
      '6824c063dfcc0109a90db00c': {
        profileURL: '',
        receiver_id: '681e20c02be764913f0e0ff6',
        receiver: null,
        last_message: '(voicemessage)',
        unread: 0
      }
    }
  },
  {
    _id: ObjectId('6825078f2be764913f0e1011'),
    profile: {
      topSection: { profileImages: [], name: 'luna', age: null, infoBubbles: {} },
      elements: []
    },
    search_filter: {},
    userdata: {
      email: 'ogl@cracki.com',
      password_hash: '$2y$10$Vl602byl8L58hBVkxEVpEejlBDfRRP1PVgr3BhOoPzBxte7KbfY/q',
      verification_code: '6412',
      verification_code_expiry: ISODate('2025-05-14T21:28:51.000Z'),
      is_verified: 0,
      fingerprint_hash: null
    }
  },
  {
    _id: ObjectId('6825dc4adfcc0109a90db00d'),
    profile: {
      topSection: {
        profileImages: [
          'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6825dc83bf2a1_image_cropper_1747311734793.jpg',
          'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6825dc83bf8e1_image_cropper_1747311742714.jpg'
        ],
        name: 'luna',
        age: 23,
        infoBubbles: {
          gender: 'Female',
          languages: [ 'German', 'English' ],
          politics: 'Socialist',
          religion: 'Other'
        }
      },
      elements: [
        {
          _id: ObjectId('6825dc87e8126a01fc04ff5b'),
          type: 'icebreaker',
          content: {
            question: 'Wenn ich ein Tier wäre, dann wäre ich ein …',
            answer: 'zbab'
          }
        }
      ]
    },
    search_filter: {},
    userdata: {
      email: 'lunalala@ji5.de',
      password_hash: '$2y$10$PuYiFz.IWUKFk8ehm5R3ceXzsmLfhhyADpvgZHhSRtKbFZ16GIpYK',
      verification_code: null,
      verification_code_expiry: null,
      is_verified: true,
      fingerprint_hash: '$2y$10$0IXvrbVJvMVBXctVkjeMQuRHhvoUX5/TR8LoIm2/E2KBLcUpFTJj.'
    }
  }
]
"


Please keep the new "update_voicememo.php" consistent with my other scripts here an example:
"

<?php

/*───────────────────────────────────────────────────────────────────────────*/

/*  update_loc.php – writes user’s current location + dating radius         */

/*  NEW MODEL: search_filter.location_radius                                */

/*───────────────────────────────────────────────────────────────────────────*/

declare(strict_types=1);

  

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';          // → $mongoURI

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\{ObjectId};

use MongoDB\Driver\{Manager, BulkWrite};

  

header('Content-Type: application/json');

ini_set('display_errors',1);  error_reporting(E_ALL);

  

/*──────── 1 · allow only POST ─────────────────────────────────────────────*/

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(['error'=>'Method not allowed – use POST']); exit;

}

  

/*──────── 2 · JWT auth ────────────────────────────────────────────────────*/

$hdr = getallheaders();

if (!preg_match('/Bearer\s+(\S+)/', $hdr['Authorization'] ?? $hdr['authorization'] ?? '', $m)) {

    http_response_code(401); echo json_encode(['error'=>'Unauthorized']); exit;

}

$secret = 'hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==';

try   { $uid = JWT::decode($m[1], new Key($secret,'HS256'))->user_id ?? null; }

catch (\Throwable $e) { $uid = null; }

if (!$uid) { http_response_code(401); echo json_encode(['error'=>'Invalid token']); exit; }

  

/*──────── 3 · validate input ──────────────────────────────────────────────*/

$payload = json_decode(file_get_contents('php://input'), true);

$lat = $payload['latitude']  ?? null;

$lon = $payload['longitude'] ?? null;

$rad = $payload['dating_radius'] ?? null;

  

if ($lat===null || $lon===null || $rad===null) {

    http_response_code(400);

    echo json_encode(['error'=>'latitude, longitude and dating_radius required']); exit;

}

$lat = (float)$lat;  $lon = (float)$lon;  $rad = (float)$rad;

  

/*──────── 4 · build $set doc in NEW structure ─────────────────────────────*/

$set = [

    'search_filter.location_radius' => [

        'location'=>[

            'type'=>'Point',

            'coordinates'=>[$lon,$lat]          // GeoJSON → [lng,lat]

        ],

        'radius'=>$rad                         // km ( float )

    ]

];

  

/*──────── 5 · write to MongoDB ────────────────────────────────────────────*/

try {

    $mgr  = new Manager($mongoURI);

    $bulk = new BulkWrite();

    $bulk->update(

        ['_id'=>new ObjectId($uid)],

        ['$set'=>$set],

        ['multi'=>false,'upsert'=>true]

    );

    $mgr->executeBulkWrite('swipe_chat_play.users', $bulk);

  

    echo json_encode(['success'=>true,'message'=>'Location & radius saved']);

} catch (\Throwable $e) {

    http_response_code(500);

    echo json_encode(['error'=>'DB update failed: '.$e->getMessage()]);

}"