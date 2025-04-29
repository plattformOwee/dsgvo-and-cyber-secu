import 'dart:async';

import 'dart:convert';

import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/fonts.dart';

// Chat page + popup

import 'package:swipe_chat_play/tabs/chat/layouts/chat.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat_popup.dart';

// Profile data & card

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/tabs/profile/layouts/profile_layout.dart'

show ProfileCard;

/// A swipeable stack of profile cards that supports both swipe left/right

/// and in‑card reactions (e.g. icebreaker questions). Tapping an element

/// inside the card opens a reaction chat popup.

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

// Show at most 2 cards at once: back + top

final int maxVisibleCards = 2;

final int prefetchThreshold = 2;

// Drag offset notifier for top card

final ValueNotifier<Offset> _dragPositionNotifier =

ValueNotifier(Offset.zero);

ScrollController _topCardScrollController = ScrollController();

// Swipe thresholds

double get swipeThreshold => MediaQuery.of(context).size.width * 0.07;

double get directionChangeThreshold =>

MediaQuery.of(context).size.width * 0.02;

// Gesture detection

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

if (isFetchingMore || (!reset && !hasMore)) return;

setState(() => reset ? isLoading = true : isFetchingMore = true);

final storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) {

setState(() => isLoading = false);

return;

}

final limit = reset ? 50 : 10;

try {

final response = await http.get(

Uri.parse(

'${Config.backendBaseUrl}/match_algorithm/match.php?page=$currentPage&limit=$limit'),

headers: {"Authorization": "Bearer $token"},

);

if (response.statusCode == 200) {

final data = jsonDecode(response.body);

if (data["status"] == "success") {

final profilesList = (data["data"] as List)

.map((j) => profile.ProfileData.fromJson(j))

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

final top = cardData.last;

setState(() => cardData.removeLast());

_sendSwipe(top, swipeValue);

_topCardScrollController.dispose();

_topCardScrollController = ScrollController();

_debouncer.run(() {

if (mounted && cardData.length <= prefetchThreshold && hasMore) {

_fetchProfiles();

}

});

}

Future<void> _sendSwipe(profile.ProfileData data, String swipeValue) async {

final storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) return;

final url = Uri.parse('${Config.backendBaseUrl}/match_algorithm/swipe.php');

final body = jsonEncode({

"receiver_id": data.topSection.id,

"swipe": swipeValue,

});

try {

final resp = await http.post(url,

headers: {

"Content-Type": "application/json",

"Authorization": "Bearer $token"

},

body: body);

if (resp.statusCode == 200) {

final clean = resp.body.replaceAll(RegExp(r'<br\s*/?>'), '').trim();

final rd = jsonDecode(clean);

if (rd["status"] == "match") {

Navigator.push(

context,

MaterialPageRoute(

builder: (_) => ChatPage(

chatId: rd["chat_id"],

profileURL: rd["profileURL"] ?? "",

receiver: rd["receiver"] ?? "",

),

),

);

}

}

} catch (e) {

debugPrint("Error sending swipe: $e");

}

}

void _onPanStart(DragStartDetails d) {

_gestureStart = d.globalPosition;

_decisionMade = false;

_isHorizontalSwipe = false;

}

void _onPanUpdate(DragUpdateDetails d) {

if (_gestureStart == null) return;

final dx = d.globalPosition.dx - _gestureStart!.dx;

final dy = d.globalPosition.dy - _gestureStart!.dy;

if (!_decisionMade) {

final dist = math.sqrt(dx * dx + dy * dy);

if (dist < _slopDistance) return;

_decisionMade = true;

final angle = math.atan(dy.abs() / dx.abs()) * (180 / math.pi);

_isHorizontalSwipe = angle <= 45;

}

if (_isHorizontalSwipe) {

final cur = _dragPositionNotifier.value;

final newDx = cur.dx + d.delta.dx;

final newDy = cur.dy + d.delta.dy * 0.7;

_dragPositionNotifier.value = Offset(newDx, newDy);

_updateSwipeDirection(newDx);

} else {

if (_topCardScrollController.hasClients) {

final change = -d.delta.dy;

final curOff = _topCardScrollController.offset;

final maxExt = _topCardScrollController.position.maxScrollExtent;

final newOff = math.min(math.max(curOff + change, 0.0), maxExt);

_topCardScrollController.jumpTo(newOff);

}

}

}

void _updateSwipeDirection(double dx) {

if (_currentDirection == null) {

if (dx > 0) {

_currentDirection = "right";

_furthestRightSoFar = dx;

} else {

_currentDirection = "left";

_furthestLeftSoFar = dx;

}

return;

}

if (_currentDirection == "right") {

if (dx > _furthestRightSoFar) {

_furthestRightSoFar = dx;

}

if (dx < 0 && dx < (_furthestRightSoFar - directionChangeThreshold)) {

_currentDirection = "left";

_furthestLeftSoFar = dx;

}

} else {

if (dx < _furthestLeftSoFar) {

_furthestLeftSoFar = dx;

}

if (dx > 0 && dx > (_furthestLeftSoFar + directionChangeThreshold)) {

_currentDirection = "right";

_furthestRightSoFar = dx;

}

}

}

void _onPanEnd(DragEndDetails d) {

if (_isHorizontalSwipe && cardData.isNotEmpty) {

final dx = _dragPositionNotifier.value.dx;

if (dx.abs() > swipeThreshold) {

_dismissCard(dx > 0 ? "yes" : "no");

}

} else {

if (_topCardScrollController.hasClients) {

final velY = d.velocity.pixelsPerSecond.dy;

final cur = _topCardScrollController.offset;

final maxE = _topCardScrollController.position.maxScrollExtent;

double tgt = cur - velY * 0.2;

tgt = math.max(0.0, math.min(tgt, maxE));

_topCardScrollController.animateTo(

tgt,

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

/// reactionPayload must be:

/// {

/// "type" : "react_bubble" or "react_icebreaker",

/// "content" : { bubble_text: "...", question: "...", answer: "..." },

/// "receiver_id": "<Mongo _id string>"

/// }

void _openReactionChat(Map<String, dynamic> reactionPayload) async {

// --- FIX: ensure that `reactionPayload["receiver_id"]` is not null ---

// You should normally set this in `ProfileCard` or wherever you build `reactionPayload`.

// But to be safe, let's do it here if it's missing:

if (!reactionPayload.containsKey("receiver_id") ||

reactionPayload["receiver_id"] == null) {

// We'll assume the top card is the card the user tapped:

if (visibleCards.isNotEmpty) {

final topProfile = visibleCards.last;

reactionPayload["receiver_id"] = topProfile.topSection.id;

}

}

// Ensure reaction type is properly set

if (!reactionPayload.containsKey("type")) {

debugPrint("Warning: reactionPayload is missing 'type'");

reactionPayload["type"] = "react_unknown";

}

// Ensure content exists

if (!reactionPayload.containsKey("content")) {

reactionPayload["content"] = {};

}

// Log what we're sending to the dialog

debugPrint("Opening chat popup with payload: $reactionPayload");

final result = await showDialog(

context: context,

barrierDismissible: true,

builder: (context) {

return ChatPopupDialog(reactionPayload: reactionPayload);

},

);

if (result == "NO_MATCH") {

// Dismiss the card after reaction if no match

_dismissCard("yes");

}

}

@override

Widget build(BuildContext context) {

if (!isLoading && cardData.isEmpty) {

return Container(

color: AppColors.background,

child: Center(

child: Text("No profiles left, change filters",

style: AppFonts.secondaryFont),

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

child: Stack(children: [

if (visibleCards.length > 1)

Positioned.fill(

child: AnimatedScale(

duration: const Duration(milliseconds: 200),

scale: 0.95,

child: ProfileCard(

data: visibleCards.first,

scrollController: ScrollController(),

onElementTap: _openReactionChat,

),

),

),

if (visibleCards.isNotEmpty)

Positioned.fill(

child: GestureDetector(

behavior: HitTestBehavior.translucent,

onPanStart: _onPanStart,

onPanUpdate: _onPanUpdate,

onPanEnd: _onPanEnd,

child: ValueListenableBuilder<Offset>(

valueListenable: _dragPositionNotifier,

builder: (_, offset, child) {

final dx = offset.dx;

final factor = (dx.abs() / swipeThreshold).clamp(0.0, 1.0);

return Opacity(

opacity: 1.0 - 0.2 * factor,

child: Transform.translate(

offset: offset,

child: AnimatedSwitcher(

duration: const Duration(milliseconds: 300),

transitionBuilder: (c, a) => ScaleTransition(

scale: Tween(begin: 0.95, end: 1.0).animate(

CurvedAnimation(

parent: a, curve: Curves.easeOutBack),

),

child: c,

),

child: RepaintBoundary(

key: ValueKey(visibleCards.last),

child: ProfileCard(

data: visibleCards.last,

scrollController: _topCardScrollController,

onElementTap: _openReactionChat,

),

),

),

),

);

},

),

),

),

SwipeIndicators(

dragNotifier: _dragPositionNotifier,

swipeThreshold: swipeThreshold,

),

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

]),

),

);

}

}

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

builder: (_, offset, __) {

final dx = offset.dx;

final dy = offset.dy;

final isH = dx.abs() >= dy.abs();

final prog = (dx.abs() / swipeThreshold).clamp(0.0, 1.0);

final leftOp = (dx < 0 && isH) ? prog : 0.0;

final rightOp = (dx > 0 && isH) ? prog : 0.0;

final vp = MediaQuery.of(context).size.height / 2 - 30;

return IgnorePointer(

child: Stack(children: [

Positioned(

left: 20,

top: vp,

child: AnimatedOpacity(

opacity: leftOp,

duration: const Duration(milliseconds: 100),

child: CircleAvatar(

radius: 30,

backgroundColor: Colors.red.withOpacity(0.9),

child: const Icon(Icons.close, color: Colors.white, size: 30),

),

),

),

Positioned(

right: 20,

top: vp,

child: AnimatedOpacity(

opacity: rightOp,

duration: const Duration(milliseconds: 100),

child: CircleAvatar(

radius: 30,

backgroundColor: Colors.green.withOpacity(0.9),

child: const Icon(Icons.check, color: Colors.white, size: 30),

),

),

),

]),

);

},

);

}

}

class Debouncer {

final int milliseconds;

Timer? _timer;

Debouncer({required this.milliseconds});

void run(VoidCallback action) {

_timer?.cancel();

_timer = Timer(Duration(milliseconds: milliseconds), action);

}

void cancel() => _timer?.cancel();

}

"

// profile_layout.dart

import 'package:flutter/material.dart';

import 'package:cached_network_image/cached_network_image.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

/// A single card for a user's profile.

/// Accepts an onElementTap callback that child layouts can use to send properly formatted reaction payloads.

class ProfileCard extends StatelessWidget {

final profile.ProfileData data;

final ScrollController scrollController;

/// Callback when an element is tapped.

final Function(Map<String, dynamic>)? onElementTap;

const ProfileCard({

Key? key,

required this.data,

required this.scrollController,

this.onElementTap,

}) : super(key: key);

@override

Widget build(BuildContext context) {

// The swiped user's username is stored in topSection.username.

final swipedUserUsername = data.topSection.name;

return Padding(

padding: const EdgeInsets.all(8.0),

child: Card(

color: AppColors.cardBackground,

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(16),

),

child: DefaultTextStyle(

style: AppFonts.primaryFont,

child: SingleChildScrollView(

controller: scrollController,

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

// Top section

TopSectionWidget(

topSection: data.topSection,

swipedUserUsername: swipedUserUsername,

onTopSectionTap: (bubbleText) {

// Process the bubble tap with proper reaction format

_onTopSectionBubbleTap(bubbleText);

},

),

for (final entry in data.elements.entries) ...[

const Divider(

height: 2,

thickness: 2,

color: AppColors.dividerLine,

),

switch (entry.value.type) {

'icebreaker' => IceBreakerLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onIcebreakerTap: (question, answer) {

// Process icebreaker tap with proper reaction format

_onIcebreakerTap(question, answer);

},

),

'image' => _buildImageLayout(

entry.value.content, swipedUserUsername),

'bubbles' => BubbleGridLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onBubbleTap: (bubbleText) {

// Process bubble tap with proper reaction format

_onBubbleTap(bubbleText);

},

),

_ => const SizedBox(),

}

],

],

),

),

),

),

);

}

/// Build image layout for a section

