Now i want to integrate this into my chat.
I want the following userflow:
User clicks "attach_file" icon in chat.dart > attach file layout on "GameInviteLayout" opens > user clicks on the "tiktaktoe.svg" icon > game-invite-bubble appears in the chat (with text "tiktaktoe" and a "join" button) > user clicks join > tiktaktoe opens (if they where the first they are "x" if they where the second, they are "o"

To make this possible we need to following files / adjustments to files:

1. GameInviteLayout
	1. add tiktaktoe.svg as icon of one of the tiles (list of games) >
		- onClick > calls "sendInviteMessage.php" with jwt as auth bearer and the message as payload
		- message structure:
		  {
			  "type":"gameInvite"
			  "sender_id": "asdf",
			  "timestamp":"sajdflk"
			  "status":"delivered" // or not delivered
			  "content":{
				  "game":"tiktaktoe"
				  "game_id":"the game id"
			  }
		  }
	2. 