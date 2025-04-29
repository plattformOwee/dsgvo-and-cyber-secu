1. change "save_profile.php 
1.1 please change it to take jwt token as auth bearer as input instead of email and the json as payload. 
1.2 add function to decode the _id back from the token.
- the following is how that token was generated when user logged in please make coherent with that:
"
<?php
require __DIR__ . '/../../vendor/autoload.php';
require 'config.php'; // MongoDB configuration

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

header('Content-Type: application/json');
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

// Logging setup
$logFile = __DIR__ . '/fingerprint_login_error.log';
ini_set('log_errors', 1);
ini_set('error_log', $logFile);

error_log("fingerprintLogin.php script started");

function logMessage($message) {
    $logFile = '/home/container/webroot/logfile.log';
    file_put_contents($logFile, date('Y-m-d H:i:s') . " - " . $message . PHP_EOL, FILE_APPEND);
    error_log($message);
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $fingerprint = $_POST['fingerprint'] ?? null;

    if (!$fingerprint) {
        $errorMsg = "Fingerprint is required";
        logMessage("Error: $errorMsg");
        http_response_code(400);
        echo json_encode(["error" => $errorMsg]);
        exit;
    }

    try {
        logMessage("Received fingerprint: $fingerprint");

        // Connect to MongoDB
        $manager = new MongoDB\Driver\Manager($mongoURI);
        $namespace = "swipe_chat_play.users";

        // Fetch all users and their fingerprint hashes
        $query = new MongoDB\Driver\Query([]);
        $cursor = $manager->executeQuery($namespace, $query);
        $users = $cursor->toArray();

        $userId = null;
        foreach ($users as $user) {
            if (empty($user->fingerprint_hash)) {
                // Skip users with no fingerprint hash
                logMessage("Skipping user ID: {$user->_id} due to null/empty fingerprint hash");
                continue;
            }

            if (password_verify($fingerprint, $user->fingerprint_hash)) {
                $userId = (string)$user->_id; // Use the ObjectId as the user ID
                break;
            }
        }

        if (!$userId) {
            $errorMsg = "Invalid fingerprint";
            logMessage("Error: $errorMsg");
            http_response_code(401);
            echo json_encode(["error" => $errorMsg]);
            exit;
        }

        logMessage("Valid fingerprint for user ID: $userId");

        // Generate JWT
        $payload = [
            "iss" => "http://panel.krasserserver.com",
            "aud" => "http://panel.krasserserver.com",
            "iat" => time(),
            "exp" => time() + (60 * 60), // Token expires in 1 hour
            "user_id" => $userId
        ];
        $jwt = JWT::encode($payload, $secretKey, 'HS256');

        logMessage("JWT generated successfully for user ID: $userId");

        // Respond with token
        http_response_code(200);
        echo json_encode(["token" => $jwt]);
    } catch (MongoDB\Driver\Exception\Exception $e) {
        $errorMsg = "MongoDB error: " . $e->getMessage();
        logMessage("Error: $errorMsg");
        http_response_code(500);
        echo json_encode(["error" => $errorMsg]);
    } catch (Exception $e) {
        $errorMsg = "Internal server error: " . $e->getMessage();
        logMessage("Error: $errorMsg");
        http_response_code(500);
        echo json_encode(["error" => $errorMsg]);
    }
} else {
    $errorMsg = "Method not allowed";
    logMessage("Error: $errorMsg");
    http_response_code(405);
    echo json_encode(["error" => $errorMsg]);
}
"
1.3 save the profilejson into mongodb in user with this _id
1.4 return success message

2. rewrite the appbar of my create profile page so it calls this script correctly
- token is saved at "await _secureStorage.write(key: 'jwt_token', value: token);"