Widget _buildImageLayout(Map<String, dynamic> content, String username) {

final singleUrl = content['imageUrl'] as String?;

final imageUrls = content['imageUrls'] as List<dynamic>?;

final urls = <String>[];

if (singleUrl != null) {

urls.add(singleUrl);

} else if (imageUrls != null) {

urls.addAll(imageUrls.whereType<String>());

}

if (urls.isEmpty) return const SizedBox.shrink();

return SizedBox(

height: 200,

child: PageView.builder(

itemCount: urls.length,

itemBuilder: (context, index) {

return GestureDetector(

onTap: () => _onProfileImageTap(urls[index]),

child: CachedNetworkImage(

imageUrl: urls[index],

fit: BoxFit.cover,

placeholder: (ctx, url) => Container(color: Colors.grey[200]),

errorWidget: (ctx, url, error) => Container(

color: Colors.grey[400],

child: const Icon(Icons.error),

),

),

);

},

),

);

}

/// Process bubble tap with proper reaction format

void _onBubbleTap(String bubbleText) {

if (onElementTap != null) {

onElementTap!({

"type": "react_bubble",

"content": {

"bubble_text": bubbleText,

},

"receiver_id": data.topSection.id,

});

}

}

/// Process top section bubble tap with proper reaction format

void _onTopSectionBubbleTap(String bubbleText) {

if (onElementTap != null) {

onElementTap!({

"type": "react_top_section_bubble",

"content": {

"bubble_text": bubbleText,

},

"receiver_id": data.topSection.id,

});

}

}

/// Process icebreaker tap with proper reaction format

void _onIcebreakerTap(String question, String answer) {

if (onElementTap != null) {

onElementTap!({

"type": "react_icebreaker",

"content": {

"question": question,

"answer": answer,

},

"receiver_id": data.topSection.id,

});

}

}

/// Process profile image tap with proper reaction format

void _onProfileImageTap(String imageUrl) {

if (onElementTap != null) {

onElementTap!({

"type": "react_profile_image",

"content": {

"image_url": imageUrl,

},

"receiver_id": data.topSection.id,

});

}

}

}

"

"

// bubble_grid_layout.dart

import 'package:flutter/material.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

class BubbleGridLayout extends StatelessWidget {

final Map<String, dynamic> content;

/// The swiped user's username.

final String swipedUserUsername;

/// Callback when a bubble is tapped.

/// Now takes just the bubble text as parameter.

final Function(String)? onBubbleTap;

const BubbleGridLayout({

super.key,

required this.content,

required this.swipedUserUsername,

this.onBubbleTap,

});

@override

Widget build(BuildContext context) {

final title = content['title'] as String? ?? '';

final bubbles = (content['bubbles'] as List<dynamic>?) ?? [];

return Container(

width: double.infinity,

padding: const EdgeInsets.symmetric(horizontal: 25, vertical: 15),

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

if (title.isNotEmpty)

Padding(

padding: const EdgeInsets.only(left: 8),

child: Text(

title,

style: AppFonts.secondaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w400,

color: AppColors.text,

),

),

),

if (title.isNotEmpty) const SizedBox(height: 8),

Wrap(

spacing: 7,

runSpacing: 0,

alignment: WrapAlignment.spaceBetween,

children: [

for (final bubble in bubbles)

GestureDetector(

onTap: () {

// Use simplified callback with just the bubble text

onBubbleTap?.call(bubble.toString().trim());

},

child: Chip(

label: Text(

bubble.toString().trim(),

style: AppFonts.tertiaryFont.copyWith(

fontSize: 14,

fontWeight: FontWeight.w600,

color: AppColors.text,

),

),

backgroundColor: const Color(0xFFF5F5F5),

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(100),

side:

const BorderSide(color: Colors.transparent, width: 0),

),

),

),

],

),

],

),

);

}

}

"

"

"

// ice_breaker_layout.dart

import 'package:flutter/material.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

class IceBreakerLayout extends StatelessWidget {

static const double kHorizontalPadding = 25.0;

final Map<String, dynamic> content;

/// The swiped user's username.

final String swipedUserUsername;

/// Callback when the icebreaker is tapped.

/// Now takes question and answer as separate parameters.

final Function(String, String)? onIcebreakerTap;

const IceBreakerLayout({

super.key,

required this.content,

required this.swipedUserUsername,

this.onIcebreakerTap,

});

@override

Widget build(BuildContext context) {

final question = content['question'] ?? 'No question';

final answer = content['answer'] ?? 'No answer';

return GestureDetector(

onTap: () {

// Use simplified callback with just the question and answer

onIcebreakerTap?.call(question, answer);

},

child: Container(

width: double.infinity,

alignment: Alignment.topLeft,

padding: const EdgeInsets.fromLTRB(

kHorizontalPadding, 25, kHorizontalPadding, 25),

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

Text(

question,

style: AppFonts.secondaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w400,

color: AppColors.text,

),

),

const SizedBox(height: 5),

Text(

answer,

style: AppFonts.tertiaryFont.copyWith(

fontSize: 22,

fontWeight: FontWeight.w400,

color: AppColors.text,

),

),

const SizedBox(height: 20),

],

),

),

);

}

}"

"

// top_section.dart

import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

import '../models/profile_data.dart';

class TopSectionWidget extends StatelessWidget {

static const double kHorizontalPadding = 20.0;

final TopSection topSection;

/// The swiped user's username.

final String swipedUserUsername;

/// Callback for tapping parts of the top section.

final Function(String)? onTopSectionTap;

const TopSectionWidget({

super.key,

required this.topSection,

required this.swipedUserUsername,

this.onTopSectionTap,

});

@override

Widget build(BuildContext context) {

const double sectionSpacing = 20.0;

final pageController = PageController();

final currentPage = ValueNotifier<int>(0);

return Padding(

padding: const EdgeInsets.only(top: 0.0, bottom: 5.0),

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

// IMAGE STACK

Stack(

clipBehavior: Clip.none,

children: [

ClipRRect(

borderRadius: const BorderRadius.only(

topLeft: Radius.circular(16.0),

topRight: Radius.circular(16.0),

),

child: AspectRatio(

aspectRatio: 3 / 4,

child: PageView.builder(

controller: pageController,

itemCount: topSection.profileImages.length,

onPageChanged: (index) => currentPage.value = index,

physics: const NeverScrollableScrollPhysics(),

itemBuilder: (context, index) => GestureDetector(

onTapDown: (details) {

final screenWidth = MediaQuery.of(context).size.width;

final tapPosition = details.localPosition.dx;

if (tapPosition < screenWidth / 3 && index > 0) {

pageController.jumpToPage(index - 1);

} else if (tapPosition > 2 * screenWidth / 3 &&

index < topSection.profileImages.length - 1) {

pageController.jumpToPage(index + 1);

}

},

child: Image.network(

topSection.profileImages[index],

fit: BoxFit.cover,

),

),

),

),

),

Positioned(

top: 10,

left: 0,

right: 0,

child: Padding(

padding: const EdgeInsets.symmetric(horizontal: 5),

child: ValueListenableBuilder<int>(

valueListenable: currentPage,

builder: (context, value, child) {

return Row(

children: List.generate(

topSection.profileImages.length,

(idx) => Expanded(

child: Container(

height: 3,

margin: const EdgeInsets.symmetric(horizontal: 1),

decoration: BoxDecoration(

color: idx == value

? Colors.white

: Colors.grey[400],

borderRadius: BorderRadius.circular(3),

),

),

),

),

);

},

),

),

),

// Overlapping icons.

Positioned(

bottom: -22,

right: 10,

child: Row(

children: [

GestureDetector(

onTap: () {

// Use simplified callback with just the bubble text

onTopSectionTap?.call("Female");

},

child: CircleAvatar(

radius: 22,

backgroundColor: Colors.white,

child: Icon(Icons.navigation, color: Colors.grey[800]),

),

),

const SizedBox(width: 15),

CircleAvatar(

radius: 22,

backgroundColor: Colors.white,

child: Icon(Icons.chat_bubble, color: Colors.pinkAccent),

),

],

),

),

],

),

const SizedBox(height: sectionSpacing + 2),

// NAME, AGE

Padding(

padding: const EdgeInsets.symmetric(horizontal: kHorizontalPadding),

child: Text(

"${topSection.name}, ${topSection.age}",

style: AppFonts.primaryFont.copyWith(

fontSize: 26,

fontWeight: FontWeight.w400,

color: AppColors.text,

),

),

),

const SizedBox(height: sectionSpacing),

// INFO BUBBLES

Padding(

padding: const EdgeInsets.symmetric(horizontal: kHorizontalPadding),

child: Wrap(

spacing: 10,

children: topSection.infoBubbles.asMap().entries.map((entry) {

final index = entry.key;

final bubble = entry.value;

final iconAsset = _getIconForInfoBubble(bubble, index);

return GestureDetector(

onTap: () {

// Use simplified callback with just the bubble text

onTopSectionTap?.call(bubble);

},

child: Chip(

avatar: iconAsset.isNotEmpty

? SvgPicture.asset(iconAsset, width: 16, height: 16)

: null,

label: Text(

bubble,

style: AppFonts.secondaryFont.copyWith(

fontSize: 14,

fontWeight: FontWeight.w600,

color: AppColors.text,

),

),

backgroundColor: const Color(0xFFF5F5F5),

padding:

const EdgeInsets.symmetric(horizontal: 12, vertical: 8),

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(100),

side:

const BorderSide(color: Colors.transparent, width: 0),

),

),

);

}).toList(),

),

),

const SizedBox(height: 40),

],

),

);

}

String _extractIconName(String path) {

final parts = path.split('/');

return parts.isNotEmpty ? parts.last : "";

}

String _getIconForInfoBubble(String bubble, int index) {

if (index == 0) {

final lower = bubble.toLowerCase();

if (lower == 'female') return 'assets/icons/female.svg';

if (lower == 'male') return 'assets/icons/male.svg';

return 'assets/icons/transgender.svg';

}

switch (index) {

case 1:

return 'assets/icons/language.svg';

case 2:

return 'assets/icons/religion.svg';

case 3:

return 'assets/icons/politics.svg';

default:

return '';

}

}

}

""

"

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'dart:convert';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat.dart';

/// Popup that lets the user add a text/audio reaction to any

/// profile element (bubble, ice‑breaker, profile image, …).

class ChatPopupDialog extends StatefulWidget {

/// The element the user is reacting to:

/// ```

/// {

/// "type" : "react_bubble" | "react_icebreaker" | "react_profile_image",

/// "content" : { … }, // element‑specific data

/// "receiver_id": "abc123" // profile owner

/// }

/// ```

final Map<String, dynamic> reactionPayload;

const ChatPopupDialog({

Key? key,

required this.reactionPayload,

}) : super(key: key);

@override

State<ChatPopupDialog> createState() => _ChatPopupDialogState();

}

class _ChatPopupDialogState extends State<ChatPopupDialog> {

final storage = const FlutterSecureStorage();

final TextEditingController _commentController = TextEditingController();

late FocusNode _focusNode;

bool _isRecording = false;

bool _isSending = false;

@override

void initState() {

super.initState();

_focusNode = FocusNode();

debugPrint("[ChatPopupDialog] initState → ${widget.reactionPayload}");

}

@override

void dispose() {

_commentController.dispose();

_focusNode.dispose();

super.dispose();

}

@override

Widget build(BuildContext context) {

final reactionType = widget.reactionPayload['type'] ?? 'react_unknown';

final content = widget.reactionPayload['content'] ?? {};

return Dialog(

elevation: 0,

backgroundColor: Colors.white,

shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),

insetPadding: const EdgeInsets.symmetric(horizontal: 10, vertical: 40),

child: SizedBox(

width: double.infinity,

height: 360,

child: Column(

mainAxisSize: MainAxisSize.min,

children: [

_buildCloseButton(),

_buildPinnedElement(reactionType, content),

const Divider(thickness: 1, height: 1, color: Colors.grey),

_buildCommentInput(),

],

),

),

);

}

//---------------------------------------------------------------------------

// UI PARTS

//---------------------------------------------------------------------------

Widget _buildCloseButton() => Row(

mainAxisAlignment: MainAxisAlignment.end,

children: [

IconButton(

icon: const Icon(Icons.close, color: Colors.grey),

onPressed: () => Navigator.of(context).pop(),

),

],

);

/// Renders the element the user is reacting to (bubble, ice‑breaker, …)

Widget _buildPinnedElement(

String reactionType, Map<String, dynamic> content) {

switch (reactionType) {

case 'react_bubble':

case 'react_top_section_bubble':

final label = content['bubble_label'] ?? content['bubble_text'] ?? '';

return Padding(

padding: const EdgeInsets.only(bottom: 10),

child: Chip(

label: Text(

label,

style: AppFonts.tertiaryFont.copyWith(

fontSize: 14,

fontWeight: FontWeight.w600,

color: AppColors.text,

),

),

backgroundColor: const Color(0xFFF5F5F5),

padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(100),

side: BorderSide.none,

),

),

);

case 'react_icebreaker':

final question = content['question'] ?? '';

final answer = content['answer'] ?? '';

return Container(

width: double.infinity,

alignment: Alignment.topLeft,

padding: const EdgeInsets.all(25),

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

const SizedBox(height: 20),

Text(question,

style: AppFonts.secondaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w400,

color: AppColors.text,

)),

const SizedBox(height: 5),

Text(answer,

style: AppFonts.tertiaryFont.copyWith(

fontSize: 22,

fontWeight: FontWeight.w400,

color: AppColors.text,

)),

const SizedBox(height: 20),

],

),

);

