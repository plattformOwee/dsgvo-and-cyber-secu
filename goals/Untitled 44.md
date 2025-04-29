Problem:

When i answer to any message in the chat, the AnswerToMessageLayout-bubble shows my answer but where the answer_to_message should be, i see "unknown" instead.

To fix this, we have to do a few things.

1. when sending the "answer_to" message to the server, its aleady wrong.

For example, i just answered to a voicemail and this is what was saved on the server:

"
{

sender_id: '67b8c7579ea65af156007c4e',

type: 'answer_to',

timestamp: '2025-04-04T19:27:12.334684',

status: 'delivered',

content: {

message_id: '67f014a62babd',

message: '',

answer: 'answer to voicememo'

}"

correct would have been:

{

sender_id: '67b8c7579ea65af156007c4e',

type: 'answer_to',

timestamp: '2025-04-04T19:27:12.334684',

status: 'delivered',

content: {

message_id: '67f014a62babd',

answered_to: {

"type":"here the type of message that was answered to"

"id":"id of message that is being answered to"
"content":"the content will be different with each type"
},
answer: 'answer to voicememo'
}"

What we want is, to save different json depending on which type of message is being answered to.

When answering to image:

{
	sender_id: '67b8c7579ea65af156007c4e',
	type: 'answer_to',
	timestamp: '2025-04-04T17:56:06.311491',
	status: 'delivered',
	content: {
		answered_to: "photo"
		answered_to_id:"id of the message thats beig answered to"
		answer:"the actual answer"
	}
}