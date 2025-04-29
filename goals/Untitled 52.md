import 'dart:async';

import 'dart:convert';

import 'dart:math' as math;

import 'package:swipe_chat_play/tabs/chat/layouts/chat.dart';

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/fonts.dart';

// We use the NEW profile_data.dart from version 3:

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/tabs/chat/layouts/chat.dart';

// Chat popup or chat page references:

import 'package:swipe_chat_play/tabs/chat/layouts/chat_popup.dart';

// IMPORTANT: from version 2, we rely on usage of `profile_layout.dart`

// which exports a "ProfileCard" widget that supports onElementTap:

import 'package:swipe_chat_play/tabs/profile/layouts/profile_layout.dart'

show ProfileCard;

/// This merges scroll & swipe from version 1, reaction chat usage from version 2,

/// and the new user_id logic from version 3.

class CardStack extends StatefulWidget {

const CardStack({Key? key}) : super(key: key);

@override

_CardStackState createState() => _CardStackState();

}

class _CardStackState extends State<CardStack> {

List<profile.ProfileData> cardData = [];

bool isLoading = true;

bool isFetchingMore = false;

bool hasMore = true;

int currentPage = 1;

final Debouncer _debouncer = Debouncer(milliseconds: 500);

// Show at most 2 cards at once (back + top).

final int maxVisibleCards = 2;

final int prefetchThreshold = 2;

// Tracks the drag offset of the top card (for X & partial Y movement).

final ValueNotifier<Offset> _dragPositionNotifier =

ValueNotifier<Offset>(Offset.zero);

// Controller for the top card’s vertical scrolling.

ScrollController _topCardScrollController = ScrollController();

// --- Swipe thresholds ---

/// We consider a swipe “done” if the user’s final dx goes beyond this.

double get swipeThreshold => MediaQuery.of(context).size.width * 0.07;

/// We only switch direction if user crosses 2% beyond the furthest point.

double get directionChangeThreshold =>

MediaQuery.of(context).size.width * 0.02;

// --- Gesture detection fields ---

Offset? _gestureStart;

bool _isHorizontalSwipe = false;

bool _decisionMade = false;

final double _slopDistance = 10;

String? _currentDirection;

double _furthestRightSoFar = 0.0;

double _furthestLeftSoFar = 0.0;

@override

void initState() {

super.initState();

_fetchProfiles(reset: true);

}

Future<void> _fetchProfiles({bool reset = false}) async {

if (isFetchingMore || (reset == false && !hasMore)) return;

setState(() => reset ? isLoading = true : isFetchingMore = true);

const storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) {

setState(() => isLoading = false);

return;

}

final int limit = reset ? 50 : 10;

try {

final response = await http.get(

Uri.parse(

'${Config.backendBaseUrl}/match_algorithm/match.php?page=$currentPage&limit=$limit'),

headers: {"Authorization": "Bearer $token"},

);

if (response.statusCode == 200) {

final data = jsonDecode(response.body);

if (data["status"] == "success") {

final profilesList = (data["data"] as List<dynamic>)

.map((json) => profile.ProfileData.fromJson(json))

.toList();

setState(() {

if (reset) {

cardData = profilesList;

currentPage = 1;

} else {

cardData.addAll(profilesList);

}

hasMore = data["has_more"] ?? false;

currentPage++;

});

}

}

} catch (e) {

debugPrint("Error fetching profiles: $e");

} finally {

setState(() => reset ? isLoading = false : isFetchingMore = false);

}

}

List<profile.ProfileData> get visibleCards =>

cardData.reversed.take(maxVisibleCards).toList().reversed.toList();

void _dismissCard(String swipeValue) {

if (cardData.isEmpty) return;

final topProfile = cardData.last;

setState(() {

cardData.removeLast();

});

_sendSwipe(topProfile, swipeValue);

_topCardScrollController.dispose();

_topCardScrollController = ScrollController();

_debouncer.run(() {

if (mounted && cardData.length <= prefetchThreshold && hasMore) {

_fetchProfiles();

}

});

}

Future<void> _sendSwipe(

profile.ProfileData? profileData, String swipeValue) async {

if (profileData == null) return;

const storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) return;

final url = Uri.parse('${Config.backendBaseUrl}/match_algorithm/swipe.php');