case 'react_profile_image':

final url = content['image_url'] ?? '';

return Padding(

padding: const EdgeInsets.only(bottom: 10),

child: Column(

children: [

Text("Reacting to image:", style: AppFonts.primaryFont),

const SizedBox(height: 5),

ClipRRect(

borderRadius: BorderRadius.circular(8),

child: Image.network(

url,

width: 120,

height: 120,

fit: BoxFit.cover,

),

),

],

),

);

default:

return const SizedBox.shrink();

}

}

/// Text‑input row (matches Chat page styling)

Widget _buildCommentInput() {

return Container(

decoration: const BoxDecoration(

color: Colors.white,

boxShadow: [

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

hintText: "Add a comment…",

hintStyle: TextStyle(color: Colors.grey),

border: InputBorder.none,

contentPadding: EdgeInsets.only(left: 8),

),

),

),

GestureDetector(

onTapUp: (_) async {

if (_commentController.text.trim().isNotEmpty) {

await _sendReactionMessage();

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

colors: [Colors.pinkAccent, Colors.orangeAccent],

begin: Alignment.topLeft,

end: Alignment.bottomRight,

),

),

child: Icon(

_commentController.text.trim().isNotEmpty

? Icons.send

: (_isRecording ? Icons.mic : Icons.mic_none),

color: Colors.white,

),

),

),

],

),

);

}

//---------------------------------------------------------------------------

// NETWORK PART

//---------------------------------------------------------------------------

/// POSTs a reaction to `match_algorithm/sendReaction.php`

Future<void> _sendReactionMessage() async {

final comment = _commentController.text.trim();

if (comment.isEmpty) return;

setState(() => _isSending = true);

try {

final token = await storage.read(key: 'jwt_token');

if (token == null) throw Exception("Missing JWT token");

// --- build request body ------------------------------------------------

final receiverId = widget.reactionPayload['receiver_id'];

final reactionType = widget.reactionPayload['type'];

final content =

Map<String, dynamic>.from(widget.reactionPayload['content'] ?? {});

content['comment'] = comment; // merge user text in

final requestBody = {

"type": reactionType, // react_bubble | react_icebreaker | …

"content": content,

"receiver_id": receiverId,

};

debugPrint("→ sendReaction.php payload: $requestBody");

// ----------------------------------------------------------------------

final res = await http.post(

Uri.parse("${Config.backendBaseUrl}/match_algorithm/sendReaction.php"),

headers: {

"Authorization": "Bearer $token",

"Content-Type": "application/json",

},

body: jsonEncode(requestBody),

);

debugPrint("← ${res.statusCode}: ${res.body}");

if (res.statusCode != 200) throw Exception(res.body);

final resp = jsonDecode(res.body);

if (resp['status'] == 'match') {

// already matched – open chat

if (mounted) Navigator.of(context).pop();

if (mounted) {

Navigator.push(

context,

MaterialPageRoute(

builder: (_) => ChatPage(

chatId: resp['chat_id'],

profileURL: resp['profileURL'] ?? '',

receiver: resp['receiver'] ?? '',

),

),

);

}

} else {

// no match yet – just close the popup

if (mounted) Navigator.of(context).pop("NO_MATCH");

}

_commentController.clear(); // reset input

} catch (e, st) {

debugPrint("sendReaction error: $e\n$st");

if (mounted) {

ScaffoldMessenger.of(context).showSnackBar(

SnackBar(content: Text("Couldn’t send reaction. Please try again.")),

);

}

} finally {

if (mounted) setState(() => _isSending = false);

}

}

}

"

"

// profile_layout.dart

import 'package:flutter/material.dart';

import 'package:cached_network_image/cached_network_image.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

/// A single card for a user's profile.

/// Accepts an onElementTap callback that child layouts can use to send "react_..." payloads.

class ProfileCard extends StatelessWidget {

final profile.ProfileData data;

final ScrollController scrollController;

/// Callback when an element is tapped.

final Function(Map<String, dynamic>)? onElementTap;

const ProfileCard({

Key? key,

required this.data,

required this.scrollController,

this.onElementTap,

}) : super(key: key);

@override

Widget build(BuildContext context) {

// The swiped user's username is stored in topSection.username.

final swipedUserUsername = data.topSection.username;

return Padding(

padding: const EdgeInsets.all(8.0),

child: Card(

color: AppColors.cardBackground,

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(16),

),

child: DefaultTextStyle(

style: AppFonts.primaryFont,

child: SingleChildScrollView(

controller: scrollController,

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

// Top section

TopSectionWidget(

topSection: data.topSection,

swipedUserUsername: swipedUserUsername,

onTopSectionTap: _onTopSectionBubbleTap,

),

for (final entry in data.elements.entries) ...[

const Divider(

height: 2,

thickness: 2,

color: AppColors.dividerLine,

),

switch (entry.value.type) {

'icebreaker' => IceBreakerLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onIcebreakerTap: _onIcebreakerTap,

),

'image' => _CachedImageLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onImageTap: _onProfileImageTap,

),

'bubbles' => BubbleGridLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onBubbleTap: _onBubbleTap,

),

_ => const SizedBox(),

}

],

],

),

),

),

),

);

}

/// Process bubble tap with proper reaction format

void _onBubbleTap(String bubbleText) {

if (onElementTap != null) {

onElementTap!({

"type": "react_bubble",

"content": {

"bubble_text": bubbleText,

},

"receiver_id": data.topSection.id,

});

}

}

/// Process top section bubble tap with proper reaction format

void _onTopSectionBubbleTap(String bubbleText) {

if (onElementTap != null) {

onElementTap!({

"type": "react_top_section_bubble",

"content": {

"bubble_text": bubbleText,

},

"receiver_id": data.topSection.id,

});

}

}

/// Process icebreaker tap with proper reaction format

void _onIcebreakerTap(String question, String answer) {

if (onElementTap != null) {

onElementTap!({

"type": "react_icebreaker",

"content": {

"question": question,

"answer": answer,

},

"receiver_id": data.topSection.id,

});

}

}

/// Process profile image tap with proper reaction format

void _onProfileImageTap(String imageUrl) {

if (onElementTap != null) {

onElementTap!({

"type": "react_profile_image",

"content": {

"image_url": imageUrl,

},

"receiver_id": data.topSection.id,

});

}

}

}

/// Minimal "YES"/"NO" indicators.

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

/// Internal image layout that produces a "react_profile_image" payload.

class _CachedImageLayout extends StatefulWidget {

final dynamic content;

final String swipedUserUsername;

final Function(String)? onImageTap;

const _CachedImageLayout({

Key? key,

required this.content,

required this.swipedUserUsername,

this.onImageTap,

}) : super(key: key);

@override

_CachedImageLayoutState createState() => _CachedImageLayoutState();

}

class _CachedImageLayoutState extends State<_CachedImageLayout> {

late PageController _pageController;

int _currentPage = 0;

@override

void initState() {

super.initState();

_pageController = PageController();

}

@override

void dispose() {

_pageController.dispose();

super.dispose();

}

List<String> _extractUrls(dynamic content) {

if (content is Map<String, dynamic>) {

final maybeUrls = content['imageUrls'];

final singleUrl = content['imageUrl'];

if (maybeUrls is List) {

return maybeUrls.whereType<String>().toList();

} else if (singleUrl is String) {

return [singleUrl];

}

}

return [];

}

void _onImageTapped(String imageUrl) {

// Just pass the URL to the parent method

widget.onImageTap?.call(imageUrl);

}

@override

Widget build(BuildContext context) {

final urls = _extractUrls(widget.content);

if (urls.isEmpty) return const SizedBox();

return SizedBox(

height: 200,

child: PageView.builder(

controller: _pageController,

itemCount: urls.length,

onPageChanged: (index) => setState(() => _currentPage = index),

itemBuilder: (context, index) {

final imageUrl = urls[index];

return GestureDetector(

onTap: () => _onImageTapped(imageUrl),

child: CachedNetworkImage(

imageUrl: imageUrl,

fit: BoxFit.cover,

placeholder: (ctx, url) => Container(color: Colors.grey[200]),

errorWidget: (ctx, url, error) => Container(

color: Colors.grey[400],

child: const Icon(Icons.error),

),

),

);

},

),

);

}

}

"
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

  

// Loop over all messages in this chat

foreach ($chatData as $messageId => $message) {

    // Convert sender_id to "1" or "2" depending on who is logged in

    if ($message->sender_id === $userId) {

        $message->sender_id = "1";

    } else {

        $message->sender_id = "2";

    }

    // If this embedded message has its own Mongo _id, make it a string

    if (isset($message->_id)) {

        $message->_id = (string)$message->_id;

    }

}

  

// Then build the response JSON, e.g.

$response = [

    "status" => "success",

    "chat" => $chatData  // now each message has its real Mongo _id

];

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
sendReaction.php
"
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

// Set JSON header

header('Content-Type: application/json');

  

// Disable on-screen errors, enable logging to a file:

ini_set('display_errors', '0');

ini_set('display_startup_errors', '0');

ini_set('log_errors', '1');

ini_set('error_log', __DIR__ . '/my_custom_errors.log');

error_reporting(E_ALL);

  

// Only allow POST requests

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["status" => "error", "message" => "Method not allowed"]);

    exit;

}

  

// JWT check, etc.

$headers = getallheaders();

if (!isset($headers['Authorization'])) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Missing Authorization header"]);

    exit;

}

$authHeader = $headers['Authorization'];

if (strpos($authHeader, "Bearer ") !== 0) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Invalid Authorization header"]);

    exit;

}

$jwtToken = trim(str_replace("Bearer ", "", $authHeader));

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $senderId = $decodedToken->user_id ?? null;

} catch (Exception $e) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Invalid token"]);

    exit;

}

if (!$senderId) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Invalid token"]);

    exit;

}

  

// Read POST data

$input = json_decode(file_get_contents('php://input'), true);

$reactTo = $input['react_to'] ?? [];    // Extra data about the reaction

$comment = $input['comment'] ?? "";    // The user’s typed comment

$receiverIdInput = $input['receiver_id'] ?? null;

if (!$receiverIdInput) {

    http_response_code(400);

    echo json_encode(["status" => "error", "message" => "Missing receiver_id"]);

    exit;

}

  

// MongoDB connection

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

// Query sender

$senderFilter = ['_id' => new ObjectId($senderId)];

$senderQuery  = new MongoDB\Driver\Query($senderFilter);

$senderCursor = $mongoManager->executeQuery("swipe_chat_play.users", $senderQuery);

$sender       = current($senderCursor->toArray());

if (!$sender) {

    http_response_code(404);

    echo json_encode(["status" => "error", "message" => "Sender not found"]);

    exit;

}

  

// Query receiver

$receiverFilter = ['_id' => new ObjectId($receiverIdInput)];

$receiverQuery  = new MongoDB\Driver\Query($receiverFilter);

$receiverCursor = $mongoManager->executeQuery("swipe_chat_play.users", $receiverQuery);

$receiver       = current($receiverCursor->toArray());

if (!$receiver) {

    http_response_code(404);

    echo json_encode(["status" => "error", "message" => "Receiver not found"]);

    exit;

}

$receiverId = (string)$receiver->_id;

  

// Check matches doc

$matchQueryFilter = [

    "users.$senderId"   => ['$exists' => true],

    "users.$receiverId" => ['$exists' => true]

];

$matchQuery  = new MongoDB\Driver\Query($matchQueryFilter);

$matchCursor = $mongoManager->executeQuery("swipe_chat_play.matches", $matchQuery);

$matchDoc    = current($matchCursor->toArray());

  

$bulk = new MongoDB\Driver\BulkWrite();

$swipeValue = "yes"; // Reaction => right-swipe

  

