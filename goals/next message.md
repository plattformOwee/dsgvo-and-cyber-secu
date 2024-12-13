okay that makes sense.now lets create all the backend code and lets test with curl.

1. give sql code to make db
username, firstname, lastname, adress(one string), id, password(hashed), is_verified(boolean), verification code (for jwt later
2. give sql code to create example userdata and and example chatt.

3. create fetchChatt.php which is GET endpoint taking this
input:

{

"id1":"value"

"id2":"value"

}
output to curl or later flutter:

{
"messages":[
	"1":[
		"timestamp":"value"
		"message":"value"
		 "id":"value"
	],
	"2":[
		"timestamp":"value"
		"message":"value"
		 "id":"value"
	],
	"3":[
		"timestamp":"value"
		"message":"value"
		 "id":"value"
	]
]
}

4. create sendMessage.php POST endpoint to ad new message to the backend (and on success we will later not now in flutter call fetchChattapp)