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

/// A single card for a user’s profile.

/// Accepts an onElementTap callback that child layouts can use to send "react_..." payloads.

class ProfileCard extends StatelessWidget {

final profile.ProfileData data;

/// Make scrollController optional.

final ScrollController? scrollController;

/// Callback when an element is tapped.

final Function(Map<String, dynamic>)? onElementTap;

const ProfileCard({

Key? key,

required this.data,

this.scrollController, // <-- now optional

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

onTopSectionTap: (payload) => onElementTap?.call(payload),

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

onIcebreakerTap: (payload) =>

onElementTap?.call(payload),

),

'image' => _CachedImageLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onImageTap: (payload) => onElementTap?.call(payload),

),

'bubbles' => BubbleGridLayout(

content: entry.value.content,

swipedUserUsername: swipedUserUsername,

onBubbleTap: (payload) => onElementTap?.call(payload),

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

}

/// Minimal "YES"/"NO" indicators for horizontal swipes.

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

// Left "NO" indicator

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

// Right "YES" indicator

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

final Function(Map<String, dynamic>)? onImageTap;

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

final payload = {

"profile_username": widget.swipedUserUsername,

"type": "react_profile_image",

"timestamp": DateTime.now().toIso8601String(),

"content": {

"image_url": imageUrl,

"comment": "",

},

};

widget.onImageTap?.call(payload);

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




"

// cardstack.dart

import 'dart:async';

import 'dart:convert';

import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

// Adjust these as per your actual project structure:

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/tabs/chat/layouts/chat_popup.dart';

import 'package:swipe_chat_play/tabs/profile/layouts/profile_layout.dart';

// A simple Debouncer for prefetch logic:

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

}

class CardStack extends StatefulWidget {

const CardStack({Key? key}) : super(key: key);

@override

_CardStackState createState() => _CardStackState();

}

class _CardStackState extends State<CardStack>

with SingleTickerProviderStateMixin {

// ----------------------------------------------------------------

// DATA + FETCHING

// ----------------------------------------------------------------

List<profile.ProfileData> cardData = [];

bool isLoading = true;

bool isFetchingMore = false;

bool hasMore = true;

int currentPage = 1;

final Debouncer _debouncer = Debouncer(milliseconds: 500);

final int maxVisibleCards = 2; // Show at most 2 cards (top + back)

final int prefetchThreshold = 2; // Fetch more when we have <= 2 left

// ----------------------------------------------------------------

// SWIPING + SCROLLING

// ----------------------------------------------------------------

// Threshold for deciding a "horizontal" swipe

double get swipeThreshold => MediaQuery.of(context).size.width * 0.07;

// Once you switch from left->right or right->left beyond a small slop, we consider direction changed

double get directionChangeThreshold =>

MediaQuery.of(context).size.width * 0.02;

// Used to track horizontal drag offset (for yes/no swipe).

// We rely on this value to animate the top card horizontally.

final ValueNotifier<Offset> _dragPositionNotifier =

ValueNotifier<Offset>(Offset.zero);

// Pan-gesture bookkeeping

Offset? _gestureStart;

bool _decisionMade = false;

bool _isHorizontalSwipe = false;

final double _slopDistance = 10;

// We store the last recognized direction (“left” or “right”),

// so we can detect if the user changes direction mid-drag.

String? _currentDirection;

double _furthestRightSoFar = 0.0;

double _furthestLeftSoFar = 0.0;

// For custom vertical scrolling:

double _verticalOffset = 0.0; // how far the top card is scrolled

double _maxScrollExtent =

1000; // some large max scroll – adapt or measure dynamically

// You could measure the card’s actual height or content length to set this properly.

// Fling animation

AnimationController? _flingController;

Animation<double>? _flingAnimation;

@override

void initState() {

super.initState();

_fetchProfiles(reset: true);

// Setup an AnimationController for vertical “fling”:

_flingController = AnimationController(

vsync: this,

duration: const Duration(milliseconds: 500),

);

_flingController!.addListener(() {

setState(() {

_verticalOffset = _flingAnimation?.value ?? _verticalOffset;

});

});

}

@override

void dispose() {

_dragPositionNotifier.dispose();

_flingController?.dispose();

_debouncer.cancel();

super.dispose();

}

// ----------------------------------------------------------------

// FETCH PROFILES

// ----------------------------------------------------------------

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

// The “top” card is the last in the list, so we show them reversed:

List<profile.ProfileData> get visibleCards =>

cardData.reversed.take(maxVisibleCards).toList().reversed.toList();

// ----------------------------------------------------------------

// SWIPE / DISMISS

// ----------------------------------------------------------------

void _dismissCard(String swipeValue) {

if (cardData.isEmpty) return;

final topProfile = cardData.last;

setState(() {

cardData.removeLast();

// Reset vertical offset for new top card:

_verticalOffset = 0;

});

_sendSwipe(topProfile, swipeValue);

// Possibly prefetch more:

_debouncer.run(() {

if (mounted && cardData.length <= prefetchThreshold && hasMore) {

_fetchProfiles();

}

});

}

Future<void> _sendSwipe(

profile.ProfileData? profileData, String swipeValue) async {

if (profileData == null) return;

// The new approach uses "receiver_id" from topSection.id

final receiverId = profileData.topSection.id;

if (receiverId.isEmpty) return;

const storage = FlutterSecureStorage();

final token = await storage.read(key: 'jwt_token');

if (token == null) return;

final url = Uri.parse('${Config.backendBaseUrl}/match_algorithm/swipe.php');

final body = jsonEncode({

"receiver_id": receiverId,

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

// Possibly do something, e.g., show a popup or navigate to chat

} else if (status == "no match") {

debugPrint("No match or still waiting on other side to swipe yes");

}

} else {

debugPrint(

"Swipe endpoint error: ${response.statusCode} ${response.body}");

}

} catch (e) {

debugPrint("Error sending swipe: $e");

}

}