if ($matchDoc) {

    // Update existing doc

    $updateField = "users.$senderId";

    $bulk->update(

        ['_id' => $matchDoc->_id],

        ['$set' => [$updateField => $swipeValue]],

        ['multi' => false, 'upsert' => false]

    );

    $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

  

    // Reload doc

    $query     = new MongoDB\Driver\Query(['_id' => $matchDoc->_id]);

    $cursor    = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

    $updatedDoc= current($cursor->toArray());

    $users     = $updatedDoc->users;

    $swiperSwipe = $users->$senderId    ?? null;

    $swipedSwipe = $users->$receiverId  ?? null;

  

    if ($swiperSwipe === "yes" && $swipedSwipe === "yes") {

        // Already matched => either chat_id present or create new

        if (isset($updatedDoc->chat_id)) {

            $chatIdString = $updatedDoc->chat_id;

            // Insert reaction message using the "sendMessage" style

            insertChatMessage($mongoManager, $chatIdString, $senderId, $comment, $reactTo);

            echo json_encode([

                "status"     => "match",

                "chat_id"    => $chatIdString,

                "profileURL" => $updatedDoc->receiver_profileURL ?? "",

                "receiver_id"=> $receiverId,

                "receiver"   => $receiver->username

            ]);

            exit;

        }

        // Otherwise create new chat

        $chatIdString = createNewChatAndLink($mongoManager, $sender, $receiver);

        // Insert the first reaction message

        insertChatMessage($mongoManager, $chatIdString, $senderId, $comment, $reactTo);

  

        // Update matches doc to store the chat_id

        $bulkMatch = new MongoDB\Driver\BulkWrite();

        $bulkMatch->update(

            ['_id' => $updatedDoc->_id],

            ['$set' => ["chat_id" => $chatIdString]]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatch);

  

        echo json_encode([

            "status"     => "match",

            "chat_id"    => $chatIdString,

            "profileURL" => "",

            "receiver_id"=> $receiverId,

            "receiver"   => $receiver->username

        ]);

        exit;

    } else {

        // Not yet a double-yes => no match

        echo json_encode(["status" => "no match"]);

        exit;

    }

} else {

    // No matches doc => create one

    $newDoc = [

        "users" => [

            $senderId   => $swipeValue,

            $receiverId => null

        ]

    ];

    $bulk->insert($newDoc);

    $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

  

    echo json_encode(["status" => "no match"]);

    exit;

}

  

//--------------------------------------

// Helper functions

//--------------------------------------

  

function createNewChatAndLink($mongoManager, $sender, $receiver) {

    $senderId   = (string)$sender->_id;

    $receiverId = (string)$receiver->_id;

  

    $chatData = [

        "messages" => new \stdClass(),

        "participants" => [$senderId, $receiverId]

    ];

    $bulkChat = new MongoDB\Driver\BulkWrite();

    $chatId = $bulkChat->insert($chatData);

    $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkChat);

    $chatIdString = (string)$chatId;

  

    // (Optional) set profile images

    $senderProfileImage = "";

    $receiverProfileImage = "";

  

    // Link in user docs

    $bulkUpdate = new MongoDB\Driver\BulkWrite();

    $bulkUpdate->update(

        ["_id" => new ObjectId($senderId)],

        ['$set' => ["chats.$chatIdString" => [

            "profileURL"   => $receiverProfileImage,

            "receiver_id"  => $receiverId,

            "receiver"     => $receiver->username,

            "last_message" => "",

            "unread"       => 0

        ]]]

    );

    $bulkUpdate->update(

        ["_id" => new ObjectId($receiverId)],

        ['$set' => ["chats.$chatIdString" => [

            "profileURL"   => $senderProfileImage,

            "receiver_id"  => $senderId,

            "receiver"     => $sender->username,

            "last_message" => "",

            "unread"       => 0

        ]]]

    );

    $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUpdate);

  

    return $chatIdString;

}

  

/**

 * Inserts the reaction as a new message in the chat, using the same style as sendMessage.php.

 * We'll treat it as type="reaction", store the typed comment as "message",

 * and store the "react_to" data in "content".

 */

function insertChatMessage($mongoManager, $chatIdString, $senderId, $type, $content) {

    $messageId  = uniqid();

    $timestamp  = (new DateTimeImmutable('now', new DateTimeZone('UTC')))

                     ->format(DateTime::ISO8601);  // 2025‑04‑03T10:05:00+0000

  

    $newMessage = [

        "sender_id" => $senderId,

        "type"      => $type,      // react_bubble | react_icebreaker

        "timestamp" => $timestamp,

        "status"    => "delivered",

        "content"   => $content    // already contains the user’s comment

    ];

  

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update(

        ['_id' => new ObjectId($chatIdString)],

        ['$set' => ["messages.$messageId" => $newMessage]],

        ['upsert' => true]

    );

    $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk);

  

    /* Optional: use the comment itself as last_message */

    if (!empty($content['comment'])) {

        $bulkUsers = new MongoDB\Driver\BulkWrite();

        $bulkUsers->update(

            ["chats.$chatIdString" => ['$exists' => true]],

            ['$set' => ["chats.$chatIdString.last_message" => $content['comment']]],

            ['multi' => true]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUsers);

    }

}
"
swipe.php:"
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

ini_set('display_errors', '0');         // Turn off showing errors in HTML

ini_set('display_startup_errors', '0'); // Same for startup errors

ini_set('log_errors', '1');             // Enable logging

ini_set('error_log', __DIR__ . '/my_custom_errors.log');

error_reporting(E_ALL);                 // Report all errors, but only to the log file

  
  

// Only allow POST requests.

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["status" => "error", "message" => "Method not allowed"]);

    exit;

}

  

// Retrieve JWT from the Authorization header.

$headers = getallheaders();

if (!isset($headers['Authorization'])) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Missing Authorization header"]);

    exit;

}

$authHeader = $headers['Authorization'];

if (strpos($authHeader, "Bearer ") !== 0) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Invalid Authorization header"]);

    exit;

}

$jwtToken = trim(str_replace("Bearer ", "", $authHeader));

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $senderId = $decodedToken->user_id ?? null;

} catch(Exception $e) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Invalid token"]);

    exit;

}

if (!$senderId) {

    http_response_code(401);

    echo json_encode(["status" => "error", "message" => "Invalid token"]);

    exit;

}

  

// Connect to MongoDB and get sender (swiper) information.

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

$senderFilter = ['_id' => new ObjectId($senderId)];

$senderQuery = new MongoDB\Driver\Query($senderFilter);

$senderCursor = $mongoManager->executeQuery("swipe_chat_play.users", $senderQuery);

$sender = current($senderCursor->toArray());

if (!$sender) {

    http_response_code(404);

    echo json_encode(["status" => "error", "message" => "Sender not found"]);

    exit;

}

  

// Get JSON input.

$input = json_decode(file_get_contents('php://input'), true);

$inputReceiverId = $input['receiver_id'] ?? null; // Now using the unique id directly

$swipeValue = $input['swipe'] ?? null;

if (!$inputReceiverId || !$swipeValue) {

    http_response_code(400);

    echo json_encode(["status" => "error", "message" => "receiver_id and swipe are required"]);

    exit;

}

  

// Look up the receiver by _id.

$receiverFilter = ['_id' => new ObjectId($inputReceiverId)];

$receiverQuery = new MongoDB\Driver\Query($receiverFilter);

$receiverCursor = $mongoManager->executeQuery("swipe_chat_play.users", $receiverQuery);

$receiver = current($receiverCursor->toArray());

if (!$receiver) {

    http_response_code(404);

    echo json_encode(["status" => "error", "message" => "Receiver not found"]);

    exit;

}

$receiverId = (string)$receiver->_id;

  

// Build query filter using sender and receiver IDs (used as keys in the match document)

$queryFilter = [

    "users.$senderId"   => ['$exists' => true],

    "users.$receiverId" => ['$exists' => true]

];

$query = new MongoDB\Driver\Query($queryFilter);

$cursor = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

$matchDoc = current($cursor->toArray());

$bulk = new MongoDB\Driver\BulkWrite();

  

if ($matchDoc) {

    // Update the current user’s (swiper’s) swipe.

    $updateField = "users.$senderId";

    $bulk->update(

        ['_id' => $matchDoc->_id],

        ['$set' => [$updateField => $swipeValue]],

        ['multi' => false, 'upsert' => false]

    );

    $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

  

    // Re-read the document to check the swipe status.

    $query = new MongoDB\Driver\Query(['_id' => $matchDoc->_id]);

    $cursor = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

    $updatedDoc = current($cursor->toArray());

    $users = $updatedDoc->users;

    $swiperSwipe = $users->$senderId ?? null;

    $swipedSwipe = $users->$receiverId ?? null;

  

    if ($swiperSwipe === "yes" && $swipedSwipe === "yes") {

        // OPTIONAL: Check if a chat was already created to avoid duplicates.

        if (isset($updatedDoc->chat_id)) {

            // Return the already created chat details.

            echo json_encode([

                "status" => "match",

                "chat_id" => $updatedDoc->chat_id,

                "profileURL" => $updatedDoc->receiver_profileURL ?? "",

                "receiver_id" => $receiverId,

                "receiver" => $receiver->username

            ]);

            exit;

        }

  

        // Fetch receiver information and extract profile image.

        $receiverProfileImage = "";

        if (isset($receiver->profile)) {

            if (is_string($receiver->profile)) {

                $receiverProfile = json_decode($receiver->profile, true);

            } else {

                $receiverProfile = (array)$receiver->profile;

            }

            $receiverProfileImage = $receiverProfile['images'][0] ?? "";

        }

        $senderProfileImage = "";

        if (isset($sender->profile)) {

            if (is_string($sender->profile)) {

                $senderProfile = json_decode($sender->profile, true);

            } else {

                $senderProfile = (array)$sender->profile;

            }

            $senderProfileImage = $senderProfile['images'][0] ?? "";

        }

  

        // Create chat document.

        $chatData = [

            "messages" => new \stdClass(),

            "participants" => [$senderId, $receiverId]

        ];

        $bulkChat = new MongoDB\Driver\BulkWrite();

        $chatId = $bulkChat->insert($chatData);

        $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkChat);

        $chatIdString = (string)$chatId;

  

        // Prepare chat metadata.

        // For the sender, store the receiver's unique id under "receiver_id" and its username under "receiver".

        $chatMetaSender = [

            "profileURL"   => $receiverProfileImage,

            "receiver_id"  => $receiverId,

            "receiver"     => $receiver->username,

            "last_message" => "",

            "unread"       => 0

        ];

        // For the receiver, store the sender's unique id under "receiver_id" and its username under "receiver".

        $chatMetaReceiver = [

            "profileURL"   => $senderProfileImage,

            "receiver_id"  => $senderId,

            "receiver"     => $sender->username,

            "last_message" => "",

            "unread"       => 0

        ];

  

        // Update sender's and receiver's chat lists.

        $bulkUpdate = new MongoDB\Driver\BulkWrite();

        $bulkUpdate->update(

            ["_id" => new ObjectId($senderId)],

            ['$set' => ["chats.$chatIdString" => $chatMetaSender]]

        );

        $bulkUpdate->update(

            ["_id" => new ObjectId($receiverId)],

            ['$set' => ["chats.$chatIdString" => $chatMetaReceiver]]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUpdate);

  

        // OPTIONAL: Update the match document with chat details to avoid duplicate creation.

        $bulkMatchUpdate = new MongoDB\Driver\BulkWrite();

        $bulkMatchUpdate->update(

            ['_id' => $updatedDoc->_id],

            ['$set' => [

                "chat_id" => $chatIdString,

                "receiver_profileURL" => $receiverProfileImage

            ]]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatchUpdate);

  

        echo json_encode([

            "status" => "match",

            "chat_id" => $chatIdString,

            "profileURL" => $receiverProfileImage,

            "receiver_id" => $receiverId,

            "receiver" => $receiver->username

        ]);

    } elseif ($swiperSwipe === "no" || $swipedSwipe === "no") {

        echo json_encode(["status" => "no match"]);

    } else {

        // One swipe is still pending.

        echo json_encode(["status" => "no match"]);

    }

} else {

    // No document exists yet; create one with both user IDs.

    $newDoc = [

        "users" => [

            $senderId   => $swipeValue,

            $receiverId => null

        ]

    ];

    $bulk->insert($newDoc);

    $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

    echo json_encode(["status" => "no match"]);

}

?>
"
match.php:
"
<?php

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

// --- 1. Get JWT from Authorization header ---

$headers = getallheaders();

if (!isset($headers['Authorization'])) {

    http_response_code(401);

    echo json_encode(["error" => "Missing Authorization header"]);

    exit;

}

$authHeader = $headers['Authorization'];

if (strpos($authHeader, "Bearer ") !== 0) {

    http_response_code(401);

    echo json_encode(["error" => "Invalid Authorization header format"]);

    exit;

}

$jwtToken = trim(str_replace("Bearer", "", $authHeader));

  

// --- 2. Decode JWT ---

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

try {

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

} catch (Exception $e) {

    http_response_code(401);

    echo json_encode(["error" => "Invalid token: " . $e->getMessage()]);

    exit;

}

  

