now lets create all the backend code and lets test with curl.

1. give sql code to make db


3. give sql code to create example userdata and and example chatt.

4. create fetchChatt.php which is GET endpoint taking this
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
