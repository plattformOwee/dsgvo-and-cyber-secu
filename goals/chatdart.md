// ignore_for_file: avoid_print

import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:flutter_sound/flutter_sound.dart';

import 'package:permission_handler/permission_handler.dart';

import 'package:path_provider/path_provider.dart';

  

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/attach_file_layout.dart';

import 'package:swipe_chat_play/tabs/chat/new_bubbles/game_invite_bubble_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_left_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat_bubble_right_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_left_layout.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/contract_bubble_right_layout.dart';

import 'package:swipe_chat_play/tabs/chat/new_bubbles/location_bubble_layout.dart';

import 'package:swipe_chat_play/tabs/chat/new_bubbles/new_chatbubble_layouts/speech_bubble_layout.dart';

import 'package:swipe_chat_play/tabs/chat/new_bubbles/answer_to_message_layout.dart';

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

  

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

    as profile;

import '../../profile/layouts/profile_layout.dart';

  

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

  final _storage = const FlutterSecureStorage();

  final TextEditingController _msgCtrl = TextEditingController();

  final ScrollController _scrollCtrl = ScrollController();

  

  Map<String, dynamic> _chat = {};

  bool _isLoading = false;

  bool _isScrolledUp = false;

  

  late final AnimationController _menuCtrl = AnimationController(

    vsync: this,

    duration: const Duration(milliseconds: 300),

  );

  late final Animation<Offset> _menuSlide = Tween(

    begin: const Offset(0, 1),

    end: Offset.zero,

  ).animate(CurvedAnimation(parent: _menuCtrl, curve: Curves.easeInOut));

  

  final FocusNode _inputFocus = FocusNode();

  final FocusNode _moreBtnFocus = FocusNode(skipTraversal: true);

  

  bool _isRecording = false;

  final FlutterSoundRecorder _recorder = FlutterSoundRecorder();

  String? _recordedFilePath;

  

  profile.ProfileData? _receiverProfile;

  Map<String, int> _msgIndex = {}; // for scroll-to-reply

  

  Map<String, dynamic>? _replyTo;

  

  @override

  void initState() {

    super.initState();

    _msgCtrl.addListener(() => setState(() {}));

    _scrollCtrl.addListener(() {

      final up = _scrollCtrl.position.pixels <

          _scrollCtrl.position.maxScrollExtent - 50;

      if (up != _isScrolledUp) setState(() => _isScrolledUp = up);

    });

  

    _fetchChat();

    _loadLocal();

    _fetchReceiverProfile();

    _initRecorder();

  

    WidgetsBinding.instance

        .addPostFrameCallback((_) => _inputFocus.requestFocus());

  }

  

  @override

  void dispose() {

    _menuCtrl.dispose();

    _msgCtrl.dispose();

    _scrollCtrl.dispose();

    _inputFocus.dispose();

    _moreBtnFocus.dispose();

    _recorder.closeRecorder();

    super.dispose();

  }

  

  // ───────────────── Recorder ─────────────────

  Future<void> _initRecorder() async {

    await Permission.microphone.request();

    await _recorder.openRecorder();

  }

  

  Future<void> _startRecording() async {

    if (_recorder.isRecording) return;

    final dir = await getTemporaryDirectory();

    final path = "${dir.path}/${DateTime.now().millisecondsSinceEpoch}.aac";

    await _recorder.startRecorder(toFile: path, codec: Codec.aacADTS);

    setState(() {

      _isRecording = true;

      _recordedFilePath = path;

    });

  }

  

  Future<void> _stopRecordingAndSend() async {

    if (!_recorder.isRecording) return;

    final path = await _recorder.stopRecorder();

    setState(() => _isRecording = false);

    if (path != null) await _uploadVoiceMessage(path);

  }

  

  Future<void> _uploadVoiceMessage(String localFilePath) async {

    final token = await _storage.read(key: 'jwt_token');

    if (token == null) return;

    final msgId = DateTime.now().millisecondsSinceEpoch.toString();

    final ts = DateTime.now().toIso8601String();

  

    setState(() {

      _chat[msgId] = {

        "type": "voicemessage",

        "sender_id": "1",

        "timestamp": ts,

        "status": "uploading",

        "audio_url": null,

      };

    });

    _scrollBottom();

    await _saveLocal();

  

    final req = http.MultipartRequest(

      "POST",

      Uri.parse("${Config.backendBaseUrl}/chat/sendVoiceMessage.php"),

    );

    req.headers["Authorization"] = "Bearer $token";

    req.fields["chat_id"] = widget.chatId;

    req.fields["timestamp"] = ts;

    req.files

        .add(await http.MultipartFile.fromPath("voice_file", localFilePath));

  

    try {

      final res = await req.send();

      final body = await res.stream.bytesToString();

      if (res.statusCode == 200) {

        final data = jsonDecode(body);

        final url = data["voice_url"] ?? "";

        setState(() {

          _chat[msgId]["audio_url"] = url;

          _chat[msgId]["status"] = "delivered";

        });

      } else {

        setState(() => _chat[msgId]["status"] = "failed");

      }

    } catch (_) {

      setState(() => _chat[msgId]["status"] = "failed");

    }

    _scrollBottom();

    await _saveLocal();

  }

  

  // ───────────────── Networking ─────────────────

  Future<void> _fetchChat() async {

    final tok = await _storage.read(key: 'jwt_token');

    if (tok == null) return;

    setState(() => _isLoading = true);

    try {

      final res = await http.post(

        Uri.parse('${Config.backendBaseUrl}/chat/fetchChat.php'),

        headers: {

          'Authorization': 'Bearer $tok',

          'Content-Type': 'application/json'

        },

        body: jsonEncode({'chat_id': widget.chatId}),

      );

      if (!mounted) return;

      setState(() => _isLoading = false);

      final data = jsonDecode(res.body);

      if (res.statusCode == 200 && data['status'] == 'success') {

        setState(() => _chat = data['chat']);

        await _saveLocal();

        _scrollBottom();

      }

    } catch (_) {

      if (mounted) setState(() => _isLoading = false);

    }

  }

  

  Future<void> _sendInvite() async {

    final tok = await _storage.read(key: 'jwt_token');

    if (tok == null) return;

    final ts = DateTime.now().toIso8601String();

    final res = await http.post(

      Uri.parse('${Config.backendBaseUrl}/chat/sendInviteMessage.php'),

      headers: {

        'Authorization': 'Bearer $tok',

        'Content-Type': 'application/json'

      },

      body: jsonEncode({'chat_id': widget.chatId, 'timestamp': ts}),

    );

    if (!mounted) return;

    final data = jsonDecode(res.body);

    if (res.statusCode == 200 && data['status'] == 'success') {

      setState(() => _chat.addAll(data['message']));

      await _saveLocal();

      _scrollBottom();

    }

  }

  

  Future<void> _fetchReceiverProfile() async {

    final tok = await _storage.read(key: 'jwt_token');

    if (tok == null) return;

    try {

      final res = await http.post(

        Uri.parse('${Config.backendBaseUrl}/chat/get_profile_from_chatid.php'),

        headers: {

          'Authorization': 'Bearer $tok',

          'Content-Type': 'application/json'

        },

        body: jsonEncode({'chat_id': widget.chatId}),

      );

      if (res.statusCode == 200) {

        _receiverProfile = profile.ProfileData.fromJson(jsonDecode(res.body));

        await _storage.write(

            key: 'receiver_profile_${widget.chatId}', value: res.body);

        setState(() {});

      }

    } catch (_) {}

  }

  

  Future<void> _saveLocal() =>

      _storage.write(key: 'chat_${widget.chatId}', value: jsonEncode(_chat));

  

  Future<void> _loadLocal() async {

    final raw = await _storage.read(key: 'chat_${widget.chatId}');

    if (raw != null && mounted) {

      setState(() => _chat = jsonDecode(raw));

      _scrollBottom();

    }

  }

  

  // ───────────────── Reply / Swipe ─────────────────

  void _onMessageSwipedToReply(Map<String, dynamic> msg, String id) {

    setState(() {

      _replyTo = {

        "message_id": id,

        "sender_id": msg["sender_id"],

        "type": msg["type"],

        "message": msg["message"],

        "content": msg["content"],

      };

    });

    _scrollBottom();

  }

  

  void _cancelReply() => setState(() => _replyTo = null);

  

  Widget _buildReplyReferenceCard() {

    if (_replyTo == null) return const SizedBox.shrink();

    final t = _replyTo!["type"] ?? "text";

    final txt = (t == "text") ? _replyTo!["message"] ?? "" : "($t)";

    return Card(

      margin: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),

      color: Colors.grey.shade300,

      child: ListTile(

        title: const Text("Replying to:",

            style: TextStyle(fontWeight: FontWeight.bold, fontSize: 13)),

        subtitle: Text(txt, style: const TextStyle(fontSize: 12)),

        trailing:

            IconButton(icon: const Icon(Icons.close), onPressed: _cancelReply),

      ),

    );

  }

  

  // ───────────────── Sending Text / Location / Special ─────────────────

  Future<void> sendMessage(String message) async {

    final id = DateTime.now().millisecondsSinceEpoch.toString();

    final ts = DateTime.now().toIso8601String();

    final tok = await _storage.read(key: 'jwt_token');

    if (tok == null) return;

  

    final isReply = _replyTo != null;

    final type = isReply ? "answer_to" : "text";

    final payload = {

      "chat_id": widget.chatId,

      "timestamp": ts,

      "type": type,

      if (isReply)

        "content": {

          "answered_to": _replyTo!["type"],

          "answered_to_id": _replyTo!["message_id"],

          if (_replyTo!["type"] == "text")

            "answered_to_content": _replyTo!["message"],

          "answer": message,

        }

      else

        "message": message,

    };

  

    setState(() {

      _chat[id] = {

        "type": type,

        "sender_id": "1",

        "timestamp": ts,

        "status": "sending",

        if (isReply) "content": payload["content"] else "message": message,

      };

    });

    await _saveLocal();

    _scrollBottom();

  

    try {

      final res = await http

          .post(

            Uri.parse("${Config.backendBaseUrl}/chat/sendMessage.php"),

            headers: {

              'Authorization': 'Bearer $tok',

              'Content-Type': 'application/json'

            },

            body: jsonEncode(payload),

          )

          .timeout(const Duration(seconds: 10));

  

      if (res.statusCode == 200 && jsonDecode(res.body) == "sent") {

        setState(() => _chat[id]["status"] = "delivered");

      } else {

        setState(() => _chat[id]["status"] = "failed");

      }

    } catch (_) {

      setState(() => _chat[id]["status"] = "failed");

    }

    _scrollBottom();

    _cancelReply();

    await _saveLocal();

  }

  

  Future<void> _sendLocationMessage(double lat, double lng) async {

    final id = DateTime.now().millisecondsSinceEpoch.toString();

    final ts = DateTime.now().toIso8601String();

    final tok = await _storage.read(key: 'jwt_token');

    if (tok == null) return;

  

    setState(() {

      _chat[id] = {

        "type": "location",

        "sender_id": "1",

        "timestamp": ts,

        "status": "sending",

        "content": {"lat": lat, "lng": lng},

      };

    });

    await _saveLocal();

    _scrollBottom();

  

    try {

      final res = await http

          .post(

            Uri.parse("${Config.backendBaseUrl}/chat/sendMessage.php"),

            headers: {

              'Authorization': 'Bearer $tok',

              'Content-Type': 'application/json'

            },

            body: jsonEncode({

              "chat_id": widget.chatId,

              "timestamp": ts,

              "type": "location",

              "content": {"lat": lat, "lng": lng}

            }),

          )

          .timeout(const Duration(seconds: 10));

  

      if (res.statusCode == 200 && jsonDecode(res.body) == "sent") {

        setState(() => _chat[id]["status"] = "delivered");

      } else {

        setState(() => _chat[id]["status"] = "failed");

      }

    } catch (_) {

      setState(() => _chat[id]["status"] = "failed");

    }

    await _saveLocal();

    _scrollBottom();

  }

  

  // ───────────────── UI Helpers ─────────────────

  void _scrollBottom() {

    if (!_scrollCtrl.hasClients) return;

    WidgetsBinding.instance.addPostFrameCallback((_) {

      _scrollCtrl.animateTo(

        _scrollCtrl.position.maxScrollExtent,

        duration: const Duration(milliseconds: 300),

        curve: Curves.easeOut,

      );

    });

  }

  

  String _fmtTime(String iso) {

    final d = DateTime.parse(iso);

    return "${d.hour}:${d.minute.toString().padLeft(2, '0')}";

  }

  

  String _fmtDate(String iso) {

    final d = DateTime.parse(iso);

    return "${d.day}/${d.month}/${d.year}";

  }

  

  void _scrollToReferencedMessage(String refId) {

    final idx = _msgIndex[refId];

    if (idx == null) return;

    _scrollCtrl.animateTo(

      idx * 100.0,

      duration: const Duration(milliseconds: 300),

      curve: Curves.easeInOut,

    );

  }

  

  // ───────────────── Build ─────────────────

  @override

  Widget build(BuildContext context) {

    return SafeArea(

      child: Scaffold(

        backgroundColor: Colors.grey.shade200,

        body: Stack(

          children: [

            Column(

              children: [

                // ▸ top bar

                Container(

                  decoration: const BoxDecoration(

                    color: Colors.white,

                    boxShadow: [

                      BoxShadow(color: Colors.black12, blurRadius: 4)

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

                          onTap: () {

                            if (_receiverProfile == null) return;

                            showModalBottomSheet(

                              context: context,

                              isScrollControlled: true,

                              backgroundColor: Colors.transparent,

                              builder: (_) => ProfileCard(

                                data: _receiverProfile!,

                                scrollController: ScrollController(),

                              ),

                            );

                          },

                          child: Container(

                            padding: const EdgeInsets.all(10),

                            child: Row(

                              children: [

                                CircleAvatar(

                                  backgroundImage:

                                      NetworkImage(widget.profileURL),

                                ),

                                const SizedBox(width: 8),

                                Text(widget.receiver,

                                    style: const TextStyle(

                                        fontWeight: FontWeight.bold,

                                        fontSize: 16)),

                                const Spacer(),

                              ],

                            ),

                          ),

                        ),

                      ),

                      IconButton(

                        focusNode: _moreBtnFocus,

                        icon: const Icon(Icons.more_vert, color: Colors.grey),

                        onPressed: () {/* your menu */},

                      ),

                    ],

                  ),

                ),

  

                // ▸ messages list

                Expanded(

                  child: _isLoading

                      ? const Center(child: CircularProgressIndicator())

                      : ListView.builder(

                          controller: _scrollCtrl,

                          padding: const EdgeInsets.all(8),

                          itemCount: _chat.length,

                          itemBuilder: (ctx, i) {

                            final keys = _chat.keys.toList();

                            final msgId = keys[i];

                            final msg = _chat[msgId]!;

                            final isMe = msg["sender_id"].toString() == '1';

                            _msgIndex[msgId] = i;

  

                            final currDate = _fmtDate(msg["timestamp"]);

                            final prevDate = (i > 0)

                                ? _fmtDate(_chat[keys[i - 1]]["timestamp"])

                                : null;

                            final showDate =

                                prevDate == null || currDate != prevDate;

  

                            final type = msg["type"] ?? "text";

                            final content = msg["content"] != null

                                ? Map<String, dynamic>.from(msg["content"])

                                : <String, dynamic>{};

  

                            return Column(

                              children: [

                                if (showDate)

                                  Padding(

                                    padding:

                                        const EdgeInsets.symmetric(vertical: 8),

                                    child: Container(

                                      padding: const EdgeInsets.symmetric(

                                          horizontal: 10, vertical: 5),

                                      decoration: BoxDecoration(

                                          color: Colors.grey[300],

                                          borderRadius:

                                              BorderRadius.circular(15)),

                                      child: Text(currDate,

                                          style: const TextStyle(

                                              color: Colors.black54,

                                              fontSize: 12)),

                                    ),

                                  ),

                                Align(

                                  alignment: isMe

                                      ? Alignment.centerRight

                                      : Alignment.centerLeft,

                                  child: _SwipableBubble(

                                    isSender: isMe,

                                    onReplySwipe: () =>

                                        _onMessageSwipedToReply(msg, msgId),

                                    child: _buildBubbleByType(

                                        type, msg, content, isMe),

                                  ),

                                ),

                                const SizedBox(height: 4),

                              ],

                            );

                          },

                        ),

                ),

  

                // ▸ reply card + input row

                _buildReplyReferenceCard(),

                _inputRow(),

              ],

            ),

  

            // ▸ attach overlay

            Positioned(

              left: 0,

              right: 0,

              top: 85,

              bottom: 75,

              child: AnimatedBuilder(

                animation: _menuCtrl,

                builder: (_, child) => Opacity(

                  opacity: _menuCtrl.value,

                  child: SlideTransition(position: _menuSlide, child: child),

                ),

                child: AttachFileLayout(

                  onGamePicked: (g) {

                    if (g == 'tictactoe') _sendInvite();

                    _menuCtrl.reverse();

                  },

                  onLocationSelected: (lat, lng) {

                    _menuCtrl.reverse();

                    _sendLocationMessage(lat, lng);

                  },

                  onImagesSelected: (files) {/* your image logic */},

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

                  onPressed: _scrollBottom,

                  child: const Icon(Icons.arrow_downward, color: Colors.white),

                ),

              )

            : null,

      ),

    );

  }

  

  Widget _inputRow() => Container(

        decoration: const BoxDecoration(

            color: Colors.white,

            boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 4)]),

        padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),

        child: Row(

          children: [

            IconButton(

              icon: const Icon(Icons.attach_file),

              onPressed: () {

                _menuCtrl.forward();

                FocusScope.of(context).unfocus();

              },

            ),

            Expanded(

              child: TextField(

                controller: _msgCtrl,

                focusNode: _inputFocus,

                decoration: const InputDecoration(

                    border: InputBorder.none, hintText: 'Type…'),

                onSubmitted: (txt) {

                  if (txt.trim().isNotEmpty) {

                    sendMessage(txt.trim());

                    _msgCtrl.clear();

                  }

                },

              ),

            ),

            GestureDetector(

              onTapDown: (_) {

                if (_msgCtrl.text.trim().isEmpty) {

                  _startRecording();

                }

              },

              onTapUp: (_) async {

                if (_msgCtrl.text.trim().isNotEmpty) {

                  sendMessage(_msgCtrl.text.trim());

                  _msgCtrl.clear();

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

                          end: Alignment.bottomRight)

                      : const LinearGradient(

                          colors: [Colors.pinkAccent, Colors.orangeAccent],

                          begin: Alignment.topLeft,

                          end: Alignment.bottomRight),

                ),

                child: Icon(

                  _msgCtrl.text.trim().isNotEmpty

                      ? Icons.send

                      : (_isRecording ? Icons.mic : Icons.mic_none),

                  color: Colors.white,

                ),

              ),

            ),

          ],

        ),

      );

  

  Widget _buildBubbleByType(

    String type,

    Map<String, dynamic> msg,

    Map<String, dynamic> content,

    bool isSender,

  ) {

    final ts = _fmtTime(msg["timestamp"]);

    switch (type) {

      case "game_invite":

        // just forward the existing content map

        return GameInviteBubbleLayout(

          content: content,

          isSender: isSender,

        );

  

      case "voicemessage":

        return SpeechMemoLayout(

            key: ValueKey("voice_${msg["audio_url"]}"),

            audioUrl: msg["audio_url"] ?? "",

            isSender: isSender,

            timestamp: ts);

      case "answer_to":

        return AnswerToMessageLayout(

            message: msg,

            onTap: () {

              final ref = content["answered_to_id"];

              if (ref != null) _scrollToReferencedMessage(ref);

            });

      case "location":

        return LocationBubbleLayout(

            content: content, isSender: isSender, timestamp: ts);

  

      case "react_bubble":

        return ReactBubbleLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_icebreaker":

        return ReactIcebreakerLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_interessen":

        return ReactInteressenLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_multiple_choice":

        return ReactMultipleChoiceLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_music":

        return ReactMusicLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_profile_image_carousel":

        return ReactProfileImageCarouselLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_profile_image":

        return ReactProfileImageLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_qa":

        return ReactQALayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_recorded_message":

        return ReactRecordedMessageLayout(

            content: content, isSender: isSender, timestamp: ts);

      case "react_top_section_bubble":

        return ReactTopSectionBubbleLayout(

            content: content, isSender: isSender, timestamp: ts);

      default:

        final txt = msg["message"] ?? '';

        return isSender

            ? ChatBubbleRightLayout(message: txt, timestamp: ts)

            : ChatBubbleLeftLayout(message: txt, timestamp: ts);

    }

  }

}

  