// Assume the token payload contains the user ID (as "user_id")

$userId = $decodedToken->user_id;

  

// --- 3. Connect to MongoDB and load current user profile ---

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

$namespace = "swipe_chat_play.users"; // Change to your actual DB and collection name

  

$query = new MongoDB\Driver\Query(['_id' => new MongoDB\BSON\ObjectId($userId)]);

try {

    $cursor = $mongoManager->executeQuery($namespace, $query);

    $currentUser = current($cursor->toArray());

    if (!$currentUser) {

        http_response_code(404);

        echo json_encode(["error" => "User not found"]);

        exit;

    }

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => "Database error: " . $e->getMessage()]);

    exit;

}

  

// Ensure the current user's profile has a "searching_for" field

if (!isset($currentUser->profile->searching_for)) {

    http_response_code(400);

    echo json_encode(["error" => "Searching criteria not defined"]);

    exit;

}

$searchingFor = (array)$currentUser->profile->searching_for;

  

// --- 4. Build hard filter query using "searching_for" ---

$hardFilter = [];

  

// Gender: candidate's about_you.gender must be one of the desired genders

if (isset($searchingFor['genders'])) {

    $hardFilter['profile.about_you.gender'] = [

        '$in' => $searchingFor['genders']

    ];

}

  

// Age: candidate's about_you.age should be within the provided ageRange

if (isset($searchingFor['ageRange'])) {

    $ageRange = $searchingFor['ageRange'];

    $minAge = (int)$ageRange[0];

    $maxAge = (int)$ageRange[1];

    $hardFilter['profile.about_you.age'] = [

        '$gte' => $minAge,

        '$lte' => $maxAge

    ];

}

  

// Exclude the current user from the candidate list

$hardFilter['_id'] = [

    '$ne' => new MongoDB\BSON\ObjectId($userId)

];

  

// NOTE: The lines below are REMOVED so that religion and politics aren't hard filters.

// if (isset($searchingFor['religion'])) {

//     $hardFilter['profile.about_you.religion'] = ['$in' => $searchingFor['religion']];

// }

// if (isset($searchingFor['politics'])) {

//     $hardFilter['profile.about_you.politics'] = ['$in' => $searchingFor['politics']];

// }

  

// --- 5. Handle pagination parameters ---

$page = isset($_GET['page']) ? (int)$_GET['page'] : 1;

$limit = isset($_GET['limit']) ? (int)$_GET['limit'] : 10;

$skip = ($page - 1) * $limit;

  

// --- 5.1. Get total count of matching candidates for has_more calculation ---

$command = new MongoDB\Driver\Command([

    'count' => 'users',

    'query' => $hardFilter

]);

try {

    $cursor = $mongoManager->executeCommand('swipe_chat_play', $command);

    $result = $cursor->toArray();

    $totalCount = (isset($result[0]) && isset($result[0]->n)) ? $result[0]->n : 0;

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => "Count command failed: " . $e->getMessage()]);

    exit;

}

  

// --- 5.2. Execute query with pagination ---

$queryOptions = [

    'skip' => $skip,

    'limit' => $limit

];

$query = new MongoDB\Driver\Query($hardFilter, $queryOptions);

try {

    $cursor = $mongoManager->executeQuery($namespace, $query);

    $candidates = $cursor->toArray();

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => "Database query error: " . $e->getMessage()]);

    exit;

}

  

// --- 5.3. Calculate has_more ---

$hasMore = ($totalCount > ($skip + $limit));

  

// --- Debug: Log raw candidates ---

$debugFile = __DIR__ . '/debug.log';

$rawCandidatesJson = json_encode($candidates, JSON_PRETTY_PRINT);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Raw Candidates:\n" . $rawCandidatesJson . "\n", FILE_APPEND);

  

// --- 6. Rank the results based on additional matching attributes ---

$weightOpenTo         = 5; // Open-to interests are more important

$weightSpokenLanguage = 2;

$weightReligion       = 1;

$weightPolitics       = 1;

  

$rankedCandidates = [];

  

foreach ($candidates as $candidate) {

    $score = 0;

    // Compare "open_to" arrays

    if (isset($currentUser->profile->open_to) && isset($candidate->profile->open_to)) {

        $currentOpenTo = $currentUser->profile->open_to;

        $candidateOpenTo = $candidate->profile->open_to;

        $commonOpenTo = array_intersect($currentOpenTo, $candidateOpenTo);

        $score += count($commonOpenTo) * $weightOpenTo;

    }

    // Compare spokenLanguage

    if (isset($currentUser->profile->about_you->spokenLanguage) && isset($candidate->profile->about_you->spokenLanguage)) {

        if ($currentUser->profile->about_you->spokenLanguage === $candidate->profile->about_you->spokenLanguage) {

            $score += $weightSpokenLanguage;

        }

    }

    // Bonus for matching religion

    if (isset($currentUser->profile->about_you->religion) && isset($candidate->profile->about_you->religion)) {

        if ($currentUser->profile->about_you->religion === $candidate->profile->about_you->religion) {

            $score += $weightReligion;

        }

    }

    // Bonus for matching politics

    if (isset($currentUser->profile->about_you->politics) && isset($candidate->profile->about_you->politics)) {

        if ($currentUser->profile->about_you->politics === $candidate->profile->about_you->politics) {

            $score += $weightPolitics;

        }

    }

    // Convert candidate to array, attach the matching score

    $candidateArray = (array)$candidate;

    $candidateArray['matching_score'] = $score;

    $rankedCandidates[] = $candidateArray;

}

  

// --- Debug: Log ranked candidates ---

$rankedCandidatesJson = json_encode($rankedCandidates, JSON_PRETTY_PRINT);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Ranked Candidates:\n" . $rankedCandidatesJson . "\n", FILE_APPEND);

  

// Sort in descending order of matching score

usort($rankedCandidates, function($a, $b) {

    return $b['matching_score'] <=> $a['matching_score'];

});

  

// --- 6.5. Transform each candidate into the desired structure ---

$formattedCandidates = [];

  

foreach ($rankedCandidates as $candidate) {

    $profile = $candidate['profile'];

    $topSection = [

        'id' => (string)$candidate['_id'],  // <-- Added the unique MongoDB id

        'profileImages' => isset($profile->images) ? $profile->images : [],

        'name' => (isset($candidate['firstname']) ? $candidate['firstname'] : "") . ' ' . (isset($candidate['lastname']) ? $candidate['lastname'] : ""),

        'age' => isset($profile->about_you->age) ? (string)$profile->about_you->age : '',

        'infoBubbles' => [

            isset($profile->about_you->gender) ? $profile->about_you->gender : '',

            isset($profile->about_you->spokenLanguage) ? $profile->about_you->spokenLanguage : '',

            isset($profile->about_you->religion) ? $profile->about_you->religion : '',

            isset($profile->about_you->politics) ? $profile->about_you->politics : ''

        ]

    ];

  

    $elements = [];

    // If there's at least one Q&A

    if (

        isset($profile->questions_and_answers) &&

        is_array($profile->questions_and_answers) &&

        count($profile->questions_and_answers) > 0

    ) {

        $firstQA = $profile->questions_and_answers[0];

        $elements["1"] = [

            "type" => "icebreaker",

            "content" => [

                "question" => isset($firstQA->question) ? $firstQA->question : "",

                "answer"   => isset($firstQA->answer) ? $firstQA->answer : ""

            ]

        ];

    }

  

    // Bubbles from open_to with title "open to"

    if (isset($profile->open_to)) {

        $elements["2"] = [

            "type" => "bubbles",

            "content" => [

                "title" => "open to",

                "bubbles" => $profile->open_to

            ]

        ];

    }

  

    $formattedCandidates[] = [

        "topSection" => $topSection,

        "elements"   => $elements

    ];

}

  

// --- Debug: Log formatted candidates ---

$formattedCandidatesJson = json_encode($formattedCandidates, JSON_PRETTY_PRINT);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Formatted Candidates:\n" . $formattedCandidatesJson . "\n", FILE_APPEND);

  

// --- 7. Return the formatted list as JSON ---

$finalJson = json_encode([

    "status" => "success",

    "data" => $formattedCandidates,

    "has_more" => $hasMore

]);

file_put_contents($debugFile, date('Y-m-d H:i:s') . " - Final JSON:\n" . $finalJson . "\n", FILE_APPEND);

echo $finalJson;

?>
"

"
"
swipe_chat_play> db.users.find().pretty()

[

{

_id: ObjectId('67b8c730ed13220a4c0b5cc6'),

username: 'katrina_0',

firstname: 'Katrina',

lastname: 'Doe_0',

email: 'katrina_0@example.com',

password_hash: '$2y$10$tOWuvB3m8QlVEiggmqB1.OidCd8S8Y0VCNXpAvHJWubVN5YdW18jm',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Katrina',

age: 45,

gender: 'Male',

spokenLanguage: 'English',

religion: 'Hinduism',

career: 'Example Career 0',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'I love celebrating every day!'

},

{

question: 'If I were an animal, I would be...',

answer: "I've always wanted to be an astronaut."

},

{

question: 'Describe your perfect day.',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.660747318923933, 48.02338372476223 ]

},

radius: 1452

},

open_to: [

'cooking',

'Arts & Crafts',

'movies',

'woodworking',

'birdwatching'

],

searching_for: {

genders: [ 'Non Binary' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67cb486a9ea65af156007c84': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c730ed13220a4c0b5cc7'),

username: 'xander_1',

firstname: 'Xander',

lastname: 'Doe_1',

email: 'xander_1@example.com',

password_hash: '$2y$10$fw.LMK9c4b5ZoXeqQ1UIEuldX8Y2QX2FkLphC8Vr1W3ge8jH1wr/a',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Xander',

age: 33,

gender: 'Transgender',

spokenLanguage: 'Italian',

religion: 'Other',

career: 'Example Career 1',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.598911192892864, 48.11853566384324 ]

},

radius: 1457

},

open_to: [

'Arts & Crafts',

'dance classes',

'homebrewing',

'painting',

'card games'

],

searching_for: {

genders: [ 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf1da2dfcc0109a90da854': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c730ed13220a4c0b5cc8'),

username: 'jack_2',

firstname: 'Jack',

lastname: 'Doe_2',

email: 'jack_2@example.com',

password_hash: '$2y$10$uDrU8qDm7.cpPFYuexp2Muf4Fxgj5GpH6lEcfUzfTn/qM0wm0ad7q',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Jack',

age: 46,

gender: 'Genderqueer',

spokenLanguage: 'French',

religion: 'Islam',

career: 'Example Career 2',

politics: 'Libertarian'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.541556205963714, 48.166230222238845 ]

},

radius: 773

},

open_to: [ 'karaoke', 'zip-lining', 'board games', 'camping', 'kayaking' ],

searching_for: {

genders: [

'Transgender',

'Genderqueer',

'Non Binary',

'Female',

'Other'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf1db5dfcc0109a90da855': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c730ed13220a4c0b5cc9'),

username: 'oscar_3',

firstname: 'Oscar',

lastname: 'Doe_3',

email: 'oscar_3@example.com',

password_hash: '$2y$10$mV.L89pZXu35w.w1CsUGP.FzdjCk5siswHhiYalomNrwBo30Co0Hm',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Oscar',

age: 29,

gender: 'Transgender',

spokenLanguage: 'German',

religion: 'Judaism',

career: 'Example Career 3',

politics: 'Libertarian'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'If I were an animal, I would be...',

answer: 'I love celebrating every day!'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.452351898205613, 48.05096724311227 ]

},

radius: 917

},

open_to: [

'mixology',

'board games',

'birdwatching',

'jogging',

'snowshoeing'

],

searching_for: {

genders: [ 'Female', 'Genderqueer' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67c1f884ed13220a4c0b5d63': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(location)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cca'),

username: 'henry_4',

firstname: 'Henry',

lastname: 'Doe_4',

email: 'henry_4@example.com',

password_hash: '$2y$10$LF/eaSYkkmQfFvZlRGRCjOuJfe3vKfw/PHQcrmNye5hifR7uV4yUC',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Henry',

age: 18,

gender: 'Male',

spokenLanguage: 'Italian',

religion: 'Hinduism',

career: 'Example Career 4',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.769385039863392, 47.7683483513131 ]

},

radius: 917

},

open_to: [

'hiking',

'dance classes',

'jogging',

'camping',

'woodworking'

],

searching_for: {

genders: [ 'Genderqueer', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bdc586dfcc0109a90da84f': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccb'),

username: 'wendy_5',

firstname: 'Wendy',

lastname: 'Doe_5',

email: 'wendy_5@example.com',

password_hash: '$2y$10$W2EcRt76mYjOM4KgWkuvFOiTsgzBpyDAKS8S2tGDvjyc/Ws3Yjvja',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Wendy',

age: 35,

gender: 'Genderqueer',

spokenLanguage: 'French',

religion: 'Buddhism',

career: 'Example Career 5',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.670231480582181, 48.51572516323004 ]

},

