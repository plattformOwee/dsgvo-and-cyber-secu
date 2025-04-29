import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';

import 'dart:convert';

  

import 'package:swipe_chat_play/tabs/chat/chat.dart';

import 'package:swipe_chat_play/tabs/chat/search_user.dart';

  

class MyChatsPage extends StatefulWidget {

  const MyChatsPage({super.key});

  

  @override

  _MyChatsPageState createState() => _MyChatsPageState();

}

  

class _MyChatsPageState extends State<MyChatsPage> {

  final _secureStorage = const FlutterSecureStorage();

  List<dynamic> chats = [];

  

  @override

  void initState() {

    super.initState();

    fetchChats();

  }

  

  Future<void> fetchChats() async {

    final String? token = await _secureStorage.read(key: 'jwt_token');

  

    if (token == null) {

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Authorization token is missing.")),

        );

      }

      return;

    }

  

    final response = await http.get(

      Uri.parse("${Config.backendBaseUrl}/chat/myChats.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

    );

  

    if (response.statusCode == 200) {

      if (mounted) {

        setState(() {

          chats = jsonDecode(response.body);

        });

      }

    } else {

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text("Failed to fetch chats: ${response.reasonPhrase}"),

          ),

        );

      }

    }

  }

  

  Future<void> createChat(String receiverId) async {

    final String? token = await _secureStorage.read(key: 'jwt_token');

  

    if (token == null) {

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Authorization token is missing.")),

        );

      }

      return;

    }

  

    final response = await http.post(

      Uri.parse("${Config.backendBaseUrl}/chat/createExampleChat.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

      body: jsonEncode({

        "receiver_id": receiverId,

      }),

    );

  

    if (response.statusCode == 200) {

      final data = jsonDecode(response.body);

      if (data["status"] == "success") {

        if (mounted) {

          Navigator.push(

            context,

            MaterialPageRoute(

              builder: (context) => ChatPage(

                chatId: data["chat_id"],

                profileURL: data["profileURL"],

                receiver: data["receiver"],

              ),

            ),

          );

        }

      } else {

        if (mounted) {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(

              content: Text("Failed to create chat: ${data["message"]}"),

            ),

          );

        }

      }

    } else {

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text("Error: ${response.reasonPhrase}")),

        );

      }

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      // No 'appBar' parameter here — we build our own custom white bar at the top.

      body: SafeArea(

        child: Column(

          children: [

            // Top "bar" with pure white background and drop shadow

            Container(

              decoration: BoxDecoration(

                color: Colors.white,

                boxShadow: [

                  BoxShadow(

                    color: Colors.black12,

                    offset: const Offset(0, 2),

                    blurRadius: 4,

                  ),

                ],

              ),

              // 10px vertical padding around the pill-shaped search container

              padding: const EdgeInsets.symmetric(vertical: 10, horizontal: 16),

              child: Container(

                padding: const EdgeInsets.symmetric(horizontal: 8),

                decoration: BoxDecoration(

                  color: Colors.grey.shade200,

                  borderRadius: BorderRadius.circular(50), // pill shape

                ),

                child: Row(

                  children: [

                    const Icon(Icons.search, color: Colors.grey),

                    const SizedBox(width: 5),

                    Expanded(

                      child: TextField(

                        textAlignVertical: TextAlignVertical.center,

                        decoration: const InputDecoration(

                          hintText: 'Search...',

                          border: InputBorder.none,

                          contentPadding: EdgeInsets.symmetric(vertical: 10),

                        ),

                        onChanged: (value) {

                          // Implement search logic if needed

                        },

                      ),

                    ),

                  ],

                ),

              ),

            ),

            // Expanded area for the chats list

            Expanded(

              child: ListView.separated(

                itemCount: chats.length,

                separatorBuilder: (context, index) => Divider(

                  color: Colors.grey.shade300,

                  thickness: 2,

                  height: 2,

                ),

                itemBuilder: (context, index) {

                  final chat = chats[index];

                  return InkWell(

                    onTap: () {

                      Navigator.push(

                        context,

                        MaterialPageRoute(

                          builder: (context) => ChatPage(

                            chatId: chat["chat_id"],

                            profileURL: chat["profileURL"],

                            receiver: chat["receiver"],

                          ),

                        ),

                      );

                    },

                    child: Padding(

                      padding: const EdgeInsets.symmetric(

                        horizontal: 16,

                        vertical: 12,

                      ),

                      child: Row(

                        children: [

                          // Circle Avatar on the left

                          CircleAvatar(

                            backgroundImage: NetworkImage(chat["profileURL"]),

                            radius: 24,

                          ),

                          const SizedBox(width: 16),

  

                          // Name and last message/status in the middle

                          Expanded(

                            child: Column(

                              crossAxisAlignment: CrossAxisAlignment.start,

                              children: [

                                Text(

                                  chat["receiver"] ?? "Name",

                                  style: const TextStyle(

                                    fontWeight: FontWeight.bold,

                                    fontSize: 16,

                                  ),

                                ),

                                const SizedBox(height: 4),

                                Text(

                                  chat["last_message"]?.toString() ??

                                      "zb online oder zu gesehen",

                                  style: TextStyle(

                                    fontSize: 14,

                                    color: Colors.grey.shade600,

                                  ),

                                ),

                              ],

                            ),

                          ),

  

                          // Unread count badge on the right (shown only if unread > 0)

                          if ((chat["unread"] ?? 0) > 0)

                            Container(

                              width: 28,

                              height: 28,

                              decoration: BoxDecoration(

                                shape: BoxShape.circle,

                                gradient: const LinearGradient(

                                  colors: [

                                    Colors.pinkAccent,

                                    Colors.orangeAccent

                                  ],

                                  begin: Alignment.topLeft,

                                  end: Alignment.bottomRight,

                                ),

                              ),

                              alignment: Alignment.center,

                              child: Text(

                                chat["unread"].toString(),

                                style: const TextStyle(

                                  color: Colors.white,

                                  fontWeight: FontWeight.bold,

                                  fontSize: 12,

                                ),

                              ),

                            ),

                        ],

                      ),

                    ),

                  );

                },

              ),

            ),

          ],

        ),

      ),

    );

  }

}