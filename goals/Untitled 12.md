now:

1. make the tap in the middle in navigation bar link to new myChats.dart page

2. create myChats.dart which is a scrollable list which takes this json as input:
allChatsJson:
{
	"index":[
		"receiverName":"Username of receiver",
		"lastmessageOrUsersLastDraft":"the message itself",
		 "profilepictureURL":"URLofPircture.example",
		 "amount of unread messages":"number",
		 "last online":"time or date if not today",
		 "chat_data":[
			"receiver_name": "Username of receiver",  
			"messages": {  
		        "1": {"timestamp": "10:00 AM", "id": "1", "message": "Hello!"},  
		        "2": {"timestamp": "10:01 AM", "id": "2", "message": "Hi, how are you?"},  
		        "3": {"timestamp": "10:02 AM", "id": "1", "message": "I'm fine, thanks!"},  
			  },  
		    }
		]
	]
	"index":[
		"receiverName":"Username of receiver",
		"lastmessageOrUsersLastDraft":"the message itself",
		 "profilepictureURL":"URLofPircture.example",
		 "amount of unread messages":"number",
		 "last online":"time or date if not today",
		 "chat_data":[
			"receiver_name": "Username of receiver",  
			"messages": {  
		        "1": {"timestamp": "10:00 AM", "id": "1", "message": "Hello!"},  
		        "2": {"timestamp": "10:01 AM", "id": "2", "message": "Hi, how are you?"},  
		        "3": {"timestamp": "10:02 AM", "id": "1", "message": "I'm fine, thanks!"},  
			  },  
		    }
		]
	]
}
and then displays the chats simular to telegram:
[Profile Picture]   [Chat Name + Last Message Preview]   [Timestamp + Unread Count]

and when one chat is clicked,:
call the following chat.dart>chat(allChatsJson["index"]):
"
import 'package:flutter/material.dart';  
import 'chat_bubble_layout.dart';  
  
class chat extends StatelessWidget {  
  final Map<String, dynamic> chatData;  
  
  const chat({Key? key, required this.chatData}) : super(key: key);  
  
  @override  
  Widget build(BuildContext context) {  
    final messages = chatData['messages'] as Map<String, dynamic>;  
  
    return Scaffold(  
      appBar: AppBar(  
        title: Text(chatData['receiver_name']),  
        backgroundColor: Colors.grey[900],  
      ),  
      body: ListView.builder(  
        padding: const EdgeInsets.all(8.0),  
        itemCount: messages.length,  
        itemBuilder: (context, index) {  
          final messageData = messages[(index + 1).toString()];  
          final isLeft = messageData['id'] == '1';  
  
          return Align(  
            alignment: isLeft ? Alignment.centerLeft : Alignment.centerRight,  
            child: ChatBubbleLayout(  
              message: messageData['message'],  
              timestamp: messageData['timestamp'],  
              isLeft: isLeft,  
            ),  
          );  
        },  
      ),  
    );  
  }  
}
"

chatjson:
{ "receiver_name": "namefromjsonabove", "messages": { "1": {"timestamp": "fromjsonabove", "message": "fromjsonabove"}, "2": {"timestamp": "fromjsonabove", "message": "fromjsonabove"}, "3": {"timestamp": "fromjsonabove", "message": "fromjsonabove"}, }, };


this is an example from another of my apps of how chat.dart could be called:
"
import 'package:flutter/material.dart';  
import 'chat.dart';  
  
void main() {  
  runApp(MyApp());  
}  
  
class MyApp extends StatelessWidget {  
  @override  
  Widget build(BuildContext context) {  
    final sampleJson = {  
      "receiver_name": "John Doe",  
      "messages": {  
        "1": {"timestamp": "10:00 AM", "id": "1", "message": "Hello!"},  
        "2": {"timestamp": "10:01 AM", "id": "2", "message": "Hi, how are you?"},  
        "3": {"timestamp": "10:02 AM", "id": "1", "message": "I'm fine, thanks!"},  
      },  
    };  
  
    return MaterialApp(  
      debugShowCheckedModeBanner: false,  
      home: chat(chatData: sampleJson),  
    );  
  }  
}
"

