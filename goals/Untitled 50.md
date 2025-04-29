your task is, to take functionally from the different versions of my carstack for my dating app, the things i want and combine them to create one version which fullfills all my requirements at once called "final_version".

To do this, you must understand what part in my code, makes what i describe work.

Combine the scroll and swipe functionality from "cardstack version 1" with the react-to-profile-element functionality and the usage of "profile_layout.dart" from "cardstack version 2" with the verion 3 with new profile_data.dart and match.php and swipe.php (they now all work with the user_id from mongo unique id for each profile instead of the username please also do it like that in this final_version.)
please keep all that in mind while also following the instructions in "()" on what exactly to keep functionally from each version. 

cardstack version 1:
(here we have a custom scroll implementation, this was needed to make the filtering between horizontal and vertical scroll possible. Please take from version one both the custom scroll implementation and the functionality to differentiate between horizontal and vertical scroll)
"

import 'dart:async';

import 'dart:convert';

import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:cached_network_image/cached_network_image.dart';

// Dummy placeholders for your actual imports:

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

import 'package:swipe_chat_play/tabs/chat/chat.dart';

// Import your fonts file:

import 'package:swipe_chat_play/fonts.dart';

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

bool _isHorizontalSwipe =

false; // Are we in horizontal-swipe mode vs. vertical scroll?

bool _decisionMade = false; // Has the angle-based decision been made?

final double _slopDistance =

10; // Movement threshold before deciding H vs. V.

/// Tracks the **current** direction we have determined: "left", "right", or `null` if not set.

String? _currentDirection;

/// The furthest we’ve gone to the **right** so far (>= 0).

double _furthestRightSoFar = 0.0;

/// The furthest we’ve gone to the **left** so far (<= 0).

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

// Fetch 50 profiles on reset, or 10 if continuing.

final int limit = reset ? 50 : 10;

