<?php

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

require __DIR__ . '/../vendor/autoload.php';

use GuzzleHttp\Client;

require 'db_connection.php'; // Ensure this file defines $pdo for database connection

header("Content-Type: application/json");

ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

// Logging setup
$logFile = __DIR__ . '/create_contract_error.log';
ini_set('log_errors', 1);
ini_set('error_log', $logFile);

error_log("create_contract.php script started");
function logMessage($message) {
    // Log message to a file
    $logFile = '/home/container/webroot/logfile.log';
    file_put_contents($logFile, date('Y-m-d H:i:s') . " - " . $message . PHP_EOL, FILE_APPEND);
}


// Function to extract user ID from JWT
function id_jwt($authHeader, $secretKey) {
    if (!$authHeader) {
        error_log("Missing Authorization header");
        http_response_code(401);
        echo json_encode(["error" => "Authorization header missing"]);
        exit;
    }

    $jwtToken = str_replace('Bearer ', '', $authHeader);

    try {
        error_log("Attempting to decode JWT token: $jwtToken");
        $decoded = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));
        $userId = $decoded->user_id ?? null;

        if (!$userId) {
            error_log("Token decoded but user_id not found");
            http_response_code(401);
            echo json_encode(["error" => "Invalid token payload"]);
            exit;
        }

        error_log("JWT decoded successfully. User ID: $userId");
        return $userId;
    } catch (Exception $e) {
        error_log("JWT decoding failed: " . $e->getMessage());
        http_response_code(401);
        echo json_encode(["error" => "Invalid or missing token"]);
        exit;
    }
}

// Function to create the contract
function createContract($data, $pdo, $secretKey) {
    // Extract request data
    $token = $data['token'] ?? null;
    $amount = $data['amount'] ?? null;
    $interestRate = $data['interestrate'] ?? null;
    $months = $data['months'] ?? null;
    $sendToUsername = $data['sendto'] ?? null;

    // Validate required fields
    if (!$token || !$amount || !$interestRate || !$months || !$sendToUsername) {
        error_log("Missing required fields in the request");
        http_response_code(400);
        echo json_encode(["error" => "Missing required fields"]);
        exit;
    }

    // Extract sender ID from the token
    $userId = id_jwt("Bearer $token", $secretKey);

    // Fetch sender details
    $stmt = $pdo->prepare("SELECT `username`, `email` FROM `users` WHERE `id` = ?");
    $stmt->execute([$userId]);

    if ($stmt->rowCount() === 0) {
        error_log("Sender not found in database: ID $userId");
        http_response_code(404);
        echo json_encode(["error" => "Sender not found"]);
        exit;
    }

    $sender = $stmt->fetch(PDO::FETCH_ASSOC);
    $username1 = $sender['username'];
    $email1 = $sender['email'];

    // Fetch receiver details
    $stmt = $pdo->prepare("SELECT `username`, `email` FROM `users` WHERE `username` = ?");
    $stmt->execute([$sendToUsername]);

    if ($stmt->rowCount() === 0) {
        error_log("Receiver not found in database: Username $sendToUsername");
        http_response_code(404);
        echo json_encode(["error" => "Receiver not found"]);
        exit;
    }

    $receiver = $stmt->fetch(PDO::FETCH_ASSOC);
    $username2 = $receiver['username'];
    $email2 = $receiver['email'];

    // Build contract JSON
    $contractDetails = [
        "user1" => [
            "username" => $username1,
            "email" => $email1,
        ],
        "user2" => [
            "username" => $username2,
            "email" => $email2,
        ],
        "amount" => $amount,
        "months" => $months,
        "interestrate" => $interestRate,
    ];

    error_log("Contract details created: " . json_encode($contractDetails));
    newFromtemplate2((json_encode($contractDetails)));
}

function newFromtemplate2($contract_details,) {
    try {
        // Create a new Guzzle client
        $client = new Client();

        // Set up the request headers
        $headers = [
            'Authorization' => 'oauth_signature_method="PLAINTEXT", oauth_consumer_key="c06a740c6e7a7ea1_8124", oauth_token="c827fb9d766e2e14_35500", oauth_signature="dd3c3ae50df10485&58574ddee1609dfd"',
            'Cookie' => 'lang="en"; lang-ssn="en"'
        ];

        // Send a POST request to create a new document from template
        $response = $client->post('https://api-testbed.scrive.com/api/v2/documents/newfromtemplate/8222115557379957904', [
            'headers' => $headers,
        ]);

        logMessage("Received response from newfromtemplate API.");

        // Get the response data
        $responseData = json_decode($response->getBody()->getContents(), true);

        

        // Convert the response data to JSON
        $jsonFormatted = json_encode($responseData, JSON_PRETTY_PRINT);

        // Save original API response JSON to file
        $filePath = '/home/container/webroot/original_response_newfromtemplate2.json';
        file_put_contents($filePath, $jsonFormatted);
        logMessage("Original response saved to $filePath");
        // Extract document ID from response
        $documentId = $responseData['id'];


        // add document ID to the JSON
        $contract_details = json_decode($contract_details, true);
        $contract_details['document_id'] = $documentId;
        logMessage("newest contract_details : " . json_encode($contract_details)); 



        // Modify JSON with new details
        $modifiedJson = changejson($contract_details, $documentId, $responseData);
        logMessage(message: "Modified JSON: " . $modifiedJson);

        // Save modified JSON to a file
        $modifiedFilePath = '/home/container/webroot/changed.json';
        file_put_contents($modifiedFilePath, $modifiedJson);
        logMessage("Modified JSON saved to $modifiedFilePath");

        // Call updateDocumentScrive with the modified JSON and document ID
        updateDocumentScrive($modifiedJson, $documentId);

        // Call startDocumentScrive with the document ID
        $deliveryURL = startDocumentScrive($documentId);

        // Return the delivery URL
        echo json_encode([
            'delivery_url' => $deliveryURL
        ]);
    } catch (Exception $e) {
        logMessage("Error in newFromtemplate2: " . $e->getMessage());
    }
}


