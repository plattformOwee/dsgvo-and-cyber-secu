It appears you're facing an issue where your `consent.php` script is no longer saving permissions to your MongoDB database. After a thorough analysis of the provided code snippets, I've identified several areas for improvement in your PHP backend scripts that should resolve the problem. The primary issues seem to stem from a lack of robust error handling, inconsistent authentication logic, and potential issues in how the database operations are constructed and executed.

  

Here are the corrected and improved versions of your `consent.php` and `get_consents.php` files, incorporating best practices from your working `add_icebreaker.php` script.

  

### Corrected `consent.php`

  

This revised script includes proper logging, authentication, and error handling, which are crucial for debugging and ensuring reliability.

  

```php

<?php

declare(strict_types=1);

  

/*───────────────────────────────────────────────────────────────────────────*/

/*  consent.php – updates user's consent settings                            */

/*───────────────────────────────────────────────────────────────────────────*/

header('Content-Type: application/json; charset=utf-8');

  

/* ─── 0 · Logging setup ─────────────────────────────────────────── */

$logDir = dirname(__DIR__, 2) . '/logs';

if (!is_dir($logDir) && !mkdir($logDir, 0775, true) && !is_dir($logDir)) {

    error_log('[consent] could not create log directory');

}

ini_set('log_errors', 1);

ini_set('error_log', $logDir . '/consent.log');

error_log('[consent] request start');

  

/* ─── 1 · Bootstrap + config ────────────────────────────────────── */

require_once dirname(__DIR__, 2) . '/bootstrap.php';

require_once dirname(__DIR__, 2) . '/config.php';

  

use MongoDB\BSON\ObjectId;

use MongoDB\BSON\UTCDateTime;

use MongoDB\Driver\BulkWrite;

use MongoDB\Driver\Manager;

  

/* ─── 2 · Only POST ─────────────────────────────────────────────── */

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    error_log('[consent] wrong HTTP method');

    http_response_code(405);

    echo json_encode(['error' => 'Method not allowed – use POST']);

    exit;

}

  

/* ─── 3 · Authenticate via JWT ──────────────────────────────────── */

$claims = auth(); // will 401 & exit on failure

$userId = $claims->sub;

error_log("[consent] authenticated user={$userId}");

$userOid = new ObjectId($userId);

  

/* ─── 4 · Parse JSON body ──────────────────────────────────────── */

$rawBody = file_get_contents('php://input');

$payload = json_decode($rawBody, true) ?? [];

error_log('[consent] payload=' . $rawBody);

  

$now = new UTCDateTime();

  

$new = [

    'newsletter'         => (bool)($payload['newsletter']         ?? false),

    'profile_sensitives' => (bool)($payload['profile_sensitives'] ?? false),

    'location'           => (bool)($payload['location']           ?? false),

    'notifications'      => (bool)($payload['notifications']      ?? false),

    'microphone'         => (bool)($payload['microphone']         ?? false),

];

  

$consentsUpdate = array_map(

    fn(bool $g) => ['granted' => $g, 'timestamp' => $now],

    $new

);

  

/* ─── 5 · Build the update operation ───────────────────────────── */

$updateData = ['$set' => ['userdata.consents' => $consentsUpdate]];

if (!$new['location']) {

    $updateData['$unset'] = ['search_filter.location_radius.location' => 1];

}

  

/* ─── 6 · Execute the update ───────────────────────────────────── */

try {

    $mgr  = new Manager($mongoURI);

    $bulk = new BulkWrite();

  

    $bulk->update(

        ['_id' => $userOid],

        $updateData,

        ['upsert' => false]

    );

  

    $result = $mgr->executeBulkWrite("{$mongoDb}.users", $bulk);

    $modCnt = $result->getModifiedCount();

    error_log("[consent] Mongo modifiedCount={$modCnt}");

  

    if ($modCnt > 0) {

        http_response_code(200);

        echo json_encode([

            'success'  => true,

            'message'  => 'Consents updated successfully',

            'modified' => $modCnt,

        ]);

    } else {

        http_response_code(200); // 200 OK, but nothing changed

        echo json_encode([

            'success'  => false,

            'message'  => 'No changes to apply',

            'modified' => 0,

        ]);

    }

} catch (Throwable $e) {

    error_log('[consent] DB update failed: ' . $e->getMessage());

    http_response_code(500);

    echo json_encode(['error' => 'Could not save consent settings.']);

}

```

  