try {

final response = await http.get(

Uri.parse(

'${Config.backendBaseUrl}/match_algorithm/match.php?page=$currentPage&limit=$limit',

),

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

// "Top" profile is the last in the list:

final topProfile = cardData.last;

setState(() {

cardData.removeLast();

});

// Send the swipe to the server:

_sendSwipe(topProfile, swipeValue);

// Reset scroll controller for the new top card:

_topCardScrollController.dispose();

_topCardScrollController = ScrollController();

// Prefetch more if needed

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

final url = Uri.parse(

'http://node02.krasserserver.com:8002/swipe_chatt_play_api/match_algorithm/swipe.php',

);

final body = jsonEncode({

"receiver_username": profileData.topSection.name,

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

// If we haven't decided whether it's horizontal or vertical, do so now.

if (!_decisionMade) {

final distance = math.sqrt(dx * dx + dy * dy);

if (distance < _slopDistance) return; // Not enough movement yet

_decisionMade = true;

// If angle <= ~45°, treat it as horizontal => swipe

final angleDeg = math.atan(dy.abs() / dx.abs()) * (180 / math.pi);

_isHorizontalSwipe = (angleDeg <= 45);

}

if (_isHorizontalSwipe) {

// 1) Update the drag offset (this is the top card's position).

final current = _dragPositionNotifier.value;

final newDx = current.dx + details.delta.dx;

// Keep partial Y movement if you like:

final newDy = current.dy + details.delta.dy * 0.7;

_dragPositionNotifier.value = Offset(newDx, newDy);

// 2) Determine or switch "direction" based on dx and our 2%-beyond logic.

_updateSwipeDirection(newDx);

} else {

// Vertical scroll logic:

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

/// Switch direction after user crosses more than 2% from the furthest point

/// in the opposite direction.

void _updateSwipeDirection(double currentDx) {

// If we have no direction yet => pick it based on sign of dx

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

// If we are "right":

if (_currentDirection == "right") {

// Keep track of the furthest right point

if (currentDx > _furthestRightSoFar) {

_furthestRightSoFar = currentDx;

}

// Check if user has moved left enough to switch direction:

// i.e. dx < (furthestRightSoFar - directionChangeThreshold)

// and dx < 0 => definitely going left

if (currentDx < 0 &&

currentDx < (_furthestRightSoFar - directionChangeThreshold)) {

// Switch to left

_currentDirection = "left";

// Reset the 'furthest left' to the current dx

_furthestLeftSoFar = currentDx;

}

}

// If we are "left":

else if (_currentDirection == "left") {

// Keep track of furthest left

if (currentDx < _furthestLeftSoFar) {

_furthestLeftSoFar = currentDx;

}

// Check if user has moved right enough to switch direction:

// i.e. dx > (furthestLeftSoFar + directionChangeThreshold)

// and dx > 0 => definitely going right

if (currentDx > 0 &&

currentDx > (_furthestLeftSoFar + directionChangeThreshold)) {

// Switch to right

_currentDirection = "right";

// Reset the 'furthest right' to the current dx

_furthestRightSoFar = currentDx;

}

}

}

void _onPanEnd(DragEndDetails details) {

// If we ended in horizontal swipe mode:

if (_isHorizontalSwipe && cardData.isNotEmpty) {

final offset = _dragPositionNotifier.value;

final dx = offset.dx;

// If the absolute horizontal offset surpasses the final swipe threshold:

if (dx.abs() > swipeThreshold) {

final direction = (dx > 0) ? "yes" : "no";

_dismissCard(direction);

}

} else {

// If it was vertical scroll, apply momentum:

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

// Always reset horizontal offset and direction states.

_dragPositionNotifier.value = Offset.zero;

_gestureStart = null;

_isHorizontalSwipe = false;

_decisionMade = false;

// We'll also reset the "currentDirection" each time user lifts finger

// so next swipe can be fresh. If you prefer to keep that across drags,

// remove the following line:

_currentDirection = null;

}

@override

void dispose() {

_dragPositionNotifier.dispose();

_topCardScrollController.dispose();

_debouncer.cancel();

super.dispose();

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

// BACK CARD

if (visibleCards.length > 1)

Positioned.fill(

child: AnimatedScale(

duration: const Duration(milliseconds: 200),

scale: 0.95,

child: ProfileCard(

data: visibleCards.first,

scrollController: ScrollController(),

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

// Fade out based on how close dx is to threshold:

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

.animate(CurvedAnimation(

parent: animation,

curve: Curves.easeOutBack,

)),

child: child,

);

},

child: RepaintBoundary(

key: ValueKey(visibleCards.last),

child: ProfileCard(

data: visibleCards.last,

scrollController: _topCardScrollController,

),

),

),

),

);

},

),

),

),

// YES/NO indicators (fade in/out)

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

class ProfileCard extends StatelessWidget {

final profile.ProfileData data;

final ScrollController scrollController;

const ProfileCard({

Key? key,

required this.data,

required this.scrollController,

}) : super(key: key);

@override

Widget build(BuildContext context) {

// Wrap the card content in a DefaultTextStyle to apply a base font style.

return Padding(

padding: const EdgeInsets.all(8.0),

child: Card(

color: AppColors.cardBackground,

shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),

child: DefaultTextStyle(

style: AppFonts.primaryFont,

child: SingleChildScrollView(

physics: const NeverScrollableScrollPhysics(),

controller: scrollController,

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

// Apply a different style for the top section:

DefaultTextStyle(

style: AppFonts.quaternaryFont,

child: TopSectionWidget(topSection: data.topSection),

),

...data.elements.entries.map((entry) {

final element = entry.value;

return Column(

mainAxisSize: MainAxisSize.min,

children: [

const Divider(

height: 2,

thickness: 2,

color: AppColors.dividerLine,

),

switch (element.type) {

'icebreaker' =>

IceBreakerLayout(content: element.content),

'image' => _CachedImageLayout(content: element.content),

'bubbles' => BubbleGridLayout(content: element.content),

_ => const SizedBox(),

}

],

);

}),

],

),

),

),

),

);

}

}

/// Basic "YES"/"NO" indicators that fade in as the user swipes horizontally.

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

// We'll only show the icons strongly for horizontal movement:

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

class _CachedImageLayout extends StatefulWidget {

final dynamic content;

const _CachedImageLayout({Key? key, required this.content}) : super(key: key);

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

void _onNextPage() {

final urls = _extractUrls(widget.content);

if (urls.isEmpty) return;

if (_currentPage == urls.length - 1) {

_pageController.jumpToPage(0);

setState(() => _currentPage = 0);

} else {

_pageController.nextPage(

duration: const Duration(milliseconds: 300),

curve: Curves.easeInOut,

);

setState(() => _currentPage++);

}

}

@override

Widget build(BuildContext context) {

final urls = _extractUrls(widget.content);

if (urls.isEmpty) return const SizedBox();

return GestureDetector(

onTap: _onNextPage,

child: SizedBox(

height: 200,

child: PageView.builder(

controller: _pageController,

itemCount: urls.length,

onPageChanged: (index) => setState(() => _currentPage = index),

itemBuilder: (context, index) {

final imageUrl = urls[index];

return CachedNetworkImage(

imageUrl: imageUrl,

fit: BoxFit.cover,

placeholder: (context, url) => Container(color: Colors.grey[200]),

errorWidget: (context, url, error) => Container(

color: Colors.grey[400],

child: const Icon(Icons.error),

),

);

},

),

),

);

}

}

