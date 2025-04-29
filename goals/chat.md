import 'package:flutter/material.dart';

import 'package:flutter/services.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:http/http.dart' as http;

import 'dart:convert';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

  

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/usables/inappwebview.dart';

import 'package:swipe_chat_play/config.dart';

  

import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_left_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_right_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_left_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_right_layout.dart';

  

class ChatPage extends StatefulWidget {

  final String chatId;

  final String profileURL;

  final String receiver;

  

  const ChatPage({

    super.key,

    required this.chatId,

    required this.profileURL,

    required this.receiver,

  });

  

  @override

  _ChatPageState createState() => _ChatPageState();

}

  

class _ChatPageState extends State<ChatPage> {

  final storage = const FlutterSecureStorage();

  final TextEditingController _messageController = TextEditingController();

  final ScrollController _scrollController = ScrollController();

  

  Map<String, dynamic>? chatData = {};

  bool _isLoading = false;

  bool _isScrolledUp = false;

  

  late FocusNode _focusNode;

  

  @override

  void initState() {

    super.initState();

    _focusNode = FocusNode();

  

    _scrollController.addListener(() {

      if (_scrollController.offset <

              _scrollController.position.maxScrollExtent - 50 &&

          !_isScrolledUp) {

        setState(() => _isScrolledUp = true);

      } else if (_scrollController.offset >=

              _scrollController.position.maxScrollExtent - 50 &&

          _isScrolledUp) {

        setState(() => _isScrolledUp = false);

      }

    });

  

    fetchChatData();

    loadChatData();

  

    // Auto-focus the TextField after the first frame

    WidgetsBinding.instance.addPostFrameCallback((_) {

      FocusScope.of(context).requestFocus(_focusNode);

    });

  }

  

  @override

  void dispose() {

    _focusNode.dispose();

    _scrollController.dispose();

    _messageController.dispose();

    super.dispose();

  }

  

  Future<void> loadChatData() async {

    final storedChat = await storage.read(key: "chat_${widget.chatId}");

    if (storedChat != null) {

      setState(() {

        chatData = jsonDecode(storedChat);

      });

    }

    _scrollToBottom();

  }

  

  Future<void> saveChatData() async {

    await storage.write(

      key: "chat_${widget.chatId}",

      value: jsonEncode(chatData),

    );

  }

  