final body = jsonEncode({

"receiver_id": profileData.topSection.id, // unique user _id

"swipe": swipeValue,

});

try {

final response = await http.post(

url,

headers: {

"Content-Type": "application/json",

"Authorization": "Bearer $token",

},

body: body,

);

if (response.statusCode == 200) {

final cleanedResponse =

response.body.replaceAll(RegExp(r'<br\s*/?>'), '').trim();

final responseData = jsonDecode(cleanedResponse);

final status = responseData["status"];

if (status == "match") {

debugPrint("It's a match!");

final String chatId = responseData["chat_id"];

final String profileURL = responseData["profileURL"];

final String receiver = responseData["receiver"];

Navigator.push(

context,

MaterialPageRoute(

builder: (context) => ChatPage(

chatId: chatId,

profileURL: profileURL,

receiver: receiver,

),

),

);

} else if (status == "no match") {

debugPrint("No match");

}

} else {

debugPrint(

"Swipe endpoint error: ${response.statusCode} ${response.body}");

}

} catch (e) {

debugPrint("Error sending swipe: $e");

}

}

// -----------------------------------------------------------

// Gesture Handling: Horizontal vs. Vertical + direction changes

// -----------------------------------------------------------

void _onPanStart(DragStartDetails details) {

_gestureStart = details.globalPosition;

_decisionMade = false;

_isHorizontalSwipe = false;

}

void _onPanUpdate(DragUpdateDetails details) {

if (_gestureStart == null) return;

final dx = details.globalPosition.dx - _gestureStart!.dx;

final dy = details.globalPosition.dy - _gestureStart!.dy;

if (!_decisionMade) {

final distance = math.sqrt(dx * dx + dy * dy);

if (distance < _slopDistance) return;

_decisionMade = true;

final angleDeg = math.atan(dy.abs() / dx.abs()) * (180 / math.pi);

_isHorizontalSwipe = (angleDeg <= 45);

}

if (_isHorizontalSwipe) {

final current = _dragPositionNotifier.value;

final newDx = current.dx + details.delta.dx;

final newDy = current.dy + details.delta.dy * 0.7;

_dragPositionNotifier.value = Offset(newDx, newDy);

_updateSwipeDirection(newDx);

} else {

if (_topCardScrollController.hasClients) {

final offsetChange = -details.delta.dy;

final currentOffset = _topCardScrollController.offset;

final maxExtent = _topCardScrollController.position.maxScrollExtent;

final newOffset = math.min(

math.max(currentOffset + offsetChange, 0.0),

maxExtent,

);

_topCardScrollController.jumpTo(newOffset);

}

}

}

void _updateSwipeDirection(double currentDx) {

if (_currentDirection == null) {

if (currentDx > 0) {

_currentDirection = "right";

_furthestRightSoFar = currentDx;

_furthestLeftSoFar = 0.0;

} else {

_currentDirection = "left";

_furthestLeftSoFar = currentDx;

_furthestRightSoFar = 0.0;

}

return;

}

if (_currentDirection == "right") {

if (currentDx > _furthestRightSoFar) {

_furthestRightSoFar = currentDx;

}

if (currentDx < 0 &&

currentDx < (_furthestRightSoFar - directionChangeThreshold)) {

_currentDirection = "left";

_furthestLeftSoFar = currentDx;

}

} else if (_currentDirection == "left") {

if (currentDx < _furthestLeftSoFar) {

_furthestLeftSoFar = currentDx;

}

if (currentDx > 0 &&

currentDx > (_furthestLeftSoFar + directionChangeThreshold)) {

_currentDirection = "right";

_furthestRightSoFar = currentDx;

}

}

}

void _onPanEnd(DragEndDetails details) {

if (_isHorizontalSwipe && cardData.isNotEmpty) {

final offset = _dragPositionNotifier.value;

final dx = offset.dx;

if (dx.abs() > swipeThreshold) {

final direction = (dx > 0) ? "yes" : "no";

_dismissCard(direction);

}

} else {

if (_topCardScrollController.hasClients) {

final velocityY = details.velocity.pixelsPerSecond.dy;

final currentOffset = _topCardScrollController.offset;

final maxExtent = _topCardScrollController.position.maxScrollExtent;

double targetOffset = currentOffset - velocityY * 0.2;

targetOffset = math.max(0.0, math.min(targetOffset, maxExtent));

_topCardScrollController.animateTo(

targetOffset,

duration: const Duration(milliseconds: 500),

curve: Curves.easeOut,

);

}

}

_dragPositionNotifier.value = Offset.zero;

_gestureStart = null;

_isHorizontalSwipe = false;

_decisionMade = false;

_currentDirection = null;

}

