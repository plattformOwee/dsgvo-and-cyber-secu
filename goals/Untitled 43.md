Mission: Now lets actually connect the react_to_profile functionality to my backend.


Definitions:
swiper = the user who is currently in the app and swiping

swiped = the user whos profile is being seen by swiper and either swiped or commented or whatever 


The intended user experience is:

swiper is commenting on an element on swiped's profile > the comment gets sent and the chat popup window then closes (behind it, the next profile)

What must happen to achieve that:
We sort of need a combination of swipe.php and sendMessage.php so the message is both saved and new chat created but is also taken as a swipe to the right.

When user comments on profile > react_profile.php is called which will check if its a match or not (like swipe.php) 

if match > 1. look for existing chat in requested_chats and if there is one, take that and move to chats or if none exist yes then create this chat normally(like in swipe.php). (with this first message as first message)

if no match > create a chat with same structure but in new collection called "requested_chats"  [that also means, that when a match occurs, we must check if there is an existing requeste chat to acticate (bring over to chats collection)]


references:

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

    "users.$senderUsername"   => ['$exists' => true],

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

            $senderUsername   => $swipeValue,

            $receiverUsername => null

        ]

    ];

    $bulk->insert($newDoc);

    $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

    echo json_encode(["status" => "no match"]);

}

?>
"

sendMessage.php
"
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // MongoDB connection configuration

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set error log file

$errorLogFile = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/send_message_debug.log';

ini_set('log_errors', 1);

ini_set('error_log', $errorLogFile);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    error_log("sendMessage.php script started");

  

    // 1) Check Authorization header (Bearer token)

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        http_response_code(401);

        echo json_encode(["status" => "error", "message" => "Unauthorized"]);

        exit;

    }

  

    $jwtToken  = $matches[1];

    $decoded   = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $senderId  = $decoded->user_id ?? null;

  

    if (!$senderId) {

        http_response_code(401);

        echo json_encode(["status" => "error", "message" => "Invalid token"]);

        exit;

    }

  

    // 2) Parse input JSON

    $input     = json_decode(file_get_contents('php://input'), true);

    $chatId    = $input['chat_id'] ?? null;

    $timestamp = $input['timestamp'] ?? null;

    // 'type' can be "text" or "react_bubble", etc. default to "text"

    $type      = $input['type'] ?? 'text';

    // For "text" type we read 'message', else we read 'content'

    $message   = $input['message']  ?? null;

    $content   = $input['content']  ?? [];

  

    error_log("Input received: " . json_encode($input));

  

    if (!$chatId || !$timestamp) {

        error_log("Missing required fields: chat_id=$chatId, timestamp=$timestamp");

        http_response_code(400);

        echo json_encode("Missing required fields");

        exit;

    }

    // If it's text type, we need a message

    if ($type === 'text' && !$message) {

        error_log("Missing 'message' for text type");

        http_response_code(400);

        echo json_encode("Missing message for text type");

        exit;

    }

  

    // 3) Connect to MongoDB

    error_log("Connecting to MongoDB");

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // 4) Build the new message data

    $messageId = uniqid();

    $newMessage = [

        "sender_id"  => $senderId,

        "type"       => $type,

        "timestamp"  => $timestamp,

        "status"     => "delivered"

    ];

  

    // If it's text => store "message" => $message

    // If it's not text => store "content" => $content

    if ($type === 'text') {

        $newMessage["message"] = $message;

    } else {

        $newMessage["content"] = $content;

    }

  

    // 5) Upsert the message into the chat doc

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update(

        [ "_id" => new MongoDB\BSON\ObjectId($chatId) ],

        [ '$set' => [ "messages.$messageId" => $newMessage ] ],

        [ 'upsert' => true ]

    );

  

    error_log("Updating messages in chat: chat_id=$chatId, message_id=$messageId");

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk);

  

    // 6) If the chat doc was updated, update 'last_message' for all participants

    if ($result->getModifiedCount() > 0 || $result->getUpsertedCount() > 0) {

        error_log("Message added to chat: chat_id=$chatId, message_id=$messageId");

  

        // Build a 'last_message' snippet

        // If type=text => last_message = $message

        // else => last_message = "($type)" or you can do something else

        $lastMessage = ($type === 'text') ? $message : "($type)";

  

        $userBulk = new MongoDB\Driver\BulkWrite();

        // Update all user docs that have "chats.chatId"

        $userBulk->update(

            [ "chats.$chatId" => [ '$exists' => true ] ],

            [ '$set' => [ "chats.$chatId.last_message" => $lastMessage ] ],

            [ 'multi' => true ]

        );

        error_log("Updating last_message for all participants in chat_id=$chatId");

  

        $userResult = $mongoManager->executeBulkWrite("swipe_chat_play.users", $userBulk);

  

        if ($userResult->getModifiedCount() > 0) {

            error_log("Last message updated for chat_id=$chatId in user docs");

            echo json_encode("sent");

        } else {

            error_log("Message sent, but no user docs updated for last_message");

            http_response_code(400);

            echo json_encode("Message sent, but failed to update last_message");

        }

    } else {

        error_log("Failed to add message to chat: chat_id=$chatId");

        http_response_code(400);

        echo json_encode("Failed to send message");

    }

} catch (Exception $e) {

    error_log("Exception occurred: " . $e->getMessage());

    http_response_code(500);

    echo json_encode("Internal server error: " . $e->getMessage());

}

  

error_log("sendMessage.php script ended");

?>
"