radius: 1025

},

open_to: [

'dance classes',

'hiking',

'board games',

'Arts & Crafts',

'zip-lining'

],

searching_for: {

genders: [ 'Genderqueer', 'Female', 'Other', 'Non Binary' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccc'),

username: 'frank_6',

firstname: 'Frank',

lastname: 'Doe_6',

email: 'frank_6@example.com',

password_hash: '$2y$10$9iPRNVBS/D32a67zLGn92.hnDrTTeTp4kkpuPCyRsVex2YhmddVfG',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Frank',

age: 31,

gender: 'Genderqueer',

spokenLanguage: 'English',

religion: 'Buddhism',

career: 'Example Career 6',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'If I were an animal, I would be...',

answer: 'I love celebrating every day!'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.711711240013962, 48.52132917983182 ]

},

radius: 1726

},

open_to: [

'yoga',

'painting',

'card games',

'beach volleyball',

'karaoke'

],

searching_for: {

genders: [

'Non Binary',

'Other',

'Genderqueer',

'Female',

'Transgender'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf8b07dfcc0109a90da858': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccd'),

username: 'thomas_7',

firstname: 'Thomas',

lastname: 'Doe_7',

email: 'thomas_7@example.com',

password_hash: '$2y$10$K6STz1dJl9qTU9QTumtqAemeBLTrscoCKnUoN3LTDb94oI6Ed1lU.',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Thomas',

age: 41,

gender: 'Female',

spokenLanguage: 'German',

religion: 'Islam',

career: 'Example Career 7',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your dream job?',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'What is your favorite food?',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.508167567781054, 47.753458797619764 ]

},

radius: 1118

},

open_to: [

'board games',

'mixology',

'puzzles',

'card games',

'trail running'

],

searching_for: {

genders: [ 'Other', 'Non Binary', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf1d9e9ea65af156007c7f': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cce'),

username: 'charlie_8',

firstname: 'Charlie',

lastname: 'Doe_8',

email: 'charlie_8@example.com',

password_hash: '$2y$10$NaGbff.30E5kIyr17EVWc.q3AhIiZ93imVYkkXVJD5ZPWwisU8XHC',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Charlie',

age: 34,

gender: 'Other',

spokenLanguage: 'Italian',

religion: 'Buddhism',

career: 'Example Career 8',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.760490286216386, 48.1229166991863 ]

},

radius: 1013

},

open_to: [

'card games',

'painting',

'knitting',

'snowshoeing',

'horseback riding'

],

searching_for: {

genders: [ 'Non Binary', 'Female', 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bdcee4dfcc0109a90da851': {

profileURL: 'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67bdce67b890c_image_cropper_1740492378656.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccf'),

username: 'alice_9',

firstname: 'Alice',

lastname: 'Doe_9',

email: 'alice_9@example.com',

password_hash: '$2y$10$aI6MNndIVyjm/N8n0HnoGeyAjdwRtgoBAgiCUXL6LazLUxxyBg44q',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Alice',

age: 35,

gender: 'Male',

spokenLanguage: 'English',

religion: 'Islam',

career: 'Example Career 9',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'If I were an animal, I would be...',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.558559563316784, 48.05844332018733 ]

},

radius: 1535

},

open_to: [

'homebrewing',

'whitewater rafting',

'horseback riding',

'woodworking',

'karaoke'

],

searching_for: {

genders: [

'Female',

'Transgender',

'Other',

'Non Binary',

'Genderqueer'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf8b17ed13220a4c0b5d5f': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cd0'),

username: 'frank_10',

firstname: 'Frank',

lastname: 'Doe_10',

email: 'frank_10@example.com',

password_hash: '$2y$10$Zj8soZj6IFM2h2IxG70BqOLz.BBHkdjS1UAurrq4O2F4Gr/mcNw4i',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Frank',

age: 48,

gender: 'Transgender',

spokenLanguage: 'French',

religion: 'Judaism',

career: 'Example Career 10',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'I love celebrating every day!'

},

{

question: 'My favorite holiday and why...',

answer: 'I always love going back to my hometown.'

},

{

question: 'A place I return to often...',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.531663427621124, 48.45278600002439 ]

},

radius: 1436

},

open_to: [

'karaoke',

'Arts & Crafts',

'homebrewing',

'zip-lining',

'pottery'

],

searching_for: {

genders: [

'Genderqueer',

'Other',

'Non Binary',

'Female',

'Transgender'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cd1'),

username: 'katrina_11',

firstname: 'Katrina',

lastname: 'Doe_11',

email: 'katrina_11@example.com',

password_hash: '$2y$10$9gCU73yHFf.NjMGoYHaQeeQZcBIEsjg3dQUz.MjewrOu38pPfswmi',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Katrina',

age: 45,

gender: 'Other',

spokenLanguage: 'German',

religion: 'Christianity',

career: 'Example Career 11',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'Describe your perfect day.',

answer: 'I love celebrating every day!'

},

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'What is your dream job?',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.285237049939665, 48.274944861567775 ]

},

radius: 1266

},

open_to: [

'rock climbing',

'card games',

'biking',

'homebrewing',

'games'

],

searching_for: {

genders: [ 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cd2'),

username: 'oscar_12',

firstname: 'Oscar',

lastname: 'Doe_12',

email: 'oscar_12@example.com',

password_hash: '$2y$10$77Cl4dd8i8km31C0951.G.bulXKtUF3yeFrvUjuhwhDHNLPCuWEn2',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Oscar',

age: 35,

gender: 'Non Binary',

spokenLanguage: 'English',

religion: 'Judaism',

career: 'Example Career 12',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your dream job?',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'My favorite holiday and why...',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'A place I return to often...',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.574555082708374, 48.26791393408802 ]

},

radius: 917

},

open_to: [ 'scuba diving', 'painting', 'jogging', 'kayaking', 'games' ],

searching_for: {

genders: [ 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd3'),

username: 'thomas_13',

firstname: 'Thomas',

lastname: 'Doe_13',

email: 'thomas_13@example.com',

password_hash: '$2y$10$zwj/hAxOk0KZ0xNsK1b2/eQKSZn3CVzg8pKQVBCF2CtsuLdKQ7tHW',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Thomas',

age: 50,

gender: 'Other',

spokenLanguage: 'French',

religion: 'Buddhism',

career: 'Example Career 13',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'Describe your perfect day.',

answer: 'I love celebrating every day!'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.521614661558651, 48.15658334192666 ]

},

radius: 1873

},

open_to: [

'language exchange',

'surfing',

'fishing',

'stand-up paddleboarding',

'mixology'

],

searching_for: {

genders: [ 'Non Binary', 'Genderqueer', 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67f952dadfcc0109a90da887': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver_id: '67b8c7579ea65af156007c4e',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd4'),

username: 'rafael_14',

firstname: 'Rafael',

lastname: 'Doe_14',

email: 'rafael_14@example.com',

password_hash: '$2y$10$bsqFhfFUa.345sazs3GolelByow8o6xE/P9odTwBeYFel4NHchqiK',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Rafael',

age: 49,

gender: 'Non Binary',

spokenLanguage: 'Italian',

religion: 'Other',

career: 'Example Career 14',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'If I were an animal, I would be...',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'Describe your perfect day.',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.584996747860519, 48.094898417329055 ]

},

radius: 632

},

open_to: [ 'snowshoeing', 'scuba diving', 'movies', 'biking', 'jogging' ],

searching_for: {

genders: [ 'Non Binary', 'Female', 'Genderqueer' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd5'),

username: 'leo_15',

firstname: 'Leo',

lastname: 'Doe_15',

email: 'leo_15@example.com',

password_hash: '$2y$10$kjGJ0OsrrGJmJpCE7tTkAuui20UTntPL3rbWRbilvTbsrgrW02v4q',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Leo',

age: 39,

gender: 'Non Binary',

spokenLanguage: 'German',

religion: 'Christianity',

career: 'Example Career 15',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'Describe your perfect day.',

answer: "I've always wanted to be an astronaut."

},

{

question: 'What is your dream job?',

answer: 'I always love going back to my hometown.'

},

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.537817703022634, 48.29217201260578 ]

},

radius: 1595

},

open_to: [ 'kayaking', 'birdwatching', 'surfing', 'hiking', 'fishing' ],

searching_for: {

genders: [ 'Female', 'Transgender', 'Non Binary' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67c1f610ed13220a4c0b5d62': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(answer_to)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd6'),

username: 'zane_16',

firstname: 'Zane',

lastname: 'Doe_16',

email: 'zane_16@example.com',

password_hash: '$2y$10$xyKICdpXUUasfTjQH9hmj.WSCOW.5jIMTGDJoH4nAUEvMmVrHGcci',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Zane',

age: 41,

gender: 'Non Binary',

spokenLanguage: 'English',

religion: 'Islam',

career: 'Example Career 16',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'Describe your perfect day.',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'If I were an animal, I would be...',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.949776804957516, 48.28521784530184 ]

},

radius: 552

},

open_to: [

'zip-lining',

'karaoke',

'knitting',

'homebrewing',

'rock climbing'

],

searching_for: {

genders: [ 'Genderqueer', 'Other', 'Non Binary', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd7'),

username: 'sara_17',

firstname: 'Sara',

lastname: 'Doe_17',

email: 'sara_17@example.com',

password_hash: '$2y$10$TJV7uRjGA6TzxjDlWagJI.PtvI7Wwh0/KS702n/DYYAxvVXtC./xy',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Sara',

age: 22,

gender: 'Non Binary',

spokenLanguage: 'English',

religion: 'Other',

career: 'Example Career 17',

politics: 'Conservative'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'A place I return to often...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'If I were an animal, I would be...',

answer: 'Pizza! I could eat it every day.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.13834094107348, 48.37880809856515 ]

},

radius: 1927

},

open_to: [ 'biking', 'movies', 'yoga', 'Arts & Crafts', 'mixology' ],

searching_for: {

genders: [ 'Female', 'Transgender', 'Other' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd8'),

username: 'iris_18',

firstname: 'Iris',

lastname: 'Doe_18',

email: 'iris_18@example.com',

password_hash: '$2y$10$dQsohNi7NSQ/96O8OhIK.OaHWqSB5VkI03qM02pkBJdEYQqdbYwtG',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Iris',

age: 36,

gender: 'Male',

spokenLanguage: 'French',

religion: 'Hinduism',

career: 'Example Career 18',

politics: 'Conservative'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.071615806835588, 48.20618815173136 ]

},

radius: 472

},

open_to: [

'scuba diving',

'dance classes',

'pottery',

'whitewater rafting',

'karaoke'

],

searching_for: {

genders: [ 'Other', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bde89fed13220a4c0b5d5d': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

},

'67f94f219ea65af156007cb9': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: '67b8c7579ea65af156007c4e',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd9'),

username: 'mona_19',

firstname: 'Mona',

lastname: 'Doe_19',

email: 'mona_19@example.com',

password_hash: '$2y$10$s7.ovj/cbTy3Qe02fsZR.OJiLLghveUO6PvVTe6G0lu.Cn.SJzX9K',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Mona',

age: 18,

gender: 'Other',

spokenLanguage: 'French',

religion: 'Christianity',

career: 'Example Career 19',

politics: 'Libertarian'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'A place I return to often...',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.178459502931432, 48.15828682556798 ]

},

radius: 1049

},

open_to: [

'birdwatching',

'karaoke',

'trail running',

'movies',

'scuba diving'

],

searching_for: {

genders: [ 'Genderqueer', 'Non Binary', 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bdc587ed13220a4c0b5d5b': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

},

'67eac7019966cc19fa4eeb87': {

profileURL: 'http://example.com/luna.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

}

}

}

]"

"

swipe_chat_play> db.users.find().pretty()

[

{

_id: ObjectId('67b8c730ed13220a4c0b5cc6'),

username: 'katrina_0',

firstname: 'Katrina',

lastname: 'Doe_0',

email: 'katrina_0@example.com',

password_hash: '$2y$10$tOWuvB3m8QlVEiggmqB1.OidCd8S8Y0VCNXpAvHJWubVN5YdW18jm',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Katrina',

age: 45,

gender: 'Male',

spokenLanguage: 'English',

religion: 'Hinduism',

career: 'Example Career 0',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'I love celebrating every day!'

},

{

question: 'If I were an animal, I would be...',

answer: "I've always wanted to be an astronaut."

},

{

question: 'Describe your perfect day.',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.660747318923933, 48.02338372476223 ]

},