@override

void dispose() {

_dragPositionNotifier.dispose();

_topCardScrollController.dispose();

_debouncer.cancel();

super.dispose();

}

// REACTION CHAT:

// We pass only "receiver_id" + optional "type".

void _openReactionChat(profile.ProfileData tappedProfile) {

final reactionPayload = {

"receiver_id": tappedProfile.topSection.id,

"type": "reaction",

// or "react_bubble" if you prefer

};

showDialog(

context: context,

barrierDismissible: true,

builder: (context) {

return ChatPopupDialog(

reactionPayload: reactionPayload,

);

},

);

}

@override

Widget build(BuildContext context) {

if (!isLoading && cardData.isEmpty) {

return Container(

color: AppColors.background,

child: Center(

child: Text(

"No profiles left, change filters",

style: AppFonts.secondaryFont,

),

),

);

}

if (isLoading) {

return Container(

color: AppColors.background,

child: Center(

child: Column(

mainAxisSize: MainAxisSize.min,

children: [

SizedBox(width: 200, child: LinearProgressIndicator()),

const SizedBox(height: 8),

Text("Loading profiles", style: AppFonts.tertiaryFont),

],

),

),

);

}

return Container(

color: AppColors.background,

child: SafeArea(

child: Stack(

children: [

// BACK CARD (2nd from top)

if (visibleCards.length > 1)

Positioned.fill(

child: AnimatedScale(

duration: const Duration(milliseconds: 200),

scale: 0.95,

child: ProfileCard(

data: visibleCards.first,

scrollController: ScrollController(),

// We'll pass the entire profile so we can get topSection.id

onElementTap: (elementData) {

_openReactionChat(visibleCards.first);

},

),

),

),

// TOP CARD

if (visibleCards.isNotEmpty)

Positioned.fill(

child: GestureDetector(

behavior: HitTestBehavior.translucent,

onPanStart: _onPanStart,

onPanUpdate: _onPanUpdate,

onPanEnd: _onPanEnd,

child: ValueListenableBuilder<Offset>(

valueListenable: _dragPositionNotifier,

builder: (context, dragOffset, child) {

final dx = dragOffset.dx;

final factor =

(dx.abs() / swipeThreshold).clamp(0.0, 1.0);

final opacity = 1.0 - 0.2 * factor;

return Opacity(

opacity: opacity,

child: Transform.translate(

offset: Offset(dx, dragOffset.dy),

child: AnimatedSwitcher(

duration: const Duration(milliseconds: 300),

transitionBuilder: (child, animation) {

return ScaleTransition(

scale: Tween<double>(begin: 0.95, end: 1.0)

.animate(

CurvedAnimation(

parent: animation,

curve: Curves.easeOutBack,

),

),

child: child,

);

},

// RepaintBoundary for performance

child: RepaintBoundary(

key: ValueKey(visibleCards.last),

child: ProfileCard(

data: visibleCards.last,

scrollController: _topCardScrollController,

onElementTap: (elementData) {

_openReactionChat(visibleCards.last);

},

),

),

),

),

);

},

),

),

),

// YES/NO indicators

SwipeIndicators(

dragNotifier: _dragPositionNotifier,

swipeThreshold: swipeThreshold,

),

// Loading indicator if fetching more

if (isFetchingMore)

const Positioned(

bottom: 20,

left: 0,

right: 0,

child: Center(

child: Column(

mainAxisSize: MainAxisSize.min,

children: [

CircularProgressIndicator(),

SizedBox(height: 8),

Text("New profiles loading"),

],

),

),

),

],

),

),

);

}

}

/// Displays a translucent "check" or "close" as the user drags horizontally.