### Corrected `get_consents.php`

  

This version ensures that even if a user has no consent settings saved, the application will receive a default valid response, preventing potential crashes in the frontend.

  

```php

<?php

declare(strict_types=1);

  

/*───────────────────────────────────────────────────────────────────────────*/

/*  get_consents.php – retrieves all consent settings for the current user   */

/*───────────────────────────────────────────────────────────────────────────*/

header('Content-Type: application/json; charset=utf-8');

  

/* ─── 0 · Logging setup ─────────────────────────────────────────── */

$logDir = dirname(__DIR__, 2) . '/logs';

if (!is_dir($logDir) && !mkdir($logDir, 0775, true) && !is_dir($logDir)) {

    error_log('[get_consents] could not create log directory');

}

ini_set('log_errors', 1);

ini_set('error_log', "{$logDir}/get_consents.log");

error_log('[get_consents] request start');

  

/* ─── 1 · Bootstrap + config ────────────────────────────────────── */

require_once dirname(__DIR__, 2) . '/bootstrap.php';

require_once dirname(__DIR__, 2) . '/config.php';

  

use MongoDB\Driver\Manager;

use MongoDB\Driver\Query;

use MongoDB\BSON\ObjectId;

  

/* ─── 2 · Only GET ──────────────────────────────────────────────── */

if ($_SERVER['REQUEST_METHOD'] !== 'GET') {

    http_response_code(405);

    echo json_encode(['error' => 'Method Not Allowed']);

    exit;

}

  

/* ─── 3 · Authenticate via JWT ──────────────────────────────────── */

$claims = auth(); // sets $userId

$userId = $claims->sub;

error_log("[get_consents] authenticated user={$userId}");

  

/* ─── 4 · Fetch from DB ─────────────────────────────────────────── */

try {

    $mgr = new Manager($mongoURI);

    $q = new Query(

        ['_id' => new ObjectId($userId)],

        ['projection' => ['userdata.consents' => 1]]

    );

  

    $cursor = $mgr->executeQuery("{$mongoDb}.users", $q);

    $doc = $cursor->toArray()[0] ?? null;

  

    if (!$doc || !isset($doc->userdata) || !isset($doc->userdata->consents)) {

        // Return a default structure if no consents are stored yet

        echo json_encode([

            'tos'                => false,

            'newsletter'         => false,

            'profile_sensitives' => false,

            'location'           => false,

            'notifications'      => false,

            'microphone'         => false,

        ]);

        exit;

    }

  

    $consents = $doc->userdata->consents;

  

    // The consents are stored in the DB as an object, not an array.

    // So we need to access them as properties of an object.

    echo json_encode([

        'tos'                => true, // Assuming TOS is always implicitly agreed to if they are using the app

        'newsletter'         => $consents->newsletter->granted         ?? false,

        'profile_sensitives' => $consents->profile_sensitives->granted ?? false,

        'location'           => $consents->location->granted           ?? false,

        'notifications'      => $consents->notifications->granted      ?? false,

        'microphone'         => $consents->microphone->granted         ?? false,

    ]);

  

} catch (Throwable $e) {

    error_log('[get_consents] Query failed: ' . $e->getMessage());

    http_response_code(500);

    echo json_encode(['error' => 'Could not fetch consents']);

}

```

  

### Summary of Key Improvements:

  

*   **Consistent Authentication**: Both scripts now use a more explicit and consistent method for handling user authentication and retrieving the user ID.

*   **Error Handling and Logging**: I've implemented comprehensive logging and `try...catch` blocks. This will help you quickly identify issues by checking the log files (`consent.log` and `get_consents.log`).

*   **Safe Database Operations**: The MongoDB update operation in `consent.php` is now constructed more safely, particularly the conditional `$unset`. The entire database call is wrapped in error handling to prevent script crashes.

*   **Resilient Data Retrieval**: The `get_consents.php` script now gracefully handles cases where a user may not have any consent settings saved yet, preventing errors in your Flutter application.

*   **Clearer Responses**: The `consent.php` script now provides more descriptive JSON responses, indicating whether the update was successful or if no changes were made.

  

By implementing these changes, your consent management system should function correctly and reliably. Remember to check your server's log files if you encounter any further issues, as they will provide valuable insight into the problem.