radius: 1452

},

open_to: [

'cooking',

'Arts & Crafts',

'movies',

'woodworking',

'birdwatching'

],

searching_for: {

genders: [ 'Non Binary' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67cb486a9ea65af156007c84': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c730ed13220a4c0b5cc7'),

username: 'xander_1',

firstname: 'Xander',

lastname: 'Doe_1',

email: 'xander_1@example.com',

password_hash: '$2y$10$fw.LMK9c4b5ZoXeqQ1UIEuldX8Y2QX2FkLphC8Vr1W3ge8jH1wr/a',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Xander',

age: 33,

gender: 'Transgender',

spokenLanguage: 'Italian',

religion: 'Other',

career: 'Example Career 1',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.598911192892864, 48.11853566384324 ]

},

radius: 1457

},

open_to: [

'Arts & Crafts',

'dance classes',

'homebrewing',

'painting',

'card games'

],

searching_for: {

genders: [ 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf1da2dfcc0109a90da854': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c730ed13220a4c0b5cc8'),

username: 'jack_2',

firstname: 'Jack',

lastname: 'Doe_2',

email: 'jack_2@example.com',

password_hash: '$2y$10$uDrU8qDm7.cpPFYuexp2Muf4Fxgj5GpH6lEcfUzfTn/qM0wm0ad7q',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Jack',

age: 46,

gender: 'Genderqueer',

spokenLanguage: 'French',

religion: 'Islam',

career: 'Example Career 2',

politics: 'Libertarian'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.541556205963714, 48.166230222238845 ]

},

radius: 773

},

open_to: [ 'karaoke', 'zip-lining', 'board games', 'camping', 'kayaking' ],

searching_for: {

genders: [

'Transgender',

'Genderqueer',

'Non Binary',

'Female',

'Other'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf1db5dfcc0109a90da855': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c730ed13220a4c0b5cc9'),

username: 'oscar_3',

firstname: 'Oscar',

lastname: 'Doe_3',

email: 'oscar_3@example.com',

password_hash: '$2y$10$mV.L89pZXu35w.w1CsUGP.FzdjCk5siswHhiYalomNrwBo30Co0Hm',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Oscar',

age: 29,

gender: 'Transgender',

spokenLanguage: 'German',

religion: 'Judaism',

career: 'Example Career 3',

politics: 'Libertarian'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'If I were an animal, I would be...',

answer: 'I love celebrating every day!'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.452351898205613, 48.05096724311227 ]

},

radius: 917

},

open_to: [

'mixology',

'board games',

'birdwatching',

'jogging',

'snowshoeing'

],

searching_for: {

genders: [ 'Female', 'Genderqueer' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67c1f884ed13220a4c0b5d63': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(location)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cca'),

username: 'henry_4',

firstname: 'Henry',

lastname: 'Doe_4',

email: 'henry_4@example.com',

password_hash: '$2y$10$LF/eaSYkkmQfFvZlRGRCjOuJfe3vKfw/PHQcrmNye5hifR7uV4yUC',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Henry',

age: 18,

gender: 'Male',

spokenLanguage: 'Italian',

religion: 'Hinduism',

career: 'Example Career 4',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.769385039863392, 47.7683483513131 ]

},

radius: 917

},

open_to: [

'hiking',

'dance classes',

'jogging',

'camping',

'woodworking'

],

searching_for: {

genders: [ 'Genderqueer', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bdc586dfcc0109a90da84f': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccb'),

username: 'wendy_5',

firstname: 'Wendy',

lastname: 'Doe_5',

email: 'wendy_5@example.com',

password_hash: '$2y$10$W2EcRt76mYjOM4KgWkuvFOiTsgzBpyDAKS8S2tGDvjyc/Ws3Yjvja',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Wendy',

age: 35,

gender: 'Genderqueer',

spokenLanguage: 'French',

religion: 'Buddhism',

career: 'Example Career 5',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.670231480582181, 48.51572516323004 ]

},

radius: 1025

},

open_to: [

'dance classes',

'hiking',

'board games',

'Arts & Crafts',

'zip-lining'

],

searching_for: {

genders: [ 'Genderqueer', 'Female', 'Other', 'Non Binary' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccc'),

username: 'frank_6',

firstname: 'Frank',

lastname: 'Doe_6',

email: 'frank_6@example.com',

password_hash: '$2y$10$9iPRNVBS/D32a67zLGn92.hnDrTTeTp4kkpuPCyRsVex2YhmddVfG',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Frank',

age: 31,

gender: 'Genderqueer',

spokenLanguage: 'English',

religion: 'Buddhism',

career: 'Example Career 6',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'If I were an animal, I would be...',

answer: 'I love celebrating every day!'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.711711240013962, 48.52132917983182 ]

},

radius: 1726

},

open_to: [

'yoga',

'painting',

'card games',

'beach volleyball',

'karaoke'

],

searching_for: {

genders: [

'Non Binary',

'Other',

'Genderqueer',

'Female',

'Transgender'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf8b07dfcc0109a90da858': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccd'),

username: 'thomas_7',

firstname: 'Thomas',

lastname: 'Doe_7',

email: 'thomas_7@example.com',

password_hash: '$2y$10$K6STz1dJl9qTU9QTumtqAemeBLTrscoCKnUoN3LTDb94oI6Ed1lU.',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Thomas',

age: 41,

gender: 'Female',

spokenLanguage: 'German',

religion: 'Islam',

career: 'Example Career 7',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your dream job?',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'What is your favorite food?',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.508167567781054, 47.753458797619764 ]

},

radius: 1118

},

open_to: [

'board games',

'mixology',

'puzzles',

'card games',

'trail running'

],

searching_for: {

genders: [ 'Other', 'Non Binary', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf1d9e9ea65af156007c7f': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cce'),

username: 'charlie_8',

firstname: 'Charlie',

lastname: 'Doe_8',

email: 'charlie_8@example.com',

password_hash: '$2y$10$NaGbff.30E5kIyr17EVWc.q3AhIiZ93imVYkkXVJD5ZPWwisU8XHC',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Charlie',

age: 34,

gender: 'Other',

spokenLanguage: 'Italian',

religion: 'Buddhism',

career: 'Example Career 8',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.760490286216386, 48.1229166991863 ]

},

radius: 1013

},

open_to: [

'card games',

'painting',

'knitting',

'snowshoeing',

'horseback riding'

],

searching_for: {

genders: [ 'Non Binary', 'Female', 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bdcee4dfcc0109a90da851': {

profileURL: 'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67bdce67b890c_image_cropper_1740492378656.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5ccf'),

username: 'alice_9',

firstname: 'Alice',

lastname: 'Doe_9',

email: 'alice_9@example.com',

password_hash: '$2y$10$aI6MNndIVyjm/N8n0HnoGeyAjdwRtgoBAgiCUXL6LazLUxxyBg44q',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Alice',

age: 35,

gender: 'Male',

spokenLanguage: 'English',

religion: 'Islam',

career: 'Example Career 9',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'If I were an animal, I would be...',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.558559563316784, 48.05844332018733 ]

},

radius: 1535

},

open_to: [

'homebrewing',

'whitewater rafting',

'horseback riding',

'woodworking',

'karaoke'

],

searching_for: {

genders: [

'Female',

'Transgender',

'Other',

'Non Binary',

'Genderqueer'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bf8b17ed13220a4c0b5d5f': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cd0'),

username: 'frank_10',

firstname: 'Frank',

lastname: 'Doe_10',

email: 'frank_10@example.com',

password_hash: '$2y$10$Zj8soZj6IFM2h2IxG70BqOLz.BBHkdjS1UAurrq4O2F4Gr/mcNw4i',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Frank',

age: 48,

gender: 'Transgender',

spokenLanguage: 'French',

religion: 'Judaism',

career: 'Example Career 10',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your favorite food?',

answer: 'I love celebrating every day!'

},

{

question: 'My favorite holiday and why...',

answer: 'I always love going back to my hometown.'

},

{

question: 'A place I return to often...',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.531663427621124, 48.45278600002439 ]

},

radius: 1436

},

open_to: [

'karaoke',

'Arts & Crafts',

'homebrewing',

'zip-lining',

'pottery'

],

searching_for: {

genders: [

'Genderqueer',

'Other',

'Non Binary',

'Female',

'Transgender'

],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cd1'),

username: 'katrina_11',

firstname: 'Katrina',

lastname: 'Doe_11',

email: 'katrina_11@example.com',

password_hash: '$2y$10$9gCU73yHFf.NjMGoYHaQeeQZcBIEsjg3dQUz.MjewrOu38pPfswmi',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Katrina',

age: 45,

gender: 'Other',

spokenLanguage: 'German',

religion: 'Christianity',

career: 'Example Career 11',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'Describe your perfect day.',

answer: 'I love celebrating every day!'

},

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'What is your dream job?',

answer: "I'd probably be an eagle, so I could fly freely."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.285237049939665, 48.274944861567775 ]

},

radius: 1266

},

open_to: [

'rock climbing',

'card games',

'biking',

'homebrewing',

'games'

],

searching_for: {

genders: [ 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c731ed13220a4c0b5cd2'),

username: 'oscar_12',

firstname: 'Oscar',

lastname: 'Doe_12',

email: 'oscar_12@example.com',

password_hash: '$2y$10$77Cl4dd8i8km31C0951.G.bulXKtUF3yeFrvUjuhwhDHNLPCuWEn2',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Oscar',

age: 35,

gender: 'Non Binary',

spokenLanguage: 'English',

religion: 'Judaism',

career: 'Example Career 12',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'What is your dream job?',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'My favorite holiday and why...',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'A place I return to often...',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.574555082708374, 48.26791393408802 ]

},

radius: 917

},

open_to: [ 'scuba diving', 'painting', 'jogging', 'kayaking', 'games' ],

searching_for: {

genders: [ 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd3'),

username: 'thomas_13',

firstname: 'Thomas',

lastname: 'Doe_13',

email: 'thomas_13@example.com',

password_hash: '$2y$10$zwj/hAxOk0KZ0xNsK1b2/eQKSZn3CVzg8pKQVBCF2CtsuLdKQ7tHW',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Thomas',

age: 50,

gender: 'Other',

spokenLanguage: 'French',

religion: 'Buddhism',

career: 'Example Career 13',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'Describe your perfect day.',

answer: 'I love celebrating every day!'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.521614661558651, 48.15658334192666 ]

},

radius: 1873

},

open_to: [

'language exchange',

'surfing',

'fishing',

'stand-up paddleboarding',

'mixology'

],

searching_for: {

genders: [ 'Non Binary', 'Genderqueer', 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67f952dadfcc0109a90da887': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver_id: '67b8c7579ea65af156007c4e',

receiver: 'luna',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd4'),

username: 'rafael_14',

firstname: 'Rafael',

lastname: 'Doe_14',

email: 'rafael_14@example.com',

password_hash: '$2y$10$bsqFhfFUa.345sazs3GolelByow8o6xE/P9odTwBeYFel4NHchqiK',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Rafael',

age: 49,

gender: 'Non Binary',

spokenLanguage: 'Italian',

religion: 'Other',

career: 'Example Career 14',

politics: 'Centrist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg'

],

questions_and_answers: [

{

question: 'If I were an animal, I would be...',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'Describe your perfect day.',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.584996747860519, 48.094898417329055 ]

},

radius: 632

},

open_to: [ 'snowshoeing', 'scuba diving', 'movies', 'biking', 'jogging' ],

searching_for: {

genders: [ 'Non Binary', 'Female', 'Genderqueer' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd5'),

username: 'leo_15',

firstname: 'Leo',

lastname: 'Doe_15',

email: 'leo_15@example.com',

password_hash: '$2y$10$kjGJ0OsrrGJmJpCE7tTkAuui20UTntPL3rbWRbilvTbsrgrW02v4q',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Leo',

age: 39,

gender: 'Non Binary',

spokenLanguage: 'German',

religion: 'Christianity',

career: 'Example Career 15',

politics: 'Socialist'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg'

],

questions_and_answers: [

{

question: 'Describe your perfect day.',

answer: "I've always wanted to be an astronaut."

},

{

question: 'What is your dream job?',

answer: 'I always love going back to my hometown.'

},

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.537817703022634, 48.29217201260578 ]

},

radius: 1595

},

open_to: [ 'kayaking', 'birdwatching', 'surfing', 'hiking', 'fishing' ],

