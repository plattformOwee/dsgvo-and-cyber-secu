The Goal of this chat, is to create a matching algorithm for a Dating app.

1. info: 
We will use mongoDB as a Database in the following manner:
"
<?php
require 'config.php'; // MongoDB connection configuratio
 $manager = new MongoDB\Driver\Manager($mongoURI);
    $namespace = "test_db.users";
"
2. info:
this is my config.php"
<?php
// Configuration file (config.php)
$ngrokURL = "4.tcp.eu.ngrok.io";  // ngrok TCP URL
$mongoPort = "11009";  // Port provided by ngrok
$mongoUser = "burgermeister";  // MongoDB username
$mongoPassword = "drdrjecky14";  // MongoDB password
$mongoAuthDB = "admin";  // Authentication database

// MongoDB connection URI
$mongoURI = "mongodb://$mongoUser:$mongoPassword@$ngrokURL:$mongoPort/$mongoAuthDB";
?>
"

1. task:
please write out a php script which creates 20 different users with this profile structure:
"
{
	"_id": ObjectId('mongo object id'),
	"username":"example username",
	"attributes":{
		"gender":"male",
		"height":"175",
		"location":"
		"likes":["sport","gaming","smoking","chilling"]
	},
	"searching_for":{
		"gender":"female",
		"height":"180",
		"likes":["sport", "art","chilling","]
	}
}
"
2. task:
please write out a php script which takes as input one users Json> gets "searching_for" from the json >  then gives back Profiles with attributes fitting what the user is searching for.