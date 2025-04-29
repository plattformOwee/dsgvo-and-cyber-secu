"
Future<void> _sendSpecialMessage(

      String amount, String months, String rate) async {

    final String? token = await storage.read(key: 'jwt_token');

  

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Authorization token is missing.")),

      );

      return;

    }

  

    final response = await http.post(

      Uri.parse(

          "http://node02.krasserserver.com:8002/owee/direct_lending/sendSpecialMessage.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

      body: jsonEncode({

        "chat_id": widget.chatId,

        "type": "special",

        "contract_details": {

          "amount": amount,

          "months": months,

          "rate": rate,

        },

        "timestamp": DateTime.now().toIso8601String(),

      }),

    );

  

    if (response.statusCode == 200) {

      final data = jsonDecode(response.body);

      if (data == "sent") {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text("Contract sent successfully.")),

        );

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text("Failed to send contract.")),

        );

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error: ${response.statusCode}")),

      );

    }

  }
"

"

<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php'; // MongoDB connection configuration

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

// Set error log file

$errorLogFile = __DIR__ . '/send_special_message_debug.log';

ini_set('log_errors', 1);

ini_set('error_log', $errorLogFile);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

error_log("sendSpecialMessage.php script started");

// Extract Authorization header

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? null;

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

http_response_code(401);

echo json_encode(["status" => "error", "message" => "Unauthorized"]);

exit;

}

$jwtToken = $matches[1];

$decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

$senderId = $decodedToken->user_id ?? null;

if (!$senderId) {

http_response_code(401);

echo json_encode(["status" => "error", "message" => "Invalid token"]);

exit;

}

// Get the POST data

$input = json_decode(file_get_contents('php://input'), true);

$chatId = $input['chat_id'] ?? null;

$timestamp = $input['timestamp'] ?? null;

$contractDetails = $input['contract_details'] ?? null;

error_log("Input received: " . json_encode($input));

if (!$chatId || !$timestamp || !$contractDetails) {

error_log("Missing required fields: chat_id=$chatId, timestamp=$timestamp, contract_details=" . json_encode($contractDetails));

http_response_code(400);

echo json_encode(["status" => "error", "message" => "Missing required fields"]);

exit;

}

$amount = $contractDetails['amount'] ?? null;

$months = $contractDetails['months'] ?? null;

$interestRate = $contractDetails['interest_rate'] ?? null;

if (!$amount || !$months || !$interestRate) {

error_log("Invalid contract details: " . json_encode($contractDetails));

http_response_code(400);

echo json_encode(["status" => "error", "message" => "Invalid contract details"]);

exit;

}

// Generate the text field for the message

$text = "$amount euros for $months months at an interest rate of $interestRate%";

// Construct the message JSON

$messageId = uniqid();

$message = [

"type" => "special",

"sender_id" => $senderId,

"contract_details" => $contractDetails,

"text" => $text,

"button1_label" => "Accept",

"button2_label" => "Edit",

"timestamp" => $timestamp,

"status" => "delivered"

];

// Connect to MongoDB

error_log("Connecting to MongoDB");

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

// Update the chat document

$bulk = new MongoDB\Driver\BulkWrite();

$bulk->update(

["_id" => new MongoDB\BSON\ObjectId($chatId)],

[

'$set' => [

"messages.$messageId" => $message

]

],

['upsert' => true]

);

error_log("Updating messages in chat: chat_id=$chatId, message_id=$messageId");

$result = $mongoManager->executeBulkWrite("owee_db.chats", $bulk);

if ($result->getModifiedCount() > 0) {

error_log("Message added to chat: chat_id=$chatId, message_id=$messageId");

// Update the user's last_message in their chats list

$userBulk = new MongoDB\Driver\BulkWrite();

$userBulk->update(

["chats.$chatId" => ['$exists' => true]],

[

'$set' => [

"chats.$chatId.last_message" => $text,

]

]

);

error_log("Updating last_message for chat_id=$chatId");

$userResult = $mongoManager->executeBulkWrite("owee_db.users", $userBulk);

if ($userResult->getModifiedCount() > 0) {

error_log("Last message updated for users: chat_id=$chatId");

echo json_encode("sent");

} else {

error_log("Failed to update last_message for chat_id=$chatId");

http_response_code(400);

echo json_encode(["status" => "error", "message" => "Message sent, but failed to update last_message"]);

}

} else {

error_log("Failed to add message to chat: chat_id=$chatId");

http_response_code(400);

echo json_encode(["status" => "error", "message" => "Failed to send message"]);

}

} catch (Exception $e) {

error_log("Exception occurred: " . $e->getMessage());

http_response_code(500);

echo json_encode(["status" => "error", "message" => "Internal server error: " . $e->getMessage()]);

}

error_log("sendSpecialMessage.php script ended");

?>

"

errorlog:

"