searching_for: {

genders: [ 'Female', 'Transgender', 'Non Binary' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67c1f610ed13220a4c0b5d62': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(answer_to)',

unread: 0

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd6'),

username: 'zane_16',

firstname: 'Zane',

lastname: 'Doe_16',

email: 'zane_16@example.com',

password_hash: '$2y$10$xyKICdpXUUasfTjQH9hmj.WSCOW.5jIMTGDJoH4nAUEvMmVrHGcci',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Zane',

age: 41,

gender: 'Non Binary',

spokenLanguage: 'English',

religion: 'Islam',

career: 'Example Career 16',

politics: 'Liberal'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'A place I return to often...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'Describe your perfect day.',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'If I were an animal, I would be...',

answer: 'I always love going back to my hometown.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.949776804957516, 48.28521784530184 ]

},

radius: 552

},

open_to: [

'zip-lining',

'karaoke',

'knitting',

'homebrewing',

'rock climbing'

],

searching_for: {

genders: [ 'Genderqueer', 'Other', 'Non Binary', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd7'),

username: 'sara_17',

firstname: 'Sara',

lastname: 'Doe_17',

email: 'sara_17@example.com',

password_hash: '$2y$10$TJV7uRjGA6TzxjDlWagJI.PtvI7Wwh0/KS702n/DYYAxvVXtC./xy',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Sara',

age: 22,

gender: 'Non Binary',

spokenLanguage: 'English',

religion: 'Other',

career: 'Example Career 17',

politics: 'Conservative'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: "I'd probably be an eagle, so I could fly freely."

},

{

question: 'A place I return to often...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

},

{

question: 'If I were an animal, I would be...',

answer: 'Pizza! I could eat it every day.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.13834094107348, 48.37880809856515 ]

},

radius: 1927

},

open_to: [ 'biking', 'movies', 'yoga', 'Arts & Crafts', 'mixology' ],

searching_for: {

genders: [ 'Female', 'Transgender', 'Other' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd8'),

username: 'iris_18',

firstname: 'Iris',

lastname: 'Doe_18',

email: 'iris_18@example.com',

password_hash: '$2y$10$dQsohNi7NSQ/96O8OhIK.OaHWqSB5VkI03qM02pkBJdEYQqdbYwtG',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Iris',

age: 36,

gender: 'Male',

spokenLanguage: 'French',

religion: 'Hinduism',

career: 'Example Career 18',

politics: 'Conservative'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: 'My perfect day starts with sunshine and ends with a good book.'

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.071615806835588, 48.20618815173136 ]

},

radius: 472

},

open_to: [

'scuba diving',

'dance classes',

'pottery',

'whitewater rafting',

'karaoke'

],

searching_for: {

genders: [ 'Other', 'Female' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bde89fed13220a4c0b5d5d': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

},

'67f94f219ea65af156007cb9': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: '67b8c7579ea65af156007c4e',

last_message: '',

unread: 0

}

}

},

{

_id: ObjectId('67b8c732ed13220a4c0b5cd9'),

username: 'mona_19',

firstname: 'Mona',

lastname: 'Doe_19',

email: 'mona_19@example.com',

password_hash: '$2y$10$s7.ovj/cbTy3Qe02fsZR.OJiLLghveUO6PvVTe6G0lu.Cn.SJzX9K',

verification_code: null,

verification_code_expiry: null,

is_verified: true,

fingerprint_hash: '',

profile: {

about_you: {

name: 'Mona',

age: 18,

gender: 'Other',

spokenLanguage: 'French',

religion: 'Christianity',

career: 'Example Career 19',

politics: 'Libertarian'

},

images: [

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg',

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg'

],

questions_and_answers: [

{

question: 'My favorite holiday and why...',

answer: 'Pizza! I could eat it every day.'

},

{

question: 'A place I return to often...',

answer: "I've always wanted to be an astronaut."

}

],

location_radius: {

location: {

type: 'Point',

coordinates: [ 11.178459502931432, 48.15828682556798 ]

},

radius: 1049

},

open_to: [

'birdwatching',

'karaoke',

'trail running',

'movies',

'scuba diving'

],

searching_for: {

genders: [ 'Genderqueer', 'Non Binary', 'Transgender' ],

ageRange: [ 18, 66 ],

religion: [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

],

politics: [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist'

]

}

},

chats: {

'67bdc587ed13220a4c0b5d5b': {

profileURL: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b8c7913b5b0_image_cropper_1740162953865.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

},

'67eac7019966cc19fa4eeb87': {

profileURL: 'http://example.com/luna.jpg',

receiver: 'luna',

last_message: '(voicemessage)',

unread: 0

}

}

}

]

Type "it" for more

swipe_chat_play> db.chats.find().pretty()

[

{

_id: ObjectId('67b245da41cca8e7cc02f3c8'),

messages: {

'67b245daf0eb9': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b245daf0ebe': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b245daf0ebf': {

sender_id: '67b22aa2844fefdefd04fde5',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b25c39844fefdefd04fde6'),

messages: {

'67b25c39580ff': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b25c3958101': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b25c3958102': {

sender_id: '67b22aa2844fefdefd04fde5',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b25ca6844fefdefd04fde7'),

messages: {

'67b25ca8595c0': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hey',

timestamp: '2025-02-16T22:46:16.286016',

status: 'delivered'

}

},

participants: [ '67b22aa2844fefdefd04fde5', '67b22aa2844fefdefd04fde5' ]

},

{

_id: ObjectId('67b25cc100295b23840288e8'),

messages: {

'67b25cc1d9a22': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b25cc1d9a25': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b25cc1d9a26': {

sender_id: '67b22aa2844fefdefd04fde5',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b25d4e41cca8e7cc02f3c9'),

messages: {

'67b25d5557d36': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hello',

timestamp: '2025-02-16T22:49:09.335432',

status: 'delivered'

},

'67b25d5b9eb3b': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hellooo',

timestamp: '2025-02-16T22:49:15.630059',

status: 'delivered'

}

},

participants: [ '67b22aa2844fefdefd04fde5', '67b22aa2844fefdefd04fde5' ]

},

{

_id: ObjectId('67b25d5d844fefdefd04fde8'),

messages: {},

participants: [ '67b22aa2844fefdefd04fde5', '67b22aa2844fefdefd04fde5' ]

},

{

_id: ObjectId('67b25d7100295b23840288e9'),

messages: {

'67b25d73cf846': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'user',

timestamp: '2025-02-16T22:49:39.830147',

status: 'delivered'

},

'67b25d761171f': {

sender_id: '67b22c2041cca8e7cc02f3c6',

message: 'hello',

timestamp: '2025-02-16T22:49:42.029784',

status: 'delivered'

},

'67b2789306f29': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'kelpjelepekelekdkd',

timestamp: '2025-02-17T00:45:22.947026',

status: 'delivered'

}

},

participants: [ '67b22aa2844fefdefd04fde5', '67b22c2041cca8e7cc02f3c6' ]

},

{

_id: ObjectId('67b2766000295b23840288ea'),

messages: {

'67b27660dbab8': {

sender_id: '67b22aa2844fefdefd04fde5',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b27660dbaba': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b27660dbabb': {

sender_id: '67b22aa2844fefdefd04fde5',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b7e7b4844fefdefd050083'),

messages: {

'67b7e7b4460e9': {

sender_id: '67b7abf900295b2384028c78',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b7e7b4460ec': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b7e7b4460ed': {

sender_id: '67b7abf900295b2384028c78',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b7e7bb00295b2384028cdd'),

messages: {

'67b7e7bbf3363': {

sender_id: '67b7abf900295b2384028c78',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b7e7bbf3366': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b7e7bbf3367': {

sender_id: '67b7abf900295b2384028c78',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b7e7d3844fefdefd050084'),

messages: {

'67b7e7d3617d5': {

sender_id: '67b7abf900295b2384028c78',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b7e7d3617d9': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b7e7d3617da': {

sender_id: '67b7abf900295b2384028c78',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b7e7d800295b2384028cde'),

messages: {

'67b7e7d88a8ff': {

sender_id: '67b7abf900295b2384028c78',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b7e7d88a902': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b7e7d88a904': {

sender_id: '67b7abf900295b2384028c78',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b7f248ed13220a4c0b5732'),

messages: {

'67b7f248d3938': {

sender_id: '67b7abf900295b2384028c78',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b7f248d393b': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b7f248d393c': {

sender_id: '67b7abf900295b2384028c78',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67b7f3d99ea65af1560076b2'),

messages: {

'67b7f3d9d8f80': {

sender_id: '67b7abf900295b2384028c78',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67b7f3d9d8f84': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67b7f3d9d8f86': {

sender_id: '67b7abf900295b2384028c78',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67bb367bed13220a4c0b5d44'),

messages: {

'67bb367b7b22b': {

sender_id: '67b8c80f9ea65af156007c4f',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67bb367b7b22f': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67bb367b7b230': {

sender_id: '67b8c80f9ea65af156007c4f',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67bb3c0ded13220a4c0b5d45'),

messages: {

'67bb3c0de1443': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'hey how are you?',

timestamp: '2024-12-28T12:45:00.000Z',

status: 'delivered'

},

'67bb3c0de1445': {

sender_id: 'some_receiver_id',

message: "I'm doing great and you?",

timestamp: '2024-12-28T12:46:00.000Z',

status: 'delivered'

},

'67bb3c0de1447': {

sender_id: '67b8c7579ea65af156007c4e',

message: "I'm fine, thanks for asking. Want to go out?",

timestamp: '2024-12-28T12:47:00.000Z',

status: 'delivered'

}

}

},

{

_id: ObjectId('67bdc586dfcc0109a90da84f'),

messages: {

'67bf22e07e75e': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'hi',

timestamp: '2025-02-26T15:19:11.917855',

status: 'delivered'

},

'67bf22e295f5b': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'hellooo',

timestamp: '2025-02-26T15:19:14.068770',

status: 'delivered'

},

'67bf28b8bb56b': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'tvh',

timestamp: '2025-02-26T15:44:08.157554',

status: 'delivered'

},

'67c601833849f': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'hi',

timestamp: '2025-03-03T20:22:43.239261',

status: 'delivered'

},

'67cb2e22101e9': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'hellosor. ejbeiebek22',

timestamp: '2025-03-07T18:34:26.041216',

status: 'delivered'

},

'67cb2e2511ff1': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'behebei',

timestamp: '2025-03-07T18:34:29.119306',

status: 'delivered'

},

'67d5cf8f0717a': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'vwowj',

timestamp: '2025-03-15T20:05:50.879023',

status: 'delivered'

},

'67e1e21acd57b': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'tvvy',

timestamp: '2025-03-24T23:52:10.674497',

status: 'delivered'

},

'67e85b8a9168e': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'd6ghi',

timestamp: '2025-03-29T21:43:54.255127',

status: 'delivered'

},

'67ebdd6e4b6cf': {

sender_id: '67b8c7579ea65af156007c4e',

type: 'text',

timestamp: '2025-04-01T14:34:53.873225',

status: 'delivered',

message: 'hola'

},

'67ebdd8bd296e': {

sender_id: '67b8c7579ea65af156007c4e',

message: 'hola',

timestamp: '2025-04-01T14:35:23.541622',

status: 'delivered'

},

'67ebdda6ab9ad': {

sender_id: '67b8c7579ea65af156007c4e',

type: 'text',

timestamp: '2025-04-01T14:35:50.460917',

status: 'delivered',

message: 'hdjd'

},

'67f17f9613b44': {

sender_id: '67b8c7579ea65af156007c4e',

type: 'voicemessage',

timestamp: '2025-04-05T21:08:05.541564',

status: 'delivered',

audio_url: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f17f96138772.67969131_1743880080766.aac'

}

},

participants: [ '67b8c7579ea65af156007c4e', '67b8c731ed13220a4c0b5cca' ]

},

{

_id: ObjectId('67bdc587ed13220a4c0b5d5b'),

messages: {

'67f8779ba98bb': {

sender_id: '67b8c7579ea65af156007c4e',

type: 'location',

timestamp: '2025-04-11T03:59:55.422407',

status: 'delivered',

content: { lat: 48.1617524, lng: 11.5558313 }

},

'67f877edb3729': {

sender_id: '67b8c7579ea65af156007c4e',

type: 'voicemessage',

timestamp: '2025-04-11T04:01:17.291695',

status: 'delivered',

audio_url: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f877edb34a88.05803339_1744336870367.aac'

}

},

participants: [ '67b8c7579ea65af156007c4e', '67b8c732ed13220a4c0b5cd9' ]

},

{

_id: ObjectId('67bdc58fed13220a4c0b5d5c'),

messages: {},

participants: [ '67b8c7579ea65af156007c4e', '67b8c732ed13220a4c0b5cda' ]

},

{

_id: ObjectId('67bdcee4dfcc0109a90da851'),

messages: {},

participants: [ '67bdce30dfcc0109a90da850', '67b8c731ed13220a4c0b5cce' ]

}

]

Type "it" for more

swipe_chat_play>

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

please understand the project