  Future<void> fetchChatData() async {

    final String? token = await storage.read(key: 'jwt_token');

  

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text("Authorization token is missing.")),

      );

      return;

    }

  

    setState(() => _isLoading = true);

  

    final response = await http.post(

      Uri.parse("${Config.backendBaseUrl}/chat/fetchChat.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

      body: jsonEncode({"chat_id": widget.chatId}),

    );

  

    setState(() => _isLoading = false);

  

    if (response.statusCode == 200) {

      final data = jsonDecode(response.body);

      if (data["status"] == "success") {

        setState(() {

          chatData = data["chat"];

        });

        await saveChatData();

        _scrollToBottom();

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text("Failed to fetch chat: ${data["message"]}"),

          ),

        );

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error: ${response.statusCode}")),

      );

    }

  }

  

  Future<void> sendMessage(String message) async {

    final messageId = DateTime.now().millisecondsSinceEpoch.toString();

    final timestamp = DateTime.now().toIso8601String();

    final String? token = await storage.read(key: 'jwt_token');

  

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text("Authorization token is missing.")),

      );

      return;

    }

  

    // Insert a local "sending" message

    setState(() {

      chatData![messageId] = {

        "type": "text",

        "sender_id": "1", // local user

        "message": message,

        "timestamp": timestamp,

        "status": "sending",

      };

    });

    await saveChatData();

    _scrollToBottom();

  

    try {

      final response = await http

          .post(

            Uri.parse("${Config.backendBaseUrl}/chat/sendMessage.php"),

            headers: {

              "Authorization": "Bearer $token",

              "Content-Type": "application/json",

            },

            body: jsonEncode({

              "chat_id": widget.chatId,

              "message": message,

              "timestamp": timestamp,

            }),

          )

          .timeout(const Duration(seconds: 10));

  

      if (response.statusCode == 200) {

        final data = jsonDecode(response.body);

        if (data == "sent") {

          setState(() {

            chatData![messageId]["status"] = "delivered";

          });

        } else {

          setState(() {

            chatData![messageId]["status"] = "failed";

          });

        }

      } else {

        setState(() {

          chatData![messageId]["status"] = "failed";

        });

      }

    } catch (e) {

      setState(() {

        chatData![messageId]["status"] = "failed";

      });

    }

    await saveChatData();

  }

  

  Future<void> _sendSpecialMessage(

      String amount, String months, String rate) async {

    final String? token = await storage.read(key: 'jwt_token');

  

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text("Authorization token is missing.")),

      );

      return;

    }

  

    final response = await http.post(

      Uri.parse("${Config.backendBaseUrl}/chat/sendSpecialMessage.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

      body: jsonEncode({

        "chat_id": widget.chatId,

        "type": "special",

        "contract_details": {

          "amount": amount,

          "months": months,

          "rate": rate,

        },

        "timestamp": DateTime.now().toIso8601String(),

      }),

    );

  

    if (response.statusCode == 200) {

      final data = jsonDecode(response.body);

      if (data["status"] == "success") {

        final messageId = DateTime.now().millisecondsSinceEpoch.toString();

        final timestamp = DateTime.now().toIso8601String();

  

        setState(() {

          chatData![messageId] = {

            "type": "special",

            "sender_id": "1",

            "contract_details": {

              "amount": amount,

              "months": months,

              "rate": rate,

            },

            "text":

                "$amount euros for $months months at an interest rate of $rate%",

            "button1_label": "Accept",

            "button2_label": "Edit",

            "timestamp": timestamp,

            "status": "delivered",

          };

        });

  

        await saveChatData();

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Contract sent successfully.")),

        );

  

        _scrollToBottom();

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Failed to send contract.")),

        );

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error: ${response.statusCode}")),

      );

    }

  }

  

  void _showContractPopup(BuildContext context,

      {String? initialAmount, String? initialMonths, String? initialRate}) {

    final TextEditingController amountController =

        TextEditingController(text: initialAmount);

    final TextEditingController monthsController =

        TextEditingController(text: initialMonths);

    final TextEditingController rateController =

        TextEditingController(text: initialRate);

  

    final bool isEditing = initialAmount != null;

  

    showDialog(

      context: context,

      builder: (context) {

        return AlertDialog(

          title: Text(isEditing ? "Edit Contract" : "Create Contract"),

          content: Column(

            mainAxisSize: MainAxisSize.min,

            children: [

              TextField(

                controller: amountController,

                keyboardType: TextInputType.number,

                decoration: const InputDecoration(

                  labelText: "Amount",

                  border: OutlineInputBorder(),

                ),

              ),

              const SizedBox(height: 10),

              TextField(

                controller: monthsController,

                keyboardType: TextInputType.number,

                decoration: const InputDecoration(

                  labelText: "Months",

                  border: OutlineInputBorder(),

                ),

              ),

              const SizedBox(height: 10),

              TextField(

                controller: rateController,

                keyboardType: TextInputType.number,

                decoration: const InputDecoration(

                  labelText: "Rate (%)",

                  border: OutlineInputBorder(),

                ),

              ),

            ],

          ),

          actions: [

            TextButton(

              onPressed: () => Navigator.pop(context),

              child: const Text("Cancel"),

            ),

            ElevatedButton(

              onPressed: () {

                final amount = amountController.text;

                final months = monthsController.text;

                final rate = rateController.text;

  

                if (amount.isNotEmpty && months.isNotEmpty && rate.isNotEmpty) {

                  _sendSpecialMessage(amount, months, rate);

                  Navigator.pop(context);

                } else {

                  ScaffoldMessenger.of(context).showSnackBar(

                    const SnackBar(content: Text("Please fill all fields")),

                  );

                }

              },

              child: Text(isEditing ? "Update" : "Send"),

            ),

          ],

        );

      },

    );

  }

  

  void _scrollToBottom() {

    if (_scrollController.hasClients) {

      WidgetsBinding.instance.addPostFrameCallback((_) {

        _scrollController.animateTo(

          _scrollController.position.maxScrollExtent,

          duration: const Duration(milliseconds: 300),

          curve: Curves.easeOut,

        );

      });

    }

  }

  

  String formatTime(String timestamp) {

    final time = DateTime.parse(timestamp);

    return "${time.hour}:${time.minute.toString().padLeft(2, '0')}";

  }

  

  String formatDate(String timestamp) {

    final date = DateTime.parse(timestamp);

    return "${date.day}/${date.month}/${date.year}";

  }

  

  @override

  Widget build(BuildContext context) {

    return SafeArea(

      child: Scaffold(

        backgroundColor: Colors.grey.shade200, // Light grey background

        body: Column(

          children: [

            // Custom top bar

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

              padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 10),

              child: Row(

                children: [

                  IconButton(

                    icon: const Icon(Icons.arrow_back_ios, color: Colors.grey),

                    onPressed: () => Navigator.pop(context),

                  ),

                  CircleAvatar(

                    backgroundImage: NetworkImage(widget.profileURL),

                    radius: 18,

                  ),

                  const SizedBox(width: 8),

                  Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: [

                      Text(

                        widget.receiver,

                        style: const TextStyle(

                          color: Colors.black87,

                          fontWeight: FontWeight.bold,

                          fontSize: 16,

                        ),

                      ),

                      const Text(

                        "(zb online oder zu gesehen)",

                        style: TextStyle(color: Colors.grey, fontSize: 12),

                      ),

                    ],

                  ),

                  const Spacer(),

                  if (_isLoading)

                    const Padding(

                      padding: EdgeInsets.symmetric(horizontal: 16),

                      child: SizedBox(

                        width: 20,

                        height: 20,

                        child: CircularProgressIndicator(strokeWidth: 2),

                      ),

                    )

                  else

                    IconButton(

                      icon: const Icon(Icons.more_vert, color: Colors.grey),

                      onPressed: () {

                        // Optional 3-dot menu

                      },

                    ),

                ],

              ),

            ),

  

            // Chat messages

            Expanded(

              child: chatData == null

                  ? const Center(child: CircularProgressIndicator())

                  : ListView.builder(

                      controller: _scrollController,

                      padding: const EdgeInsets.all(8.0),

                      itemCount: chatData!.length,

                      itemBuilder: (context, index) {

                        final messageId = chatData!.keys.elementAt(index);

                        final message = chatData![messageId];

                        final bool isSender = message["sender_id"] == "1";

                        final String currentDate =

                            formatDate(message["timestamp"]);

                        final String? previousDate = index > 0

                            ? formatDate(

                                chatData![chatData!.keys.elementAt(index - 1)]

                                    ["timestamp"])

                            : null;

  

                        final showDateBubble =

                            previousDate == null || currentDate != previousDate;

                        final String type = message["type"] ?? "text";

  

                        return Column(

                          crossAxisAlignment: CrossAxisAlignment.center,

                          children: [

                            if (showDateBubble)

                              Padding(

                                padding:

                                    const EdgeInsets.symmetric(vertical: 8.0),

                                child: Container(

                                  padding: const EdgeInsets.symmetric(

                                    horizontal: 10.0,

                                    vertical: 5.0,

                                  ),

                                  decoration: BoxDecoration(

                                    color: Colors.grey[300],

                                    borderRadius: BorderRadius.circular(15.0),

                                  ),

                                  child: Text(

                                    currentDate,

                                    style: const TextStyle(

                                      color: Colors.black54,

                                      fontSize: 12.0,

                                    ),

                                  ),

                                ),

                              ),

                            if (type == "special")

                              Align(

                                alignment: isSender

                                    ? Alignment.centerRight

                                    : Alignment.centerLeft,

                                child: isSender

                                    ? ContractBubbleRightLayout(

                                        message: message["text"],

                                        timestamp:

                                            formatTime(message["timestamp"]),

                                        button1Label: message["button1_label"],

                                        button1Action: () async {

                                          // Example accept contract

                                        },

                                        button2Label: message["button2_label"],

                                        button2Action: () {

                                          final contractDetails =

                                              message["contract_details"];

                                          _showContractPopup(

                                            context,

                                            initialAmount:

                                                contractDetails["amount"],

                                            initialMonths:

                                                contractDetails["months"],

                                            initialRate:

                                                contractDetails["rate"],

                                          );

                                        },

                                      )

                                    : ContractBubbleLeftLayout(

                                        message: message["text"],

                                        timestamp:

                                            formatTime(message["timestamp"]),

                                        button1Label: message["button1_label"],

                                        button1Action: () async {

                                          // Example accept contract

                                        },

                                        button2Label: message["button2_label"],

                                        button2Action: () {

                                          final contractDetails =

                                              message["contract_details"];

                                          _showContractPopup(

                                            context,

                                            initialAmount:

                                                contractDetails["amount"],

                                            initialMonths:

                                                contractDetails["months"],

                                            initialRate:

                                                contractDetails["rate"],

                                          );

                                        },

                                      ),

                              )

                            else

                              Align(

                                alignment: isSender

                                    ? Alignment.centerRight

                                    : Alignment.centerLeft,

                                child: isSender

                                    ? ChatBubbleRightLayout(

                                        message: message["message"],

                                        timestamp:

                                            formatTime(message["timestamp"]),

                                      )

                                    : ChatBubbleLeftLayout(

                                        message: message["message"],

                                        timestamp:

                                            formatTime(message["timestamp"]),

                                      ),

                              ),

                          ],

                        );

                      },

                    ),

            ),

  

            // Bottom input field

            Container(

              decoration: BoxDecoration(

                color: Colors.white,

                boxShadow: [

                  BoxShadow(

                    color: Colors.black12,

                    offset: const Offset(0, -2),

                    blurRadius: 4,

                  ),

                ],

              ),

              padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),

              child: Row(

                children: [

                  IconButton(

                    icon: Icon(Icons.attach_file, color: Colors.grey.shade800),

                    onPressed: () => _showContractPopup(context),

                  ),

                  Expanded(

                    child: TextField(

                      controller: _messageController,

                      focusNode: _focusNode,

                      cursorColor: AppColors.profileImageBackground,

                      style: const TextStyle(color: Colors.black87),

                      decoration: const InputDecoration(

                        hintText: "Type a message...",

                        hintStyle: TextStyle(color: Colors.grey),

                        border: InputBorder.none,

                        contentPadding: EdgeInsets.only(left: 8),

                      ),

                    ),

                  ),

                  InkWell(

                    onTap: () {

                      final text = _messageController.text.trim();

                      if (text.isNotEmpty) {

                        sendMessage(text);

                        _messageController.clear();

                      }

                    },

                    child: Container(

                      width: 40,

                      height: 40,

                      decoration: const BoxDecoration(

                        shape: BoxShape.circle,

                        gradient: LinearGradient(

                          colors: [Colors.pinkAccent, Colors.orangeAccent],

                          begin: Alignment.topLeft,

                          end: Alignment.bottomRight,

                        ),

                      ),

                      child: const Icon(Icons.mic, color: Colors.white),

                    ),

                  ),

                ],

              ),

            ),

          ],

        ),

  

        // Use endFloat and add bottom padding so the FAB doesn't overlap the input field

        floatingActionButtonLocation: FloatingActionButtonLocation.endFloat,

        floatingActionButton: _isScrolledUp

            ? Padding(

                // Adjust bottom padding to sit above the bottom bar

                padding: const EdgeInsets.only(bottom: 60),

                child: FloatingActionButton(

                  mini: true,

                  backgroundColor: Colors.black87,

                  onPressed: _scrollToBottom,

                  child: const Icon(Icons.arrow_downward, color: Colors.white),

                ),

              )

            : null,

      ),

    );

  }

}