// ───────────────── Swipe-to-reply ─────────────────

class _SwipableBubble extends StatefulWidget {

  final Widget child;

  final bool isSender;

  final VoidCallback onReplySwipe;

  const _SwipableBubble(

      {Key? key,

      required this.child,

      required this.isSender,

      required this.onReplySwipe})

      : super(key: key);

  @override

  __SwipableBubbleState createState() => __SwipableBubbleState();

}

  

class __SwipableBubbleState extends State<_SwipableBubble> {

  double _dx = 0;

  static const _th = 100.0;

  @override

  Widget build(BuildContext context) {

    return GestureDetector(

      onHorizontalDragUpdate: (d) {

        if (d.delta.dx > 0) setState(() => _dx += d.delta.dx);

      },

      onHorizontalDragEnd: (_) {

        if (_dx >= _th) widget.onReplySwipe();

        setState(() => _dx = 0);

      },

      child: Stack(

        alignment: Alignment.centerLeft,

        children: [

          Positioned(

            left: 0,

            child: Opacity(

                opacity: (_dx / _th).clamp(0.0, 1.0),

                child: const Text('>',

                    style: TextStyle(fontSize: 34, color: Colors.grey))),

          ),

          Transform.translate(offset: Offset(_dx, 0), child: widget.child),

        ],

      ),

    );

  }

}