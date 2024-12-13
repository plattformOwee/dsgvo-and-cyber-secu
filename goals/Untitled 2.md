Okay got it.
- Now, what is meant by bloc archetecture in this context?

- give me the filestructure for all files for flutter login and signup fo the flutter project using the bloc method and follow security as you described before.
- and all the Files needed in flutter aswell as the following api functionality:
Functionality and security login signup:
	I. php POST api to interact with mongoDB Database using the following workflow and security strategy:
		1.  workflow:
			1. - **Client Authentication**:
				- User logs in and receives a JWT token.
			    - JWT token is sent with each API request in the `Authorization` header.
			2. -**API Request Handling**:
				- PHP endpoint verifies the token.
			    - Endpoint validates the POST data.
			    - Endpoint interacts with MongoDB securely using the local MongoDB user.
			3. **Response**:
				- Returns the appropriate data or an error message.
		2. MongoDB is on the same server as the Script so bindIP:  # /etc/mongod.conf net: bindIp: 127.0.0.1
		3. Restrict MongoDB User Permissions with roles: use myDatabase; db.createUser({ user: "myPhpUser", pwd: "strongPassword", roles: [{ role: "readWrite", db: "myDatabase" }] });
		4. Secure the PHP Endpoint: Allow Only POST Requests: if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
		    http_response_code(405);
		    echo json_encode(["error" => "Method Not Allowed"]);
		    exit();
		}
		1. Add JWT Authentication: 
			1. Generate a JWT token when the user logs in: use \Firebase\JWT\JWT; $key = "your_secret_key"; $payload = [ "iss" => "http://yourdomain.com", "aud" => "http://yourdomain.com", "iat" => time(), "exp" => time() + 3600, "user_id" => $userId ]; $jwt = JWT::encode($payload, $key, 'HS256'); echo json_encode(["token" => $jwt]);
			2. Verify the token for every request: try { $decoded = JWT::decode($token, $key, ['HS256']); $userId = $decoded->user_id; // Proceed with the request } catch (Exception $e) { http_response_code(401); echo json_encode(["error" => "Unauthorized"]); exit(); }
		2. Store JWT Securely
		3. Error Handling: try { // MongoDB query } catch (Exception $e) { http_response_code(500); echo json_encode(["error" => "Internal Server Error"]); // Log the actual error to a secure log file for analysis }
 