function changejson($contract_details, $documentId, $responseData){
    logMessage("Starting changejson function...");

    // Ensure $responseData is a JSON string
    if (is_array($responseData)) {
        $responseData = json_encode($responseData);
    }

    // Decode the input JSON
    $data = json_decode($responseData, true);

    // Change the first and last names for party1
    foreach ($data['parties'][1]['fields'] as &$field) {
        if ($field['type'] == 'name' && $field['order'] == 1) {
            $field['value'] = $contract_details['user1']['username'];
        }
        if ($field['type'] == 'name' && $field['order'] == 2) {
            $field['value'] =  $contract_details['user1']['email'];
        }
    }

    // Change the first and last names for party2
    foreach ($data['parties'][2]['fields'] as &$field) {
        if ($field['type'] == 'name' && $field['order'] == 1) {
            $field['value'] = $contract_details['user2']['username'];
        }
        if ($field['type'] == 'name' && $field['order'] == 2) {
            $field['value'] =  $contract_details['user2']['email'];
        }
    }

    // Update the api_callback_url, sign_success_redirect_url, and delivery_method
    $data['api_callback_url'] = "http://krasserserver.com:8004/callback.php";
    foreach ($data['parties'] as &$party) {
        $party['sign_success_redirect_url'] = "https://fahrschule-peppermint.com/successfully-signed/";
        $party['delivery_method'] = "api";
    }

    // Encode the modified data back to JSON in a well-formatted way
    $modifiedJson = json_encode($data, JSON_PRETTY_PRINT);

    logMessage("changejson function completed successfully.");
    //logMessage("Modified JSON: " . $modifiedJson);

    return $modifiedJson;
}

function updateDocumentScrive($latestjson, $documentId) {
    try {
        logMessage("Starting updateDocumentScrive function...");

        // Create a new Guzzle client
        $client = new GuzzleHttp\Client();

        // Set up the request headers
        $headers = [
            'Authorization' => 'oauth_signature_method="PLAINTEXT", oauth_consumer_key="c06a740c6e7a7ea1_8124", oauth_token="c827fb9d766e2e14_35500", oauth_signature="dd3c3ae50df10485&58574ddee1609dfd"',
            'Content-Type' => 'application/x-www-form-urlencoded',
            'Cookie' => 'lang="en"; lang-ssn="en"'
        ];

        // Prepare the body with the JSON data for the document update
        $body = [
            'document' => $latestjson // The JSON string should be URL-encoded
        ];

        // Send a POST request to update the document
        $response = $client->post("https://api-testbed.scrive.com/api/v2/documents/$documentId/update", [
            'headers' => $headers,
            'form_params' => $body // Use 'form_params' to send the body as URL-encoded form data
        ]);

        // Get the response
        $updateResponse = $response->getBody()->getContents();
        //logMessage("Update response: " . $updateResponse);

        // Save the update response to a file
        $updateFilePath = "/home/container/webroot/update_response.json";
        file_put_contents($updateFilePath, json_encode(json_decode($updateResponse), JSON_PRETTY_PRINT));

        logMessage("Update response saved to $updateFilePath");

    } catch (Exception $e) {
        logMessage("Error in updateDocumentScrive: " . $e->getMessage());
    }
}

function startDocumentScrive($documentId) {
    try {
        logMessage("Starting startDocumentScrive function...");

        // Create a new Guzzle client
        $client = new GuzzleHttp\Client();

        // Set up the request headers
        $headers = [
            'Authorization' => 'oauth_signature_method="PLAINTEXT", oauth_consumer_key="c06a740c6e7a7ea1_8124", oauth_token="c827fb9d766e2e14_35500", oauth_signature="dd3c3ae50df10485&58574ddee1609dfd"',
            'Cookie' => 'lang="en"; lang-ssn="en"'
        ];

        // Send a POST request to start the document
        $response = $client->post("https://api-testbed.scrive.com/api/v2/documents/$documentId/start", [
            'headers' => $headers,
        ]);

        // Get the response
        $startResponse = $response->getBody()->getContents();
        //logMessage("Start response: " . $startResponse);

        // Save the start response to a file
        $startFilePath = "/home/container/webroot/start_response_$documentId.json";
        file_put_contents($startFilePath, json_encode(json_decode($startResponse), JSON_PRETTY_PRINT));

        logMessage("Start response saved to $startFilePath");

        logMessage("Delivery URL: " . $startResponse);
        $deliveryURL = "https://api-testbed.scrive.com" . deliveryURL1($startResponse);
        
        return $deliveryURL;

    } catch (Exception $e) {
        logMessage("Error in startDocumentScrive: " . $e->getMessage());
    }
}


//later needs to change to supply each user with their own delivery URL
function deliveryURL1($json){
    $data = json_decode($json, true);

    if (isset($data['parties'][1]['api_delivery_url'])) {
        return $data['parties'][1]['api_delivery_url'];
    } else {
        return null; // Return null if the second party or api_delivery_url doesn't exist
    }
}

// Get request data
$input = file_get_contents("php://input");
$data = json_decode($input, true);

// Call the function and output the result
$contract = createContract($data, $pdo, $secretKey);
logMessage("contract: " . json_encode(createContract($data, $pdo, $secretKey)));
echo json_encode($contract);
?>
