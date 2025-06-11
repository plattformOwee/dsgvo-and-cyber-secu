The sending of the bubble now works. But now lets make it functional to join the game.

You remember our tiktaktoe test with a create game button and a join game button?

Here as a refresher that old test project:

"
import 'package:flutter/material.dart';

import 'package:flutter_svg/svg.dart';

import 'package:swipe_chat_play/games/tic_tac_toe_layout.dart';

  

class GameInviteBubbleLayout extends StatelessWidget {

  final Map<String, dynamic> content;

  final bool isSender;

  final String timestamp;

  const GameInviteBubbleLayout({

    Key? key,

    required this.content,

    required this.isSender,

    required this.timestamp,

  }) : super(key: key);

  

  @override

  Widget build(BuildContext context) {

    final game = content['game'] ?? 'game';

    return Container(

      padding: const EdgeInsets.all(10),

      decoration: BoxDecoration(

        color: isSender ? Colors.blue[50] : Colors.white,

        borderRadius: BorderRadius.circular(12),

        border: Border.all(color: Colors.grey.shade300),

      ),

      child: Row(

        mainAxisSize: MainAxisSize.min,

        children: [

          Text(game, style: const TextStyle(fontWeight: FontWeight.bold)),

          const SizedBox(width: 8),

          ElevatedButton(

            onPressed: () {

              Navigator.of(context).pushNamed(

                '/tictactoe',

                arguments: content, // {game_id, game}

              );

            },

            child: const Text('Join'),

          ),

          const SizedBox(width: 8),

          Text(timestamp,

              style: const TextStyle(fontSize: 10, color: Colors.grey)),

        ],

      ),

    );

  }

}
"

createGame.php:

"

<?php

declare(strict_types=1);

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\Driver\Manager;

use MongoDB\Driver\BulkWrite;

use MongoDB\BSON\ObjectId;

use MongoDB\BSON\UTCDateTime;

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // defines $mongoURI

header('Content-Type: application/json');

ini_set('log_errors', 1);

ini_set('error_log', __DIR__ . '/logs/createGame.log');

error_reporting(E_ALL);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

// 1) Auth

$h = getallheaders();

if (!isset($h['Authorization']) || !preg_match('/Bearer\s(\S+)/', $h['Authorization'], $m)) {

http_response_code(401);

echo json_encode(['error'=>'Unauthorized']);

exit;

}

$token = $m[1];

$payload = JWT::decode($token, new Key($secretKey,'HS256'));

$senderId = $payload->user_id ?? null;

if (!$senderId) { throw new Exception('Invalid token'); }

// 2) Input

$in = json_decode(file_get_contents('php://input'), true) ?? [];

$invitee = $in['invitee_id'] ?? null;

if (!$invitee) {

http_response_code(400);

echo json_encode(['error'=>'invitee_id required']);

exit;

}

// 3) Insert game

$mgr = new Manager($mongoURI);

$bulk = new BulkWrite();

$id = new ObjectId();

$doc = [

'_id' => $id,

'player_one' => (string)$senderId,

'player_two' => (string)$invitee,

'createdAt' => new UTCDateTime()

];

$bulk->insert($doc);

$mgr->executeBulkWrite('swipe_chat_play.games', $bulk);

// 4) Respond

echo json_encode(['gameId'=>(string)$id]);

} catch (\Firebase\JWT\ExpiredException $e) {

http_response_code(401); echo json_encode(['error'=>'Token expired']);

} catch (\Firebase\JWT\SignatureInvalidException $e) {

http_response_code(401); echo json_encode(['error'=>'Invalid signature']);

} catch (\Exception $e) {

error_log("createGame error: ".$e->getMessage());

http_response_code(500);

echo json_encode(['error'=>'Server error']);

}

"

joinGame.php:

"

<?php

declare(strict_types=1);

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\Driver\Manager;

use MongoDB\BSON\ObjectId;

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // defines $mongoURI

header('Content-Type: application/json');

ini_set('log_errors', 1);

ini_set('error_log', __DIR__ . '/logs/joinGame.log');

error_reporting(E_ALL);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

// Auth

$h = getallheaders();

if (!isset($h['Authorization']) || !preg_match('/Bearer\s(\S+)/',$h['Authorization'],$m)) {

http_response_code(401); echo json_encode(['error'=>'Unauthorized']); exit;

}

$payload = JWT::decode($m[1], new Key($secretKey,'HS256'));

$userId = $payload->user_id ?? null;

if (!$userId) throw new Exception('Invalid token');

// Input

$in = json_decode(file_get_contents('php://input'), true) ?? [];

$gameId = $in['gameId'] ?? null;

if (!$gameId) { http_response_code(400); echo json_encode(['error'=>'gameId required']); exit; }

// Fetch game

$mgr = new Manager($mongoURI);

$filter = ['_id' => new ObjectId($gameId)];

$query = new MongoDB\Driver\Query($filter);

$cursor = $mgr->executeQuery('swipe_chat_play.games', $query);

$game = current($cursor->toArray());

if (!$game) { http_response_code(404); echo json_encode(['error'=>'Game not found']); exit; }

// Determine role

if ((string)$game->player_one === $userId) {

$mark = 'X';

} elseif ((string)$game->player_two === $userId) {

$mark = 'O';

} else {

http_response_code(403); echo json_encode(['error'=>'Not a participant']); exit;

}

// Return

echo json_encode([

'roomId' => $gameId,

'mark' => $mark

]);

} catch (\Exception $e) {

error_log("joinGame error: ".$e->getMessage());

http_response_code(500);

echo json_encode(['error'=>'Server error']);

}

"

Now lets integrate this functionality into the sent gameInviteBubble. The Game shall be created when the message is sent and join button should lead to that game room.

1. rewrite sendInviteMessage.php to also actually create the game like "createGame.php" does and then add the resulting game id into the response json (at "message.content.game_id)
2. write a new seperate "tiktaktoeRoom.dart" file which can be called with neccessary data. 
3. rewrite GameInviteBubbleLayout to make the "join" button open the tiktaktoeRoom with the game_id 