class SwipeIndicators extends StatelessWidget {

final ValueNotifier<Offset> dragNotifier;

final double swipeThreshold;

const SwipeIndicators({

Key? key,

required this.dragNotifier,

required this.swipeThreshold,

}) : super(key: key);

@override

Widget build(BuildContext context) {

return ValueListenableBuilder<Offset>(

valueListenable: dragNotifier,

builder: (context, offset, child) {

final dx = offset.dx;

final dy = offset.dy;

final isHorizontal = dx.abs() >= dy.abs();

final progress = (dx.abs() / swipeThreshold).clamp(0.0, 1.0);

final leftOpacity = (dx < 0 && isHorizontal) ? progress : 0.0;

final rightOpacity = (dx > 0 && isHorizontal) ? progress : 0.0;

final screenHeight = MediaQuery.of(context).size.height;

final verticalPosition = screenHeight / 2 - 30;

return IgnorePointer(

child: Stack(

children: [

// NO indicator (left)

Positioned(

left: 20,

top: verticalPosition,

child: AnimatedOpacity(

opacity: leftOpacity,

duration: const Duration(milliseconds: 100),

child: CircleAvatar(

radius: 30,

backgroundColor: Colors.red.withOpacity(0.9),

child:

const Icon(Icons.close, color: Colors.white, size: 30),

),

),

),

// YES indicator (right)

Positioned(

right: 20,

top: verticalPosition,

child: AnimatedOpacity(

opacity: rightOpacity,

duration: const Duration(milliseconds: 100),

child: CircleAvatar(

radius: 30,

backgroundColor: Colors.green.withOpacity(0.9),

child:

const Icon(Icons.check, color: Colors.white, size: 30),

),

),

),

],

),

);

},

);

}

}

/// Simple Debouncer for prefetch logic.

class Debouncer {

final int milliseconds;

Timer? _timer;

Debouncer({required this.milliseconds});

void run(VoidCallback action) {

_timer?.cancel();

_timer = Timer(Duration(milliseconds: milliseconds), action);

}

void cancel() {

_timer?.cancel();

}

}"

"

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'dart:convert';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat.dart';

class ChatPopupDialog extends StatefulWidget {

final Map<String, dynamic> reactionPayload;

const ChatPopupDialog({

Key? key,

required this.reactionPayload,

}) : super(key: key);

@override

_ChatPopupDialogState createState() => _ChatPopupDialogState();

}

class _ChatPopupDialogState extends State<ChatPopupDialog> {

final storage = const FlutterSecureStorage();

final TextEditingController _commentController = TextEditingController();

late FocusNode _focusNode;

bool _isSending = false;

@override

void initState() {

super.initState();

_focusNode = FocusNode();

}

@override

void dispose() {

_commentController.dispose();

_focusNode.dispose();

super.dispose();

}

Future<void> _sendReactionMessage() async {

final comment = _commentController.text.trim();

if (comment.isEmpty) return;

setState(() {

_isSending = true;

});

final token = await storage.read(key: 'jwt_token');

if (token == null) {

setState(() {

_isSending = false;

});

debugPrint("Authorization token missing");

return;

}

final receiverId = widget.reactionPayload['receiver_id'];

if (receiverId == null || receiverId.isEmpty) {

debugPrint("Missing 'receiver_id' in reactionPayload");

setState(() {

_isSending = false;

});

return;

}

final timestamp =

widget.reactionPayload['timestamp'] ?? DateTime.now().toIso8601String();

final type = widget.reactionPayload['type'] ?? 'reaction';

final requestBody = {

"receiver_id": receiverId,

"timestamp": timestamp,

"type": type,

"message": comment,

};

final url = Uri.parse("${Config.backendBaseUrl}/chat/sendReaction.php");

try {

final response = await http.post(

url,

headers: {

"Authorization": "Bearer $token",

"Content-Type": "application/json",

},

body: jsonEncode(requestBody),

);

if (response.statusCode == 200) {

final data = jsonDecode(response.body);

if (data["status"] == "sent") {

debugPrint("Reaction sent => chat created: ${data["chat_id"]}");

Navigator.pushReplacement(

context,

MaterialPageRoute(

builder: (context) => ChatPage(

chatId: data["chat_id"],

profileURL: data["profileURL"],

receiver: data["receiver"],

),

),

);

} else {

debugPrint("Error sending reaction: ${data["message"]}");

}

} else {

debugPrint(

"sendReaction.php error: ${response.statusCode} ${response.body}");

}

} catch (e) {

debugPrint("Exception in _sendReactionMessage: $e");

}

setState(() {

_isSending = false;

});

}

Widget _buildCommentInput() {

return Container(

decoration: BoxDecoration(

color: Colors.white,

boxShadow: const [

BoxShadow(

color: Colors.black12, offset: Offset(0, -2), blurRadius: 4),

],

),

padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),

child: Row(

children: [

Expanded(

child: TextField(

controller: _commentController,

focusNode: _focusNode,

cursorColor: Colors.pinkAccent,

style: const TextStyle(color: Colors.black87),

decoration: const InputDecoration(

hintText: "Add a comment...",

hintStyle: TextStyle(color: Colors.grey),

border: InputBorder.none,

contentPadding: EdgeInsets.only(left: 8),

),

),

),

GestureDetector(

onTap: _sendReactionMessage,

child: Container(

width: _isSending ? 60 : 40,

height: _isSending ? 60 : 40,

decoration: BoxDecoration(

shape: BoxShape.circle,

gradient: const LinearGradient(

colors: [Colors.pinkAccent, Colors.orangeAccent],

begin: Alignment.topLeft,

end: Alignment.bottomRight,

),

),

child: const Icon(Icons.send, color: Colors.white),

),

),

],

),

);

}

