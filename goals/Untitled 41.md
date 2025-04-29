import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_svg/flutter_svg.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/colors.dart';
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

import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_left_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_right_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_left_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_right_layout.dart';
import 'package:swipe_chat_play/tabs/chat/layouts/attach_file_layout.dart';

import 'package:flutter_sound/flutter_sound.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:path_provider/path_provider.dart';
import 'package:swipe_chat_play/tabs/chat/new_chatbubble_layouts/speech_bubble_layout.dart';

import 'package:swipe_chat_play/usables/inappwebview.dart';
import 'package:swipe_chat_play/config.dart';

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

    // Initialize textfield listener
    _messageController.addListener(() {
      setState(() {}); // rebuild to toggle mic/send icon
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

    // Request focus on textfield
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

  // ---------------------------
  // Voice recording methods
  // ---------------------------
  Future<void> _initRecorder() async {
    // ask permission
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
      debugPrint("No token found, can't upload voice");
      return;
    }

    // 1) Create a local bubble with "uploading" status.
    final msgId = DateTime.now().millisecondsSinceEpoch.toString();
    final localTimestamp = DateTime.now().toIso8601String();
    setState(() {
      chatData![msgId] = {
        "type": "voicemessage",
        "sender_id": "1", // Adjust based on your user logic.
        "timestamp": localTimestamp,
        "status": "uploading",
        "audio_url": null, // Unknown until server returns
      };
    });
    _scrollToBottom();
    await saveChatData();

    // 2) Prepare the multipart request.
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
          debugPrint("Voice message sent successfully => $realUrl");

          // 3) Update the local bubble with the real URL and mark as delivered.
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
  // EXAMPLE: fetchDebugChatExample() - omitted for brevity
  // --------------------------------------------------------------------------

  // --------------------------------------------------------------------------
  //            FETCH / LOAD / SAVE Chat & Profile
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

    // Add message locally
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

    contentWidgets.add(
      TopSectionWidget(topSection: userProfile.topSection),
    );
    contentWidgets.add(const Divider(
      height: 2,
      thickness: 2,
      color: Colors.grey,
    ));

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
  // BUILD UI
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
                              overlay.size.width - 160 - 55,
                              30,
                              55,
                              overlay.size.height - 10,
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
                              // handle logic
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
                          // If there's text, send text, else record
                          if (_messageController.text.trim().isNotEmpty) {
                            // do nothing here - maybe send onTapUp
                          } else {
                            _startRecording();
                          }
                        },
                        onTapUp: (details) async {
                          if (_messageController.text.trim().isNotEmpty) {
                            // Send text
                            final text = _messageController.text.trim();
                            sendMessage(text);
                            _messageController.clear();
                          } else {
                            // stop recording
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

  // Decide which bubble layout widget to show based on message["type"]
  Widget _buildBubbleByType({
    required String type,
    required Map<String, dynamic> message,
    required Map<String, dynamic> content,
    required bool isSender,
  }) {
    final timestamp =
        formatTime(message["timestamp"] ?? DateTime.now().toString());

    switch (type) {
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

      case "voicemessage":
        // show the speech memo bubble
        final String audioUrl = message["audio_url"] ?? "";
        return SpeechMemoLayout(
          audioUrl: audioUrl,
          isSender: isSender,
          timestamp: timestamp,
        );

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
fetchChat.php:
"
<?php

require __DIR__ . '/../../vendor/autoload.php'; // Adjust path to Composer autoload

require_once '../config.php'; // Include configuration for MongoDB connection

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set the error log file path

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/fetch_chat_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

error_log("fetchChat.php script started");

  

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    try {

        // Check for Authorization header

        $headers = getallheaders();

        $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

        if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

            $response = ["status" => "error", "message" => "Unauthorized"];

            error_log("chatjson: " . json_encode($response));

            http_response_code(401);

            echo json_encode($response);

            exit;

        }

  

        $jwtToken = $matches[1];

        $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

        $userId = $decodedToken->user_id ?? null;

  

        if (!$userId) {

            $response = ["status" => "error", "message" => "Invalid token"];

            error_log("chatjson: " . json_encode($response));

            http_response_code(401);

            echo json_encode($response);

            exit;

        }

  

        // Read the input JSON

        $input = json_decode(file_get_contents('php://input'), true);

        $chatId = $input['chat_id'] ?? null;

  

        if (!$chatId) {

            error_log("Missing chat_id in request");

            $response = ["status" => "error", "message" => "chat_id is required"];

            error_log("chatjson: " . json_encode($response));

            http_response_code(400);

            echo json_encode($response);

            exit;

        }

  

        error_log("Fetching chat with ID: $chatId");

  

        // Connect to MongoDB using the URI from config.php

        $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

        // Query to find the chat by chat_id

        $filter = ["_id" => new MongoDB\BSON\ObjectId($chatId)];

        $query = new MongoDB\Driver\Query($filter);

  

        // Execute the query

        $cursor = $mongoManager->executeQuery("swipe_chat_play.chats", $query);

        $chat = $cursor->toArray();

  

        if (empty($chat)) {

            error_log("Chat not found with ID: $chatId");

            $response = ["status" => "error", "message" => "Chat not found"];

            error_log("chatjson: " . json_encode($response));

            http_response_code(404);

            echo json_encode($response);

            exit;

        }

  

        $chatData = $chat[0]->messages;

  

        // Determine sender (ID 1) and receiver (ID 2) dynamically

        foreach ($chatData as $messageId => $message) {

            if ($message->sender_id === $userId) {

                $chatData->$messageId->sender_id = "1";

            } else {

                $chatData->$messageId->sender_id = "2";

            }

        }

  

        error_log("Chat successfully fetched for ID: $chatId");

        $response = ["status" => "success", "chat" => $chatData];

        error_log("chatjson: " . json_encode($response));

        echo json_encode($response);

    } catch (MongoDB\Driver\Exception\Exception $e) {

        error_log("MongoDB error: " . $e->getMessage());

        $response = ["status" => "error", "message" => "Internal server error"];

        error_log("chatjson: " . json_encode($response));

        http_response_code(500);

        echo json_encode($response);

    } catch (Exception $e) {

        error_log("General error: " . $e->getMessage());

        $response = ["status" => "error", "message" => "Internal server error"];

        error_log("chatjson: " . json_encode($response));

        http_response_code(500);

        echo json_encode($response);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    $response = ["status" => "error", "message" => "Method not allowed"];

    error_log("chatjson: " . json_encode($response));

    http_response_code(405);

    echo json_encode($response);

}

  

error_log("fetchChat.php script finished");

?>
"
sendVoiceMessage.php
"
<?php
require __DIR__ . '/../../vendor/autoload.php';
require_once '../config.php'; // MongoDB connection config

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

header('Content-Type: application/json');
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

// Log file (optional)
$errorLogFile = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/send_voicemessage_debug.log';
ini_set('log_errors', 1);
ini_set('error_log', $errorLogFile);

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {
    error_log("sendVoiceMessage.php started");

    // 1) Check Bearer token
    $headers = getallheaders();
    $authHeader = $headers['Authorization'] ?? null;
    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {
        http_response_code(401);
        echo json_encode(["status" => "error", "message" => "Unauthorized"]);
        exit;
    }
    $jwtToken = $matches[1];
    $decoded = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));
    $senderId = $decoded->user_id ?? null;
    if (!$senderId) {
        http_response_code(401);
        echo json_encode(["status" => "error", "message" => "Invalid token"]);
        exit;
    }

    // 2) Validate POST fields
    $chatId = $_POST['chat_id'] ?? null;
    $timestamp = $_POST['timestamp'] ?? null;
    if (!$chatId || !$timestamp) {
        error_log("Missing chat_id or timestamp");
        http_response_code(400);
        echo json_encode(["status" => "error", "message" => "Missing required fields"]);
        exit;
    }

    // Check file upload
    if (!isset($_FILES['voice_file'])) {
        error_log("No voice_file uploaded");
        http_response_code(400);
        echo json_encode(["status" => "error", "message" => "No voice_file uploaded"]);
        exit;
    }

    // 3) Move the uploaded file
    $fileTmpPath = $_FILES['voice_file']['tmp_name'];
    $fileName = $_FILES['voice_file']['name'];
    // For uniqueness, rename with a timestamp:
    $fileNewName = uniqid("voice_", true) . "_" . $fileName;
    $uploadFolder = __DIR__ . "/uploads/voicememos"; // adjust path if needed
    if (!is_dir($uploadFolder)) {
        mkdir($uploadFolder, 0777, true);
    }
    $destinationPath = $uploadFolder . "/" . $fileNewName;
    if (!move_uploaded_file($fileTmpPath, $destinationPath)) {
        error_log("Failed to move uploaded voice file");
        http_response_code(500);
        echo json_encode(["status" => "error", "message" => "Failed to move uploaded file"]);
        exit;
    }

    // The URL that references the newly saved file:
    $domain = "http://node02.krasserserver.com:8002/swipe_chatt_play_api"; // adjust as needed
    $voiceUrl = "$domain/chat/uploads/voicememos/$fileNewName";

    // 4) Connect to MongoDB, insert the message into the chat document
    $mongoManager = new MongoDB\Driver\Manager($mongoURI);
    $bulk = new MongoDB\Driver\BulkWrite();
    $messageId = uniqid();

    $newMessage = [
        "sender_id" => $senderId,
        "type"      => "voicemessage",
        "timestamp" => $timestamp,
        "status"    => "delivered",
        "audio_url" => $voiceUrl
    ];

    $bulk->update(
        ["_id" => new MongoDB\BSON\ObjectId($chatId)],
        ['$set' => [ "messages.$messageId" => $newMessage ]],
        ['upsert' => false]
    );

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk);
    if ($result->getModifiedCount() < 1) {
        error_log("No chat doc modified: chat_id=$chatId");
        http_response_code(400);
        echo json_encode(["status" => "error", "message" => "Failed to add voicemessage to chat"]);
        exit;
    }

    // 5) Update last_message for all participants
    $lastMessage = "(voicemessage)";
    $userBulk = new MongoDB\Driver\BulkWrite();
    $userBulk->update(
        [ "chats.$chatId" => ['$exists' => true] ],
        [ '$set' => [ "chats.$chatId.last_message" => $lastMessage ] ],
        [ 'multi' => true ]
    );
    $mongoManager->executeBulkWrite("swipe_chat_play.users", $userBulk);

    // 6) Return success with the voice URL
    echo json_encode([
        "status" => "sent",
        "voice_url" => $voiceUrl
    ]);
    error_log("Voice message saved, chat_id=$chatId message_id=$messageId => $voiceUrl");

} catch (Exception $e) {
    error_log("Exception: " . $e->getMessage());
    http_response_code(500);
    echo json_encode(["status" => "error", "message" => "Internal server error: " . $e->getMessage()]);
}
?>
"
now when i open this chat, i can listen to old messages (i.e messages that where sent and i have opened and closed the chat in the meantime) But when i send a new message, it doesnt play.

Whats the difference, how can that be?