#  featurea planen
## chat feature planen
{
	"chats":{
		"chat_id(`_id` mongo)":{
			"mongo_unique_id_message":{
				"sender_id":"user1_id",
				"message":"hey how are you?",
				"timestamp":"2024-12-28T12:45:00.000Z",
				"status": "delivered",
			},
			"mongo_unique_id_message":{
				"sender_id":"user2_id",
				"message":"im doing great and you?",
				"timestamp":"2024-12-28T12:45:00.000Z",
				"status": "delivered",
			},
			"mongo_unique_id_message":{
				"sender_id":"user1_id",
				"message":"im fine, thanks for asking. want to go out?",
				"timestamp":"2024-12-28T12:45:00.000Z",
				"status": "delivered",
			}
			"mongo_unique_id_message":{
				"sender_id":"user1_id",
				"message":"im fine, thanks for asking. want to go out?",
				"timestamp":"2024-12-28T12:45:00.000Z",
				"status": "delivered",
			}
		}
	}
}

{
	"all other userdata":"example"
	"chats":{
		"chat_id(`_id` mongo)":{
			"profileURL":"urlofprofilepicture.com"
			"receiver":"receiverUsername"
			"last_message":"the last message sent in chat"
			"unread":"number of new messages"
		},
		"chat_id(`_id` mongo)":{
			"profileURL":"urlofprofilepicture.com"
			"receiver":"receiverUsername"
			"last_message":"the last message sent in chat"
			"unread":"number of new messages"
		},
		"chat_id(`_id` mongo)":{
			"profileURL":"urlofprofilepicture.com"
			"receiver":"receiverUsername"
			"last_message":"the last message sent in chat"
			"unread":"number of new messages"
		},
	}
}

