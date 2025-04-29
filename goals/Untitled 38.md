example of how to do propper swipe scroll differenciation:
"
import 'package:flutter/material.dart';
import 'dart:math' as math;

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Custom Gesture Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage>
    with SingleTickerProviderStateMixin {
  String _gestureStatus = "No gesture detected";

  // Track gesture start position
  Offset? _gestureStartPosition;
  // Create scroll controller to manually handle scrolling
  ScrollController _scrollController = ScrollController();
  // Flag to determine if we're in a swipe or scroll
  bool _isSwipe = false;

  // Card position
  double _cardOffsetX = 0.0;
  double _cardOffsetY = 0.0;

  // Animation controller for returning card to center
  late AnimationController _animationController;

  // Card visibility
  bool _isCardVisible = true;

  // Card counter to track replacements
  int _cardCounter = 1;

  // Screen width - will be initialized in build
  double _screenWidth = 0;

  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );

    _animationController.addListener(() {
      setState(() {
        _cardOffsetX = _cardOffsetX * (1 - _animationController.value);
        _cardOffsetY = _cardOffsetY * (1 - _animationController.value);
      });
    });
  }

  @override
  void dispose() {
    _scrollController.dispose();
    _animationController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // Get screen width
    _screenWidth = MediaQuery.of(context).size.width;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Gesture Demo'),
      ),
      body: Center(
        child: AnimatedSwitcher(
          duration: const Duration(milliseconds: 300),
          child: _isCardVisible ? _buildCard() : _buildNewCardAnimation(),
        ),
      ),
    );
  }

  Widget _buildCard() {
    return Transform.translate(
      key: ValueKey<int>(_cardCounter),
      offset: Offset(_cardOffsetX, _cardOffsetY),
      child: Card(
        elevation: 5,
        child: Container(
          width: 300,
          height: 400,
          padding: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              Text(
                "$_gestureStatus (Card #$_cardCounter)",
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                ),
              ),
              const SizedBox(height: 12),
              Expanded(
                child: GestureDetector(
                  onPanStart: _handlePanStart,
                  onPanUpdate: _handlePanUpdate,
                  onPanEnd: _handlePanEnd,
                  child: SingleChildScrollView(
                    controller: _scrollController,
                    physics:
                        const NeverScrollableScrollPhysics(), // We'll handle scrolling manually
                    child: Text(
                      _generateLongText(),
                      style: const TextStyle(fontSize: 16),
                    ),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildNewCardAnimation() {
    // This widget shows briefly while we're switching cards
    return const CircularProgressIndicator();
  }

  void _createNewCard() {
    // Hide current card
    setState(() {
      _isCardVisible = false;
    });

    // After a short delay, create a new card
    Future.delayed(const Duration(milliseconds: 300), () {
      // Reset position and increment counter
      _cardOffsetX = 0;
      _cardOffsetY = 0;
      _cardCounter++;

      // Create a new scroll controller for the new card
      _scrollController = ScrollController();

      // Show the new card
      setState(() {
        _isCardVisible = true;
        _gestureStatus = "New card created";
      });
    });
  }

  void _handlePanStart(DragStartDetails details) {
    _gestureStartPosition = details.globalPosition;
    _isSwipe = false;
    _animationController.reset();
    setState(() {
      _gestureStatus = "Gesture started";
    });
  }

  void _handlePanUpdate(DragUpdateDetails details) {
    if (_gestureStartPosition == null) return;

    // Calculate overall displacement from start position
    final dx = details.globalPosition.dx - _gestureStartPosition!.dx;
    final dy = details.globalPosition.dy - _gestureStartPosition!.dy;

    // Avoid division by zero and check for minimum movement
    if (dx.abs() < 10 && dy.abs() < 10) return; // Ignore very small movements

    if (!_isSwipe && dx.abs() > 20) {
      // Calculate angle of the overall gesture
      double angle = math.atan(dy.abs() / dx.abs()) * (180 / math.pi);

      // Determine if it's a swipe or scroll based on overall angle
      if (angle <= 45) {
        _isSwipe = true;
        setState(() {
          _gestureStatus = "Swiping (angle: ${angle.toStringAsFixed(1)}°)";
        });
      }
    }

    // Handle the gesture based on classification
    if (_isSwipe) {
      // Move the card with the swipe
      setState(() {
        _cardOffsetX = dx;
        // Add a small vertical movement based on the angle for more natural feel
        _cardOffsetY = dy * 0.2;
      });

      // Check if card should disappear (swiped more than 20% of screen width)
      if (dx.abs() > _screenWidth * 0.2) {
        _createNewCard();
        return;
      }
    } else {
      // Handle scrolling manually
      _scrollController.jumpTo(_scrollController.offset - details.delta.dy);
      setState(() {
        _gestureStatus = "Scrolling";
      });
    }
  }

  void _handlePanEnd(DragEndDetails details) {
    if (_isSwipe) {
      // If card is still visible, animate it back to center
      if (_isCardVisible) {
        // Get total displacement and velocity
        final dx = _gestureStartPosition != null
            ? details.velocity.pixelsPerSecond.dx
            : 0;

        // Handle swipe end based on direction
        if (dx > 0) {
          setState(() {
            _gestureStatus = "Swipe right completed";
          });
        } else if (dx < 0) {
          setState(() {
            _gestureStatus = "Swipe left completed";
          });
        }

        // Animate the card back to center
        _animationController.forward(from: 0.0);
      }
    }

    // Reset gesture tracking
    _gestureStartPosition = null;
    _isSwipe = false;
  }

  String _generateLongText() {
    return """
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam in dui mauris. Vivamus hendrerit arcu sed erat molestie vehicula. Sed auctor neque eu tellus rhoncus ut eleifend nibh porttitor. Ut in nulla enim.

Phasellus molestie magna non est bibendum non venenatis nisl tempor. Suspendisse dictum feugiat nisl ut dapibus. Mauris iaculis porttitor posuere. Praesent id metus massa, ut blandit odio.

Proin quis tortor orci. Etiam at risus et justo dignissim congue. Donec congue lacinia dui, a porttitor lectus condimentum laoreet. Nunc eu ullamcorper orci. Quisque eget odio ac lectus vestibulum faucibus eget in metus.

In pellentesque faucibus vestibulum. Nulla at nulla justo, eget luctus tortor. Nulla facilisi. Duis aliquet egestas purus in blandit. Curabitur vulputate, ligula lacinia scelerisque tempor, lacus lacus ornare ante, ac egestas est urna sit amet arcu.

Sed mollis, eros et ultrices tempus, mauris ipsum aliquam libero, non adipiscing dolor urna a orci. Curabitur sagittis sapien in est. Praesent pellentesque. Sed dictum.

Etiam elit elit, elementum sed varius at, adipiscing vitae est. Sed nec felis pellentesque, lacinia dui sed, ultricies sapien. Pellentesque orci lectus, consectetur vel posuere posuere, rutrum eu ipsum.

Aliquam eget odio odio, tincidunt pharetra, cursus ac, tristique sit amet, nisi. Sed arcu. Cras consequat.

Mauris pretium quam et urna. Fusce nibh. Aenean accumsan ultrices consectetur. Nullam at leo quis purus ultricies luctus. Aenean lectus elit, fermentum et, consectetur sit amet, porta at, neque. Aenean porta.

Praesent sagittis nonummy sapien. Praesent sagittis nonummy sapien. Aliquam tincidunt posuere sem. Aliquam tincidunt posuere sem. Aliquam tincidunt posuere sem. Aliquam tincidunt posuere sem. Aliquam tincidunt posuere sem.
""";
  }
} "

Please apply this gesturedetection logic to this (while keeping all else the same like the check and cross indicator when swiping and sorting on release of card and how next card move from back to front and so on only change the detectoon of swipe vs scroll):
"
import 'package:flutter/material.dart';
import 'package:cached_network_image/cached_network_image.dart';
import 'package:flutter_svg/flutter_svg.dart';
import 'package:google_fonts/google_fonts.dart';
import 'package:http/http.dart' as http;
import 'dart:async';
import 'dart:convert';
import 'dart:math' as math;
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:swipe_chat_play/colors.dart';
import 'package:swipe_chat_play/config.dart';

// Import your model & widgets
import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'
    as profile;
import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';
import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';
import 'package:swipe_chat_play/profile_layout/widgets/image_layout.dart';
import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';
import 'package:swipe_chat_play/tabs/chat/chat.dart';

class CardStack extends StatefulWidget {
  const CardStack({super.key});

  @override
  _CardStackState createState() => _CardStackState();
}

class _CardStackState extends State<CardStack> {
  List<profile.ProfileData> cardData = [];
  bool isLoading = true;
  bool isFetchingMore = false;

  // For controlling swipe offset on the top card.
  final ValueNotifier<Offset> _dragPositionNotifier =
      ValueNotifier<Offset>(Offset.zero);

  // Where the user's drag began.
  Offset _dragStart = Offset.zero;

  // Whether we've decided to swipe or scroll for this gesture.
  bool _isSwiping = false;
  bool _isScrolling = false;

  // Whether we’ve committed to a direction yet (horizontal vs. vertical).
  bool _decisionMade = false;

  // A small distance the user must drag before we decide swipe vs scroll.
  final double _slopDistance = 10;

  // Pagination / fetch logic
  int currentPage = 1;
  bool hasMore = true;

  // Distance threshold before we consider it a complete swipe.
  final double swipeThreshold = 93;

  // Only two cards visible at any time (front + back).
  final int maxVisibleCards = 2;

  // Fetch more profiles when we get down to 2 left.
  final int prefetchThreshold = 2;

  final Debouncer _debouncer = Debouncer(milliseconds: 500);

  // Which cards to show on screen: top + the one behind it
  List<profile.ProfileData> get visibleCards =>
      cardData.reversed.take(maxVisibleCards).toList().reversed.toList();

  @override
  void initState() {
    super.initState();
    _fetchProfiles(reset: true);
  }

  Future<void> _fetchProfiles({bool reset = false}) async {
    if (isFetchingMore || (!reset && !hasMore)) return;
    if (mounted) {
      setState(() => reset ? isLoading = true : isFetchingMore = true);
    }

    const storage = FlutterSecureStorage();
    final token = await storage.read(key: 'jwt_token');
    if (token == null) {
      if (mounted) {
        setState(() => isLoading = false);
      }
      return;
    }

    // Fetch 50 on reset, or 10 if continuing.
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

          if (mounted) {
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

          // Prefetch images
          final imageUrlsToPrefetch = <String>[];
          for (var profileData in profilesList) {
            for (var elementEntry in profileData.elements.entries) {
              var element = elementEntry.value;
              if (element.type == 'image') {
                final content = element.content;
                if (content is Map<String, dynamic>) {
                  final maybeUrls = content['imageUrls'];
                  final singleUrl = content['imageUrl'];

                  if (maybeUrls is List) {
                    final urls = maybeUrls.whereType<String>().toList();
                    imageUrlsToPrefetch.addAll(urls);
                  } else if (singleUrl is String) {
                    imageUrlsToPrefetch.add(singleUrl);
                  }
                }
              }
            }
          }

          for (final url in imageUrlsToPrefetch) {
            precacheImage(CachedNetworkImageProvider(url), context)
                .catchError((e) {
              debugPrint("Prefetch error for $url: $e");
            });
          }
        }
      }
    } catch (e) {
      debugPrint("Error fetching profiles: $e");
    } finally {
      if (mounted) {
        setState(() => reset ? isLoading = false : isFetchingMore = false);
      }
    }
  }

  /// Remove top card from the list & send the "swipe" to server.
  void _dismissCard(String swipeValue) {
    if (cardData.isNotEmpty) {
      final dismissedProfile = cardData.last;
      setState(() {
        cardData.removeLast();
      });
      _sendSwipe(dismissedProfile, swipeValue);
    }

    // Possibly fetch more if we’re getting low
    _debouncer.run(() {
      if (mounted && cardData.length <= prefetchThreshold && hasMore) {
        _fetchProfiles();
      }
    });
  }

  /// Send the swipe result to the server
  Future<void> _sendSwipe(
      profile.ProfileData profileData, String swipeValue) async {
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

  // -------------------------
  // GESTURE HANDLING LOGIC
  // -------------------------

  void _onPanStart(DragStartDetails details) {
    _dragStart = details.globalPosition;
    _isSwiping = false;
    _isScrolling = false;
    _decisionMade = false; // We haven't decided which direction yet
  }

  void _onPanUpdate(DragUpdateDetails details) {
    final dx = details.globalPosition.dx - _dragStart.dx;
    final dy = details.globalPosition.dy - _dragStart.dy;
    final distance = math.sqrt(dx * dx + dy * dy);

    // If we haven't moved enough to decide, don't do anything yet
    if (!_decisionMade) {
      if (distance < _slopDistance) {
        return; // wait until we pass the slop
      }
      _decisionMade = true;

      // Determine angle
      final angleDeg = math.atan2(dy, dx) * 180 / math.pi;
      final absAngle = angleDeg.abs();

      // If angle is near horizontal => SWIPE
      // i.e. ±45° from 0 or 180 => (absAngle <= 45 || absAngle >= 135)
      if (absAngle <= 45 || absAngle >= 135) {
        _isSwiping = true;
      } else {
        // else => SCROLL
        _isScrolling = true;
      }
    }

    // If we decided SWIPE => update offset
    if (_isSwiping) {
      final currentOffset = _dragPositionNotifier.value;
      final newOffset = currentOffset + details.delta;

      final screenSize = MediaQuery.of(context).size;
      // Prevent the card from moving off-screen too far
      if (newOffset.dx.abs() < screenSize.width &&
          newOffset.dy.abs() < screenSize.height * 1.5) {
        _dragPositionNotifier.value = newOffset;
      }
    } else {
      // If SCROLL => do nothing here, let SingleChildScrollView handle vertical
    }
  }

  void _onPanEnd(DragEndDetails details) {
    // If we never decided swipe => no dismiss
    if (!_isSwiping) {
      // Reset offset
      _dragPositionNotifier.value = Offset.zero;
      return;
    }

    // We are swiping => check distance
    final offset = _dragPositionNotifier.value;
    final distance = offset.distance;

    if (distance > swipeThreshold) {
      final swipeValue = (offset.dx > 0) ? "yes" : "no";
      _dismissCard(swipeValue);
    }
    // Reset offset no matter what
    _dragPositionNotifier.value = Offset.zero;
  }

  @override
  void dispose() {
    _debouncer.cancel();
    _dragPositionNotifier.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // If loading finished but no data
    if (!isLoading && cardData.isEmpty) {
      return Container(
        color: AppColors.background,
        child: const Center(
          child: Text("No profiles left, change filters"),
        ),
      );
    }

    // If still loading initial
    if (isLoading) {
      return Container(
        color: AppColors.background,
        child: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              SizedBox(
                width: 200,
                child: LinearProgressIndicator(),
              ),
              const SizedBox(height: 8),
              const Text("Loading profiles"),
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
            // BACK CARD (scaled slightly)
            if (visibleCards.length > 1)
              Positioned.fill(
                child: AnimatedScale(
                  duration: const Duration(milliseconds: 300),
                  scale: 0.95,
                  child: ProfileCard(data: visibleCards.first),
                ),
              ),

            // TOP CARD (swipable)
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
                      final factor = (dragOffset.dx.abs() / swipeThreshold)
                          .clamp(0.0, 1.0);
                      final double opacity = 1.0 - 0.2 * factor;
                      // Calculate rotation angle based on horizontal drag.
                      // When drag offset equals swipeThreshold, rotate ~10° (pi/18 radians)
                      final double rotationAngle =
                          (dragOffset.dx / swipeThreshold) * (math.pi / 18);

                      return Opacity(
                        opacity: opacity,
                        child: Transform.translate(
                          // Only apply 70% of the vertical offset while keeping horizontal movement the same.
                          offset: Offset(dragOffset.dx, dragOffset.dy * 0.6),
                          child: Transform.rotate(
                            angle: rotationAngle,
                            child: AnimatedSwitcher(
                              duration: const Duration(milliseconds: 300),
                              transitionBuilder:
                                  (Widget child, Animation<double> animation) {
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
                                child: ProfileCard(data: visibleCards.last),
                              ),
                            ),
                          ),
                        ),
                      );
                    },
                  ),
                ),
              ),

            // The YES/NO indicators
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

/// Shows a "ProfileCard" with scrollable content.
class ProfileCard extends StatelessWidget {
  final profile.ProfileData data;

  const ProfileCard({super.key, required this.data});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Card(
        color: AppColors.cardBackground, // Set card background to white
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
        child: _ProfileCardContent(data: data),
      ),
    );
  }
}

class _ProfileCardContent extends StatelessWidget {
  final profile.ProfileData data;
  const _ProfileCardContent({required this.data});

  @override
  Widget build(BuildContext context) {
    // The entire card is scrollable (vertical).
    return SingleChildScrollView(
      physics: const BouncingScrollPhysics(),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          TopSectionWidget(topSection: data.topSection),
          ...data.elements.entries.map((entry) => _buildElement(entry.value)),
        ],
      ),
    );
  }

  // ADD BACK THE DIVIDER LINES HERE
  Widget _buildElement(profile.ProfileElement element) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        // Divider before each element
        const Divider(
          height: 2,
          thickness: 2,
          color: AppColors.dividerLine,
        ),
        switch (element.type) {
          'icebreaker' => IceBreakerLayout(content: element.content),
          'image' =>
            _CachedImageLayout(key: UniqueKey(), content: element.content),
          'bubbles' => BubbleGridLayout(content: element.content),
          _ => const SizedBox(),
        },
      ],
    );
  }
}

