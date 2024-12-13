<?php
// callback.php

// Set up a log file for debugging purposes
$logFile = 'scrive_callback.log';

// Write to log file
function logMessage($message) {
    global $logFile;
    file_put_contents($logFile, date('Y-m-d H:i:s') . " - " . $message . "\n", FILE_APPEND);
}

// Check if this is a POST request
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    echo json_encode(["error" => "Only POST requests are allowed"]);
    logMessage("Received non-POST request.");
    exit();
}

// Read the incoming JSON data
$input = file_get_contents('php://input');
$data = json_decode($input, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(["error" => "Invalid JSON data"]);
    logMessage("Invalid JSON data received.");
    exit();
}

// Log the received data for debugging
logMessage("Received callback data: " . print_r($data, true));

// Check if document_signed_and_sealed is true
if (isset($data['document_signed_and_sealed']) && $data['document_signed_and_sealed'] === true) {
    $documentId = $data['document_id'] ?? null;

    if (!$documentId) {
        http_response_code(400);
        echo json_encode(["error" => "Document ID is missing"]);
        logMessage("Document ID is missing in callback data.");
        exit();
    }

    // Handle the document signing confirmation, e.g., update your database or notify the user
    logMessage("Document with ID $documentId has been signed and sealed.");

    // Send a success response back to Scrive
    http_response_code(200);
    echo json_encode(["status" => "success", "message" => "Document signed and sealed."]);
} else {
    // If document_signed_and_sealed is not true, log the event
    logMessage("Received callback, but document_signed_and_sealed is not true.");
    http_response_code(400);
    echo json_encode(["error" => "Document is not signed and sealed"]);
}
?>