// ----------------------------------------------------------------

// GESTURE DETECTION

// ----------------------------------------------------------------

void _onPanStart(DragStartDetails details) {

_gestureStart = details.globalPosition;

_decisionMade = false;

_isHorizontalSwipe = false;

_currentDirection = null;

_furthestRightSoFar = 0.0;

_furthestLeftSoFar = 0.0;

// Cancel any fling that’s in progress:

_flingController?.stop();

}

void _onPanUpdate(DragUpdateDetails details) {

if (_gestureStart == null) return;

final dx = details.globalPosition.dx - _gestureStart!.dx;

final dy = details.globalPosition.dy - _gestureStart!.dy;

if (!_decisionMade) {

final distance = math.sqrt(dx * dx + dy * dy);

if (distance < _slopDistance) {

// Not big enough to decide

return;

}

_decisionMade = true;

// Calculate angle in degrees:

final angleDeg = math.atan(dy.abs() / dx.abs()) * (180 / math.pi);

// If angle <= 45°, we treat it as horizontal swipe

_isHorizontalSwipe = (angleDeg <= 45);

}

if (_isHorizontalSwipe) {

// Move the top card horizontally

final current = _dragPositionNotifier.value;

final newDx = current.dx + details.delta.dx;

final newDy =

current.dy + details.delta.dy * 0.7; // slight vertical shift is okay

_dragPositionNotifier.value = Offset(newDx, newDy);

_updateSwipeDirection(newDx);

} else {

// Custom vertical scroll

_verticalOffset -= details.delta.dy;

// Clamp so we can’t scroll above 0 or beyond _maxScrollExtent

_verticalOffset = _verticalOffset.clamp(0.0, _maxScrollExtent);

setState(() {});

}

}

void _updateSwipeDirection(double currentDx) {

if (_currentDirection == null) {

if (currentDx > 0) {

_currentDirection = "right";

_furthestRightSoFar = currentDx;

} else {

_currentDirection = "left";

_furthestLeftSoFar = currentDx;

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

} else {

// left

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

// Decide yes/no

final direction = (dx > 0) ? "yes" : "no";

_dismissCard(direction);

}

// Reset horizontal offset

_dragPositionNotifier.value = Offset.zero;

} else {

// We treat it as a vertical fling if velocity is large enough

final velocityY = details.velocity.pixelsPerSecond.dy;

if (velocityY.abs() > 200) {

// Animate a fling

_startVerticalFling(velocityY);

}

}

_gestureStart = null;

_decisionMade = false;

_isHorizontalSwipe = false;

_currentDirection = null;

}

void _startVerticalFling(double initialVelocity) {

// We’ll create a simple tween from current verticalOffset to some target.

// The target is unbounded here; we clamp it in the listener or we

// pick a quick estimate. For simplicity, let’s assume we fling

// but still clamp between 0.._maxScrollExtent.

final currentOffset = _verticalOffset;

// We just do an estimate: if velocity is positive, we fling downward to max. If negative, fling upward to 0.

final endOffset = (initialVelocity > 0) ? _maxScrollExtent : 0.0;

_flingAnimation =

Tween<double>(begin: currentOffset, end: endOffset).animate(

CurvedAnimation(

parent: _flingController!,

curve: Curves.easeOut,

),

);

// Start the controller:

_flingController!.forward(from: 0.0);

}

// ----------------------------------------------------------------

// REACTION POPUP

// ----------------------------------------------------------------

void _openReactionChat(Map<String, dynamic> reactionPayload) {

if (!reactionPayload.containsKey("profile_username")) {

debugPrint("Missing profile_username in reactionPayload");

} else {

debugPrint("Profile username: ${reactionPayload["profile_username"]}");

}

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

// ----------------------------------------------------------------

// BUILD

// ----------------------------------------------------------------

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

// BACK CARD (second-to-last)

if (visibleCards.length > 1)

Positioned.fill(

child: AnimatedScale(

duration: const Duration(milliseconds: 200),

scale: 0.95,

child: ProfileCard(

data: visibleCards.first,

// We do not use a ScrollController now;

// We rely on the “back card” being static or partially behind the top card.

onElementTap: _openReactionChat,

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

// Combine horizontal offset with vertical offset

child: Transform.translate(

offset: Offset(dx, -_verticalOffset),

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

child: RepaintBoundary(

key: ValueKey(visibleCards.last),

child: ProfileCard(

data: visibleCards.last,

// no scrollController, we do custom

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

// If fetching more, show a spinner at bottom

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

"