- fetch_chats.php
	- get sender id from jwt
	- get the chat_id`s
	- get all messages (later should be only new messages)
	- return all messages of each chat to app:
		{
			"chats":{
			"chat_id(`_id` mongo)":{
				"profileURL":"urlofprofilepicture.com"
				"receiver":"receiverUsername"
				"last_message":"the last message sent in chat"
				"unread":"number of new messages"
				"messages":{
					""
				}
			},
			"chat_id(`_id` mongo)":{
				"profileURL":"urlofprofilepicture.com"
				"receiver":"receiverUsername"
				"last_message":"the last message sent in chat"
				"unread":"number of new messages"
			},
			"chat_id(`_id` mongo)":{
				"profileURL":"urlofprofilepicture.com"
				"receiver":"receiverUsername"
				"last_message":"the last message sent in chat"
				"unread":"number of new messages"
			},
		}

- sendMessage.php
	- get sender id from jwt
	- get chat_id from payload
	- check if sender is participand in that chat
	- save new message into that chat
		- on fail > try again (5 times)
		- success > send newest message to user (websocket) & return "sent"
			- fail > send pushnote
			- success > return "sent"


- my_chats.dart
	- get my_chats.json from securestorage
	- display all chats as list with: profilepicture on the right, amount of new messages on the left and last message under name just to the left of the profilepicture
	- if chat is clicked:
		- open chat.dart give into, the json of this chat

- chat.dart
	- sort messages into dates make new json
	- display the messages in bubbles left right which is layout file
	- inputfield> opens keyboard

- PUT mark message as read

- my_chats.json should be fetched together with all other data, when user logges in successfully. also the newest 30 messages of each chat.

## profile filter logic

I want to create a simple matching algorithm php script.
payload:
"userdata":{
	"id":"some id"
	"attributes":{
		"height":"175"
		"age":"24"
		"seek":"relationship" // relationship / fun / friends
	},
	"searching_for":{
		"height":{"min":"160", "max":"220"}
		"age":{"min":"20", "max":"35"}
		"should_seek":"relationship"
	}
}

1. extract userdata(searching for)
2. make query to mongo_db to get all users that fit these criteria
3. return list of all fitting profiles



https://stadt.muenchen.de/dam/Home/Stadtverwaltung/Kreisverwaltungsreferat/fachspezifisch/HA-III/Dokumente/Fuehrerschein/Aenderungsantrag.pdf

https://stadt.muenchen.de/dam/jcr:5dfdcf99-1a84-47f5-96c4-43e23415ee40/Fahrerlaubnis%20erstmalig.pdf

erweiterrung: https://stadt.muenchen.de/dam/jcr:b3e77888-686d-4ffc-98f0-899702859a85/Fahrerlaubnis%20erweitern.pdf

17: https://stadt.muenchen.de/dam/jcr:c572e153-99ca-4b46-832b-9a0b28afc952/Fuehrerschein%20mit%2017.pdf

neu erteilung:
https://stadt.muenchen.de/dam/jcr:97d79a3a-03aa-44fe-a962-64b99f9029ac/Neuerteilung%20einer%20Fahrerlaubnis.pdf


## profile logic

For my Dating app profile page i want to be able to fleixbly control the layout for each person (order and content).
To make that possible, the entire layout should be driven by one json like:

## todos idk
vorgehen:
- [x] php script to create example chat with id 1
- [x] rewrite chat.dart to fetch from there and save in securestorage then display
- [x] make sendMessage.php which saves new message in mongodb
- [x] make it work with jwt 
- [x] when creating a new chat, save it into users data 
	- [ ] update last message when send message.php
	- [ ] update number of unread messages also
- [x] change myChats.php which loads all the chats from userdata
- [x] sendmassage also save newest message into user data
- [x] make examplechat create get username of receiver later
- [ ] get two devices to test
	- [ ] make receiver a real receiver
- [ ] löschen, zurückrufen(platzhalter bleibt übrig), kopieren
- [ ] show latest message also without reloading (get it local from chat and only on new open, get from cloud)
- [ ] make sendMessage.php which sends message using websockets after message has been saved
	- [ ] write php script for it 
	- [ ] rewrite chat.dart to append the incomming message to the chats json in secureStorage
- [ ] wenn man mit selben phone neuen acc macht wird noch altes profiel von securestorage gelesen das muss geändert werden.

# create profile
Von dem was es gibt weiter arbeiten.

My Goal is, to create a "create_profile.dart" and "my_profile.dart" aswell as "save_profile.php" "fetch_profile.php".

They should look the same but in "create_profile.dart" every element will be editable by clicking on it and have an x on the top left corner (overlayed) to delete the element.

Lets start by building the "my_profile.dart" page before moving on and making it editable.

In every step, i want the layout to be read from and written to a my_profile.json (saved locally) of the following format:

{
	"profile_picture_key":"path_to_storage"
	"username":"example"
	"info_bubbles":{"x years old","relationship","etc"}
	"first_segment":"example text long"
	"aditional elements":{
		"1":{
			"type":"text/image"
			"content":"keystore/exampletext"
		},
		"2":{
			"type":"text/image"
			"content":"keystore/exampletext"
		}
		"3":{
			"type":"text/image"
			"content":"keystore/exampletext"
		}
	}
}


give me dart code which takes as input the following json:

"

{

"profile_picture_key":"path_to_storage"

"username":"example"

"info_bubbles":{"x years old","relationship","etc"}

"first_segment":"example text long"

"aditional elements":{

"1":{

"type":"text/image"

"content":"path_to_storage/exampletext"

},

"2":{

"type":"text/image"

"content":"path_to_storage/exampletext"

}

"3":{

"type":"text/image"

"content":"path_to_storage/exampletext"

}

}

}

" (path_to_storage is a path to where the image is saved on the phone use "key":"value" type of storage.)

# work with chatgpt

" <?php require __DIR__ . '/../../vendor/autoload.php'; // Adjust path to Composer autoload require_once 'config.php'; // Include configuration for MongoDB connection use Firebase\JWT\JWT; use Firebase\JWT\Key; header('Content-Type: application/json'); ini_set('display_errors', 1); ini_set('display_startup_errors', 1); error_reporting(E_ALL); // Set the error log file path $error_log_file = __DIR__ . '/create_chat_log.log'; ini_set('log_errors', 1); ini_set('error_log', $error_log_file); $secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA=="; try { // Check for Authorization header $headers = getallheaders(); $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null; if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) { http_response_code(401); echo json_encode(["error" => "Unauthorized"]); exit; } $jwtToken = $matches[1]; // Decode the JWT token to get sender ID $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256')); $senderId = $decodedToken->user_id ?? null; if (!$senderId) { http_response_code(401); echo json_encode(["error" => "Invalid token"]); exit; } // Read the receiver ID from the input $input = json_decode(file_get_contents('php://input'), true); $receiverId = $input['receiver_id'] ?? null; if (!$receiverId) { http_response_code(400); echo json_encode(["error" => "Receiver ID is required"]); exit; } // Connect to MongoDB $mongoManager = new MongoDB\Driver\Manager($mongoURI); // Create the example chat data $exampleChat = [ "messages" => [ uniqid() => [ "sender_id" => $senderId, "message" => "hey how are you?", "timestamp" => "2024-12-28T12:45:00.000Z", "status" => "delivered", ], uniqid() => [ "sender_id" => $receiverId, "message" => "I'm doing great and you?", "timestamp" => "2024-12-28T12:46:00.000Z", "status" => "delivered", ], uniqid() => [ "sender_id" => $senderId, "message" => "I'm fine, thanks for asking. Want to go out?", "timestamp" => "2024-12-28T12:47:00.000Z", "status" => "delivered", ], ] ]; // Insert the chat into MongoDB $bulk = new MongoDB\Driver\BulkWrite(); $chatId = $bulk->insert($exampleChat); $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk); echo json_encode([ "status" => "success", "chat_id" => (string)$chatId, // Convert ObjectId to string ]); } catch (Exception $e) { error_log("Error: " . $e->getMessage()); http_response_code(500); echo json_encode([ "status" => "error", "message" => "Internal server error", ]); } " additionally to creating the chat in db chats, i also want the above script, to save the newly created chat in a list of chats in the users data. by adding "chats" to userdata and then the chats relevant data into that: " "chats":{ "chat_id(`_id` mongo)":{ "profileURL":"urlofprofilepicture.com" "receiver":"receiverUsername" "last_message":"the last message sent in chat" "unread":"number of new messages" }, "chat_id(`_id` mongo)":{ "profileURL":"urlofprofilepicture.com" "receiver":"receiverUsername" "last_message":"the last message sent in chat" "unread":"number of new messages" }, "chat_id(`_id` mongo)":{ "profileURL":"urlofprofilepicture.com" "receiver":"receiverUsername" "last_message":"the last message sent in chat" "unread":"number of new messages" }, }" this is what my users db currently looks like: " { _id: ObjectId('67894f0367ef03835b05b15c'), username: 'zach', firstname: 'example', lastname: 'mample', email: 'zach.vogl.35@gmail.com', password_hash: '$2y$10$14oCpnbTBZP8SrJVZiioN.62O/IPau.nVXcNJXKMi9VODNcWcpK62', verification_code: null, verification_code_expiry: null, is_verified: true, fingerprint_hash: '$2y$10$L5FEOMYYAX66J/6PQ0A6U.Mc13FcgbKZZMhX6kDR0ThqHJeFR4ZWC' } " this is and example of what it should look like after user creates a new chat: " { _id: ObjectId('67894f0367ef03835b05b15c'), username: 'zach', firstname: 'example', lastname: 'mample', email: 'zach.vogl.35@gmail.com', password_hash: '$2y$10$14oCpnbTBZP8SrJVZiioN.62O/IPau.nVXcNJXKMi9VODNcWcpK62', verification_code: null, verification_code_expiry: null, is_verified: true, fingerprint_hash: '$2y$10$L5FEOMYYAX66J/6PQ0A6U.Mc13FcgbKZZMhX6kDR0ThqHJeFR4ZWC' "chats":{ "chat_id(`_id` mongo)":{ "mongo_unique_id_message":{ "sender_id":"user1_id", "message":"hey how are you?", "timestamp":"2024-12-28T12:45:00.000Z", "status": "delivered", }, "mongo_unique_id_message":{ "sender_id":"user2_id", "message":"im doing great and you?", "timestamp":"2024-12-28T12:45:00.000Z", "status": "delivered", }, "mongo_unique_id_message":{ "sender_id":"user1_id", "message":"im fine, thanks for asking. want to go out?", "timestamp":"2024-12-28T12:45:00.000Z", "status": "delivered", } }, } }"


" <?php require __DIR__ . '/../../vendor/autoload.php'; // Adjust path to Composer autoload require_once 'config.php'; // Include configuration for MongoDB connection use Firebase\JWT\JWT; use Firebase\JWT\Key; header('Content-Type: application/json'); ini_set('display_errors', 1); ini_set('display_startup_errors', 1); error_reporting(E_ALL); // Set the error log file path $error_log_file = __DIR__ . '/create_chat_log.log'; ini_set('log_errors', 1); ini_set('error_log', $error_log_file); $secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA=="; try { // Check for Authorization header $headers = getallheaders(); $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null; if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) { http_response_code(401); echo json_encode(["error" => "Unauthorized"]); exit; } $jwtToken = $matches[1]; // Decode the JWT token to get sender ID $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256')); $senderId = $decodedToken->user_id ?? null; if (!$senderId) { http_response_code(401); echo json_encode(["error" => "Invalid token"]); exit; } // Read the receiver ID from the input $input = json_decode(file_get_contents('php://input'), true); $receiverId = $input['receiver_id'] ?? null; if (!$receiverId) { http_response_code(400); echo json_encode(["error" => "Receiver ID is required"]); exit; } // Connect to MongoDB $mongoManager = new MongoDB\Driver\Manager($mongoURI); // Create the example chat data $exampleChat = [ "messages" => [ uniqid() => [ "sender_id" => $senderId, "message" => "hey how are you?", "timestamp" => "2024-12-28T12:45:00.000Z", "status" => "delivered", ], uniqid() => [ "sender_id" => $receiverId, "message" => "I'm doing great and you?", "timestamp" => "2024-12-28T12:46:00.000Z", "status" => "delivered", ], uniqid() => [ "sender_id" => $senderId, "message" => "I'm fine, thanks for asking. Want to go out?", "timestamp" => "2024-12-28T12:47:00.000Z", "status" => "delivered", ], ] ]; // Insert the chat into MongoDB $bulk = new MongoDB\Driver\BulkWrite(); $chatId = $bulk->insert($exampleChat); $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk); echo json_encode([ "status" => "success", "chat_id" => (string)$chatId, // Convert ObjectId to string ]); } catch (Exception $e) { error_log("Error: " . $e->getMessage()); http_response_code(500); echo json_encode([ "status" => "error", "message" => "Internal server error", ]); } " additionally to creating the chat in db chats, i also want the above script, to save the newly created chat in a list of chats in the users data. by adding "chats" to userdata and then the chats relevant data into that: " "chats":{ "chat_id(`_id` mongo)":{ "profileURL":"urlofprofilepicture.com" "receiver":"receiverUsername" "last_message":"the last message sent in chat" "unread":"number of new messages" }, "chat_id(`_id` mongo)":{ "profileURL":"urlofprofilepicture.com" "receiver":"receiverUsername" "last_message":"the last message sent in chat" "unread":"number of new messages" }, "chat_id(`_id` mongo)":{ "profileURL":"urlofprofilepicture.com" "receiver":"receiverUsername" "last_message":"the last message sent in chat" "unread":"number of new messages" }, }" this is what my users db currently looks like: " { _id: ObjectId('67894f0367ef03835b05b15c'), username: 'zach', firstname: 'example', lastname: 'mample', email: 'zach.vogl.35@gmail.com', password_hash: '$2y$10$14oCpnbTBZP8SrJVZiioN.62O/IPau.nVXcNJXKMi9VODNcWcpK62', verification_code: null, verification_code_expiry: null, is_verified: true, fingerprint_hash: '$2y$10$L5FEOMYYAX66J/6PQ0A6U.Mc13FcgbKZZMhX6kDR0ThqHJeFR4ZWC' } " this is and example of what it should look like after user creates a new chat: " { _id: ObjectId('67894f0367ef03835b05b15c'), username: 'zach', firstname: 'example', lastname: 'mample', email: 'zach.vogl.35@gmail.com', password_hash: '$2y$10$14oCpnbTBZP8SrJVZiioN.62O/IPau.nVXcNJXKMi9VODNcWcpK62', verification_code: null, verification_code_expiry: null, is_verified: true, fingerprint_hash: '$2y$10$L5FEOMYYAX66J/6PQ0A6U.Mc13FcgbKZZMhX6kDR0ThqHJeFR4ZWC' "chats":{ "chat_id(`_id` mongo)":{ "mongo_unique_id_message":{ "sender_id":"user1_id", "message":"hey how are you?", "timestamp":"2024-12-28T12:45:00.000Z", "status": "delivered", }, "mongo_unique_id_message":{ "sender_id":"user2_id", "message":"im doing great and you?", "timestamp":"2024-12-28T12:45:00.000Z", "status": "delivered", }, "mongo_unique_id_message":{ "sender_id":"user1_id", "message":"im fine, thanks for asking. want to go out?", "timestamp":"2024-12-28T12:45:00.000Z", "status": "delivered", } }, } }"


<?php
require __DIR__ . '/../../vendor/autoload.php'; // Adjust path to Composer autoload
require 'config.php'; // MongoDB connection configuration

header('Content-Type: application/json');
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

$mongoURI = "mongodb://localhost:27017"; // Update with your MongoDB URI

try {
    // Connect to MongoDB
    $manager = new MongoDB\Driver\Manager($mongoURI);

    // Create the example chat data
    $exampleChat = [
        "messages" => [
            uniqid() => [
                "sender_id" => "1",
                "message" => "hey how are you?",
                "timestamp" => "2024-12-28T12:45:00.000Z",
                "status" => "delivered",
            ],
            uniqid() => [
                "sender_id" => "2",
                "message" => "im doing great and you?",
                "timestamp" => "2024-12-28T12:45:00.000Z",
                "status" => "delivered",
            ],
            uniqid() => [
                "sender_id" => "1",
                "message" => "im fine, thanks for asking. want to go out?",
                "timestamp" => "2024-12-28T12:45:00.000Z",
                "status" => "delivered",
            ],
        ]
    ];

    // Prepare the insert command
    $bulk = new MongoDB\Driver\BulkWrite();
    $chatId = $bulk->insert($exampleChat);

    // Execute the insert
    $manager->executeBulkWrite("swipe_chat_play.chats", $bulk);

    // Return the chat ID
    echo json_encode([
        "status" => "success",
        "chat_id" => (string)$chatId, // Convert ObjectId to string
    ]);
} catch (MongoDB\Driver\Exception\Exception $e) {
    // Handle exceptions
    echo json_encode([
        "status" => "error",
        "message" => $e->getMessage(),
    ]);
}
?>

# wichtige übrige todos
- [ ] wenn man mit selben phone neuen acc macht wird noch altes profiel von securestorage gelesen das muss geändert werden.
- [ ] 