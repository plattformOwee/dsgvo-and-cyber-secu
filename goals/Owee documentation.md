# Mitschriften
## 20/10/24 erstes mal flutter auf android laufen und iframe zum unterschreiben

#### scripts
- C:\xampp\htdocs\flutter_api/api.php:
	- <?php
header('Content-Type: application/json');

$host = 'localhost';
$user = 'root'; // Default XAMPP username
$pass = ''; // Default XAMPP password (empty)
$db_name = 'flutter_db'; // Your database name

// Create a connection
$conn = new mysqli($host, $user, $pass, $db_name);

// Check the connection
if ($conn->connect_error) {
    die(json_encode(['error' => $conn->connect_error]));
}

// Query to get data (change this according to your table structure)
$sql = "SELECT * FROM users"; // Replace 'users' with your table name
$result = $conn->query($sql);

$data = [];
if ($result->num_rows > 0) {
    while ($row = $result->fetch_assoc()) {
        $data[] = $row;
    }
}

echo json_encode($data);
$conn->close();
?>