/// Simple Debouncer for prefetch logic

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

"


cardstack version2:
(use the react to elements functionality and also use the import 'package:swipe_chat_play/tabs/profile/layouts/profile_layout.dart in the same way to build the layout)
"
// cardstack.dart

import 'dart:async';

import 'dart:convert';

import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

  

// Dummy placeholders for your actual imports:

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/fonts.dart';

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

    as profile;

// ^---- The file where ProfileCard is defined

import 'package:swipe_chat_play/tabs/chat/layouts/chat_popup.dart';

import 'package:swipe_chat_play/tabs/profile/layouts/profile_layout.dart';

// ^---- The new popup for sending "react_..." messages

  

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

  

  // Tracks the drag offset

  final ValueNotifier<Offset> _dragPositionNotifier =

      ValueNotifier<Offset>(Offset.zero);

  

  // Controller for the top card’s vertical scrolling

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

  

    // Send the swipe to server

    _sendSwipe(topProfile, swipeValue);

  

    // Reset scroll controller for new top card

    _topCardScrollController.dispose();

    _topCardScrollController = ScrollController();

  

    // Prefetch more if needed

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

  

    final url = Uri.parse(

      '${Config.backendBaseUrl}/match_algorithm/swipe.php',

    );

    final body = jsonEncode({

      "receiver_username": profileData.topSection.name,

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

          // If it's a match, do something...

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

  // GESTURE DETECTION

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

  

  // This method is called whenever a child element is tapped and wants to open reaction chat

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

            // BACK CARD

            if (visibleCards.length > 1)

              Positioned.fill(

                child: AnimatedScale(

                  duration: const Duration(milliseconds: 200),

                  scale: 0.95,

                  child: ProfileCard(

                    data: visibleCards.first,

                    scrollController: ScrollController(),

                    onElementTap: _openReactionChat, // <--- Key line

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

                            child: RepaintBoundary(

                              key: ValueKey(visibleCards.last),

                              child: ProfileCard(

                                data: visibleCards.last,

                                scrollController: _topCardScrollController,

                                onElementTap:

                                    _openReactionChat, // <--- Key line

                              ),

                            ),

                          ),

                        ),

                      );

                    },

                  ),

                ),

              ),

  

            // If fetching more

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

  

/// Simple Debouncer

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
"


version 3
profile_data.dart:
"
// profile_data.dart

  

class ProfileData {

  final TopSection topSection;

  final Map<String, ProfileElement> elements;

  

  ProfileData({required this.topSection, required this.elements});

  

  factory ProfileData.fromJson(Map<String, dynamic> json) {

    // Use an empty map if 'topSection' is missing.

    final topSectionJson = (json['topSection'] as Map<String, dynamic>?) ?? {};

    final topSection = TopSection.fromJson(topSectionJson);

  

    // Use an empty map if 'elements' is missing.

    final elementsJson = (json['elements'] as Map<String, dynamic>?) ?? {};

    final elementsMap = elementsJson.map((key, value) {

      final elementMap = (value as Map<String, dynamic>?) ?? {};

      return MapEntry(key, ProfileElement.fromJson(elementMap));

    });

  

    return ProfileData(

      topSection: topSection,

      elements: elementsMap,

    );

  }

}

  

class TopSection {

  final List<String> profileImages;

  final String name;

  final String username;

  final String id; // Unique MongoDB identifier

  final String age;

  final List<String> infoBubbles;

  

  TopSection({

    required this.profileImages,

    required this.name,

    required this.username,

    required this.id,

    required this.age,

    required this.infoBubbles,

  });

  

  factory TopSection.fromJson(Map<String, dynamic> json) {

    return TopSection(

      profileImages: List<String>.from(json['profileImages'] ?? []),

      name: json['name']?.toString() ?? '',

      username: json['username']?.toString() ?? '',

      id: json['id']?.toString() ?? '', // Ensure the unique id is captured

      age: json['age']?.toString() ?? '',

      infoBubbles: List<String>.from(json['infoBubbles'] ?? []),

    );

  }

}

  

class ProfileElement {

  final String type;

  final Map<String, dynamic> content;

  

  ProfileElement({required this.type, required this.content});

  

  factory ProfileElement.fromJson(Map<String, dynamic> json) {

    return ProfileElement(

      type: json['type']?.toString() ?? '',

      content: (json['content'] as Map<String, dynamic>?) ?? {},

    );

  }

}
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

swipe.php:
"
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

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