// Optional pinned element or header

Widget _buildPinnedElement() {

final reactionType = widget.reactionPayload['type'] ?? 'reaction';

final content = widget.reactionPayload['content'] ?? {};

if (reactionType == 'react_bubble' ||

reactionType == 'react_top_section_bubble') {

final bubbleText =

content['bubble_label'] ?? content['bubble_text'] ?? 'React';

return Padding(

padding: const EdgeInsets.symmetric(vertical: 8),

child: Chip(

label: Text(

bubbleText,

style: AppFonts.tertiaryFont.copyWith(

fontSize: 14,

fontWeight: FontWeight.w600,

color: AppColors.text),

),

backgroundColor: const Color(0xFFF5F5F5),

padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),

),

);

}

return const SizedBox.shrink();

}

@override

Widget build(BuildContext context) {

return Dialog(

elevation: 0,

backgroundColor: Colors.white,

shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),

insetPadding: const EdgeInsets.symmetric(horizontal: 10, vertical: 40),

child: SizedBox(

height: 500,

child: Column(

crossAxisAlignment: CrossAxisAlignment.stretch,

children: [

// (Optional) add a header or pinned element

_buildPinnedElement(),

const Divider(thickness: 1, height: 1, color: Colors.grey),

// The input field

_buildCommentInput(),

],

),

),

);

}

}

"

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

import 'package:swipe_chat_play/tabs/chat/new_bubbles/answer_to_message_layout.dart';

import 'package:swipe_chat_play/tabs/chat/new_bubbles/location_bubble_layout.dart';

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

// Shared ProfileData & ProfileCard

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import '../../profile/layouts/profile_layout.dart';

// Ensure this import points to the file containing your ProfileCard code.

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

// Mapping from message id to its index in the list view

final Map<String, int> _messageIdToIndex = {};

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

// --------------------

// Reply/Answer State

// --------------------

Map<String, dynamic>? _replyTo;

@override

void initState() {

super.initState();

_focusNode = FocusNode();

// Listen to text changes to toggle send/mic button

_messageController.addListener(() {

setState(() {});

});

// Attach-file menu animation

_menuController = AnimationController(

vsync: this,

duration: const Duration(milliseconds: 300),

);

_slideAnimation =

Tween<Offset>(begin: const Offset(0, 1), end: Offset.zero).animate(

CurvedAnimation(parent: _menuController, curve: Curves.easeInOut),

);

// Scroll listener for “scroll to bottom” button logic

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

// Fetch chat data & load local

fetchChatData();

loadChatData();

_fetchReceiverProfile();

// Initialize the recorder

_initRecorder();

// Focus textfield immediately after build

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

_recorder.closeRecorder();

super.dispose();

}

// --------------------------------------------------------------------------

// Voice Recording Methods

// --------------------------------------------------------------------------

Future<void> _initRecorder() async {

await Permission.microphone.request();

await _recorder.openRecorder();

}

