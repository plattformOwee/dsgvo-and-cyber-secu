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

import 'package:swipe_chat_play/tabs/chat/layouts/attach_file_layout.dart';

  

// Shared profile layout

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

    as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/image_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

  

class ReceiverProfileData {

  final Map<String, dynamic> jsonData;

  ReceiverProfileData(this.jsonData);

}

  

class ChatPage extends StatefulWidget {

  final String chatId;

  final String profileURL;

  final String receiver;

  

  const ChatPage({

    Key? key,

    required this.chatId,

    required this.profileURL,

    required this.receiver,

  }) : super(key: key);

  

  @override

  _ChatPageState createState() => _ChatPageState();

}

  

class _ChatPageState extends State<ChatPage>

    with SingleTickerProviderStateMixin {

  final storage = const FlutterSecureStorage();

  final TextEditingController _messageController = TextEditingController();

  final ScrollController _scrollController = ScrollController();

  

  Map<String, dynamic>? chatData = {};

  bool _isLoading = false;

  bool _isScrolledUp = false;

  

  late AnimationController _menuController;

  late Animation<Offset> _slideAnimation;

  late FocusNode _focusNode;

  bool _wasKeyboardActive = false;

  

  // We'll store the raw JSON for the receiver's profile.

  Map<String, dynamic>? _receiverProfile;

  

  // A focus node that does not request focus so tapping the menu icon does not unfocus the input field.

  final FocusNode _menuIconFocusNode = FocusNode(

    canRequestFocus: false,

    skipTraversal: true,

  );

  

  @override

  void initState() {

    super.initState();

    _focusNode = FocusNode();

  

    _messageController.addListener(() {

      setState(() {}); // rebuild to toggle mic/send icon

    });

  

    _menuController = AnimationController(

      vsync: this,

      duration: const Duration(milliseconds: 300),

    );

    _slideAnimation = Tween<Offset>(

      begin: const Offset(0, 1),

      end: Offset.zero,

    ).animate(

      CurvedAnimation(parent: _menuController, curve: Curves.easeInOut),

    );

  

    _scrollController.addListener(() {

      if (_scrollController.position.pixels <

              _scrollController.position.maxScrollExtent - 50 &&

          !_isScrolledUp) {

        setState(() => _isScrolledUp = true);

      } else if (_scrollController.position.pixels >=

              _scrollController.position.maxScrollExtent - 50 &&

          _isScrolledUp) {

        setState(() => _isScrolledUp = false);

      }

    });

  

    fetchChatData();

    loadChatData();

    _fetchReceiverProfile();

  

    WidgetsBinding.instance.addPostFrameCallback((_) {

      FocusScope.of(context).requestFocus(_focusNode);

    });

  }

  

  @override

  void dispose() {

    _menuController.dispose();

    _focusNode.dispose();

    _scrollController.dispose();

    _messageController.dispose();

    _menuIconFocusNode.dispose();

    super.dispose();

  }

  

  // --------------------------------------------------------------------------

  //               FETCH / LOAD / SAVE CHAT & PROFILE

  // --------------------------------------------------------------------------

  Future<void> _fetchReceiverProfile() async {

    final String? token = await storage.read(key: 'jwt_token');

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Authorization token is missing.")));

      return;

    }

    try {

      final url = "${Config.backendBaseUrl}/chat/get_profile_from_chatid.php";

      final response = await http.post(

        Uri.parse(url),

        headers: {

          "Authorization": "Bearer $token",

          "Content-Type": "application/json",

        },

        body: jsonEncode({"chat_id": widget.chatId}),

      );

  

      if (response.statusCode == 200) {

        await storage.write(

          key: "receiver_profile_${widget.chatId}",

          value: response.body,

        );

        setState(() {

          _receiverProfile = jsonDecode(response.body);

        });

      } else {

        debugPrint(

            "Failed to fetch receiver profile. ${response.statusCode}: ${response.body}");

      }

    } catch (e) {

      debugPrint("Exception while fetching receiver profile: $e");

    }

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

          const SnackBar(content: Text("Authorization token is missing.")));

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

          SnackBar(content: Text("Failed to fetch chat: ${data["message"]}")),

        );

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error: ${response.statusCode}")),

      );

    }

  }

  

  // --------------------------------------------------------------------------

  //                              SEND MESSAGES

  // --------------------------------------------------------------------------

  Future<void> sendMessage(String message) async {

    final messageId = DateTime.now().millisecondsSinceEpoch.toString();

    final timestamp = DateTime.now().toIso8601String();

  

    final String? token = await storage.read(key: 'jwt_token');

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Authorization token is missing.")));

      return;

    }

  

    setState(() {

      chatData![messageId] = {

        "type": "text",

        "sender_id": "1",

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

        setState(() {

          chatData![messageId]["status"] =

              data == "sent" ? "delivered" : "failed";

        });

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

          const SnackBar(content: Text("Authorization token is missing.")));

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

              "rate": rate

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

            const SnackBar(content: Text("Contract sent successfully.")));

        _scrollToBottom();

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(content: Text("Failed to send contract.")));

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text("Error: ${response.statusCode}")));

    }

  }

  

  void _showContractPopup(BuildContext context,

      {String? initialAmount, String? initialMonths, String? initialRate}) {

    final amountController = TextEditingController(text: initialAmount);

    final monthsController = TextEditingController(text: initialMonths);

    final rateController = TextEditingController(text: initialRate);

    final isEditing = initialAmount != null;

  

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

  

  // --------------------------------------------------------------------------

  //              SHOW PROFILE BOTTOM SHEET (with animated slide-in)

  // --------------------------------------------------------------------------

  void _showReceiverProfileCard() async {

    final stored = await storage.read(key: "receiver_profile_${widget.chatId}");

    final Map<String, dynamic>? profileJson =

        stored != null ? jsonDecode(stored) : _receiverProfile;

  

    if (profileJson == null) {

      ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("No receiver profile found.")));

      return;

    }

  

    try {

      final profileData = profile.ProfileData.fromJson(profileJson);

  

      showModalBottomSheet(

        context: context,

        isScrollControlled: true,

        backgroundColor: Colors.transparent,

        barrierColor: Colors.black.withOpacity(0.3),

        isDismissible: true,

        enableDrag: true,

        builder: (BuildContext context) {

          return GestureDetector(

            behavior: HitTestBehavior.opaque,

            onTap: () => Navigator.of(context).pop(),

            child: Stack(

              children: [

                Positioned.fill(child: Container(color: Colors.transparent)),

                Align(

                  alignment: Alignment.bottomCenter,

                  child: _buildProfileCardWithViewMyProfileLayout(profileData),

                ),

              ],

            ),

          );

        },

      );

    } catch (e) {

      debugPrint('Error parsing profile JSON: $e');

      ScaffoldMessenger.of(context)

          .showSnackBar(const SnackBar(content: Text("Invalid profile data.")));

    }

  }

  

  // --------------------------------------------------------------------------

  //                BUILD PROFILE CARD (DraggableScrollableSheet)

  // --------------------------------------------------------------------------

  Widget _buildProfileCardWithViewMyProfileLayout(

      profile.ProfileData userProfile) {

    List<Widget> contentWidgets = [];

  

    // Top section

    contentWidgets.add(TopSectionWidget(topSection: userProfile.topSection));

    contentWidgets.add(const Divider(

      height: 2,

      thickness: 2,

      color: Colors.grey,

    ));

  

    // Elements

    final elementEntries = userProfile.elements.entries.toList();

    for (int i = 0; i < elementEntries.length; i++) {

      final element = elementEntries[i].value;

      contentWidgets.add(_buildElementWidget(element));

      if (i != elementEntries.length - 1) {

        contentWidgets.add(const Divider(

          height: 2,

          thickness: 2,

          color: Colors.grey,

        ));

      }

    }

  

    return DraggableScrollableSheet(

      initialChildSize: 0.8, // start at 80% of screen

      minChildSize: 0.3, // can be dragged down to 30%

      maxChildSize: 0.95, // or up to 95%

      builder: (context, scrollController) {

        return Container(

          decoration: BoxDecoration(

            color: Colors.transparent,

            borderRadius: BorderRadius.circular(16),

          ),

          child: SingleChildScrollView(

            controller: scrollController,

            child: Padding(

              padding: const EdgeInsets.all(12),

              child: Card(

                color: Colors.white,

                shape: RoundedRectangleBorder(

                  borderRadius: BorderRadius.circular(16),

                ),

                child: Column(

                  crossAxisAlignment: CrossAxisAlignment.stretch,

                  children: contentWidgets,

                ),

              ),

            ),

          ),

        );

      },

    );

  }

  

  Widget _buildElementWidget(profile.ProfileElement element) {

    switch (element.type) {

      case 'icebreaker':

        return IceBreakerLayout(content: element.content);

      case 'image':

        return ImageLayout(content: element.content);

      case 'bubbles':

        return BubbleGridLayout(content: element.content);

      default:

        debugPrint('Unknown element type: ${element.type}');

        return const SizedBox.shrink();

    }

  }

  

  // --------------------------------------------------------------------------

  //                        SCROLL HELPERS / FORMATTING

  // --------------------------------------------------------------------------

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

  

  // --------------------------------------------------------------------------

  //                              BUILD MAIN WIDGET

  // --------------------------------------------------------------------------

  @override

  Widget build(BuildContext context) {

    return SafeArea(

      child: Scaffold(

        resizeToAvoidBottomInset: true,

        backgroundColor: Colors.grey.shade200,

        body: Stack(

          children: [

            Column(

              children: [

                // Top bar with expanded hitbox and inner padding

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

                  child: Row(

                    children: [

                      IconButton(

                        icon: const Icon(Icons.arrow_back_ios,

                            color: Colors.grey),

                        onPressed: () => Navigator.pop(context),

                      ),

                      Expanded(

                        child: GestureDetector(

                          onTap: _showReceiverProfileCard,

                          behavior: HitTestBehavior.translucent,

                          child: Container(

                            padding: const EdgeInsets.symmetric(

                                vertical: 10, horizontal: 10),

                            child: Row(

                              children: [

                                CircleAvatar(

                                  backgroundImage:

                                      NetworkImage(widget.profileURL),

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

                                      "(online status etc.)",

                                      style: TextStyle(

                                        color: Colors.grey,

                                        fontSize: 12,

                                      ),

                                    ),

                                  ],

                                ),

                                const Spacer(),

                              ],

                            ),

                          ),

                        ),

                      ),

                      // 3-dot icon using manual showMenu; keyboard stays active.

                      IconButton(

                        focusNode: _menuIconFocusNode,

                        icon: const Icon(Icons.more_vert, color: Colors.grey),

                        onPressed: () {

                          final RenderBox buttonBox =

                              context.findRenderObject() as RenderBox;

                          final RenderBox overlay = Overlay.of(context)

                              .context

                              .findRenderObject() as RenderBox;

                          final Offset buttonPos =

                              buttonBox.localToGlobal(Offset.zero);

                          final Size buttonSize = buttonBox.size;

                          // Position the menu so that its right side aligns with the left side of the button.

                          // Assume menu width of 200.

                          final RelativeRect position = RelativeRect.fromLTRB(

                            buttonPos.dx - 200,

                            buttonPos.dy,

                            overlay.size.width - buttonPos.dx,

                            overlay.size.height - buttonPos.dy,

                          );

  

                          showMenu<String>(

                            context: context,

                            position: position,

                            color: Colors.white,

                            shape: RoundedRectangleBorder(

                              borderRadius: BorderRadius.circular(16),

                            ),

                            items: [

                              const PopupMenuItem<String>(

                                value: 'block_and_report',

                                child: Text("block & report"),

                              ),

                              const PopupMenuItem<String>(

                                value: 'block',

                                child: Text("block"),

                              ),

                              const PopupMenuItem<String>(

                                value: 'delete_chat',

                                child: Text("delete chat for both"),

                              ),

                            ],

                          ).then((selectedValue) {

                            if (selectedValue != null) {

                              debugPrint("Menu item tapped: $selectedValue");

                              // TODO: handle each option's logic if needed.

                            }

                          });

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

                                ? formatDate(chatData![chatData!.keys

                                    .elementAt(index - 1)]["timestamp"])

                                : null;

                            final showDateBubble = previousDate == null ||

                                currentDate != previousDate;

                            final String type = message["type"] ?? "text";

  

                            return Column(

                              crossAxisAlignment: CrossAxisAlignment.center,

                              children: [

                                if (showDateBubble)

                                  Padding(

                                    padding: const EdgeInsets.symmetric(

                                        vertical: 8.0),

                                    child: Container(

                                      padding: const EdgeInsets.symmetric(

                                          horizontal: 10.0, vertical: 5.0),

                                      decoration: BoxDecoration(

                                        color: Colors.grey[300],

                                        borderRadius:

                                            BorderRadius.circular(15.0),

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

                                            timestamp: formatTime(

                                                message["timestamp"]),

                                            button1Label:

                                                message["button1_label"],

                                            button1Action: () {},

                                            button2Label:

                                                message["button2_label"],

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

                                            timestamp: formatTime(

                                                message["timestamp"]),

                                            button1Label:

                                                message["button1_label"],

                                            button1Action: () {},

                                            button2Label:

                                                message["button2_label"],

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

                                            timestamp: formatTime(

                                                message["timestamp"]),

                                          )

                                        : ChatBubbleLeftLayout(

                                            message: message["message"],

                                            timestamp: formatTime(

                                                message["timestamp"]),

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

                  padding:

                      const EdgeInsets.symmetric(horizontal: 8, vertical: 8),

                  child: Row(

                    children: [

                      IconButton(

                        icon: Icon(Icons.attach_file,

                            color: Colors.grey.shade800),

                        onPressed: () {

                          if (_menuController.status ==

                              AnimationStatus.completed) {

                            _menuController.reverse().then((_) {

                              if (_wasKeyboardActive) {

                                FocusScope.of(context).requestFocus(_focusNode);

                                _wasKeyboardActive = false;

                              }

                            });

                          } else {

                            _wasKeyboardActive = _focusNode.hasFocus;

                            // Do not unfocus so keyboard stays active

                            _menuController.forward();

                          }

                        },

                      ),

                      Expanded(

                        child: TextField(

                          controller: _messageController,

                          focusNode: _focusNode,

                          cursorColor: Colors.pinkAccent,

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

                          } else {

                            debugPrint("Record audio...");

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

                          child: Icon(

                            _messageController.text.trim().isNotEmpty

                                ? Icons.send

                                : Icons.mic,

                            color: Colors.white,

                          ),

                        ),

                      ),

                    ],

                  ),

                ),

              ],

            ),

            // The attach menu overlay

            Positioned(

              left: 0,

              right: 0,

              top: 85,

              bottom: 75,

              child: AnimatedBuilder(

                animation: _menuController,

                builder: (context, child) {

                  return Opacity(

                    opacity: _menuController.value,

                    child: SlideTransition(

                      position: _slideAnimation,

                      child: child,

                    ),

                  );

                },

                child: Container(

                  margin: const EdgeInsets.symmetric(horizontal: 8),

                  child: const AttachFileLayout(),

                ),

              ),

            ),

          ],

        ),

        floatingActionButtonLocation: FloatingActionButtonLocation.endFloat,

        floatingActionButton: _isScrolledUp

            ? Padding(

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