what i want: 
1. when any bubble is swiped away from border on its side, make an arrow symbole (">" text for now) appear(get less transparent)  with a answer_swipe_threshhold at which it would count as a swipe and the arrow reaches full visibility.
2. when a message is answered to, make the bubble which was referenced appear above inputfield in a card.
3. when user sends their answer, send a new message type "answer_to" example:
msg_016: {
        type: 'answer_to',
        sender_id: '67b8c732ed13220a4c0b5cd9',
        timestamp: '2025-04-03T10:39:00Z',
        content: {
          "message_id":"id of the referenced message"
          "message":"the actual referenced message"
          "answer":"the answer to that message"
        }
      },

task1: rewrite chat.dar tto make this backswiping functionality as described above possible:
"
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_svg/flutter_svg.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

// Colors, config, attach-file layout, etc.
import 'package:swipe_chat_play/colors.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/attach_file_layout.dart';
import 'package:swipe_chat_play/usables/inappwebview.dart';
import 'package:swipe_chat_play/config.dart';

// Old bubble layouts
import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_left_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_right_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_left_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_right_layout.dart';

// New bubble layouts
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_bubble_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_icebreaker_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_interessen_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_multiple_choice_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_music_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_profile_image_carousel_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_profile_image_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_qa_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_recorded_message_layout.dart';
import 'package:swipe_chat_play/tabs/chat/new_bubbles/react_top_section_bubble_layout.dart';