Future<void> _startRecording() async {

if (_recorder.isRecording) return;

final tempDir = await getTemporaryDirectory();

final filePath =

"${tempDir.path}/${DateTime.now().millisecondsSinceEpoch}.aac";

await _recorder.startRecorder(toFile: filePath, codec: Codec.aacADTS);

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

final msgId = DateTime.now().millisecondsSinceEpoch.toString();

final localTimestamp = DateTime.now().toIso8601String();

setState(() {

chatData![msgId] = {

"type": "voicemessage",

"sender_id": "1",

"timestamp": localTimestamp,

"status": "uploading",

"audio_url": null,

};

});

_scrollToBottom();

await saveChatData();

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

// FETCH / LOAD / SAVE Chat & Profile

// --------------------------------------------------------------------------

Future<void> _fetchReceiverProfile() async {

final token = await storage.read(key: 'jwt_token');

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

"Content-Type": "application/json"

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

final token = await storage.read(key: 'jwt_token');

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

"Content-Type": "application/json"

},

body: jsonEncode({"chat_id": widget.chatId}),

);

setState(() => _isLoading = false);

if (response.statusCode == 200) {

final data = jsonDecode(response.body);

if (data["status"] == "success") {

setState(() => chatData = data["chat"]);

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

// Reply / Swipe-to-Reply Methods

// --------------------------------------------------------------------------

void _onMessageSwipedToReply(

Map<String, dynamic> messageData, String messageId) {

setState(() {

_replyTo = {

"message_id": messageId,

"sender_id": messageData["sender_id"],

"type": messageData["type"],

"message": messageData["message"],

"content": messageData["content"],

};

});

_scrollToBottom();

}

void _cancelReply() {

setState(() {

_replyTo = null;

});

}

Widget _buildReplyReferenceCard() {

if (_replyTo == null) return const SizedBox.shrink();

final referencedType = _replyTo!["type"] ?? "text";

final referencedText = (referencedType == "text")

? _replyTo!["message"] ?? ""

: "($referencedType)";

return Card(

margin: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),

color: Colors.grey.shade300,

child: ListTile(

title: const Text(

"Replying to:",

style: TextStyle(fontWeight: FontWeight.bold, fontSize: 13),

),

subtitle: Text(

referencedText.toString(),

style: const TextStyle(fontSize: 12),

),

trailing: IconButton(

icon: const Icon(Icons.close),

onPressed: _cancelReply,

),

),

);

}

// --------------------------------------------------------------------------

// SEND MESSAGES

// --------------------------------------------------------------------------

Future<void> sendMessage(String message) async {

final messageId = DateTime.now().millisecondsSinceEpoch.toString();

final timestamp = DateTime.now().toIso8601String();

final token = await storage.read(key: 'jwt_token');

if (token == null) {

ScaffoldMessenger.of(context).showSnackBar(

const SnackBar(content: Text("Authorization token is missing.")),

);

return;

}

final bool isReply = _replyTo != null;

final String messageType = isReply ? "answer_to" : "text";

Map<String, dynamic> newMsgData = {

"type": messageType,

"sender_id": "1",

"timestamp": timestamp,

"status": "sending",

};

if (isReply) {

newMsgData["content"] = {

"answered_to": _replyTo!["type"],

"answered_to_id": _replyTo!["message_id"],

if (_replyTo!["type"] == "text")

"answered_to_content": _replyTo!["message"],

"answer": message,

};

} else {

newMsgData["message"] = message;

}

setState(() {

chatData![messageId] = newMsgData;

});

await saveChatData();

_scrollToBottom();

try {

final response = await http

.post(

Uri.parse("${Config.backendBaseUrl}/chat/sendMessage.php"),

headers: {

"Authorization": "Bearer $token",

"Content-Type": "application/json"

},

body: jsonEncode({

"chat_id": widget.chatId,

"timestamp": timestamp,

"type": messageType,

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

// Clear reply

setState(() {

_replyTo = null;

});

await saveChatData();

}

Future<void> _sendLocationMessage(double lat, double lng) async {

final messageId = DateTime.now().millisecondsSinceEpoch.toString();

final timestamp = DateTime.now().toIso8601String();

final token = await storage.read(key: 'jwt_token');

if (token == null) {

ScaffoldMessenger.of(context).showSnackBar(

const SnackBar(content: Text("Authorization token is missing.")),

);

return;

}

// Locally add bubble

setState(() {

chatData![messageId] = {

"type": "location",

"sender_id": "1",

"timestamp": timestamp,

"status": "sending",

"content": {"lat": lat, "lng": lng},

};

});

await saveChatData();

_scrollToBottom();

// Send to server

try {

final response = await http

.post(

Uri.parse("${Config.backendBaseUrl}/chat/sendMessage.php"),

headers: {

"Authorization": "Bearer $token",

"Content-Type": "application/json"

},

body: jsonEncode({

"chat_id": widget.chatId,

"timestamp": timestamp,

"type": "location",

"content": {"lat": lat, "lng": lng},

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

_scrollToBottom();

}

// Example "special" message (contract)

Future<void> _sendSpecialMessage(

String amount, String months, String rate) async {

final token = await storage.read(key: 'jwt_token');

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

"Content-Type": "application/json"

},

body: jsonEncode({

"chat_id": widget.chatId,

"type": "special",

"contract_details": {"amount": amount, "months": months, "rate": rate},

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

// BOTTOM SHEET: OPENS at 80% => DRAGS UP to 95% => content scrolls

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

// Convert raw JSON → ProfileData

profile.ProfileData? profileData;

try {

profileData = profile.ProfileData.fromJson(profileJson);

} catch (e) {

debugPrint('Error parsing profile JSON: $e');

ScaffoldMessenger.of(context).showSnackBar(

const SnackBar(content: Text("Invalid profile data.")),

);

return;

}

// Show modal bottom sheet from 80% to 95%.

showModalBottomSheet(

context: context,

isScrollControlled: true,

backgroundColor: Colors.transparent,

barrierColor: Colors.black.withOpacity(0.3),

builder: (BuildContext bottomCtx) {

return GestureDetector(

behavior: HitTestBehavior.opaque,

onTap: () => Navigator.of(bottomCtx).pop(),

child: Stack(

children: [

// tapping outside closes sheet

Positioned.fill(child: Container(color: Colors.transparent)),

Align(

alignment: Alignment.bottomCenter,

child: DraggableScrollableSheet(

initialChildSize: 0.8, // starts at 80% height

minChildSize: 0.8, // cannot go below 80%

maxChildSize: 0.95, // can drag up to 95%

// If on Flutter >= 3.7, you can snap to [0.8, 0.95].

snap: true,

snapSizes: const [0.8, 0.95],

expand: false,

builder: (ctx, scrollController) {

return ProfileCard(

data: profileData!,

scrollController: scrollController,

);

},

),

),

],

),

);

},

);

}

// --------------------------------------------------------------------------

// SCROLL / FORMAT HELPERS

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

void _scrollToReferencedMessage(String referencedId) {

final idx = _messageIdToIndex[referencedId];

if (idx == null) return;

final itemOffset = idx * 100.0;

_scrollController.animateTo(

itemOffset,

duration: const Duration(milliseconds: 300),

curve: Curves.easeInOut,

);

}

// --------------------------------------------------------------------------

// MAIN BUILD

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

child: (chatData == null)

? const Center(child: CircularProgressIndicator())

: ListView.builder(

controller: _scrollController,

padding: const EdgeInsets.all(8.0),

itemCount: chatData!.length,

itemBuilder: (context, index) {

final allKeys = chatData!.keys.toList();

final msgKey = allKeys[index];

final message = chatData![msgKey];

final bool isSender = (message["sender_id"] == "1");

final String currentDate =

formatDate(message["timestamp"]);

final String? previousDate = (index > 0)

? formatDate(

chatData![allKeys[index - 1]]["timestamp"])

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

final String actualId = message["id"] ?? msgKey;

_messageIdToIndex[actualId] = index;

return Column(

crossAxisAlignment: CrossAxisAlignment.center,

children: [

if (showDateBubble)

Padding(

padding:

const EdgeInsets.symmetric(vertical: 8),

child: Container(

padding: const EdgeInsets.symmetric(

horizontal: 10, vertical: 5),

decoration: BoxDecoration(

color: Colors.grey[300],

borderRadius: BorderRadius.circular(15),

),

child: Text(

currentDate,

style: const TextStyle(

color: Colors.black54,

fontSize: 12,

),

),

),

),

Align(

alignment: alignment,

child: _SwipableBubble(

isSender: isSender,

onReplySwipe: () => _onMessageSwipedToReply(

message,

msgKey,

),

child: _buildBubbleByType(

type: type,

message: message,

content: content,

isSender: isSender,

),

),

),

],

);

},

),

),

// Bottom input + reply reference

_buildReplyReferenceCard(),

Container(

decoration: BoxDecoration(

color: Colors.white,

boxShadow: const [

BoxShadow(

color: Colors.black12,

offset: Offset(0, -2),

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

if (_messageController.text.trim().isEmpty) {

_startRecording();

}

},

onTapUp: (details) async {

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

// Attach-file overlay

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

child: AttachFileLayout(

onLocationSelected: (double lat, double lng) {

_menuController.reverse().then((_) {

if (_wasKeyboardActive) {

FocusScope.of(context).requestFocus(_focusNode);

_wasKeyboardActive = false;

}

});

_sendLocationMessage(lat, lng);

},

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

// Decide bubble layout by "type"

Widget _buildBubbleByType({

required String type,

required Map<String, dynamic> message,

required Map<String, dynamic> content,

required bool isSender,

}) {

final timestamp =

formatTime(message["timestamp"] ?? DateTime.now().toIso8601String());

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

case "voicemessage":

final audioUrl = message["audio_url"] ?? "";

return SpeechMemoLayout(

key: ValueKey("voice_$audioUrl"),

audioUrl: audioUrl,

isSender: isSender,

timestamp: timestamp,

);

case "react_bubble":

return ReactBubbleLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_top_section_bubble":

return ReactTopSectionBubbleLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_icebreaker":

return ReactIcebreakerLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_recorded_message":

return ReactRecordedMessageLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_qa":

return ReactQALayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_interessen":

return ReactInteressenLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_multiple_choice":

return ReactMultipleChoiceLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_music":

return ReactMusicLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_profile_image_carousel":

return ReactProfileImageCarouselLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "react_profile_image":

return ReactProfileImageLayout(

content: content, isSender: isSender, timestamp: timestamp);

case "answer_to":

return AnswerToMessageLayout(

message: message,

onTap: () {

final referencedId = message["content"]?["answered_to_id"];

if (referencedId != null) {

_scrollToReferencedMessage(referencedId);

}

},

);

case "location":

return ChatLocationBubble(

content: content, isSender: isSender, timestamp: timestamp);

default:

final fallbackMsg = message["message"] ?? "[No text provided]";

return isSender

? ChatBubbleRightLayout(message: fallbackMsg, timestamp: timestamp)

: ChatBubbleLeftLayout(message: fallbackMsg, timestamp: timestamp);

}

}

}

// For swipe-to-reply detection

class _SwipableBubble extends StatefulWidget {

final Widget child;

final bool isSender;

final VoidCallback onReplySwipe;

const _SwipableBubble({

Key? key,

required this.child,

required this.isSender,

required this.onReplySwipe,

}) : super(key: key);

@override

__SwipableBubbleState createState() => __SwipableBubbleState();

}

class __SwipableBubbleState extends State<_SwipableBubble> {

double _dragOffset = 0.0;

static const double _answerSwipeThreshold = 100.0;

@override

Widget build(BuildContext context) {

return GestureDetector(

onHorizontalDragUpdate: (details) {

final delta = details.delta.dx;

if (delta > 0) {

setState(() => _dragOffset += delta);

}

},

onHorizontalDragEnd: (details) {

if (_dragOffset >= _answerSwipeThreshold) {

widget.onReplySwipe();

}

setState(() => _dragOffset = 0.0);

},

child: Stack(

alignment: Alignment.centerLeft,

children: [

Positioned(

left: 0,

child: Opacity(

opacity: (_dragOffset / _answerSwipeThreshold).clamp(0.0, 1.0),

child: const Text(

">",

style: TextStyle(

fontSize: 40,

fontWeight: FontWeight.bold,

color: Colors.grey,

),

),

),

),

Transform.translate(

offset: Offset(_dragOffset, 0),

child: widget.child,

),

],

),

);

}

}

"

Task: rewrite ChatPopupDialog. It should take as input the receiver_id and