/// A separate widget for handling images in a PageView with caching.
class _CachedImageLayout extends StatefulWidget {
  final dynamic content;
  const _CachedImageLayout({super.key, required this.content});

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
  void didUpdateWidget(covariant _CachedImageLayout oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.content != widget.content) {
      _pageController.jumpToPage(0);
      setState(() => _currentPage = 0);
    }
  }

  @override
  void dispose() {
    _pageController.dispose();
    super.dispose();
  }

  /// Safely extract either `['imageUrls']` or `['imageUrl']`.
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
              placeholder: (context, url) => Container(
                color: Colors.grey[200],
              ),
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

/// YES / NO indicators based on horizontal drag offset.
class SwipeIndicators extends StatelessWidget {
  final ValueNotifier<Offset> dragNotifier;
  final double swipeThreshold;

  const SwipeIndicators({
    super.key,
    required this.dragNotifier,
    required this.swipeThreshold,
  });

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder<Offset>(
      valueListenable: dragNotifier,
      builder: (context, offset, child) {
        final dx = offset.dx;
        final dy = offset.dy;
        final distance = offset.distance;

        final isHorizontal = dx.abs() >= dy.abs();
        final progress =
            isHorizontal ? (distance / swipeThreshold).clamp(0.0, 1.0) : 0.0;

        final leftOpacity = dx < 0 ? progress : 0.0;
        final rightOpacity = dx > 0 ? progress : 0.0;

        final screenHeight = MediaQuery.of(context).size.height;
        final verticalPosition = screenHeight / 2 - 30;

        return IgnorePointer(
          child: Stack(
            children: [
              // NO
              Positioned(
                left: 20,
                top: verticalPosition,
                child: AnimatedOpacity(
                  opacity: leftOpacity,
                  duration: const Duration(milliseconds: 200),
                  child: CircleAvatar(
                    radius: 30,
                    backgroundColor: Colors.red.withOpacity(0.9),
                    child:
                        const Icon(Icons.close, color: Colors.white, size: 30),
                  ),
                ),
              ),
              // YES
              Positioned(
                right: 20,
                top: verticalPosition,
                child: AnimatedOpacity(
                  opacity: rightOpacity,
                  duration: const Duration(milliseconds: 200),
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

/// Helper class for debouncing async calls (like fetch requests).
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