// Voice recording + custom speech bubble
import 'package:flutter_sound/flutter_sound.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:path_provider/path_provider.dart';
import 'package:swipe_chat_play/tabs/chat/new_chatbubble_layouts/speech_bubble_layout.dart';

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

  // We'll store the raw JSON for the receiver's profile
  Map<String, dynamic>? _receiverProfile;

  // Focus node for the menu icon that does NOT request focus
  final FocusNode _menuIconFocusNode = FocusNode(
    canRequestFocus: false,
    skipTraversal: true,
  );

  // --------------------
  // Voice Recording State
  // --------------------
  bool _isRecording = false;
  final FlutterSoundRecorder _recorder = FlutterSoundRecorder();
  String? _recordedFilePath;

  @override
  void initState() {
    super.initState();
    _focusNode = FocusNode();

    // Listen to text changes to toggle send/mic button
    _messageController.addListener(() {
      setState(() {});
    });

    // Animation controller for attach_file menu
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

    // Scroll listener
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

    // Start fetching chat data
    fetchChatData();
    loadChatData();
    _fetchReceiverProfile();

    // Initialize the recorder
    _initRecorder();

    // Focus textfield as soon as the widget is built
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

    // close recorder
    _recorder.closeRecorder();
    super.dispose();
  }

  // --------------------------------------------------------------------------
  //                      Voice Recording Methods
  // --------------------------------------------------------------------------
  Future<void> _initRecorder() async {
    // Ask for microphone permission
    await Permission.microphone.request();
    await _recorder.openRecorder();
  }

  Future<void> _startRecording() async {
    if (_recorder.isRecording) return;
    final tempDir = await getTemporaryDirectory();
    final filePath =
        "${tempDir.path}/${DateTime.now().millisecondsSinceEpoch}.aac";
    await _recorder.startRecorder(
      toFile: filePath,
      codec: Codec.aacADTS,
    );
    setState(() {
      _isRecording = true;
      _recordedFilePath = filePath;
    });
  }

  Future<void> _stopRecordingAndSend() async {
    if (!_recorder.isRecording) return;
    final path = await _recorder.stopRecorder();
    setState(() => _isRecording = false);

    if (path == null) {
      debugPrint("No recording path found, aborting");
      return;
    }
    _recordedFilePath = path;
    await _uploadVoiceMessage(path);
  }

  Future<void> _uploadVoiceMessage(String localFilePath) async {
    final token = await storage.read(key: 'jwt_token');
    if (token == null) {
      debugPrint("No token found, can't upload voice message");
      return;
    }

    // 1) Create a local bubble with "uploading" status.
    final msgId = DateTime.now().millisecondsSinceEpoch.toString();
    final localTimestamp = DateTime.now().toIso8601String();
    setState(() {
      chatData![msgId] = {
        "type": "voicemessage",
        "sender_id": "1", // <--- your sender logic
        "timestamp": localTimestamp,
        "status": "uploading",
        "audio_url": null,
      };
    });
    _scrollToBottom();
    await saveChatData();

    // 2) Send the multipart request
    final request = http.MultipartRequest(
      "POST",
      Uri.parse("${Config.backendBaseUrl}/chat/sendVoiceMessage.php"),
    );
    request.headers["Authorization"] = "Bearer $token";
    request.fields["chat_id"] = widget.chatId;
    request.fields["timestamp"] = localTimestamp;
    request.files.add(
      await http.MultipartFile.fromPath("voice_file", localFilePath),
    );

    try {
      final response = await request.send();
      final responseBody = await response.stream.bytesToString();
      if (response.statusCode == 200) {
        final parsed = jsonDecode(responseBody);
        if (parsed["status"] == "sent") {
          final realUrl = parsed["voice_url"] ?? "";
          debugPrint("Voice message sent => $realUrl");
          setState(() {
            if (chatData!.containsKey(msgId)) {
              chatData![msgId]["audio_url"] = realUrl;
              chatData![msgId]["status"] = "delivered";
            }
          });
          await saveChatData();
        } else {
          debugPrint("Voice message unexpected response: $responseBody");
          setState(() {
            if (chatData!.containsKey(msgId)) {
              chatData![msgId]["status"] = "failed";
            }
          });
        }
      } else {
        debugPrint(
            "Voice msg upload error: ${response.statusCode} $responseBody");
        setState(() {
          if (chatData!.containsKey(msgId)) {
            chatData![msgId]["status"] = "failed";
          }
        });
      }
    } catch (e) {
      debugPrint("Exception uploading voice message: $e");
      setState(() {
        if (chatData!.containsKey(msgId)) {
          chatData![msgId]["status"] = "failed";
        }
      });
    }

    _scrollToBottom();
  }

  // --------------------------------------------------------------------------
  //          FETCH / LOAD / SAVE Chat & Profile
  // --------------------------------------------------------------------------
  Future<void> _fetchReceiverProfile() async {
    final String? token = await storage.read(key: 'jwt_token');
    if (token == null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text("Authorization token is missing.")),
      );
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
          "Failed to fetch receiver profile. ${response.statusCode}: ${response.body}",
        );
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
        const SnackBar(content: Text("Authorization token is missing.")),
      );
      return;
    }

    setState(() => _isLoading = true);

    try {
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
    } catch (e) {
      setState(() => _isLoading = false);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("Network error: $e")),
      );
    }
  }

  // --------------------------------------------------------------------------
  //                             SEND MESSAGES
  // --------------------------------------------------------------------------
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

    // Add message locally first
    setState(() {
      chatData![messageId] = {
        "type": "text",
        "sender_id": "1", // adapt to your logic
        "message": message,
        "timestamp": timestamp,
        "status": "sending",
      };
    });
    await saveChatData();
    _scrollToBottom();

    // Attempt sending to server
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

  // Example "special" message (like a contract)
  Future<void> _sendSpecialMessage(
    String amount,
    String months,
    String rate,
  ) async {
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
        // Add to chat locally
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

  // Show a small popup to create or edit a contract
  void _showContractPopup(
    BuildContext context, {
    String? initialAmount,
    String? initialMonths,
    String? initialRate,
  }) {
    final amountController = TextEditingController(text: initialAmount);
    final monthsController = TextEditingController(text: initialMonths);
    final rateController = TextEditingController(text: initialRate);
    final isEditing = (initialAmount != null);

    showDialog(
      context: context,
      builder: (ctx) {
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
              onPressed: () => Navigator.pop(ctx),
              child: const Text("Cancel"),
            ),
            ElevatedButton(
              onPressed: () {
                final amount = amountController.text;
                final months = monthsController.text;
                final rate = rateController.text;
                if (amount.isNotEmpty && months.isNotEmpty && rate.isNotEmpty) {
                  _sendSpecialMessage(amount, months, rate);
                  Navigator.pop(ctx);
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
  //          SHOW RECEIVER'S PROFILE BOTTOM SHEET (animated)
  // --------------------------------------------------------------------------
  void _showReceiverProfileCard() async {
    final stored = await storage.read(key: "receiver_profile_${widget.chatId}");
    final Map<String, dynamic>? profileJson =
        (stored != null) ? jsonDecode(stored) : _receiverProfile;

    if (profileJson == null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text("No receiver profile found.")),
      );
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
        builder: (BuildContext bottomCtx) {
          return GestureDetector(
            behavior: HitTestBehavior.opaque,
            onTap: () => Navigator.of(bottomCtx).pop(),
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
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text("Invalid profile data.")),
      );
    }
  }

  Widget _buildProfileCardWithViewMyProfileLayout(
      profile.ProfileData userProfile) {
    List<Widget> contentWidgets = [];

    // Top section
    contentWidgets.add(
      TopSectionWidget(topSection: userProfile.topSection),
    );
    contentWidgets.add(const Divider(
      height: 2,
      thickness: 2,
      color: Colors.grey,
    ));

    // Additional elements
    final elementEntries = userProfile.elements.entries.toList();
    for (int i = 0; i < elementEntries.length; i++) {
      final element = elementEntries[i].value;
      contentWidgets.add(_buildElementWidget(element));
      if (i != elementEntries.length - 1) {
        contentWidgets.add(
          const Divider(height: 2, thickness: 2, color: Colors.grey),
        );
      }
    }

    return DraggableScrollableSheet(
      initialChildSize: 0.8,
      minChildSize: 0.3,
      maxChildSize: 0.95,
      builder: (ctx, scrollController) {
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
  //                        SCROLL + FORMAT HELPERS
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
  //                             BUILD UI
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
                // Top bar
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
                              vertical: 10,
                              horizontal: 10,
                            ),
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
                      IconButton(
                        focusNode: _menuIconFocusNode,
                        icon: const Icon(Icons.more_vert, color: Colors.grey),
                        onPressed: () {
                          final RenderBox overlay = Overlay.of(context)
                              .context
                              .findRenderObject() as RenderBox;

                          showMenu<String>(
                            context: context,
                            position: RelativeRect.fromLTRB(
                              overlay.size.width - 160 - 55, // left
                              30, // top
                              55, // right
                              overlay.size.height - 10, // bottom
                            ),
                            color: Colors.white,
                            shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.circular(16),
                            ),
                            items: const [
                              PopupMenuItem<String>(
                                value: 'block_and_report',
                                child: Text("block & report"),
                              ),
                              PopupMenuItem<String>(
                                value: 'block',
                                child: Text("block"),
                              ),
                              PopupMenuItem<String>(
                                value: 'delete_chat',
                                child: Text("delete chat for both"),
                              ),
                            ],
                          ).then((selectedValue) {
                            if (selectedValue != null) {
                              debugPrint("Menu item tapped: $selectedValue");
                              // handle logic if needed
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
                            final bool isSender = (message["sender_id"] == "1");

                            final String currentDate =
                                formatDate(message["timestamp"]);
                            final String? previousDate = index > 0
                                ? formatDate(chatData![chatData!.keys
                                    .elementAt(index - 1)]["timestamp"])
                                : null;
                            final showDateBubble = (previousDate == null ||
                                currentDate != previousDate);

                            final String type = message["type"] ?? "text";
                            final Map<String, dynamic> content = {};

                            if (message.containsKey("content")) {
                              content.addAll(message["content"]);
                            }

                            final alignment = isSender
                                ? Alignment.centerRight
                                : Alignment.centerLeft;

                            return Column(
                              crossAxisAlignment: CrossAxisAlignment.center,
                              children: [
                                // Show date bubble if needed
                                if (showDateBubble)
                                  Padding(
                                    padding: const EdgeInsets.symmetric(
                                        vertical: 8.0),
                                    child: Container(
                                      padding: const EdgeInsets.symmetric(
                                        horizontal: 10.0,
                                        vertical: 5.0,
                                      ),
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

                                // Actual bubble
                                Align(
                                  alignment: alignment,
                                  child: _buildBubbleByType(
                                    type: type,
                                    message: message,
                                    content: content,
                                    isSender: isSender,
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
                  padding: const EdgeInsets.symmetric(
                    horizontal: 8,
                    vertical: 8,
                  ),
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
                            FocusScope.of(context).unfocus();
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
                      // The mic/send button
                      GestureDetector(
                        onTapDown: (details) {
                          // If there's text, do nothing special here
                          // else start recording
                          if (_messageController.text.trim().isEmpty) {
                            _startRecording();
                          }
                        },
                        onTapUp: (details) async {
                          // If there's text, send text
                          // else stop recording
                          if (_messageController.text.trim().isNotEmpty) {
                            final text = _messageController.text.trim();
                            sendMessage(text);
                            _messageController.clear();
                          } else {
                            await _stopRecordingAndSend();
                          }
                        },
                        child: Container(
                          width: _isRecording ? 60 : 40,
                          height: _isRecording ? 60 : 40,
                          decoration: BoxDecoration(
                            shape: BoxShape.circle,
                            gradient: _isRecording
                                ? const LinearGradient(
                                    colors: [Colors.red, Colors.orange],
                                    begin: Alignment.topLeft,
                                    end: Alignment.bottomRight,
                                  )
                                : const LinearGradient(
                                    colors: [
                                      Colors.pinkAccent,
                                      Colors.orangeAccent
                                    ],
                                    begin: Alignment.topLeft,
                                    end: Alignment.bottomRight,
                                  ),
                          ),
                          child: Icon(
                            _messageController.text.trim().isNotEmpty
                                ? Icons.send
                                : (_isRecording ? Icons.mic : Icons.mic_none),
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

  // --------------------------------------------------------------------------
  // Decide which bubble layout widget to show based on message["type"]
  // --------------------------------------------------------------------------
  Widget _buildBubbleByType({
    required String type,
    required Map<String, dynamic> message,
    required Map<String, dynamic> content,
    required bool isSender,
  }) {
    final timestamp = formatTime(
      message["timestamp"] ?? DateTime.now().toIso8601String(),
    );

    switch (type) {
      // ----------------------------------------------------------------------
      // Special (contract) bubble
      // ----------------------------------------------------------------------
      case "special":
        if (isSender) {
          return ContractBubbleRightLayout(
            message: message["text"],
            timestamp: timestamp,
            button1Label: message["button1_label"],
            button1Action: () {},
            button2Label: message["button2_label"],
            button2Action: () {
              final contractDetails = message["contract_details"];
              _showContractPopup(
                context,
                initialAmount: contractDetails["amount"],
                initialMonths: contractDetails["months"],
                initialRate: contractDetails["rate"],
              );
            },
          );
        } else {
          return ContractBubbleLeftLayout(
            message: message["text"],
            timestamp: timestamp,
            button1Label: message["button1_label"],
            button1Action: () {},
            button2Label: message["button2_label"],
            button2Action: () {
              final contractDetails = message["contract_details"];
              _showContractPopup(
                context,
                initialAmount: contractDetails["amount"],
                initialMonths: contractDetails["months"],
                initialRate: contractDetails["rate"],
              );
            },
          );
        }

      // ----------------------------------------------------------------------
      // Voice Message bubble
      // ----------------------------------------------------------------------
      case "voicemessage":
        final String audioUrl = message["audio_url"] ?? "";
        return SpeechMemoLayout(
          // Include a unique key so it rebuilds when audioUrl changes
          key: ValueKey("voice_${audioUrl}"),
          audioUrl: audioUrl,
          isSender: isSender,
          timestamp: timestamp,
        );

      // ----------------------------------------------------------------------
      // React / new bubble types (all your custom “react_” ones)
      // ----------------------------------------------------------------------
      case "react_bubble":
        return ReactBubbleLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_top_section_bubble":
        return ReactTopSectionBubbleLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_icebreaker":
        return ReactIcebreakerLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_recorded_message":
        return ReactRecordedMessageLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_qa":
        return ReactQALayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_interessen":
        return ReactInteressenLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_multiple_choice":
        return ReactMultipleChoiceLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_music":
        return ReactMusicLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_profile_image_carousel":
        return ReactProfileImageCarouselLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );
      case "react_profile_image":
        return ReactProfileImageLayout(
          content: content,
          isSender: isSender,
          timestamp: timestamp,
        );

      // ----------------------------------------------------------------------
      // Default fallback => normal text bubble
      // ----------------------------------------------------------------------
      default:
        final fallbackMsg = message["message"] ?? "[No text provided]";
        return isSender
            ? ChatBubbleRightLayout(
                message: fallbackMsg,
                timestamp: timestamp,
              )
            : ChatBubbleLeftLayout(
                message: fallbackMsg,
                timestamp: timestamp,
              );
    }
  }
}"

task2: change sendmessage.php to also accept this new type of message if needed:
"
<?php
require __DIR__ . '/../../vendor/autoload.php';
require_once '../config.php'; // MongoDB connection configuration

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

header('Content-Type: application/json');
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

// Set error log file
$errorLogFile = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/send_message_debug.log';
ini_set('log_errors', 1);
ini_set('error_log', $errorLogFile);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {
    error_log("sendMessage.php script started");

    // 1) Check Authorization header (Bearer token)
    $headers = getallheaders();
    $authHeader = $headers['Authorization'] ?? null;

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {
        http_response_code(401);
        echo json_encode(["status" => "error", "message" => "Unauthorized"]);
        exit;
    }

    $jwtToken  = $matches[1];
    $decoded   = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));
    $senderId  = $decoded->user_id ?? null;

    if (!$senderId) {
        http_response_code(401);
        echo json_encode(["status" => "error", "message" => "Invalid token"]);
        exit;
    }

    // 2) Parse input JSON
    $input     = json_decode(file_get_contents('php://input'), true);
    $chatId    = $input['chat_id'] ?? null;
    $timestamp = $input['timestamp'] ?? null;
    // 'type' can be "text" or "react_bubble", etc. default to "text"
    $type      = $input['type'] ?? 'text';
    // For "text" type we read 'message', else we read 'content'
    $message   = $input['message']  ?? null;
    $content   = $input['content']  ?? [];

    error_log("Input received: " . json_encode($input));

    if (!$chatId || !$timestamp) {
        error_log("Missing required fields: chat_id=$chatId, timestamp=$timestamp");
        http_response_code(400);
        echo json_encode("Missing required fields");
        exit;
    }
    // If it's text type, we need a message
    if ($type === 'text' && !$message) {
        error_log("Missing 'message' for text type");
        http_response_code(400);
        echo json_encode("Missing message for text type");
        exit;
    }

    // 3) Connect to MongoDB
    error_log("Connecting to MongoDB");
    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

    // 4) Build the new message data
    $messageId = uniqid();
    $newMessage = [
        "sender_id"  => $senderId,
        "type"       => $type,
        "timestamp"  => $timestamp,
        "status"     => "delivered"
    ];

    // If it's text => store "message" => $message
    // If it's not text => store "content" => $content
    if ($type === 'text') {
        $newMessage["message"] = $message;
    } else {
        $newMessage["content"] = $content;
    }

    // 5) Upsert the message into the chat doc
    $bulk = new MongoDB\Driver\BulkWrite();
    $bulk->update(
        [ "_id" => new MongoDB\BSON\ObjectId($chatId) ],
        [ '$set' => [ "messages.$messageId" => $newMessage ] ],
        [ 'upsert' => true ]
    );

    error_log("Updating messages in chat: chat_id=$chatId, message_id=$messageId");
    $result = $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk);

    // 6) If the chat doc was updated, update 'last_message' for all participants
    if ($result->getModifiedCount() > 0 || $result->getUpsertedCount() > 0) {
        error_log("Message added to chat: chat_id=$chatId, message_id=$messageId");

        // Build a 'last_message' snippet
        // If type=text => last_message = $message
        // else => last_message = "($type)" or you can do something else
        $lastMessage = ($type === 'text') ? $message : "($type)";

        $userBulk = new MongoDB\Driver\BulkWrite();
        // Update all user docs that have "chats.chatId"
        $userBulk->update(
            [ "chats.$chatId" => [ '$exists' => true ] ],
            [ '$set' => [ "chats.$chatId.last_message" => $lastMessage ] ],
            [ 'multi' => true ]
        );
        error_log("Updating last_message for all participants in chat_id=$chatId");

        $userResult = $mongoManager->executeBulkWrite("swipe_chat_play.users", $userBulk);

        if ($userResult->getModifiedCount() > 0) {
            error_log("Last message updated for chat_id=$chatId in user docs");
            echo json_encode("sent");
        } else {
            error_log("Message sent, but no user docs updated for last_message");
            http_response_code(400);
            echo json_encode("Message sent, but failed to update last_message");
        }
    } else {
        error_log("Failed to add message to chat: chat_id=$chatId");
        http_response_code(400);
        echo json_encode("Failed to send message");
    }
} catch (Exception $e) {
    error_log("Exception occurred: " . $e->getMessage());
    http_response_code(500);
    echo json_encode("Internal server error: " . $e->getMessage());
}

error_log("sendMessage.php script ended");
?>
"