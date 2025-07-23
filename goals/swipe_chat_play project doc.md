# flutter client
## project structure

lib:
|   colors.dart
|   config.dart
|   fonts.dart
|   main.dart
|   main_page.dart
|
+---login_signup
|       addFingerprint.dart
|       login.dart
|       signup.dart
|       verify_code.dart
|       verify_code_login.dart
|
+---mainScreen
|       create_profile.dart
|       edit_image.dart
|       edit_main_image.dart
|       navigation_bar.dart
|       profilepage.dart
|
+---profile_creation
|       aboutYouLayout.dart
|       choose_question.dart
|       dating_radius.dart
|       open_to.dart
|       profilbilder.dart
|       profile.dart
|       profil_elemente.dart
|       qa_item.dart
|       searching.dart
|
+---profile_layout
|   +---models
|   |       profile_data.dart
|   |
|   \---widgets
|           bubble_grid_layout.dart
|           ice_breaker_layout.dart
|           image_layout.dart
|           top_section.dart
|
+---tabs
|   +---chat
|   |   |   my_chats.dart
|   |   |   search_user.dart
|   |   |
|   |   +---layouts
|   |   |       attach_file_layout.dart
|   |   |       chat.dart
|   |   |       chat_bubble_left_layout.dart
|   |   |       chat_bubble_right_layout.dart
|   |   |       chat_popup.dart
|   |   |       contract_bubble_left_layout.dart
|   |   |       contract_bubble_right_layout.dart
|   |   |       grid_of_cards_layout.dart
|   |   |       opening_messages_layout.dart
|   |   |       send_images_layout.dart
|   |   |       send_location_layout.dart
|   |   |
|   |   \---new_bubbles
|   |       |   answer_to_message_layout.dart
|   |       |   location_bubble_layout.dart
|   |       |   react_bubble_layout.dart
|   |       |   react_icebreaker_layout.dart
|   |       |   react_interessen_layout.dart
|   |       |   react_multiple_choice_layout.dart
|   |       |   react_music_layout.dart
|   |       |   react_profile_image_carousel_layout.dart
|   |       |   react_profile_image_layout.dart
|   |       |   react_qa_layout.dart
|   |       |   react_recorded_message_layout.dart
|   |       |   react_top_section_bubble_layout.dart
|   |       |
|   |       \---new_chatbubble_layouts
|   |               speech_bubble_layout.dart
|   |
|   +---profile
|   |   |   edit_profile.dart
|   |   |   edit_profile_activities_page.dart
|   |   |   edit_profile_filters_page.dart
|   |   |   edit_profile_photos_page.dart
|   |   |   edit_profile_qa_page.dart
|   |   |   edit_profile_radius_page.dart
|   |   |   profile.dart
|   |   |   profile_tab.dart
|   |   |   view_my_profile.dart
|   |   |
|   |   \---layouts
|   |           profile_layout.dart
|   |
|   \---swipe
|           card_stack.dart
|           profileCard.dart
|
+---test
\---usables
        inappwebview.dart

[the rest is just standart flutter project structure]
## files

- config.dart
```
class Config {

  static const String _backendBaseUrl =

      'http://panel.krasserserver.com:8002/swipe_chatt_play_api';

  

  static String get backendBaseUrl => _backendBaseUrl;

}
```

- main.dart
```
import 'package:flutter/material.dart';

import 'package:flutter/services.dart'; // Required for SystemUiOverlayStyle

import 'package:swipe_chat_play/login_signup/signup.dart';

  

void main() {

  runApp(SwipeableCardsApp());

  SystemChrome.setSystemUIOverlayStyle(

    const SystemUiOverlayStyle(

      statusBarColor: Colors.white, // White status bar background

      statusBarIconBrightness: Brightness.dark, // Dark icons (for white bg)

      systemNavigationBarColor: Colors.white, // White navigation bar

      systemNavigationBarIconBrightness: Brightness.dark, // Dark icons

    ),

  );

}

  

class SwipeableCardsApp extends StatelessWidget {

  const SwipeableCardsApp({super.key});

  

  @override

  Widget build(BuildContext context) {

    SystemChrome.setSystemUIOverlayStyle(

      const SystemUiOverlayStyle(

        statusBarColor: Colors.white, // White status bar

        statusBarIconBrightness:

            Brightness.dark, // Dark icons for white background

        systemNavigationBarColor: Colors.white,

        systemNavigationBarIconBrightness: Brightness.dark,

      ),

    );

  

    return MaterialApp(

      debugShowCheckedModeBanner: false,

      theme: ThemeData(

        scaffoldBackgroundColor: Colors.white, // Background color for safe area

      ),

      home: SignupScreen(),

    );

  }

}

  

/*

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

}

*/
```
- main_page.dart
```
import 'package:flutter/material.dart';

import 'package:swipe_chat_play/profile_creation/profile.dart';

import 'package:swipe_chat_play/tabs/profile/profile.dart';

import 'package:swipe_chat_play/tabs/profile/view_my_profile.dart';

import 'package:swipe_chat_play/tabs/swipe/card_stack.dart';

import 'package:swipe_chat_play/tabs/chat/my_chats.dart';

import 'package:swipe_chat_play/mainScreen/navigation_bar.dart';

  

class MainPage extends StatefulWidget {

  const MainPage({super.key});

  

  @override

  _MainPageState createState() => _MainPageState();

}

  

class _MainPageState extends State<MainPage> {

  int _selectedIndex = 0;

  bool _isPreloading = false; // Track preloading state

  

  final List<Widget> _pages = [

    ProfileScreen(), // profile

    CardStack(), // swipe

    MyChatsPage(), // chat

  ];

  

  @override

  void initState() {

    super.initState();

    // Preload images or any initialization here if needed

  }

  

  void _onItemTapped(int index) {

    setState(() {

      _selectedIndex = index;

    });

  }

  

  Future<void> _preloadProfileImages() async {

    setState(() {

      _isPreloading = true;

    });

  

    try {

      await ProfilePage.preloadImages(context);

    } catch (e) {

      print('Failed to preload images: $e');

    } finally {

      setState(() {

        _isPreloading = false;

      });

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      body: SafeArea(

        child: Stack(

          children: [

            _pages[_selectedIndex], // Display the selected page

            if (_isPreloading && _selectedIndex == 0)

              const Center(

                child: CircularProgressIndicator(), // Show loading indicator

              ),

          ],

        ),

      ),

      bottomNavigationBar: CustomNavigationBar(

        selectedIndex: _selectedIndex,

        onTap: _onItemTapped,

      ),

    );

  }

}
```
- login_signup/addFingerprint.dart
```
import 'package:flutter/material.dart';

import 'package:http/http.dart' as http;

import 'package:local_auth/local_auth.dart'; // For fingerprint authentication

import 'package:flutter_secure_storage/flutter_secure_storage.dart'; // For secure storage

import 'package:device_info_plus/device_info_plus.dart'; // For device info

import 'dart:convert';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/main_page.dart';

import 'package:swipe_chat_play/profile_creation/aboutYouLayout.dart';

  

class AddFingerprintScreen extends StatefulWidget {

  const AddFingerprintScreen({super.key});

  

  @override

  _AddFingerprintScreenState createState() => _AddFingerprintScreenState();

}

  

class _AddFingerprintScreenState extends State<AddFingerprintScreen> {

  final LocalAuthentication _auth = LocalAuthentication();

  final FlutterSecureStorage _secureStorage = FlutterSecureStorage();

  bool _isLoading = false;

  

  Future<String> getFingerprintHash() async {

    DeviceInfoPlugin deviceInfo = DeviceInfoPlugin();

    String fingerprintHash = "unknown";

  

    try {

      if (Theme.of(context).platform == TargetPlatform.android) {

        // Android: Use Android ID

        AndroidDeviceInfo androidInfo = await deviceInfo.androidInfo;

        fingerprintHash = androidInfo.id ?? "unknown-android-id";

      } else if (Theme.of(context).platform == TargetPlatform.iOS) {

        // iOS: Use identifierForVendor

        IosDeviceInfo iosInfo = await deviceInfo.iosInfo;

        fingerprintHash = iosInfo.identifierForVendor ?? "unknown-ios-id";

      }

    } catch (e) {

      print("Error fetching device-specific ID: $e");

    }

  

    return fingerprintHash;

  }

  

  Future<void> addFingerprint() async {

    try {

      setState(() {

        _isLoading = true;

      });

  

      // Check if device supports biometric authentication

      bool canAuthenticate = await _auth.canCheckBiometrics;

      if (!canAuthenticate) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content:

                Text('Your device does not support fingerprint authentication'),

          ),

        );

        return;

      }

  

      // Authenticate the user

      bool authenticated = await _auth.authenticate(

        localizedReason: 'Please authenticate to add your fingerprint',

        options: const AuthenticationOptions(biometricOnly: true),

      );

  

      if (!authenticated) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text('Authentication failed')),

        );

        return;

      }

  

      // Get JWT token from secure storage

      final String? token = await _secureStorage.read(key: 'jwt_token');

      if (token == null) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content:

                Text('Authentication token not found. Please login again.'),

          ),

        );

        return;

      }

  

      // Get device-specific fingerprint hash

      final String fingerprintHash = await getFingerprintHash();

  

      // Call the addFingerprint.php API

      final response = await http.post(

        Uri.parse('${Config.backendBaseUrl}/login_signup/addFingerprint.php'),

        headers: {

          "Authorization": "Bearer $token",

          "Content-Type": "application/x-www-form-urlencoded",

        },

        body: {

          "fingerprint": fingerprintHash,

        },

      );

  

      if (response.statusCode == 200) {

        final responseData = json.decode(response.body);

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text('Fingerprint added successfully')),

        );

  

        // Navigate to the main page

        Navigator.pushReplacement(

          context,

          MaterialPageRoute(builder: (context) => AboutYouLayout()),

        );

      } else {

        final errorMessage =

            json.decode(response.body)['error'] ?? 'Unknown error';

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text('Error: $errorMessage')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('An error occurred: $e')),

      );

    } finally {

      setState(() {

        _isLoading = false;

      });

    }

  }

  

  Widget _buildLoadingIndicator() {

    return Center(

      child: CircularProgressIndicator(

        valueColor: AlwaysStoppedAnimation<Color>(Colors.white),

      ),

    );

  }

  

  Widget _buildContent() {

    return Column(

      mainAxisAlignment: MainAxisAlignment.center,

      children: [

        Icon(

          Icons.fingerprint,

          size: 100,

          color: Colors.white,

        ),

        SizedBox(height: 20),

        Text(

          'Add Your Fingerprint',

          style: TextStyle(

            fontSize: 24,

            color: Colors.white,

            fontWeight: FontWeight.bold,

          ),

        ),

        SizedBox(height: 10),

        Padding(

          padding: EdgeInsets.symmetric(horizontal: 32.0),

          child: Text(

            'Securely add your fingerprint to quickly and easily access your account.',

            textAlign: TextAlign.center,

            style: TextStyle(

              color: Colors.white70,

              fontSize: 16,

            ),

          ),

        ),

        SizedBox(height: 30),

        ElevatedButton(

          onPressed: addFingerprint,

          style: ElevatedButton.styleFrom(

            backgroundColor: Colors.white,

            foregroundColor: Theme.of(context).primaryColor,

            padding: EdgeInsets.symmetric(horizontal: 40, vertical: 16),

            shape: RoundedRectangleBorder(

              borderRadius: BorderRadius.circular(25),

            ),

          ),

          child: Text(

            'Add Fingerprint',

            style: TextStyle(

              fontSize: 18,

              fontWeight: FontWeight.w600,

            ),

          ),

        ),

        SizedBox(height: 20),

        TextButton(

          onPressed: () {

            Navigator.pushReplacement(

              context,

              MaterialPageRoute(builder: (context) => MainPage()),

            );

          },

          child: Text(

            'Skip',

            style: TextStyle(

              color: Colors.white70,

              fontSize: 16,

            ),

          ),

        ),

      ],

    );

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      // Optional: Customize your app bar if you want a different style

      appBar: AppBar(

        title: Text(

          'Add Fingerprint',

          style: TextStyle(color: Colors.white),

        ),

        backgroundColor: Colors.deepPurple,

        centerTitle: true,

        iconTheme: IconThemeData(color: Colors.white),

      ),

      body: Container(

        // Use a gradient background

        decoration: BoxDecoration(

          gradient: LinearGradient(

            colors: [

              AppColors.secondaryGradientStart,

              AppColors.secondaryGradientEnd

            ],

            begin: Alignment.topCenter,

            end: Alignment.bottomCenter,

          ),

        ),

        child: Center(

          child: SingleChildScrollView(

            child: _isLoading ? _buildLoadingIndicator() : _buildContent(),

          ),

        ),

      ),

    );

  }

}
```
- login_signup/login.dart
```
import 'package:flutter/material.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';

import 'dart:convert';

  

import 'verify_code_login.dart'; // Import VerifyCodeScreen

  

class LoginScreen extends StatefulWidget {

  const LoginScreen({super.key});

  

  @override

  _LoginScreenState createState() => _LoginScreenState();

}

  

class _LoginScreenState extends State<LoginScreen> {

  final TextEditingController _emailController = TextEditingController();

  final TextEditingController _passwordController = TextEditingController();

  bool _isLoading = false;

  

  Future<void> loginUser() async {

    final String email = _emailController.text.trim();

    final String password = _passwordController.text.trim();

  

    if (email.isEmpty || password.isEmpty) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Please fill in all fields')),

      );

      return;

    }

  

    setState(() {

      _isLoading = true;

    });

  

    final url = Uri.parse('${Config.backendBaseUrl}/login_signup/login.php');

  

    try {

      final response = await http.post(

        url,

        body: {'email': email, 'password': password},

      );

  

      if (response.statusCode == 200) {

        final Map<String, dynamic> responseJson = json.decode(response.body);

  

        if (responseJson['message'] == 'Verification code sent') {

          Navigator.push(

            context,

            MaterialPageRoute(

              builder: (context) => VerifyCodeScreenLogin(email: email),

            ),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(content: Text('Unexpected response')),

          );

        }

      } else {

        final error = json.decode(response.body)['error'];

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text(error ?? 'Login failed')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('An error occurred: $e')),

      );

    } finally {

      setState(() {

        _isLoading = false;

      });

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: Text('Login')),

      body: Padding(

        padding: const EdgeInsets.all(16.0),

        child: Column(

          children: [

            TextField(

              controller: _emailController,

              decoration: InputDecoration(labelText: 'Email'),

            ),

            TextField(

              controller: _passwordController,

              decoration: InputDecoration(labelText: 'Password'),

              obscureText: true,

            ),

            SizedBox(height: 20),

            _isLoading

                ? CircularProgressIndicator()

                : ElevatedButton(

                    onPressed: loginUser,

                    child: Text('Login'),

                  ),

          ],

        ),

      ),

    );

  }

}
```

- login_signup/signup.dart
```
import 'dart:convert';

import 'package:device_info_plus/device_info_plus.dart';

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:local_auth/local_auth.dart'; // For fingerprint authentication

import 'package:http/http.dart' as http;

  

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/main_page.dart';

import 'verify_code.dart';

import 'login.dart';

  

class SignupScreen extends StatefulWidget {

  const SignupScreen({super.key});

  

  @override

  _SignupScreenState createState() => _SignupScreenState();

}

  

class _SignupScreenState extends State<SignupScreen> {

  final TextEditingController _usernameController = TextEditingController();

  final TextEditingController _emailController = TextEditingController();

  final TextEditingController _passwordController = TextEditingController();

  

  final LocalAuthentication _auth = LocalAuthentication();

  final FlutterSecureStorage _secureStorage = const FlutterSecureStorage();

  

  bool _isLoading = false;

  bool _isFingerprintLoading = false;

  

  // ==============================================================

  // SIGN UP LOGIC

  // ==============================================================

  Future<void> signupUser() async {

    final String username = _usernameController.text.trim();

    final String email = _emailController.text.trim();

    final String password = _passwordController.text.trim();

  

    if (username.isEmpty || email.isEmpty || password.isEmpty) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('Please fill in all fields')),

      );

      return;

    }

  

    if (password.length < 8) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

            content: Text('Password must be at least 8 characters long')),

      );

      return;

    }

  

    setState(() => _isLoading = true);

  

    final url = Uri.parse('${Config.backendBaseUrl}/login_signup/signup.php');

  

    try {

      final response = await http.post(

        url,

        headers: {'Content-Type': 'application/x-www-form-urlencoded'},

        body: {

          'username': username,

          'email': email,

          'password': password,

        },

      );

  

      // Check for successful or duplicate responses

      if (response.statusCode == 200 || response.statusCode == 201) {

        if (response.body.isNotEmpty) {

          final Map<String, dynamic> responseJson = json.decode(response.body);

          final String message = responseJson['message'] ?? '';

  

          if (message == "Success" || message == "not verified") {

            // Navigate to VerifyCodeScreen for new registrations or if the user exists but not verified

            Navigator.push(

              context,

              MaterialPageRoute(

                builder: (context) => VerifyCodeScreen(email: email),

              ),

            );

          } else if (message == "is verified") {

            // Navigate to LoginScreen if the user already exists & is verified

            Navigator.pushReplacement(

              context,

              MaterialPageRoute(builder: (context) => const LoginScreen()),

            );

          } else {

            ScaffoldMessenger.of(context).showSnackBar(

              const SnackBar(content: Text('Unexpected server response')),

            );

          }

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(

                content: Text('Signup successful, but no response body')),

          );

        }

      } else if (response.statusCode == 400 || response.statusCode == 500) {

        if (response.body.isNotEmpty) {

          final error = json.decode(response.body)['error'];

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(content: Text(error ?? 'Error occurred')),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(

                content: Text('Error occurred, but no response body')),

          );

        }

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text('Unexpected error occurred')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('An error occurred: $e')),

      );

    } finally {

      setState(() => _isLoading = false);

    }

  }

  

  // ==============================================================

  // GET FINGERPRINT HASH (DEVICE-SPECIFIC ID)

  // ==============================================================

  Future<String> getFingerprintHash() async {

    final deviceInfo = DeviceInfoPlugin();

    String fingerprintHash = "unknown";

  

    try {

      if (Theme.of(context).platform == TargetPlatform.android) {

        final androidInfo = await deviceInfo.androidInfo;

        fingerprintHash = androidInfo.id ?? "unknown-android-id";

      } else if (Theme.of(context).platform == TargetPlatform.iOS) {

        final iosInfo = await deviceInfo.iosInfo;

        fingerprintHash = iosInfo.identifierForVendor ?? "unknown-ios-id";

      }

    } catch (e) {

      debugPrint("Error fetching device-specific ID: $e");

    }

    return fingerprintHash;

  }

  

  // ==============================================================

  // FINGERPRINT LOGIN WITH LOCKOUT HANDLING

  // ==============================================================

  Future<void> fingerprintLogin() async {

    try {

      setState(() => _isFingerprintLoading = true);

  

      final bool canAuthenticate = await _auth.canCheckBiometrics;

      if (!canAuthenticate) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(

              content: Text(

                  'Your device does not support fingerprint authentication')),

        );

        return;

      }

  

      bool authenticated = false;

  

      try {

        authenticated = await _auth.authenticate(

          localizedReason: 'Please authenticate to log in',

          options: const AuthenticationOptions(

            biometricOnly: true, // no passcode fallback

            stickyAuth: true,

          ),

        );

      } catch (e) {

        // local_auth throws PlatformExceptions for lockouts, etc.

        final errorString = e.toString().toLowerCase();

        if (errorString.contains('lockedout') ||

            errorString.contains('permanentlylockedout') ||

            errorString.contains('biometriclockout')) {

          // Show a message and send the user to normal login

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(

                content: Text('Too many attempts. Please use your password.')),

          );

          Navigator.pushReplacement(

            context,

            MaterialPageRoute(builder: (context) => const LoginScreen()),

          );

          return;

        } else {

          // some other error, rethrow or handle differently

          rethrow;

        }

      }

  

      if (!authenticated) {

        // The user either canceled or it outright failed

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text('Authentication failed or canceled')),

        );

        return;

      }

  

      // If here, the user was successfully authenticated

      final String deviceFingerprint = await getFingerprintHash();

      final response = await http.post(

        Uri.parse('${Config.backendBaseUrl}/login_signup/fingerprintLogin.php'),

        headers: {"Content-Type": "application/x-www-form-urlencoded"},

        body: {

          "fingerprint": deviceFingerprint,

        },

      );

  

      if (response.statusCode == 200) {

        final responseData = json.decode(response.body);

        if (responseData != null && responseData['token'] != null) {

          final String token = responseData['token'];

          await _secureStorage.write(key: 'jwt_token', value: token);

  

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(content: Text('Fingerprint login successful')),

          );

          Navigator.pushReplacement(

            context,

            MaterialPageRoute(builder: (context) => const MainPage()),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(content: Text('No token returned from server')),

          );

        }

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text('Error: ${response.body}')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('An error occurred: $e')),

      );

    } finally {

      setState(() => _isFingerprintLoading = false);

    }

  }

  

  // ==============================================================

  // UI BUILD

  // ==============================================================

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      body: Stack(

        children: [

          // Background gradient

          Container(

            width: double.infinity,

            height: double.infinity,

            decoration: BoxDecoration(

              gradient: LinearGradient(

                colors: [

                  AppColors.primaryGradientStart,

                  AppColors.primaryGradientEnd,

                ],

                begin: Alignment.topCenter,

                end: Alignment.bottomCenter,

              ),

            ),

            // Scrollable signup form

            child: SingleChildScrollView(

              padding:

                  const EdgeInsets.symmetric(horizontal: 24.0, vertical: 48.0),

              child: Column(

                children: [

                  const SizedBox(height: 60),

                  _buildRoundedTextField(

                    controller: _usernameController,

                    labelText: 'Username',

                    icon: Icons.person_outline,

                  ),

                  const SizedBox(height: 16),

                  _buildRoundedTextField(

                    controller: _emailController,

                    labelText: 'Email',

                    icon: Icons.email_outlined,

                    keyboardType: TextInputType.emailAddress,

                  ),

                  const SizedBox(height: 16),

                  _buildRoundedTextField(

                    controller: _passwordController,

                    labelText: 'Password',

                    icon: Icons.lock_outline,

                    obscureText: true,

                  ),

                  const SizedBox(height: 30),

                  _isLoading

                      ? const CircularProgressIndicator(

                          valueColor:

                              AlwaysStoppedAnimation<Color>(Colors.white),

                        )

                      : ElevatedButton(

                          onPressed: signupUser,

                          style: ElevatedButton.styleFrom(

                            backgroundColor: Colors.white,

                            foregroundColor: Colors.teal,

                            padding: const EdgeInsets.symmetric(

                                horizontal: 40.0, vertical: 16.0),

                            shape: RoundedRectangleBorder(

                              borderRadius: BorderRadius.circular(30),

                            ),

                          ),

                          child: Text(

                            'Sign Up',

                            style: TextStyle(

                              fontSize: 16,

                              fontWeight: FontWeight.bold,

                              color: AppColors.primaryColorDark,

                            ),

                          ),

                        ),

                  const SizedBox(height: 16),

                  TextButton(

                    onPressed: () {

                      Navigator.push(

                        context,

                        MaterialPageRoute(

                            builder: (context) => const LoginScreen()),

                      );

                    },

                    child: const Text(

                      'Login instead',

                      style: TextStyle(color: Colors.white),

                    ),

                  ),

                ],

              ),

            ),

          ),

  

          // Fingerprint button at the bottom

          Positioned(

            bottom: 30,

            left: 0,

            right: 0,

            child: Center(

              child: CircleAvatar(

                radius: 35,

                backgroundColor: AppColors.white,

                child: IconButton(

                  onPressed: fingerprintLogin,

                  icon: Icon(

                    Icons.fingerprint,

                    color: AppColors.textOnWhite,

                    size: 35,

                  ),

                ),

              ),

            ),

          ),

        ],

      ),

    );

  }

  

  // ==============================================================

  // HELPER: ROUNDED TEXT FIELD

  // ==============================================================

  Widget _buildRoundedTextField({

    required TextEditingController controller,

    required String labelText,

    IconData? icon,

    bool obscureText = false,

    TextInputType? keyboardType,

  }) {

    return TextField(

      controller: controller,

      obscureText: obscureText,

      keyboardType: keyboardType ?? TextInputType.text,

      style: const TextStyle(color: Colors.white),

      decoration: InputDecoration(

        prefixIcon: icon != null ? Icon(icon, color: Colors.white) : null,

        labelText: labelText,

        labelStyle: const TextStyle(color: Colors.white70),

        filled: true,

        fillColor: Colors.white.withOpacity(0.1),

        focusedBorder: OutlineInputBorder(

          borderRadius: BorderRadius.circular(30),

          borderSide: const BorderSide(color: Colors.white70),

        ),

        enabledBorder: OutlineInputBorder(

          borderRadius: BorderRadius.circular(30),

          borderSide: const BorderSide(color: Colors.white54),

        ),

      ),

    );

  }

}
```

- login_signup/verify_code.dart
```
import 'package:flutter/material.dart';

import 'package:http/http.dart' as http;

import 'dart:convert';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/login_signup/addFingerprint.dart';

  

class VerifyCodeScreen extends StatefulWidget {

  final String email;

  

  const VerifyCodeScreen({super.key, required this.email});

  

  @override

  _VerifyCodeScreenState createState() => _VerifyCodeScreenState();

}

  

class _VerifyCodeScreenState extends State<VerifyCodeScreen> {

  final TextEditingController _codeController = TextEditingController();

  final FlutterSecureStorage _secureStorage = FlutterSecureStorage();

  bool _isLoading = false;

  

  Future<void> verifyCode() async {

    final String code = _codeController.text.trim();

  

    if (code.isEmpty) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Please enter the verification code')),

      );

      return;

    }

  

    setState(() {

      _isLoading = true;

    });

  

    final url =

        Uri.parse('${Config.backendBaseUrl}/login_signup/verify_code.php');

    try {

      final response = await http.post(

        url,

        headers: {'Content-Type': 'application/x-www-form-urlencoded'},

        body: {'email': widget.email, 'code': code},

      );

  

      if (response.statusCode == 200) {

        final Map<String, dynamic> responseData = json.decode(response.body);

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text(responseData['message'])),

        );

  

        if (response.body.isNotEmpty && responseData['message'] == "Success") {

          // Save the token using FlutterSecureStorage

          String token = responseData['token'];

          await _secureStorage.write(key: 'jwt_token', value: token);

  

          // Navigate to the desired page

          Navigator.push(

            context,

            MaterialPageRoute(builder: (context) => AddFingerprintScreen()),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(content: Text('Something went wrong!')),

          );

        }

      } else {

        final error = json.decode(response.body)['error'];

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text(error ?? 'Verification failed')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('An error occurred: $e')),

      );

    } finally {

      setState(() {

        _isLoading = false;

      });

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: Text('Verify Code')),

      body: Padding(

        padding: const EdgeInsets.all(16.0),

        child: Column(

          children: [

            TextField(

              controller: _codeController,

              decoration: InputDecoration(labelText: 'Verification Code'),

            ),

            SizedBox(height: 20),

            _isLoading

                ? CircularProgressIndicator()

                : ElevatedButton(

                    onPressed: verifyCode,

                    child: Text('Verify'),

                  ),

          ],

        ),

      ),

    );

  }

}
```

- login_signup/verify_code_login.dart
```
import 'package:flutter/material.dart';

import 'package:http/http.dart' as http;

import 'dart:convert';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/login_signup/addFingerprint.dart';

//import 'chat/openChat.dart'; // Import OpenChat module

  

class VerifyCodeScreenLogin extends StatefulWidget {

  final String email;

  

  const VerifyCodeScreenLogin({super.key, required this.email});

  

  @override

  _VerifyCodeScreenState createState() => _VerifyCodeScreenState();

}

  

class _VerifyCodeScreenState extends State<VerifyCodeScreenLogin> {

  final TextEditingController _codeController = TextEditingController();

  final FlutterSecureStorage _secureStorage = FlutterSecureStorage();

  bool _isLoading = false;

  

  Future<void> verifyCode() async {

    final String code = _codeController.text.trim();

  

    if (code.isEmpty) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Please enter the verification code')),

      );

      return;

    }

  

    setState(() {

      _isLoading = true;

    });

  

    final url =

        Uri.parse('${Config.backendBaseUrl}/login_signup/verifycode_login.php');

  

    try {

      final response = await http.post(

        url,

        headers: {'Content-Type': 'application/x-www-form-urlencoded'},

        body: {'email': widget.email, 'code': code},

      );

  

      if (response.statusCode == 200) {

        final Map<String, dynamic> responseData = json.decode(response.body);

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text(responseData['message'])),

        );

  

        if (response.body.isNotEmpty && responseData['message'] == "Success") {

          // Save the token using FlutterSecureStorage

          String token = responseData['token'];

          print("jwt token:$token");

          await _secureStorage.write(key: 'jwt_token', value: token);

  

          // Navigate to OpenChat

          Navigator.push(

            context,

            MaterialPageRoute(builder: (context) => AddFingerprintScreen()),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(content: Text('Something went wrong!')),

          );

        }

      } else {

        final error = json.decode(response.body)['error'];

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text(error ?? 'Verification failed')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('An error occurred: $e')),

      );

    } finally {

      setState(() {

        _isLoading = false;

      });

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: Text('Verify Code')),

      body: Padding(

        padding: const EdgeInsets.all(16.0),

        child: Column(

          children: [

            TextField(

              controller: _codeController,

              decoration: InputDecoration(labelText: 'Verification Code'),

            ),

            SizedBox(height: 20),

            _isLoading

                ? CircularProgressIndicator()

                : ElevatedButton(

                    onPressed: verifyCode,

                    child: Text('Verify'),

                  ),

          ],

        ),

      ),

    );

  }

}
```
- mainScreen/create_profile.dart
```
import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:image_cropper/image_cropper.dart';

import 'dart:io'; // For File

import 'package:image_picker/image_picker.dart'; // For image picker

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/mainScreen/profilepage.dart'; // For making POST requests

import 'package:swipe_chat_play/mainScreen/edit_image.dart'; // For free aspect ratio cropping

import 'package:swipe_chat_play/mainScreen/edit_main_image.dart'; // For fixed 12:6 aspect ratio cropping

import 'package:path_provider/path_provider.dart';

import 'package:swipe_chat_play/tabs/profile/profile_tab.dart'; // For getTemporaryDirectory

import 'package:swipe_chat_play/colors.dart';

  

class CreateProfileScreen extends StatefulWidget {

  const CreateProfileScreen({super.key});

  

  @override

  _CreateProfileScreenState createState() => _CreateProfileScreenState();

}

  

class _CreateProfileScreenState extends State<CreateProfileScreen> {

  final FlutterSecureStorage _secureStorage = const FlutterSecureStorage();

  bool isInputFieldFocused = false;

  Map<String, dynamic> profileData = {};

  int selectedProfileImageIndex = 0; // Track selected image

  

  final TextEditingController _textController = TextEditingController();

  final FocusNode _focusNode = FocusNode();

  int? _editingIndex; // Track which text element is being edited

  String? _editingKey;

  

  final String startJson = '''

  {

    "profile_images": [

      "https://fahrschule-peppermint.com/wp-content/uploads/2025/01/Screenshot-2025-01-02-182049.png",

      "https://fahrschule-peppermint.com/wp-content/uploads/2025/01/Screenshot-2025-01-02-182049.png"

    ],

    "username": "| add username",

    "age": "how old are you?",

    "searching": "what are you Looking for?",

    "element": [

      {

        "type": "text",

        "content": "Click here to add some text about yourself. |"

      },

      {

        "type": "image",

        "content": "https://fahrschule-peppermint.com/wp-content/uploads/2025/01/Screenshot-2025-01-02-182049.png",

        "subtitle": "| add subtitle"

      }

    ],

    "last": "+ add element"

  }

  ''';

  

  @override

  void initState() {

    super.initState();

    _loadProfileData();

  }

  

  @override

  void dispose() {

    _textController.dispose();

    _focusNode.dispose();

    super.dispose();

  }

  

  Widget _buildProfileImageGallery() {

    final List<String> profileImages =

        List<String>.from(profileData["profile_images"] ?? []);

  

    return Column(

      children: [

        GestureDetector(

          onTap: () {

            // User taps the main image to change it

            _pickAndUploadImage(jsonKey: "profile_images");

          },

          child: Container(

            width: double.infinity,

            height: MediaQuery.of(context).size.width *

                1.4, // Main image aspect ratio

            color: Colors.grey.shade800,

            child: profileImages.isEmpty

                ? const Center(

                    child: Text("+ add image",

                        style: TextStyle(color: Colors.grey)))

                : Image.network(profileImages[selectedProfileImageIndex],

                    fit: BoxFit.cover),

          ),

        ),

        const SizedBox(height: 16),

        // Thumbnails row

        SizedBox(

          height: 100, // Height for the image thumbnails

          child: ListView.builder(

            scrollDirection: Axis.horizontal,

            itemCount: profileImages.length + 1, // +1 for "+ add image" button

            itemBuilder: (context, index) {

              if (index == profileImages.length) {

                // "+ add image" button

                return GestureDetector(

                  onTap: () {

                    _pickAndUploadImage(

                        jsonKey: "profile_images"); // Add new image

                  },

                  child: Container(

                    width: 100,

                    margin: const EdgeInsets.only(right: 8),

                    decoration: BoxDecoration(

                      color: Colors.grey.shade800,

                      borderRadius: BorderRadius.circular(8),

                    ),

                    child: const Center(

                      child: Icon(Icons.add, color: Colors.white),

                    ),

                  ),

                );

              }

  

              return GestureDetector(

                onTap: () {

                  // Set selected image index

                  setState(() {

                    selectedProfileImageIndex = index;

                  });

                },

                child: Container(

                  width: 100,

                  margin: const EdgeInsets.only(right: 8),

                  decoration: BoxDecoration(

                    border: Border.all(

                      color: selectedProfileImageIndex == index

                          ? Colors.blue

                          : Colors.transparent,

                      width: 2,

                    ),

                    borderRadius: BorderRadius.circular(8),

                  ),

                  child: ClipRRect(

                    borderRadius: BorderRadius.circular(8),

                    child: Image.network(

                      profileImages[index],

                      fit: BoxFit.cover,

                    ),

                  ),

                ),

              );

            },

          ),

        ),

      ],

    );

  }

  

  Future<void> _loadProfileData() async {

    try {

      // Check if profile_json exists in secure storage

      String? localJson = await _secureStorage.read(key: 'profile_json');

  

      if (localJson != null) {

        print("Loaded profile from local storage.");

        setState(() {

          profileData = jsonDecode(localJson);

        });

      } else {

        print("Local profile not found. Loading from server...");

        // Attempt to load profile from the server

        final serverData = await _fetchProfileFromServer();

  

        if (serverData != null) {

          print("Loaded profile from server.");

          await _secureStorage.write(

              key: 'profile_json',

              value: jsonEncode(serverData)); // Cache in local storage

          setState(() {

            profileData = serverData;

          });

        } else {

          print("Profile not found on server. Loading default startJson.");

          setState(() {

            profileData = jsonDecode(startJson); // Load default JSON

          });

        }

      }

    } catch (e) {

      print("Error reading profile data: $e");

      setState(() {

        profileData =

            jsonDecode(startJson); // Fallback to default JSON on error

      });

    }

  }

  

  Future<Map<String, dynamic>?> _fetchProfileFromServer() async {

    try {

      String? jwtToken = await _secureStorage.read(key: 'jwt_token');

      if (jwtToken == null) {

        print("JWT token not found. Cannot load profile from server.");

        return null;

      }

  

      final response = await http.get(

        Uri.parse('${Config.backendBaseUrl}/profile/retrieve_profile.php'),

        headers: {'Authorization': 'Bearer $jwtToken'},

      );

  

      if (response.statusCode == 200) {

        final responseJson = jsonDecode(response.body);

        if (responseJson['profile'] != null) {

          return jsonDecode(

              responseJson['profile']); // Parse and return profile data

        }

      }

    } catch (e) {

      print("Error fetching profile from server: $e");

    }

  

    return null; // Return null if server call fails or no data

  }

  

  Future<bool> _showExitConfirmationDialog() async {

    return await showDialog<bool>(

          context: context,

          builder: (context) => AlertDialog(

            title: const Text("Save changes?"),

            content: const Text("Do you want to save changes before leaving?"),

            actions: [

              // No: Navigate to MyProfileScreen without saving

              TextButton(

                onPressed: () {

                  Navigator.of(context).pop(false); // Close the dialog

                  Navigator.of(context).pushReplacement(

                    MaterialPageRoute(builder: (context) => MyProfileScreen()),

                  ); // Redirect to profile page

                },

                child: const Text("No"),

              ),

              // Yes: Save profile and then navigate

              TextButton(

                onPressed: () async {

                  await _saveProfileToServer(); // Save the profile

                },

                child: const Text("Yes"),

              ),

            ],

          ),

        ) ??

        false; // Default value if dismissed

  }

  

  /*

 Future<void> _loadProfileData() async {

    try {

      String? savedJson = await _secureStorage.read(key: 'profile_json');

      setState(() {

        profileData =

            savedJson != null ? jsonDecode(savedJson) : jsonDecode(startJson);

      });

    } catch (e) {

      print("Error reading secure storage: $e");

      setState(() {

        profileData = jsonDecode(startJson);

      });

    }

  }

  */

  

  Future<void> _saveProfileToServer() async {

    try {

      String? jwtToken = await _secureStorage.read(key: 'jwt_token');

      if (jwtToken == null) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("You need to log in first!")),

        );

        return;

      }

  

      final response = await http.post(

        Uri.parse('${Config.backendBaseUrl}/profile/save_profile.php'),

        headers: {

          'Content-Type': 'application/json',

          'Authorization': 'Bearer $jwtToken',

        },

        body: jsonEncode(profileData),

      );

  

      if (response.statusCode == 200) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Profile saved successfully!")),

        );

  

        // save profile data to secure storage

        await _saveProfileData();

  

        // Navigate to MyProfileScreen on success

        Navigator.of(context).pushReplacement(

          MaterialPageRoute(builder: (context) => MyProfileScreen()),

        );

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text("Failed to save profile: ${response.body}"),

          ),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(

          content: Text("Error saving profile: $e"),

        ),

      );

    }

  }

  

  Future<void> _saveProfileData() async {

    try {

      await _secureStorage.write(

          key: 'profile_json', value: jsonEncode(profileData));

    } catch (e) {

      print("Error writing to secure storage: $e");

    }

  }

  

  void _addElement() {

    setState(() {

      profileData["element"].add({

        "type": "text",

        "content": "Click here to add some text about yourself. |",

      });

    });

    _saveProfileData();

  }

  

  void _toggleElementType(int index) {

    setState(() {

      profileData["element"][index]["type"] =

          profileData["element"][index]["type"] == "text" ? "image" : "text";

      profileData["element"][index]["content"] =

          profileData["element"][index]["type"] == "image"

              ? "path.jpg"

              : "Click here to add some text about yourself. |";

    });

    _saveProfileData();

  }

  

  void _onTextFieldTapped(dynamic keyOrIndex) {

    setState(() {

      if (keyOrIndex is String) {

        _editingKey = keyOrIndex; // For non-list elements

        _editingIndex = null;

      } else if (keyOrIndex is Map) {

        // Handle subtitle editing

        _editingIndex = keyOrIndex["index"];

        _editingKey = keyOrIndex["field"];

      } else {

        _editingIndex = keyOrIndex; // For list elements (main content)

        _editingKey = null;

      }

      _textController.clear(); // Clear the input field

      _focusNode.requestFocus(); // Open the keyboard

    });

  }

  

  void _saveText() {

    if (_textController.text.isNotEmpty) {

      setState(() {

        if (_editingKey != null &&

            _editingKey == "subtitle" &&

            _editingIndex != null) {

          // Save subtitle to the specific element

          profileData["element"][_editingIndex!]["subtitle"] =

              _textController.text;

          debugPrint(

              "Updated subtitle at index $_editingIndex: ${profileData["element"][_editingIndex!]}");

        } else if (_editingKey != null) {

          // Save to "username", "age", or "searching"

          profileData[_editingKey!] = _textController.text;

          debugPrint("Updated $_editingKey: ${profileData[_editingKey!]}");

        } else if (_editingIndex != null) {

          // Save to element list (main content)

          profileData["element"][_editingIndex!]["content"] =

              _textController.text;

          debugPrint(

              "Updated element at index $_editingIndex: ${profileData["element"][_editingIndex!]}");

        }

        _editingKey = null;

        _editingIndex = null; // Close input after saving

      });

      _saveProfileData(); // Save updated text to secure storage

      _focusNode.unfocus(); // Close the keyboard

    } else {

      debugPrint("Text field is empty, ignoring save");

    }

  }

  

  Future<void> _pickAndUploadImage(

      {required String jsonKey, int? elementIndex}) async {

    final ImagePicker picker = ImagePicker();

    final XFile? pickedFile =

        await picker.pickImage(source: ImageSource.gallery);

  

    if (pickedFile != null) {

      final File imageFile = File(pickedFile.path);

  

      final croppedImageFile = await Navigator.push<File?>(

        context,

        MaterialPageRoute(

          builder: (context) => elementIndex == null

              ? EditMainImageScreen(imageFile: imageFile)

              : EditImageScreen(imageFile: imageFile),

        ),

      );

  

      if (croppedImageFile != null) {

        try {

          final request = http.MultipartRequest(

            'POST',

            Uri.parse('${Config.backendBaseUrl}/profile/upload_image.php'),

          );

          request.files.add(await http.MultipartFile.fromPath(

              'image', croppedImageFile.path));

  

          final response = await request.send();

          final responseBody = await response.stream.bytesToString();

  

          if (response.statusCode == 200) {

            final responseJson = jsonDecode(responseBody);

            final String imageUrl = responseJson['url'];

  

            setState(() {

              if (elementIndex != null && jsonKey == "element") {

                profileData["element"][elementIndex]["content"] =

                    imageUrl; // Update the specific element

              } else if (jsonKey == "profile_images") {

                profileData["profile_images"]

                    .add(imageUrl); // Add the image to the profile images list

              } else {

                profileData[jsonKey] = imageUrl; // General case for other keys

              }

            });

  

            _saveProfileData(); // Save updated profile data

          } else {

            debugPrint("Image upload failed: $responseBody");

          }

        } catch (e) {

          debugPrint("Error during image upload: $e");

        }

      }

    } else {

      debugPrint("No image selected.");

    }

  }

  

  Future<void> _cropExistingImage() async {

    try {

      final String? imageUrl = profileData["image"];

      if (imageUrl == null || !imageUrl.startsWith('http')) {

        throw Exception("No valid image URL to crop.");

      }

  

      // Download the image from the URL

      final tempDir = await getTemporaryDirectory(); // Get temporary directory

      final localImagePath = '${tempDir.path}/temp_cropped_image.jpg';

  

      final response = await http.get(Uri.parse(imageUrl));

      if (response.statusCode == 200) {

        // Save the image file locally

        final file = File(localImagePath);

        await file.writeAsBytes(response.bodyBytes);

  

        // Crop the local image file

        final croppedFile = await ImageCropper().cropImage(

          sourcePath: file.path,

          aspectRatio: const CropAspectRatio(ratioX: 9, ratioY: 16),

          uiSettings: [

            if (Platform.isAndroid)

              AndroidUiSettings(

                toolbarTitle: 'Crop Image',

                toolbarColor: Colors.black,

                toolbarWidgetColor: Colors.black,

                lockAspectRatio: true,

              ),

            if (Platform.isIOS)

              IOSUiSettings(

                title: 'Crop Image',

                aspectRatioLockEnabled: true,

              ),

          ],

        );

  

        if (croppedFile != null) {

          // Upload the cropped image and update the URL

          await _uploadCroppedImage(File(croppedFile.path));

        }

      } else {

        throw Exception("Failed to download image from URL.");

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error during cropping: $e")),

      );

    }

  }

  

  Future<void> _uploadCroppedImage(File imageFile) async {

    try {

      // Create a Multipart Request

      final request = http.MultipartRequest(

        'POST',

        Uri.parse('${Config.backendBaseUrl}/profile/upload_image.php'),

      );

  

      request.files

          .add(await http.MultipartFile.fromPath('image', imageFile.path));

  

      // Send the request

      final response = await request.send();

      final responseBody = await response.stream.bytesToString();

  

      if (response.statusCode == 200) {

        final responseJson = jsonDecode(responseBody);

        final String imageUrl = responseJson['url'];

  

        // Update the profileData with the new image URL

        setState(() {

          profileData["image"] = imageUrl; // Save the new image URL

        });

  

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Image updated successfully!")),

        );

      } else {

        throw Exception("Failed to upload cropped image: $responseBody");

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error uploading image: $e")),

      );

    }

  }

  

  Future<String?> _showEditDialog(String keyName, String initialText) async {

    TextEditingController controller = TextEditingController(text: initialText);

    return showDialog<String>(

      context: context,

      builder: (context) => AlertDialog(

        backgroundColor: Colors.grey.shade900,

        title:

            Text("Edit $keyName", style: const TextStyle(color: Colors.black)),

        content: TextField(

          controller: controller,

          style: const TextStyle(color: AppColors.text),

          decoration: const InputDecoration(

            hintText: "Enter your input",

            hintStyle: TextStyle(color: AppColors.text),

          ),

        ),

        actions: [

          TextButton(

            onPressed: () => Navigator.of(context).pop(),

            child: const Text("Cancel"),

          ),

          TextButton(

            onPressed: () => Navigator.of(context).pop(controller.text),

            child: const Text("Save"),

          ),

        ],

      ),

    );

  }

  

  void _closeKeyboard() {

    FocusScope.of(context).unfocus(); // Close the keyboard

    setState(() {

      isInputFieldFocused = false; // Track input focus state

      _editingKey = null; // Clear editing key to hide the input field

      _editingIndex = null; // Clear editing index to hide the input field

    });

  }

  

  @override

  Widget build(BuildContext context) {

    return WillPopScope(

      onWillPop: _showExitConfirmationDialog, // Intercept back button

      child: GestureDetector(

        onTap: _closeKeyboard, // Close keyboard on tap anywhere

        child: Scaffold(

          body: Stack(

            children: [

              // Gradient background

              Container(

                decoration: const BoxDecoration(

                  gradient: LinearGradient(

                    colors: [

                      AppColors.primaryGradientStart,

                      AppColors.primaryGradientEnd,

                    ], // Gradient colors

                    begin: Alignment.topCenter,

                    end: Alignment.bottomCenter,

                  ),

                ),

              ),

  

              // Top buttons (close and check/save)

              Positioned(

                top: 20,

                left: 16,

                child: CircleButton(

                  assetPath: 'assets/icons/close.svg',

                  onPressed: () async {

                    await _showExitConfirmationDialog();

                  },

                ),

              ),

              Positioned(

                top: 20,

                right: 16,

                child: CircleButton(

                  assetPath: 'assets/icons/check.svg',

                  onPressed: _saveProfileToServer, // Save profile function

                ),

              ),

  

              // White card with profile content

              Positioned.fill(

                top: 100, // Space for the gradient and top buttons

                bottom: 20,

                child: Padding(

                  padding: const EdgeInsets.symmetric(horizontal: 16.0),

                  child: Container(

                    decoration: BoxDecoration(

                      color: AppColors.cardBackground,

                      borderRadius: BorderRadius.circular(24.0),

                      boxShadow: [

                        BoxShadow(

                          color: AppColors.shadowColor.withOpacity(0.5),

                          spreadRadius: 1,

                          blurRadius: 8,

                          offset: const Offset(0, 4),

                        ),

                      ],

                    ),

                    child: Padding(

                      padding: const EdgeInsets.all(16.0),

                      child: SingleChildScrollView(

                        child: Column(

                          crossAxisAlignment: CrossAxisAlignment.start,

                          children: [

                            // Profile Image Section (Main Image)

                            GestureDetector(

                              onTap: () {

                                _pickAndUploadImage(jsonKey: "image");

                              },

                              child: Stack(

                                children: [

                                  Container(

                                    width: double.infinity,

                                    height:

                                        MediaQuery.of(context).size.width * 1.4,

                                    decoration: BoxDecoration(

                                      color: Colors.grey.shade800,

                                      borderRadius: BorderRadius.circular(12),

                                    ),

                                    child: profileData["image"] == null

                                        ? const Center(

                                            child: Text(

                                              "+ add image",

                                              style: TextStyle(

                                                color: AppColors

                                                    .placeholderTextColor,

                                              ),

                                            ),

                                          )

                                        : ClipRRect(

                                            borderRadius:

                                                BorderRadius.circular(12),

                                            child: Image.network(

                                              profileData["image"],

                                              fit: BoxFit.cover,

                                            ),

                                          ),

                                  ),

                                  if (profileData["image"] != null)

                                    Positioned(

                                      top: 8,

                                      right: 8,

                                      child: IconButton(

                                        icon: const Icon(

                                          Icons.crop,

                                          color: Colors.white,

                                        ),

                                        onPressed: _cropExistingImage,

                                      ),

                                    ),

                                ],

                              ),

                            ),

                            const SizedBox(height: 16),

  

                            // Username TextField

                            GestureDetector(

                              onTap: () {

                                _onTextFieldTapped("username");

                              },

                              child: Container(

                                width: double.infinity,

                                padding:

                                    const EdgeInsets.symmetric(vertical: 8.0),

                                child: Text(

                                  profileData["username"] ?? "| add username",

                                  style: const TextStyle(

                                      color: AppColors.text, fontSize: 18),

                                ),

                              ),

                            ),

                            const SizedBox(height: 16),

  

                            // Chip Elements (age, searching)

                            Wrap(

                              spacing: 8,

                              children: [

                                GestureDetector(

                                  onTap: () {

                                    _onTextFieldTapped("age");

                                  },

                                  child: Chip(

                                    backgroundColor: Colors.grey.shade200,

                                    label: Text(

                                      profileData["age"] ?? "how old are you?",

                                      style: const TextStyle(

                                          color: AppColors.text),

                                    ),

                                  ),

                                ),

                                GestureDetector(

                                  onTap: () {

                                    _onTextFieldTapped("searching");

                                  },

                                  child: Chip(

                                    backgroundColor: Colors.grey.shade200,

                                    label: Text(

                                      profileData["searching"] ??

                                          "what are you looking for?",

                                      style: const TextStyle(

                                          color: AppColors.text),

                                    ),

                                  ),

                                ),

                              ],

                            ),

                            const SizedBox(height: 16),

  

                            // Elements list (text or image cards)

                            ListView.builder(

                              shrinkWrap: true,

                              physics: const NeverScrollableScrollPhysics(),

                              itemCount: profileData["element"]?.length ?? 0,

                              itemBuilder: (context, index) {

                                final element = profileData["element"][index];

                                return Padding(

                                  padding: const EdgeInsets.only(bottom: 16.0),

                                  child: _buildElementCard(element, index),

                                );

                              },

                            ),

  

                            // Add new element button

                            Center(

                              child: TextButton(

                                onPressed: _addElement,

                                child: const Text(

                                  "+ add element",

                                  style: TextStyle(color: AppColors.text),

                                ),

                              ),

                            ),

                          ],

                        ),

                      ),

                    ),

                  ),

                ),

              ),

  

              // Semi-transparent overlay and input field

              if (_editingIndex != null || _editingKey != null)

                Column(

                  children: [

                    Expanded(

                      child: GestureDetector(

                        onTap: _closeKeyboard,

                        child: Container(

                          color: Colors.black.withOpacity(0.8),

                        ),

                      ),

                    ),

                    Container(

                      color:

                          AppColors.inputFieldBackground, // Overlay background

                      padding: const EdgeInsets.symmetric(

                          horizontal: 8.0, vertical: 8.0),

                      child: Row(

                        children: [

                          Expanded(

                            child: TextField(

                              controller: _textController,

                              focusNode: _focusNode,

                              style: const TextStyle(

                                  color:

                                      AppColors.inputFieldText), // Input color

                              decoration: const InputDecoration(

                                hintText: "Type your text...",

                                hintStyle: TextStyle(

                                    color: AppColors

                                        .placeholderTextColor), // Placeholder color

                                border: InputBorder.none,

                              ),

                            ),

                          ),

                          IconButton(

                            icon: const Icon(

                              Icons.send,

                              color: AppColors.sendButtonColor, // Send button

                            ),

                            onPressed: _saveText,

                          ),

                        ],

                      ),

                    ),

                  ],

                ),

            ],

          ),

        ),

      ),

    );

  }

  

  Widget _buildEditableChip({required String label, required String keyName}) {

    return GestureDetector(

      onTap: () async {

        String? newValue = await _showEditDialog(keyName, label);

        if (newValue != null) {

          setState(() {

            profileData[keyName] = newValue;

          });

          _saveProfileData();

        }

      },

      child: Chip(

        backgroundColor: Colors.grey.shade800,

        label: Text(label, style: const TextStyle(color: AppColors.text)),

      ),

    );

  }

  

  Widget _buildElementCard(Map<String, dynamic> element, int index) {

    bool isTextElement = element["type"] == "text";

  

    return Padding(

      padding: const EdgeInsets.symmetric(vertical: 8.0),

      child: Column(

        crossAxisAlignment: CrossAxisAlignment.start,

        children: [

          Row(

            mainAxisAlignment: MainAxisAlignment.spaceBetween,

            children: [

              Row(

                children: [

                  GestureDetector(

                    onTap: () {

                      if (!isTextElement) _toggleElementType(index);

                    },

                    child: Text(

                      "text",

                      style: TextStyle(

                        color: isTextElement

                            ? AppColors.text_light

                            : AppColors.text,

                        fontSize: 16,

                        decoration: isTextElement

                            ? TextDecoration.underline

                            : TextDecoration.none,

                      ),

                    ),

                  ),

                  const SizedBox(width: 16),

                  GestureDetector(

                    onTap: () {

                      if (isTextElement) _toggleElementType(index);

                    },

                    child: Text(

                      "image",

                      style: TextStyle(

                        color: !isTextElement

                            ? AppColors.text_light

                            : AppColors.text,

                        fontSize: 16,

                        decoration: !isTextElement

                            ? TextDecoration.underline

                            : TextDecoration.none,

                      ),

                    ),

                  ),

                ],

              ),

              IconButton(

                icon: const Icon(Icons.close, color: AppColors.closeIconColor),

                onPressed: () {

                  setState(() {

                    profileData["element"].removeAt(index);

                  });

                  _saveProfileData();

                },

              ),

            ],

          ),

          if (isTextElement)

            GestureDetector(

              onTap: () {

                _onTextFieldTapped(index); // Open bottom input for text

              },

              child: Container(

                width: double.infinity, // Full width

                padding:

                    const EdgeInsets.symmetric(vertical: 2.0), // Add padding

                child: Text(

                  element["content"],

                  style: const TextStyle(color: AppColors.text),

                  textAlign: TextAlign.left,

                ),

              ),

            )

          else

            Column(

              crossAxisAlignment: CrossAxisAlignment.start,

              children: [

                GestureDetector(

                  onTap: () {

                    _pickAndUploadImage(

                        jsonKey: "element", elementIndex: index);

                  },

                  child: element["content"] != null &&

                          element["content"] != "path.jpg"

                      ? ClipRRect(

                          borderRadius: BorderRadius.circular(8),

                          child: Image.network(

                            element["content"],

                            fit: BoxFit.cover,

                            loadingBuilder: (context, child, loadingProgress) {

                              if (loadingProgress == null) return child;

                              return const Center(

                                  child: CircularProgressIndicator(

                                      color: AppColors.text));

                            },

                            errorBuilder: (context, error, stackTrace) {

                              return Container(

                                color: Colors.grey.shade800,

                                child: const Center(

                                  child: Text(

                                    "Failed to load image",

                                    style: TextStyle(color: Colors.red),

                                  ),

                                ),

                              );

                            },

                          ),

                        )

                      : Container(

                          width: double.infinity,

                          height: 200,

                          decoration: BoxDecoration(

                            color: const Color(0xFF2E2E2E),

                            borderRadius: BorderRadius.circular(8),

                          ),

                          child: const Center(

                            child: Text(

                              "+ add image",

                              style: TextStyle(

                                color: Color(0xFF877171),

                                fontSize: 18,

                              ),

                            ),

                          ),

                        ),

                ),

                const SizedBox(height: 8),

                GestureDetector(

                  onTap: () {

                    // Pass index and flag to indicate subtitle editing

                    _onTextFieldTapped({"index": index, "field": "subtitle"});

                  },

                  child: Container(

                    width: double.infinity, // Full width for subtitle tap area

                    padding: const EdgeInsets.symmetric(

                        vertical: 4.0), // Padding for easier tapping

                    child: Text(

                      element["subtitle"] ?? "| add subtitle",

                      style: const TextStyle(

                        color: AppColors.text,

                        fontSize: 16,

                      ),

                      textAlign: TextAlign.left,

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
```
- mainScreen/edit_image.dart
```
import 'dart:io';

import 'package:flutter/material.dart';

import 'package:image_cropper/image_cropper.dart';

  

class EditImageScreen extends StatefulWidget {

  final File imageFile;

  

  const EditImageScreen({required this.imageFile, super.key});

  

  @override

  _EditImageScreenState createState() => _EditImageScreenState();

}

  

class _EditImageScreenState extends State<EditImageScreen> {

  File? _croppedImage;

  

  @override

  void initState() {

    super.initState();

    _cropImage(); // Automatically start cropping when the screen opens

  }

  

  Future<void> _cropImage() async {

    final croppedFile = await ImageCropper().cropImage(

      sourcePath: widget.imageFile.path,

      //cropStyle: CropStyle.rectangle, // Use rectangle or circle for cropping

      aspectRatio:

          const CropAspectRatio(ratioX: 1, ratioY: 1), // Square crop (1:1)

      uiSettings: [

        if (Platform.isAndroid)

          AndroidUiSettings(

            toolbarTitle: 'Edit Photo',

            toolbarColor: Colors.black,

            toolbarWidgetColor: Colors.white,

            initAspectRatio: CropAspectRatioPreset.original,

            lockAspectRatio: false, // Allow free resizing

          ),

        if (Platform.isIOS)

          IOSUiSettings(

            title: 'Edit Photo',

            minimumAspectRatio: 0.2, // Prevent extremely narrow crops

          ),

      ],

    );

  

    if (croppedFile != null) {

      setState(() {

        _croppedImage = File(croppedFile.path);

      });

      Navigator.pop(context, _croppedImage); // Return cropped image

    } else {

      Navigator.pop(context, null); // Close screen if cropping was canceled

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: Colors.black,

      body: Center(

        child: _croppedImage == null

            ? const CircularProgressIndicator() // Show loader while cropping

            : Image.file(_croppedImage!),

      ),

    );

  }

}
```
- mainScreen/edit_main_image.dart
```
import 'dart:io';

import 'package:flutter/material.dart';

import 'package:image_cropper/image_cropper.dart';

  

class EditMainImageScreen extends StatefulWidget {

  final File imageFile;

  

  const EditMainImageScreen({super.key, required this.imageFile});

  

  @override

  _EditMainImageScreenState createState() => _EditMainImageScreenState();

}

  

class _EditMainImageScreenState extends State<EditMainImageScreen> {

  File? _croppedImage;

  

  @override

  void initState() {

    super.initState();

    _cropImage(); // Automatically start cropping when the screen opens

  }

  

  Future<void> _cropImage() async {

    final croppedFile = await ImageCropper().cropImage(

      sourcePath: widget.imageFile.path,

      aspectRatio:

          const CropAspectRatio(ratioX: 6, ratioY: 12), // Vertical 6:12 ratio

      uiSettings: [

        if (Platform.isAndroid)

          AndroidUiSettings(

            toolbarTitle: 'Edit Main Image',

            toolbarColor: Colors.black,

            toolbarWidgetColor: Colors.white,

            lockAspectRatio: true, // Lock aspect ratio while allowing resizing

            showCropGrid: true, // Show guiding grid

          ),

        if (Platform.isIOS)

          IOSUiSettings(

            title: 'Edit Main Image',

            aspectRatioLockEnabled: true, // Lock aspect ratio while resizing

          ),

      ],

    );

  

    if (croppedFile != null) {

      setState(() {

        _croppedImage = File(croppedFile.path);

      });

      Navigator.pop(context, _croppedImage); // Return cropped image

    } else {

      Navigator.pop(context, null); // Close screen if cropping was canceled

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: Colors.black,

      body: Center(

        child: _croppedImage == null

            ? const CircularProgressIndicator() // Show loader while cropping

            : Image.file(_croppedImage!),

      ),

    );

  }

}
```
- mainScreen/profilepage.dart
```
import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/mainScreen/create_profile.dart';

import 'package:swipe_chat_play/main_page.dart'; // Import MainPage

  

class MyProfileScreen extends StatefulWidget {

  const MyProfileScreen({super.key});

  

  @override

  _MyProfileScreenState createState() => _MyProfileScreenState();

}

  

class _MyProfileScreenState extends State<MyProfileScreen> {

  final FlutterSecureStorage _secureStorage = const FlutterSecureStorage();

  Map<String, dynamic> profileData = {};

  

  @override

  void initState() {

    super.initState();

    _loadProfileFromServer();

  }

  

  @override

  void didChangeDependencies() {

    super.didChangeDependencies();

    _loadProfileFromServer(); // Ensure refresh on navigation

  }

  

  Future<void> _loadProfileFromServer() async {

    final String apiUrl =

        '${Config.backendBaseUrl}/profile/retrieve_profile.php';

  

    try {

      String? jwtToken = await _secureStorage.read(key: 'jwt_token');

      if (jwtToken == null) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("You need to log in first!")),

        );

        return;

      }

  

      final response = await http.get(

        Uri.parse(apiUrl),

        headers: {'Authorization': 'Bearer $jwtToken'},

      );

  

      if (response.statusCode == 200) {

        final responseJson = jsonDecode(response.body);

        setState(() {

          profileData = jsonDecode(responseJson['profile']);

        });

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

              content: Text("Failed to retrieve profile: ${response.body}")),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error retrieving profile: $e")),

      );

    }

  }

  

  Future<void> _navigateToEditProfile() async {

    await Navigator.of(context).push(

      MaterialPageRoute(builder: (context) => CreateProfileScreen()),

    );

    _loadProfileFromServer(); // Reload profile after returning from edit screen

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: Colors.black,

      appBar: AppBar(

        backgroundColor: Colors.black,

        elevation: 0,

        leading: IconButton(

          icon:

              const Icon(Icons.arrow_back, color: Colors.white), // Back button

          onPressed: () {

            Navigator.pushReplacement(

              context,

              MaterialPageRoute(builder: (context) => MainPage()),

            ); // Navigate to MainPage

          },

        ),

        actions: [

          IconButton(

            icon: const Icon(Icons.edit, color: Colors.white), // Edit button

            onPressed: _navigateToEditProfile, // Navigate to edit screen

          ),

        ],

        title: const Text('My Profile', style: TextStyle(color: Colors.white)),

      ),

      body: RefreshIndicator(

        onRefresh: _loadProfileFromServer, // Reload when pulled down

        child: ListView(

          physics: const AlwaysScrollableScrollPhysics(), // Enable overscroll

          padding: EdgeInsets.all(0),

          children: [

            profileData.isEmpty

                ? const Center(child: CircularProgressIndicator())

                : Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: [

                      const SizedBox(height: 16),

                      // Profile Image Section (read-only)

                      Container(

                        width: double.infinity,

                        height: MediaQuery.of(context).size.width * 1.4,

                        color: Colors.grey.shade800,

                        child: profileData["image"] == null

                            ? const Center(

                                child: Text("+ add image",

                                    style: TextStyle(color: Colors.grey)),

                              )

                            : ClipRRect(

                                borderRadius: BorderRadius.circular(8),

                                child: Image.network(

                                  profileData["image"],

                                  fit: BoxFit.cover,

                                  loadingBuilder:

                                      (context, child, loadingProgress) {

                                    if (loadingProgress == null) return child;

                                    return const Center(

                                        child: CircularProgressIndicator(

                                            color: Colors.white));

                                  },

                                  errorBuilder: (context, error, stackTrace) {

                                    return Container(

                                      color: Colors.grey.shade800,

                                      child: const Center(

                                        child: Text(

                                          "Failed to load image",

                                          style: TextStyle(color: Colors.red),

                                        ),

                                      ),

                                    );

                                  },

                                ),

                              ),

                      ),

  

                      const SizedBox(height: 16),

                      // Username (read-only)

                      Container(

                        width: double.infinity,

                        padding: const EdgeInsets.symmetric(vertical: 8.0),

                        child: Text(

                          profileData["username"] ?? "| add username",

                          style: const TextStyle(

                              color: Colors.white, fontSize: 18),

                        ),

                      ),

                      const SizedBox(height: 16),

                      // Chip Elements (age, searching)

                      Wrap(

                        spacing: 8,

                        children: [

                          Chip(

                            backgroundColor: Colors.grey.shade800,

                            label: Text(

                              profileData["age"] ?? "how old are you?",

                              style: const TextStyle(color: Colors.white),

                            ),

                          ),

                          Chip(

                            backgroundColor: Colors.grey.shade800,

                            label: Text(

                              profileData["searching"] ??

                                  "what are you looking for?",

                              style: const TextStyle(color: Colors.white),

                            ),

                          ),

                        ],

                      ),

                      const SizedBox(height: 16),

                      // Elements list (read-only text and images)

                      ListView.builder(

                        shrinkWrap: true,

                        physics: const NeverScrollableScrollPhysics(),

                        itemCount: profileData["element"]?.length ?? 0,

                        itemBuilder: (context, index) {

                          final element = profileData["element"][index];

                          return Padding(

                            padding: const EdgeInsets.only(bottom: 16.0),

                            child: _buildElementCard(element),

                          );

                        },

                      ),

                    ],

                  ),

          ],

        ),

      ),

    );

  }

  

  Widget _buildElementCard(Map<String, dynamic> element) {

    bool isTextElement = element["type"] == "text";

  

    return Padding(

      padding: const EdgeInsets.symmetric(vertical: 8.0),

      child: Column(

        crossAxisAlignment: CrossAxisAlignment.start,

        children: [

          // Removed "text/image" toggle switch here

          const SizedBox(height: 8),

          if (isTextElement)

            Container(

              width: double.infinity,

              padding: const EdgeInsets.symmetric(vertical: 2.0),

              child: Text(

                element["content"],

                style: const TextStyle(color: Colors.white),

                textAlign: TextAlign.left,

              ),

            )

          else

            Column(

              crossAxisAlignment: CrossAxisAlignment.start,

              children: [

                element["content"] != null && element["content"] != "path.jpg"

                    ? ClipRRect(

                        borderRadius: BorderRadius.circular(8),

                        child: Image.network(

                          element["content"],

                          fit: BoxFit.cover,

                          loadingBuilder: (context, child, loadingProgress) {

                            if (loadingProgress == null) return child;

                            return const Center(

                                child: CircularProgressIndicator(

                                    color: Colors.white));

                          },

                          errorBuilder: (context, error, stackTrace) {

                            return Container(

                              color: Colors.grey.shade800,

                              child: const Center(

                                child: Text(

                                  "Failed to load image",

                                  style: TextStyle(color: Colors.red),

                                ),

                              ),

                            );

                          },

                        ),

                      )

                    : Container(

                        width: double.infinity,

                        height: 200,

                        decoration: BoxDecoration(

                          color: const Color(0xFF2E2E2E),

                          borderRadius: BorderRadius.circular(8),

                        ),

                        child: const Center(

                          child: Text(

                            "+ add image",

                            style: TextStyle(

                              color: Color(0xFF877171),

                              fontSize: 18,

                            ),

                          ),

                        ),

                      ),

                const SizedBox(height: 8),

                Container(

                  width: double.infinity,

                  padding: const EdgeInsets.symmetric(vertical: 4.0),

                  child: Text(

                    element["subtitle"] ?? "| add subtitle",

                    style: const TextStyle(

                      color: Colors.grey,

                      fontSize: 16,

                    ),

                    textAlign: TextAlign.left,

                  ),

                ),

              ],

            ),

        ],

      ),

    );

  }

}
```
- profile_creation/aboutYouLayout.dart
```
import 'dart:convert'; // for jsonEncode

import 'package:flutter/material.dart';

import 'package:flutter_svg/svg.dart';

import 'package:http/http.dart' as http; // for the POST request

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/profile_creation/profilbilder.dart';

  

class AboutYouLayout extends StatefulWidget {

  const AboutYouLayout({super.key});

  

  @override

  _AboutYouLayoutState createState() => _AboutYouLayoutState();

}

  

class _AboutYouLayoutState extends State<AboutYouLayout> {

  String? name;

  String? age;

  String? gender;

  String? spokenLanguage;

  String? religion;

  String? career;

  String? politics;

  

  final FlutterSecureStorage _secureStorage = FlutterSecureStorage();

  String? _jwtToken;

  

  @override

  void initState() {

    super.initState();

    _fetchToken();

  }

  

  Future<void> _fetchToken() async {

    final token = await _secureStorage.read(key: 'jwt_token');

    setState(() {

      _jwtToken = token;

    });

  

    if (_jwtToken == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

          content: Text('Authentication token not found. Please login again.'),

        ),

      );

    }

  }

  

  /// When the arrow button is clicked, send the answers to the backend

  Future<void> _submitProfile() async {

    if (_jwtToken == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

          content: Text('Authentication token not found. Please login again.'),

        ),

      );

      return;

    }

  

    // 1) Construct the JSON

    final Map<String, dynamic> aboutYouPayload = {

      'name': name,

      'age': age,

      'gender': gender,

      'spokenLanguage': spokenLanguage,

      'religion': religion,

      'career': career,

      'politics': politics,

    };

  

    // Remove null or empty values

    aboutYouPayload

        .removeWhere((key, value) => value == null || value.toString().isEmpty);

  

    // The final JSON to send

    final Map<String, dynamic> requestBody = {

      'about_you': aboutYouPayload,

    };

  

    // Debug: print the JSON

    print('Sending JSON: ${jsonEncode(requestBody)}');

  

    try {

      // 2) Send POST request

      final response = await http.post(

        Uri.parse('${Config.backendBaseUrl}/create_profile/add_answer.php'),

        headers: {

          'Content-Type': 'application/json',

          'Authorization': 'Bearer $_jwtToken',

        },

        body: jsonEncode(requestBody),

      );

  

      // 3) Check the response

      if (response.statusCode == 200) {

        final responseData = jsonDecode(response.body);

        if (responseData['success'] ?? false) {

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(content: Text('Profile updated successfully!')),

          );

          // Navigate to the main page

          Navigator.pushReplacement(

            context,

            MaterialPageRoute(builder: (context) => ProfilbilderLayout()),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(content: Text('Error: ${responseData['error']}')),

          );

        }

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

              content: Text(

                  'Failed to update profile. Status: ${response.statusCode}')),

        );

        print('Response body: ${response.body}');

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Network error: $e')),

      );

      print('Exception while updating profile: $e');

    }

  }

  

  // A helper function for showing a text input bottom sheet.

  Future<void> _showTextInput({

    required String title,

    String initialValue = '',

    TextInputType keyboardType = TextInputType.text,

    required Function(String) onSubmitted,

  }) async {

    final TextEditingController controller =

        TextEditingController(text: initialValue);

    await showModalBottomSheet(

      context: context,

      isScrollControlled: true, // allows the bottom sheet to resize

      builder: (BuildContext context) {

        return Padding(

          // so the bottom sheet sits above the keyboard

          padding: EdgeInsets.only(

            bottom: MediaQuery.of(context).viewInsets.bottom,

            left: 16,

            right: 16,

            top: 16,

          ),

          child: Column(

            mainAxisSize: MainAxisSize.min,

            children: [

              Text(title, style: const TextStyle(fontSize: 18)),

              const SizedBox(height: 10),

              TextField(

                controller: controller,

                autofocus: true,

                keyboardType: keyboardType,

                decoration: const InputDecoration(

                  border: OutlineInputBorder(),

                ),

              ),

              const SizedBox(height: 10),

              ElevatedButton(

                child: const Text('Done'),

                onPressed: () {

                  onSubmitted(controller.text);

                  Navigator.pop(context);

                },

              ),

              const SizedBox(height: 10),

            ],

          ),

        );

      },

    );

  }

  

  // A helper function to show a list of options in a bottom sheet.

  Future<void> _showOptions({

    required List<String> options,

    required Function(String) onSelected,

  }) async {

    await showModalBottomSheet(

      context: context,

      builder: (BuildContext context) {

        return Column(

          mainAxisSize: MainAxisSize.min,

          children: [

            ...options.map(

              (option) => ListTile(

                title: Text(option),

                onTap: () {

                  onSelected(option);

                  Navigator.pop(context);

                },

              ),

            ),

            const SizedBox(height: 10),

          ],

        );

      },

    );

  }

  

  // region "edit item" methods

  void _editName() {

    _showTextInput(

      title: 'Enter your name',

      initialValue: name ?? '',

      onSubmitted: (value) => setState(() => name = value),

    );

  }

  

  void _editAge() {

    _showTextInput(

      title: 'Enter your age',

      initialValue: age ?? '',

      keyboardType: TextInputType.number,

      onSubmitted: (value) => setState(() => age = value),

    );

  }

  

  void _selectGender() {

    const List<String> genderOptions = [

      'Male',

      'Female',

      'Non-binary',

      'Transgender',

      'Genderqueer',

      'Other'

    ];

    _showOptions(

      options: genderOptions,

      onSelected: (selected) => setState(() => gender = selected),

    );

  }

  

  Future<void> _selectSpokenLanguage() async {

    final List<String> allLanguages = [

      'Afrikaans',

      'Albanian',

      'Amharic',

      'Arabic',

      'Armenian',

      'Azerbaijani',

      'Basque',

      'Belarusian',

      'Bengali',

      'Bosnian',

      'Bulgarian',

      'Burmese',

      'Catalan',

      'Cebuano',

      'Chichewa',

      'Chinese (Simplified)',

      'Chinese (Traditional)',

      'Corsican',

      'Croatian',

      'Czech',

      'Danish',

      'Dutch',

      'English',

      'Esperanto',

      'Estonian',

      'Filipino',

      'Finnish',

      'French',

      'Frisian',

      'Galician',

      'Georgian',

      'German',

      'Greek',

      'Gujarati',

      'Haitian Creole',

      'Hausa',

      'Hawaiian',

      'Hebrew',

      'Hindi',

      'Hmong',

      'Hungarian',

      'Icelandic',

      'Igbo',

      'Indonesian',

      'Irish',

      'Italian',

      'Japanese',

      'Javanese',

      'Kannada',

      'Kazakh',

      'Khmer',

      'Kinyarwanda',

      'Korean',

      'Kurdish (Kurmanji)',

      'Kyrgyz',

      'Lao',

      'Latin',

      'Latvian',

      'Lithuanian',

      'Luxembourgish',

      'Macedonian',

      'Malagasy',

      'Malay',

      'Malayalam',

      'Maltese',

      'Maori',

      'Marathi',

      'Mongolian',

      'Nepali',

      'Norwegian',

      'Nyanja',

      'Odia (Oriya)',

      'Pashto',

      'Persian',

      'Polish',

      'Portuguese',

      'Punjabi',

      'Romanian',

      'Russian',

      'Samoan',

      'Scots Gaelic',

      'Serbian',

      'Sesotho',

      'Shona',

      'Sindhi',

      'Sinhala',

      'Slovak',

      'Slovenian',

      'Somali',

      'Spanish',

      'Sundanese',

      'Swahili',

      'Swedish',

      'Tagalog (Filipino)',

      'Tajik',

      'Tamil',

      'Tatar',

      'Telugu',

      'Thai',

      'Turkish',

      'Turkmen',

      'Ukrainian',

      'Urdu',

      'Uyghur',

      'Uzbek',

      'Vietnamese',

      'Welsh',

      'Xhosa',

      'Yiddish',

      'Yoruba',

      'Zulu'

    ];

  

    String searchQuery = '';

    final Set<String> selectedLanguages = {};

  

    final result = await showModalBottomSheet<Set<String>>(

      context: context,

      isScrollControlled: true,

      builder: (BuildContext context) {

        return StatefulBuilder(

          builder: (BuildContext context, StateSetter setModalState) {

            final List<String> filteredLanguages = allLanguages

                .where((lang) =>

                    lang.toLowerCase().contains(searchQuery.toLowerCase()))

                .toList();

  

            return Padding(

              padding: EdgeInsets.only(

                bottom: MediaQuery.of(context).viewInsets.bottom,

                top: 16,

                left: 16,

                right: 16,

              ),

              child: SizedBox(

                height: 400,

                child: Column(

                  children: [

                    Row(

                      children: [

                        Expanded(

                          child: TextField(

                            decoration: const InputDecoration(

                              hintText: 'Search language',

                              prefixIcon: Icon(Icons.search),

                            ),

                            onChanged: (value) {

                              setModalState(() {

                                searchQuery = value;

                              });

                            },

                          ),

                        ),

                        IconButton(

                          icon: const Icon(Icons.add),

                          onPressed: () {

                            Navigator.pop(context, selectedLanguages);

                          },

                        )

                      ],

                    ),

                    const SizedBox(height: 10),

                    Expanded(

                      child: ListView.builder(

                        itemCount: filteredLanguages.length,

                        itemBuilder: (BuildContext context, int index) {

                          final language = filteredLanguages[index];

                          final isSelected =

                              selectedLanguages.contains(language);

                          return ListTile(

                            title: Text(language),

                            trailing: isSelected

                                ? const Icon(Icons.check, color: Colors.green)

                                : null,

                            onTap: () {

                              setModalState(() {

                                if (isSelected) {

                                  selectedLanguages.remove(language);

                                } else {

                                  selectedLanguages.add(language);

                                }

                              });

                            },

                          );

                        },

                      ),

                    ),

                  ],

                ),

              ),

            );

          },

        );

      },

    );

  

    if (result != null && result.isNotEmpty) {

      setState(() {

        spokenLanguage = result.join(', ');

      });

    }

  }

  

  void _selectReligion() {

    const List<String> religionOptions = [

      'Christianity',

      'Islam',

      'Hinduism',

      'Buddhism',

      'Judaism',

      'Other'

    ];

    _showOptions(

      options: religionOptions,

      onSelected: (selected) => setState(() => religion = selected),

    );

  }

  

  void _editCareer() {

    _showTextInput(

      title: 'Enter your career',

      initialValue: career ?? '',

      onSubmitted: (value) => setState(() => career = value),

    );

  }

  

  void _selectPolitics() {

    const List<String> politicsOptions = [

      'Liberal',

      'Conservative',

      'Centrist',

      'Libertarian',

      'Socialist',

      'Other'

    ];

    _showOptions(

      options: politicsOptions,

      onSelected: (selected) => setState(() => politics = selected),

    );

  }

  // endregion

  

  Widget _buildItem(

    String title,

    String subtitle, {

    bool showDivider = true,

    required VoidCallback onTap,

  }) {

    return Material(

      color: Colors.transparent,

      child: InkWell(

        onTap: onTap,

        child: Container(

          width: double.infinity,

          constraints: const BoxConstraints(minHeight: 60),

          child: Column(

            mainAxisSize: MainAxisSize.min,

            children: [

              Container(

                width: double.infinity,

                padding:

                    const EdgeInsets.symmetric(horizontal: 16, vertical: 16),

                child: Column(

                  crossAxisAlignment: CrossAxisAlignment.start,

                  children: [

                    Text(

                      title,

                      style: const TextStyle(

                        fontSize: 16,

                        fontWeight: FontWeight.w500,

                        color: Colors.black87,

                      ),

                    ),

                    const SizedBox(height: 4),

                    Text(

                      subtitle,

                      style: const TextStyle(

                        fontSize: 14,

                        color: Colors.grey,

                      ),

                    ),

                  ],

                ),

              ),

              if (showDivider)

                const Divider(

                  color: Colors.grey,

                  thickness: 0.5,

                  height: 1,

                ),

            ],

          ),

        ),

      ),

    );

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      body: SafeArea(

        child: Stack(

          children: [

            // Main column

            Column(

              crossAxisAlignment: CrossAxisAlignment.start,

              children: [

                // Header

                Padding(

                  padding: const EdgeInsets.all(16.0),

                  child: Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: const [

                      Text(

                        'About you',

                        style: TextStyle(

                          fontSize: 24,

                          fontWeight: FontWeight.bold,

                        ),

                      ),

                      SizedBox(height: 4),

                      Text(

                        'This info will appear in your Profile (you can change it later)',

                        style: TextStyle(

                          fontSize: 14,

                          color: Colors.grey,

                        ),

                      ),

                      SizedBox(height: 24),

                    ],

                  ),

                ),

                // Expanded scrollable list

                Expanded(

                  child: ListView(

                    padding: const EdgeInsets.symmetric(horizontal: 16.0),

                    children: [

                      _buildItem('name', name ?? 'add your username',

                          onTap: _editName),

                      _buildItem('age', age ?? 'set your age', onTap: _editAge),

                      _buildItem('gender', gender ?? 'choose your gender',

                          onTap: _selectGender),

                      _buildItem('spoken languages',

                          spokenLanguage ?? 'what languages do you speak?',

                          onTap: _selectSpokenLanguage),

                      _buildItem(

                          'religion',

                          religion ??

                              'choose wisely or you might go to paradise',

                          onTap: _selectReligion),

                      _buildItem('career',

                          career ?? 'school, uni, work, whatever or nothing',

                          onTap: _editCareer),

                      _buildItem(

                        'politics',

                        politics ?? 'what do you stand for politically?',

                        showDivider: false,

                        onTap: _selectPolitics,

                      ),

                      const SizedBox(height: 24),

                    ],

                  ),

                ),

              ],

            ),

            // Arrow button pinned

            Positioned(

              bottom: 20,

              right: 30,

              child: IconButton(

                icon: SvgPicture.asset(

                  'assets/icons/arrow_icon.svg',

                  width: 50,

                  height: 50,

                ),

                onPressed: _submitProfile, // <--- Calls your new method

              ),

            ),

          ],

        ),

      ),

    );

  }

}
```
- profile_creation/choose_question.dart
```
import 'package:flutter/material.dart';

  

class ChoseQuestionPage extends StatefulWidget {

  final String? initialQuestion;

  final String? initialAnswer;

  

  const ChoseQuestionPage({

    super.key,

    this.initialQuestion,

    this.initialAnswer,

  });

  

  @override

  _ChoseQuestionPageState createState() => _ChoseQuestionPageState();

}

  

class _ChoseQuestionPageState extends State<ChoseQuestionPage> {

  final List<String> questions = [

    "Wenn ich ein Tier wäre, dann wäre ich ein …",

    "Was ich erst vor kurzem herausgefunden habe...",

    "Ein Ort, an den ich immer wieder zurückkehre...",

    "Mein liebster Feiertag und warum...",

    "Das verrückteste Abenteuer, das ich je erlebt habe...",

    "Ein Buch, das mein Leben verändert hat...",

    "Wenn ich eine Superkraft haben könnte, wäre es...",

    "Mein geheimes Talent...",

    "Meine liebste Art, einen Regentag zu verbringen...",

    "Das beste Essen, das ich je gekocht habe...",

    "Ein Lied, das mich immer glücklich macht...",

    "Meine ideale Art, das Wochenende zu verbringen...",

    "Etwas, das ich gerne lernen möchte...",

    "Ein Film, der mich zum Lachen oder Weinen bringt...",

    "Das Schönste, das mir jemals passiert ist...",

    "Meine größte Herausforderung und wie ich sie gemeistert habe...",

    "Ein Zitat, nach dem ich lebe...",

    "Wenn ich einen Tag lang unsichtbar wäre, würde ich...",

    "Meine früheste Kindheitserinnerung...",

    "Ein Ort, den ich unbedingt besuchen möchte...",

    "Drei Dinge, die ich immer in meiner Tasche habe..."

  ];

  

  String? selectedQuestion;

  final TextEditingController _answerController = TextEditingController();

  bool _isQuestionSelected = false;

  final FocusNode _answerFocusNode = FocusNode();

  

  @override

  void initState() {

    super.initState();

    selectedQuestion = widget.initialQuestion;

    _answerController.text = widget.initialAnswer ?? '';

    if (widget.initialAnswer?.isNotEmpty ?? false) {

      _isQuestionSelected = true;

    }

  }

  

  @override

  void dispose() {

    _answerFocusNode.dispose();

    super.dispose();

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(

        title: const Text('Fragen & Antworten'),

        backgroundColor: Colors.white,

        elevation: 0,

        iconTheme: const IconThemeData(color: Colors.black),

        actions: _isQuestionSelected

            ? [

                IconButton(

                  icon: const Icon(Icons.check, color: Colors.black),

                  onPressed: () {

                    if (_answerController.text.isNotEmpty) {

                      Navigator.pop(context, {

                        'question': selectedQuestion,

                        'answer': _answerController.text,

                      });

                    }

                  },

                )

              ]

            : null,

      ),

      backgroundColor: Colors.white,

      body: _isQuestionSelected ? _buildAnswerView() : _buildQuestionList(),

    );

  }

  

  Widget _buildQuestionList() {

    return Column(

      crossAxisAlignment: CrossAxisAlignment.start,

      children: [

        // Heading similar to "Profil-elemente" style

        const Padding(

          padding: EdgeInsets.fromLTRB(24, 24, 24, 8),

          child: Text(

            'Wähle eine Frage',

            style: TextStyle(

              fontSize: 24,

              fontWeight: FontWeight.bold,

              color: Colors.black87,

              fontFamily: 'MyAdobeFont',

            ),

          ),

        ),

        Expanded(

          child: ListView.builder(

            // Match horizontal padding to ProfileElementsPage

            padding: const EdgeInsets.symmetric(horizontal: 24),

            itemCount: questions.length,

            itemBuilder: (context, index) {

              final question = questions[index];

              return GestureDetector(

                onTap: () {

                  setState(() {

                    selectedQuestion = question;

                    _isQuestionSelected = true;

                  });

                  // Focus the answer field immediately after tapping a question

                  Future.delayed(Duration.zero, () {

                    _answerFocusNode.requestFocus();

                  });

                },

                child: Container(

                  margin: const EdgeInsets.only(bottom: 20),

                  padding: const EdgeInsets.symmetric(

                    horizontal: 16.0,

                    vertical: 14.0,

                  ),

                  decoration: BoxDecoration(

                    color: Colors.white,

                    borderRadius: BorderRadius.circular(19),

                    border: Border.all(

                      color: Colors.grey.shade400,

                      width: 2,

                    ),

                  ),

                  child: Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: [

                      Text(

                        question,

                        style: const TextStyle(

                          fontSize: 18,

                          fontWeight: FontWeight.bold,

                          color: Colors.black87,

                          fontFamily: 'MyAdobeFont',

                        ),

                      ),

                      const SizedBox(height: 10),

                      const Text(

                        'tippe um zu Antworten',

                        style: TextStyle(

                          fontSize: 16,

                          color: Colors.grey,

                          fontWeight: FontWeight.w300,

                          fontFamily: 'MyAdobeFont',

                        ),

                      ),

                    ],

                  ),

                ),

              );

            },

          ),

        ),

      ],

    );

  }

  

  Widget _buildAnswerView() {

    return SingleChildScrollView(

      child: Padding(

        // Outer padding around the card

        padding: const EdgeInsets.fromLTRB(24, 24, 24, 24),

        child: Container(

          // Container to emulate the card styling

          padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 16),

          decoration: BoxDecoration(

            color: Colors.white,

            borderRadius:

                BorderRadius.circular(19), // match ProfileElementsPage

            border: Border.all(

              color: Colors.grey.shade400,

              width: 2,

            ),

          ),

          child: Column(

            crossAxisAlignment: CrossAxisAlignment.start,

            children: [

              Text(

                selectedQuestion ?? 'Frage auswählen',

                style: const TextStyle(

                  fontSize: 18,

                  fontWeight: FontWeight.bold,

                  color: Colors.black87,

                  fontFamily: 'MyAdobeFont',

                ),

              ),

              const SizedBox(height: 16),

              TextField(

                controller: _answerController,

                focusNode: _answerFocusNode,

                maxLines: null,

                keyboardType: TextInputType.multiline,

                decoration: const InputDecoration(

                  hintText: 'Antwort eingeben...',

                  border: InputBorder.none,

                  contentPadding: EdgeInsets.zero,

                ),

                style: const TextStyle(

                  fontSize: 16,

                  color: Colors.black87,

                  fontFamily: 'MyAdobeFont',

                ),

                autofocus: true,

              ),

            ],

          ),

        ),

      ),

    );

  }

}
```
- profile_creation/dating_radius.dart
```
import 'dart:math' as math;

import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:geolocator/geolocator.dart';

import 'package:flutter_map/flutter_map.dart';

import 'package:latlong2/latlong.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/profile_creation/open_to.dart'; // For JWT

  

void main() {

  runApp(const MaterialApp(

    debugShowCheckedModeBanner: false,

    home: DatingRadiusPage(),

  ));

}

  

class DatingRadiusPage extends StatefulWidget {

  const DatingRadiusPage({super.key});

  

  @override

  State<DatingRadiusPage> createState() => _DatingRadiusPageState();

}

  

class _DatingRadiusPageState extends State<DatingRadiusPage> {

  double _currentDistance = 40.0; // The radius (km)

  Position? _currentPosition; // User’s current location

  late final MapController _mapController;

  bool _mapReady = false; // Flag for when the map has rendered

  final _storage = FlutterSecureStorage(); // For JWT token storage

  

  @override

  void initState() {

    super.initState();

    _mapController = MapController();

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Dating Radius')),

      body: Stack(

        children: [

          // Main scrollable content

          Positioned.fill(

            child: SingleChildScrollView(

              padding: const EdgeInsets.all(16.0),

              child: Column(

                crossAxisAlignment: CrossAxisAlignment.start,

                children: [

                  const Text(

                    'I think you get it but for layout-consistency, here is some text.',

                    style: TextStyle(color: Colors.grey),

                  ),

                  const SizedBox(height: 16),

  

                  // Map container

                  Container(

                    height: 300,

                    width: double.infinity,

                    decoration: BoxDecoration(

                      border: Border.all(color: Colors.grey),

                      borderRadius: BorderRadius.circular(8),

                    ),

                    child: _currentPosition == null

                        ? const Center(

                            child: Text(

                              'Enable location to show map',

                              style: TextStyle(color: Colors.brown),

                            ),

                          )

                        : _buildMap(), // Only build map if we have a location

                  ),

                  const SizedBox(height: 16),

  

                  // Enable location button

                  ElevatedButton(

                    onPressed: _requestLocation,

                    style: ElevatedButton.styleFrom(

                      padding: const EdgeInsets.symmetric(

                        horizontal: 24,

                        vertical: 12,

                      ),

                      shape: RoundedRectangleBorder(

                        borderRadius: BorderRadius.circular(24),

                      ),

                    ),

                    child: const Text('Enable Location'),

                  ),

                  const SizedBox(height: 16),

  

                  const SizedBox(height: 32),

  

                  // Slider with a distance bubble

                  Center(

                    child: SizedBox(

                      width: 300,

                      child: Stack(

                        alignment: Alignment.center,

                        children: [

                          Slider(

                            min: 10,

                            max: 100,

                            divisions: 99,

                            value: _currentDistance,

                            onChanged: (value) {

                              setState(() {

                                _currentDistance = value;

                              });

                              if (_mapReady && _currentPosition != null) {

                                _updateMapView();

                              }

                            },

                          ),

                          Positioned(

                            top: 0,

                            left: _currentDistance * 2.5,

                            child: Material(

                              color: Colors.transparent,

                              child: Container(

                                padding: const EdgeInsets.all(6),

                                decoration: BoxDecoration(

                                  color: Colors.white,

                                  borderRadius: BorderRadius.circular(12),

                                  boxShadow: const [

                                    BoxShadow(

                                      color: Colors.black26,

                                      blurRadius: 4,

                                      offset: Offset(0, 2),

                                    ),

                                  ],

                                ),

                                child: Text(

                                  '${_currentDistance.toStringAsFixed(0)} km',

                                  style: const TextStyle(

                                    fontSize: 14,

                                    color: Colors.black,

                                  ),

                                ),

                              ),

                            ),

                          ),

                        ],

                      ),

                    ),

                  ),

                ],

              ),

            ),

          ),

  

          // Pinned arrow icon at bottom-right

          Positioned(

            bottom: 20,

            right: 30,

            child: IconButton(

              icon: SvgPicture.asset(

                'assets/icons/arrow_icon.svg', // Adjust path as needed

                width: 50,

                height: 50,

              ),

              onPressed: _onArrowPressed, // Updated to use _onArrowPressed

            ),

          ),

        ],

      ),

    );

  }

  

  /// Builds the map

  Widget _buildMap() {

    final userLocation = LatLng(

      _currentPosition!.latitude,

      _currentPosition!.longitude,

    );

  

    return FlutterMap(

      mapController: _mapController,

      options: MapOptions(

        onMapReady: () {

          setState(() {

            _mapReady = true;

          });

          _updateMapView();

        },

        initialCenter: userLocation,

        initialZoom: 10,

      ),

      children: [

        TileLayer(

          urlTemplate: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',

          subdomains: ['a', 'b', 'c'],

        ),

        CircleLayer(

          circles: [

            CircleMarker(

              point: userLocation,

              color: Colors.blue.withOpacity(0.3),

              borderColor: Colors.blue,

              borderStrokeWidth: 2,

              useRadiusInMeter: true,

              radius: _currentDistance * 1000, // in meters

            ),

          ],

        ),

        MarkerLayer(

          markers: [

            Marker(

              width: 40.0,

              height: 40.0,

              point: userLocation,

              child: const Icon(

                Icons.location_pin,

                color: Colors.red,

                size: 40,

              ),

            ),

          ],

        ),

      ],

    );

  }

  

  /// Updates the map view when the radius or user location changes

  void _updateMapView() {

    if (!_mapReady || _currentPosition == null) return;

  

    final userLocation = LatLng(

      _currentPosition!.latitude,

      _currentPosition!.longitude,

    );

  

    // Compute a suitable zoom so that the circle is ~30% of map width

    final newZoom = _calculateZoomByBounds(userLocation, _currentDistance);

    _mapController.move(userLocation, newZoom);

  }

  

  /// Calculates the zoom level based on the radius

  double _calculateZoomByBounds(LatLng center, double radiusKm) {

    final latRadians = center.latitude * math.pi / 180;

    final latDiff = (radiusKm / 111.0) * 2; // total lat delta

    final lonDiff = (radiusKm / (111.0 * math.cos(latRadians))) * 2;

    final maxDiff = math.max(latDiff, lonDiff);

  

    // log2(360 / degrees)

    double zoom = math.log(360 / maxDiff) / math.log(2);

  

    // Subtract to keep circle smaller on map

    zoom = zoom - 1.2;

    if (zoom < 2.0) zoom = 2.0;

    if (zoom > 17.0) zoom = 17.0;

    return zoom;

  }

  

  /// Requests the current location from the device

  Future<void> _requestLocation() async {

    final serviceEnabled = await Geolocator.isLocationServiceEnabled();

    if (!serviceEnabled) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

          content: Text('Location services are disabled. Please enable them.'),

        ),

      );

      return;

    }

  

    var permission = await Geolocator.checkPermission();

    if (permission == LocationPermission.denied) {

      permission = await Geolocator.requestPermission();

      if (permission == LocationPermission.denied) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text('Location permission denied.')),

        );

        return;

      }

    }

  

    if (permission == LocationPermission.deniedForever) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

          content: Text('Location permission is permanently denied.'),

        ),

      );

      return;

    }

  

    final position = await Geolocator.getCurrentPosition(

      desiredAccuracy: LocationAccuracy.high,

    );

  

    setState(() {

      _currentPosition = position;

    });

  

    if (_mapReady && _currentPosition != null) {

      _updateMapView();

    }

  }

  

  /// Called when the user presses the arrow button

  Future<void> _onArrowPressed() async {

    if (_currentPosition == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('Please enable location first')),

      );

      return;

    }

  

    final token = await _storage.read(key: 'jwt_token');

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('Authentication required')),

      );

      return;

    }

  

    try {

      final response = await http.post(

        Uri.parse(

          '${Config.backendBaseUrl}/create_profile/update_loc.php',

        ),

        headers: {

          'Authorization': 'Bearer $token',

          'Content-Type': 'application/json',

        },

        body: json.encode({

          'latitude': _currentPosition!.latitude,

          'longitude': _currentPosition!.longitude,

          'dating_radius': _currentDistance,

        }),

      );

  

      if (response.statusCode == 200) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text('Location updated successfully')),

        );

  

        // Navigate to ProfilbilderLayout after successful upload

        Navigator.push(

          context,

          MaterialPageRoute(

            builder: (context) => const OpenToPage(),

          ),

        );

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text('Error: ${response.body}')),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Failed to update location: $e')),

      );

    }

  }

}

  

/// Example ProfilbilderLayout page

class ProfilbilderLayout extends StatelessWidget {

  const ProfilbilderLayout({super.key});

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Profilbilder')),

      body: const Center(

        child: Text('Profilbilder Layout'),

      ),

    );

  }

}
```
- profile_creation/open_to.dart
```
import 'package:flutter/material.dart';

import 'dart:convert';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/profile_creation/searching.dart';

  

class OpenToPage extends StatefulWidget {

  const OpenToPage({super.key});

  

  @override

  _OpenToPage createState() => _OpenToPage();

}

  

class _OpenToPage extends State<OpenToPage> {

  final String jsonString = '''

  {

    "title": "Open to",

    "subtitle": "What activities are you open to exploring together?",

    "sections": [

      {

        "categoryName": "Outdoor",

        "keywords": [

          "jogging", "hiking", "biking", "kayaking", "rock climbing", "camping", "birdwatching"

        ],

        "moreKeywords": [

          "stand-up paddleboarding", "fishing", "snowshoeing", "surfing", "horseback riding", "trail running", "scuba diving", "whitewater rafting", "beach volleyball", "zip-lining"

        ],

        "hasShowMore": true

      },

      {

        "categoryName": "Indoor Activities",

        "keywords": [

          "cooking", "yoga", "games", "movies", "Arts & Crafts", "puzzles", "board games"

        ],

        "moreKeywords": [

          "dance classes", "homebrewing", "woodworking", "pottery", "karaoke", "painting", "knitting", "mixology", "card games", "language exchange"

        ],

        "hasShowMore": true

      }

    ]

  }

  ''';

  

  late final Map<String, dynamic> data;

  late final String title;

  late final String subtitle;

  late final List<dynamic> sections;

  final Map<String, bool> isExpanded = {};

  final Set<String> selectedBubbles = {};

  final storage = const FlutterSecureStorage();

  

  @override

  void initState() {

    super.initState();

    data = json.decode(jsonString);

    title = data['title'] ?? '';

    subtitle = data['subtitle'] ?? '';

    sections = data['sections'] ?? [];

  

    for (var section in sections) {

      final categoryName = section['categoryName'] as String;

      isExpanded[categoryName] = false;

    }

  }

  

  void _toggleBubble(String keyword) {

    setState(() {

      selectedBubbles.contains(keyword)

          ? selectedBubbles.remove(keyword)

          : selectedBubbles.add(keyword);

    });

  }

  

  Widget _buildBubble(String keyword) {

    final isSelected = selectedBubbles.contains(keyword);

    return GestureDetector(

      onTap: () => _toggleBubble(keyword),

      child: Container(

        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),

        decoration: BoxDecoration(

          color: isSelected ? Colors.blue[800] : Colors.grey[200],

          borderRadius: BorderRadius.circular(20),

          border: Border.all(

            color: isSelected ? Colors.blue : Colors.grey,

            width: 1,

          ),

        ),

        child: Row(

          mainAxisSize: MainAxisSize.min,

          children: [

            Text(

              keyword,

              style: TextStyle(color: isSelected ? Colors.white : Colors.black),

            ),

            if (isSelected)

              const Padding(

                padding: EdgeInsets.only(left: 8),

                child: Icon(Icons.close, size: 16, color: Colors.white),

              ),

          ],

        ),

      ),

    );

  }

  

  Future<void> _saveSelections() async {

    final token = await storage.read(key: 'jwt_token');

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('No JWT token found. Please log in.')),

      );

      return;

    }

  

    final response = await http.post(

      Uri.parse('${Config.backendBaseUrl}/create_profile/update_open_to.php'),

      headers: {

        'Authorization': 'Bearer $token',

        'Content-Type': 'application/json',

      },

      body: json.encode({'open_to': selectedBubbles.toList()}),

    );

  

    if (response.statusCode == 200) {

      final body = json.decode(response.body);

      if (body['success'] == true) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

              content:

                  Text(body['message'] ?? 'Activities updated successfully.')),

        );

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text('No changes made or unknown response.')),

        );

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(

          content: Text(

              'Error ${response.statusCode} saving selections: ${response.body}'),

        ),

      );

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Activities')),

      body: Stack(

        children: [

          Positioned.fill(

            child: SingleChildScrollView(

              padding: const EdgeInsets.all(16),

              child: Column(

                crossAxisAlignment: CrossAxisAlignment.start,

                children: [

                  Text(title, style: Theme.of(context).textTheme.titleLarge),

                  const SizedBox(height: 8),

                  Text(subtitle),

                  const SizedBox(height: 16),

                  ...sections.map((section) {

                    final categoryName = section['categoryName'] as String;

                    final keywords =

                        List<String>.from(section['keywords'] ?? []);

                    final moreKeywords =

                        List<String>.from(section['moreKeywords'] ?? []);

                    final hasShowMore = section['hasShowMore'] ?? false;

                    final expanded = isExpanded[categoryName] ?? false;

  

                    return Padding(

                      padding: const EdgeInsets.only(bottom: 16),

                      child: Column(

                        crossAxisAlignment: CrossAxisAlignment.start,

                        children: [

                          Text(

                            categoryName,

                            style: const TextStyle(

                              fontSize: 18,

                              fontWeight: FontWeight.bold,

                            ),

                          ),

                          const SizedBox(height: 8),

                          Wrap(

                            spacing: 8,

                            runSpacing: 8,

                            children: [

                              ...keywords.map(_buildBubble),

                              if (expanded) ...moreKeywords.map(_buildBubble),

                            ],

                          ),

                          if (hasShowMore)

                            GestureDetector(

                              onTap: () => setState(

                                  () => isExpanded[categoryName] = !expanded),

                              child: Padding(

                                padding: const EdgeInsets.only(top: 8),

                                child: Text(

                                  expanded ? 'show less ...' : 'show more ...',

                                  style: const TextStyle(color: Colors.blue),

                                ),

                              ),

                            )

                        ],

                      ),

                    );

                  }),

                ],

              ),

            ),

          ),

          Positioned(

            bottom: 20,

            right: 30,

            child: IconButton(

              icon: SvgPicture.asset(

                'assets/icons/arrow_icon.svg',

                width: 50,

                height: 50,

              ),

              onPressed: () async {

                final token = await storage.read(key: 'jwt_token');

                if (token == null) return;

  

                try {

                  await _saveSelections();

  

                  // Navigate directly to MainPage after successful save

                  Navigator.pushAndRemoveUntil(

                    context,

                    MaterialPageRoute(

                        builder: (context) => FilterSelectionPage()),

                    (Route<dynamic> route) => false,

                  );

                } catch (e) {

                  ScaffoldMessenger.of(context).showSnackBar(

                    SnackBar(content: Text('Error: ${e.toString()}')),

                  );

                }

              },

            ),

          ),

        ],

      ),

    );

  }

}
```
- profile_creation/profilbilder.dart
```
import 'dart:io';

import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter_svg/svg.dart';

import 'package:image_picker/image_picker.dart';

import 'package:image_cropper/image_cropper.dart';

import 'package:reorderables/reorderables.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

  

import 'profil_elemente.dart';

  

class ProfilbilderLayout extends StatefulWidget {

  const ProfilbilderLayout({super.key});

  

  @override

  _ProfilbilderLayoutState createState() => _ProfilbilderLayoutState();

}

  

class _ProfilbilderLayoutState extends State<ProfilbilderLayout> {

  // Number of "slots" for images

  final int itemCount = 9;

  

  // Stores File references to the images chosen or null if empty

  final List<File?> _selectedImages = List.filled(9, null);

  

  // Image picker for camera/gallery

  final ImagePicker _picker = ImagePicker();

  

  // Secure storage to retrieve saved JWT token

  final FlutterSecureStorage _secureStorage = const FlutterSecureStorage();

  String? _jwtToken;

  

  @override

  void initState() {

    super.initState();

    _fetchToken();

  }

  

  /// Fetches the JWT token from secure storage

  Future<void> _fetchToken() async {

    final token = await _secureStorage.read(key: 'jwt_token');

    setState(() {

      _jwtToken = token;

    });

  

    if (_jwtToken == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

          content: Text(

            'Authentication token not found. Please login again.',

          ),

        ),

      );

    }

  }

  

  /// Allows picking & cropping an image for a specific slot

  Future<void> _pickAndCropImage(int index) async {

    if (_selectedImages[index] != null) {

      // If there's already an image in this slot, allow re-cropping or re-editing

      final croppedFile = await Navigator.push<File?>(

        context,

        MaterialPageRoute(

          builder: (context) => EditAnImage(

            imageFile: _selectedImages[index]!,

            aspectRatio: const CropAspectRatio(ratioX: 8, ratioY: 12),

          ),

        ),

      );

      if (croppedFile != null) {

        setState(() => _selectedImages[index] = croppedFile);

      }

    } else {

      // If empty, let user choose source: camera or gallery

      final source = await showModalBottomSheet<ImageSource>(

        context: context,

        isDismissible: true,

        builder: (context) => SafeArea(

          child: Column(

            mainAxisSize: MainAxisSize.min,

            children: [

              ListTile(

                leading: const Icon(Icons.camera_alt),

                title: const Text('Take Photo'),

                onTap: () => Navigator.pop(context, ImageSource.camera),

              ),

              ListTile(

                leading: const Icon(Icons.photo_library),

                title: const Text('Choose from Gallery'),

                onTap: () => Navigator.pop(context, ImageSource.gallery),

              ),

            ],

          ),

        ),

      );

  

      if (source == null) return; // User dismissed the bottom sheet

  

      final XFile? image = await _picker.pickImage(source: source);

      if (image == null) return; // User did not pick any image

  

      // Navigate to crop screen

      final croppedFile = await Navigator.push<File?>(

        context,

        MaterialPageRoute(

          builder: (context) => EditAnImage(

            imageFile: File(image.path),

            aspectRatio: const CropAspectRatio(ratioX: 8, ratioY: 12),

          ),

        ),

      );

      if (croppedFile != null) {

        setState(() => _selectedImages[index] = croppedFile);

      }

    }

  }

  

  /// Allows reordering images in the grid

  void _reorderImages(int oldIndex, int newIndex) {

    setState(() {

      if (oldIndex < newIndex) {

        newIndex -= 1;

      }

      final File? image = _selectedImages.removeAt(oldIndex);

      _selectedImages.insert(newIndex, image);

    });

  }

  

  /// Upload images to the server using a multipart POST request

  Future<void> _uploadImages() async {

    // If we have no token, show error and return

    if (_jwtToken == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(

          content: Text('No token available. Please login again.'),

        ),

      );

      return;

    }

  

    // Your actual endpoint

    final uri = Uri.parse(

      '${Config.backendBaseUrl}/create_profile/upload_profile_images.php',

    );

  

    // Create a multipart request

    var request = http.MultipartRequest('POST', uri);

  

    // Add the Authorization header with the JWT token

    request.headers['Authorization'] = 'Bearer $_jwtToken';

  

    // If you want to pass a pre-existing images array from the front-end:

    // For now, we keep an empty list. If you have existing images from DB,

    // you could pass them here in the same format.

    final existingImages = [];

  

    // Add it as a text field in the multipart request

    request.fields['images'] = json.encode(existingImages);

  

    // Add any non-null selected images to the request

    for (int i = 0; i < _selectedImages.length; i++) {

      if (_selectedImages[i] != null) {

        final file = _selectedImages[i]!;

        final fileName = file.path.split('/').last;

        request.files.add(

          await http.MultipartFile.fromPath(

            'image$i', // name/key in $_FILES array on server

            file.path,

            filename: fileName,

          ),

        );

      }

    }

  

    try {

      // Send the request

      final streamedResponse = await request.send();

      final response = await http.Response.fromStream(streamedResponse);

  

      if (response.statusCode == 200) {

        final responseData = json.decode(response.body);

  

        if (responseData['success'] == true) {

          // Successfully saved in DB, get the final images array

          final imagesArray = responseData['images'] as List<dynamic>;

          debugPrint('Uploaded images: $imagesArray');

  

          // Navigate to the next page

          Navigator.push(

            context,

            MaterialPageRoute(

              builder: (context) => const ProfileElementsPage(),

            ),

          );

        } else {

          debugPrint('Upload failed: ${response.body}');

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(

              content: Text('Error: ${responseData['error'] ?? 'Unknown'}'),

            ),

          );

        }

      } else {

        debugPrint('Server returned status ${response.statusCode}');

        debugPrint('Body: ${response.body}');

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text(

              'Upload failed with status ${response.statusCode}',

            ),

          ),

        );

      }

    } catch (e) {

      debugPrint('Exception caught: $e');

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Upload error: $e')),

      );

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      // Entire screen

      body: Stack(

        fit: StackFit.expand,

        children: [

          // Main content

          Padding(

            padding: const EdgeInsets.all(16.0),

            child: Column(

              crossAxisAlignment: CrossAxisAlignment.start,

              children: [

                const SizedBox(height: 24),

                const Text(

                  'Show yourself',

                  style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),

                ),

                const SizedBox(height: 8),

                const Text(

                  "Don't worry you will be judged but won't see it ❤️",

                  style: TextStyle(fontSize: 16),

                ),

                const SizedBox(height: 24),

                // A reorderable wrap grid

                Expanded(

                  child: ReorderableWrap(

                    spacing: 16,

                    runSpacing: 16,

                    onReorder: _reorderImages,

                    children: List.generate(itemCount, (index) {

                      return GestureDetector(

                        key: Key('$index'),

                        onTap: () => _pickAndCropImage(index),

                        child: Container(

                          width: MediaQuery.of(context).size.width / 3 - 24,

                          height: (MediaQuery.of(context).size.width / 3 - 24) *

                              (3.5 / 3),

                          decoration: BoxDecoration(

                            color: const Color(0xFFF6F6F6),

                            border: Border.all(

                              color: const Color(0xFFBFBBBB),

                              width: 2,

                            ),

                            borderRadius: BorderRadius.circular(8),

                          ),

                          child: _selectedImages[index] != null

                              ? ClipRRect(

                                  borderRadius: BorderRadius.circular(6),

                                  child: Image.file(

                                    _selectedImages[index]!,

                                    fit: BoxFit.cover,

                                  ),

                                )

                              : Center(

                                  child: SvgPicture.asset(

                                    'assets/icons/image_placeholder.svg',

                                    fit: BoxFit.contain,

                                  ),

                                ),

                        ),

                      );

                    }),

                  ),

                ),

                const SizedBox(height: 16),

                const Center(

                  child: Text(

                    'min 3 hinzufügen',

                    style: TextStyle(

                      fontSize: 16,

                      fontWeight: FontWeight.w300,

                      color: Color(0xFF7D7D7D),

                    ),

                  ),

                ),

              ],

            ),

          ),

          // Button to go next (which uploads)

          Positioned(

            bottom: 20,

            right: 30,

            child: IconButton(

              icon: SvgPicture.asset(

                'assets/icons/arrow_icon.svg', // your arrow icon

                width: 50,

                height: 50,

              ),

              onPressed: _uploadImages,

            ),

          ),

        ],

      ),

    );

  }

}

  

/// A screen for cropping an image before returning it

class EditAnImage extends StatefulWidget {

  final File imageFile;

  final CropAspectRatio aspectRatio;

  

  const EditAnImage({

    required this.imageFile,

    required this.aspectRatio,

    super.key,

  });

  

  @override

  _EditAnImageState createState() => _EditAnImageState();

}

  

class _EditAnImageState extends State<EditAnImage> {

  File? _croppedImage;

  

  @override

  void initState() {

    super.initState();

    _cropImage();

  }

  

  Future<void> _cropImage() async {

    final croppedFile = await ImageCropper().cropImage(

      sourcePath: widget.imageFile.path,

      aspectRatio: widget.aspectRatio,

      compressQuality: 90,

      uiSettings: [

        if (Platform.isAndroid)

          AndroidUiSettings(

            toolbarTitle: 'Adjust Image',

            toolbarColor: Colors.black,

            toolbarWidgetColor: Colors.white,

            initAspectRatio: CropAspectRatioPreset.original,

            lockAspectRatio: true,

          ),

        if (Platform.isIOS)

          IOSUiSettings(

            title: 'Adjust Image',

            aspectRatioLockEnabled: true,

            resetAspectRatioEnabled: false,

          ),

      ],

    );

  

    if (croppedFile != null) {

      setState(() => _croppedImage = File(croppedFile.path));

      // Return the cropped file to previous screen

      Navigator.pop(context, _croppedImage);

    } else {

      // If user cancels cropping, just pop without returning a file

      Navigator.pop(context);

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: Colors.black,

      appBar: AppBar(

        backgroundColor: Colors.black,

        actions: [

          IconButton(

            icon: const Icon(Icons.check, color: Colors.white),

            onPressed: () => Navigator.pop(context, _croppedImage),

          )

        ],

      ),

      body: Center(

        child: _croppedImage == null

            ? const CircularProgressIndicator()

            : InteractiveViewer(

                panEnabled: false,

                child: Image.file(_croppedImage!),

              ),

      ),

    );

  }

}
```
- profile_creation/profile.dart
```
import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart'; // For displaying SVG icons.

import 'package:http/http.dart' as http;

import 'dart:convert';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:google_fonts/google_fonts.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart'; // Example color file

  

// Shared constant for horizontal padding

const double kHorizontalPadding = 20.0;

  

void main() {

  runApp(MaterialApp(

    theme: ThemeData(

      textTheme: GoogleFonts.zillaSlabTextTheme(),

    ),

    home: ProfilePage(),

  ));

}

  

class ProfilePage extends StatelessWidget {

  final Future<ProfileData> profileData;

  

  ProfilePage({super.key, ProfileData? preloadedData})

      : profileData = preloadedData != null

            ? Future.value(preloadedData)

            : _fetchProfileData();

  

  static Future<void> preloadImages(BuildContext context) async {

    try {

      final profileData = await _fetchProfileData();

  

      final imageUrls = [

        ...profileData.topSection.profileImages,

        ...profileData.elements.values

            .where((element) => element.type == 'image')

            .map((element) => element.content['image_link'] as String),

      ];

  

      for (final imageUrl in imageUrls) {

        await precacheImage(NetworkImage(imageUrl), context);

      }

    } catch (e) {

      print('Failed to preload images: $e');

    }

  }

  

  static Future<ProfileData> _fetchProfileData() async {

    const storage = FlutterSecureStorage();

    final token = await storage.read(key: 'jwt_token');

  

    if (token == null) {

      throw Exception('No JWT token found. Please log in.');

    }

  

    final response = await http.get(

      Uri.parse('${Config.backendBaseUrl}/create_profile/get_profile.php'),

      headers: {'Authorization': 'Bearer $token'},

    );

  

    print('Response status: ${response.statusCode}');

    print('Full response body:');

    const chunkSize = 1000;

    for (var i = 0; i < response.body.length; i += chunkSize) {

      final end = i + chunkSize;

      print(response.body.substring(

          i, end < response.body.length ? end : response.body.length));

    }

  

    if (response.statusCode == 200) {

      final jsonData = json.decode(response.body);

      return ProfileData.fromJson(jsonData);

    } else {

      throw Exception('Failed to load profile: ${response.body}');

    }

  }

  

  @override

  Widget build(BuildContext context) {

    // Set vertical padding for all elements to 25 px.

    const double elementVerticalPadding = 25.0;

  

    return Scaffold(

      backgroundColor: Colors.grey[200],

      body: Padding(

        padding: const EdgeInsets.all(5.0),

        child: Card(

          // Card has rounded corners but does not clip overflow.

          shape: RoundedRectangleBorder(

            borderRadius: BorderRadius.circular(16.0),

          ),

          clipBehavior: Clip.none,

          elevation: 4.0,

          child: Theme(

            data: ThemeData(

              textTheme: GoogleFonts.zillaSlabTextTheme(

                Theme.of(context).textTheme,

              ),

            ),

            child: FutureBuilder<ProfileData>(

              future: profileData,

              builder: (context, snapshot) {

                if (snapshot.connectionState == ConnectionState.waiting) {

                  return const Center(child: CircularProgressIndicator());

                } else if (snapshot.hasError) {

                  return Center(child: Text('Error: ${snapshot.error}'));

                } else if (!snapshot.hasData) {

                  return const Center(child: Text('No profile data found.'));

                }

  

                final profileData = snapshot.data!;

                print('Parsed profile data: ${profileData.toJson()}');

  

                final sortedElements = profileData.elements.entries.toList()

                  ..sort((a, b) => int.parse(a.key).compareTo(int.parse(b.key)))

                  ..retainWhere(

                      (entry) => _isSupportedElementType(entry.value.type));

  

                return SingleChildScrollView(

                  child: Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: [

                      _buildTopSection(profileData.topSection),

                      ...sortedElements.map((entry) => Column(

                            crossAxisAlignment: CrossAxisAlignment.start,

                            children: [

                              Container(

                                width: double.infinity,

                                height: 2,

                                color: Colors.grey[300],

                              ),

                              Padding(

                                padding: const EdgeInsets.symmetric(

                                    vertical: elementVerticalPadding),

                                child: _buildElement(entry.value),

                              ),

                            ],

                          )),

                    ],

                  ),

                );

              },

            ),

          ),

        ),

      ),

    );

  }

  

  bool _isSupportedElementType(String type) {

    return ['icebreaker', 'image', 'bubbles'].contains(type);

  }

  

  // _buildTopSection now builds the image area with overlay elements.

  Widget _buildTopSection(TopSection topSection) {

    const double sectionSpacing = 20.0;

    final PageController pageController = PageController();

    final ValueNotifier<int> currentPage = ValueNotifier<int>(0);

  

    return Padding(

      padding: const EdgeInsets.only(top: 5.0, bottom: 5.0),

      child: Column(

        crossAxisAlignment: CrossAxisAlignment.start,

        children: [

          // Stack for the image and overlay elements.

          Stack(

            clipBehavior: Clip.none,

            children: [

              // Clip only the image for rounded top corners.

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

                    physics: const ClampingScrollPhysics(),

                    itemBuilder: (context, index) => GestureDetector(

                      behavior: HitTestBehavior.opaque,

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

                        width: double.infinity,

                        fit: BoxFit.cover,

                      ),

                    ),

                  ),

                ),

              ),

              // Progress indicator bars at the top center with 5px horizontal padding.

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

                          (index) => Expanded(

                            child: Container(

                              height: 3,

                              margin: const EdgeInsets.symmetric(horizontal: 1),

                              decoration: BoxDecoration(

                                color: index == value

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

              // Two round buttons overlayed at the bottom.

              // Their centers will be at the bottom edge of the image.

              Positioned(

                bottom: -22, // With CircleAvatar radius 22, centers at bottom.

                right: 10,

                child: Row(

                  mainAxisSize: MainAxisSize.min,

                  children: [

                    CircleAvatar(

                      radius: 22,

                      backgroundColor: Colors.white,

                      child: Icon(

                        Icons.navigation,

                        color: Colors.grey[800],

                      ),

                    ),

                    const SizedBox(width: 15),

                    CircleAvatar(

                      radius: 22,

                      backgroundColor: Colors.white,

                      child: Icon(

                        Icons.chat_bubble,

                        color: Colors.pinkAccent,

                      ),

                    ),

                  ],

                ),

              ),

            ],

          ),

          // Extra spacing to account for the overlayed buttons.

          SizedBox(height: sectionSpacing + 22),

          // Name and age.

          Padding(

            padding: const EdgeInsets.symmetric(horizontal: kHorizontalPadding),

            child: Text(

              "${topSection.name}, ${topSection.age}",

              style: GoogleFonts.zillaSlab(

                fontSize: 26,

                fontWeight: FontWeight.w700,

              ),

            ),

          ),

          const SizedBox(height: sectionSpacing),

          // Info bubbles with icons for gender, language, religion, and politics.

          Padding(

            padding: const EdgeInsets.symmetric(horizontal: kHorizontalPadding),

            child: Wrap(

              spacing: 10,

              runSpacing: 10,

              children: topSection.infoBubbles.asMap().entries.map((entry) {

                int index = entry.key;

                String bubble = entry.value;

                String iconAsset = getIconForInfoBubble(bubble, index);

                return Chip(

                  avatar: iconAsset.isNotEmpty

                      ? SvgPicture.asset(

                          iconAsset,

                          width: 16,

                          height: 16,

                        )

                      : null,

                  label: Text(

                    bubble,

                    style: GoogleFonts.zillaSlab(

                      fontSize: 14,

                      fontWeight: FontWeight.w500,

                      color: Colors.grey[800],

                    ),

                  ),

                  backgroundColor: Colors.grey[300],

                  padding: const EdgeInsets.symmetric(

                    horizontal: 10,

                    vertical: 3,

                  ),

                  shape: RoundedRectangleBorder(

                    borderRadius: BorderRadius.circular(100),

                  ),

                );

              }).toList(),

            ),

          ),

        ],

      ),

    );

  }

  

  Widget _buildElement(Element element) {

    switch (element.type) {

      case 'icebreaker':

        return IceBreakerLayout(content: element.content);

      case 'image':

        return ImageLayout(content: element.content);

      case 'bubbles':

        return BubbleGridLayout(content: element.content);

      default:

        return const SizedBox.shrink();

    }

  }

}

  

// Helper function to get the appropriate icon asset based on bubble text and its index.

String getIconForInfoBubble(String bubble, int index) {

  // We assume the first bubble represents gender.

  if (index == 0) {

    final lower = bubble.toLowerCase();

    if (lower == 'female') {

      return 'assets/icons/female.svg';

    } else if (lower == 'male') {

      return 'assets/icons/male.svg';

    } else {

      return 'assets/icons/transgender.svg';

    }

  } else if (index == 1) {

    return 'assets/icons/language.svg';

  } else if (index == 2) {

    return 'assets/icons/religion.svg';

  } else if (index == 3) {

    return 'assets/icons/politics.svg';

  }

  return '';

}

  

class ProfileData {

  final TopSection topSection;

  final Map<String, Element> elements;

  

  ProfileData({required this.topSection, required this.elements});

  

  factory ProfileData.fromJson(Map<String, dynamic> json) {

    return ProfileData(

      topSection: TopSection.fromJson(json['top_section']),

      elements: (json['elements'] as Map<String, dynamic>).map(

        (key, value) => MapEntry(

          key,

          Element.fromJson(value as Map<String, dynamic>),

        ),

      ),

    );

  }

  

  Map<String, dynamic> toJson() {

    return {

      'top_section': topSection.toJson(),

      'elements': elements.map((key, value) => MapEntry(key, value.toJson())),

    };

  }

}

  

class TopSection {

  final List<String> profileImages;

  final String name;

  final String age;

  final List<String> infoBubbles;

  

  TopSection({

    required this.profileImages,

    required this.name,

    required this.age,

    required this.infoBubbles,

  });

  

  factory TopSection.fromJson(Map<String, dynamic> json) {

    return TopSection(

      profileImages: List<String>.from(json['profile_images'] ?? []),

      name: json['name'] ?? '',

      age: json['age'] ?? '',

      infoBubbles: List<String>.from(json['info_bubbles'] ?? []),

    );

  }

  

  Map<String, dynamic> toJson() {

    return {

      'profile_images': profileImages,

      'name': name,

      'age': age,

      'info_bubbles': infoBubbles,

    };

  }

}

  

class Element {

  final String type;

  final Map<String, dynamic> content;

  

  Element({required this.type, required this.content});

  

  factory Element.fromJson(Map<String, dynamic> json) {

    return Element(

      type: json['type'] ?? '',

      content: Map<String, dynamic>.from(json['content'] ?? {}),

    );

  }

  

  Map<String, dynamic> toJson() {

    return {

      'type': type,

      'content': content,

    };

  }

}

  

class IceBreakerLayout extends StatelessWidget {

  final Map<String, dynamic> content;

  

  const IceBreakerLayout({super.key, required this.content});

  

  @override

  Widget build(BuildContext context) {

    final question = content['question'] ?? 'No question provided';

    final answer = content['answer'] ?? 'No answer provided';

  

    return Padding(

      padding: const EdgeInsets.fromLTRB(

          kHorizontalPadding, 25, kHorizontalPadding, 25),

      child: Column(

        crossAxisAlignment: CrossAxisAlignment.start,

        children: [

          Text(

            question,

            style: GoogleFonts.zillaSlab(

              fontSize: 18,

              fontWeight: FontWeight.w600,

              color: AppColors.text,

            ),

          ),

          const SizedBox(height: 5),

          Text(

            answer,

            style: GoogleFonts.zillaSlab(

              fontSize: 16,

              fontWeight: FontWeight.w400,

              color: Colors.grey[700],

              height: 1.4,

            ),

          ),

        ],

      ),

    );

  }

}

  

class ImageLayout extends StatelessWidget {

  final Map<String, dynamic> content;

  

  const ImageLayout({super.key, required this.content});

  

  @override

  Widget build(BuildContext context) {

    return Column(

      children: [

        Image.network(

          content['image_link'] ?? '',

          width: double.infinity,

          fit: BoxFit.cover,

        ),

        const SizedBox(height: 20),

        Padding(

          padding: const EdgeInsets.symmetric(horizontal: kHorizontalPadding),

          child: Text(

            content['subtitle'] ?? '',

            style: GoogleFonts.zillaSlab(

              fontSize: 14,

              color: Colors.grey[600],

            ),

          ),

        ),

      ],

    );

  }

}

  

class BubbleGridLayout extends StatelessWidget {

  final Map<String, dynamic> content;

  

  const BubbleGridLayout({super.key, required this.content});

  

  @override

  Widget build(BuildContext context) {

    final List<dynamic> bubbles = (content['bubbles'] as List<dynamic>?) ?? [];

  

    return Padding(

      padding: const EdgeInsets.symmetric(

        horizontal: kHorizontalPadding,

        vertical: 15,

      ),

      child: Wrap(

        spacing: 12, // horizontal spacing between bubbles

        runSpacing: 12, // vertical spacing between lines

        children: bubbles.map<Widget>((bubble) {

          return Chip(

            label: Text(

              bubble.toString().trim(),

              style: GoogleFonts.zillaSlab(

                fontSize: 14,

                fontWeight: FontWeight.w500,

                color: Colors.grey[800],

              ),

            ),

            backgroundColor: Colors.grey[300],

            padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),

            shape: RoundedRectangleBorder(

              borderRadius: BorderRadius.circular(100),

            ),

          );

        }).toList(),

      ),

    );

  }

}
```
- profile_creation/profil_elemente.dart
```
import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:flutter_svg/svg.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';

import 'dart:convert';

  

import 'package:swipe_chat_play/profile_creation/choose_question.dart';

import 'package:swipe_chat_play/profile_creation/dating_radius.dart';

import 'qa_item.dart';

  

class ProfileElementsPage extends StatefulWidget {

  const ProfileElementsPage({super.key});

  

  @override

  _ProfileElementsPageState createState() => _ProfileElementsPageState();

}

  

class _ProfileElementsPageState extends State<ProfileElementsPage> {

  // We'll keep 3 QAItems for demonstration

  List<QAItem?> qaItems = [null, null, null];

  

  // Secure storage instance for reading the JWT token

  final storage = const FlutterSecureStorage();

  

  // This function will gather the user's Q&A data and post it to the server

  Future<void> _saveQuestionsAndAnswers() async {

    final token = await storage.read(key: 'jwt_token');

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('No JWT token found. Please log in.')),

      );

      return;

    }

  

    // Convert each QAItem to a simple map

    // If you want to exclude null entries, you can filter them out

    final List<Map<String, String>> questionsAndAnswers = qaItems

        .where((item) => item != null)

        .map((item) => {

              'question': item!.question,

              'answer': item.answer,

            })

        .toList();

  

    try {

      final response = await http.post(

        Uri.parse(

          '${Config.backendBaseUrl}/create_profile/elements_update.php',

        ),

        headers: {

          'Authorization': 'Bearer $token',

          'Content-Type': 'application/json',

        },

        body: json.encode({

          'questions_and_answers': questionsAndAnswers,

        }),

      );

  

      if (response.statusCode == 200) {

        final body = json.decode(response.body);

        if (body['success'] == true) {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(content: Text(body['message'])),

          );

          // Optionally navigate to the next page

          Navigator.push(

            context,

            MaterialPageRoute(builder: (context) => const DatingRadiusPage()),

          );

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(content: Text('Update failed or no changes made.')),

          );

        }

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text(

              'Error ${response.statusCode} saving Q&As: ${response.body}',

            ),

          ),

        );

      }

    } catch (error) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Error saving Q&As: $error')),

      );

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: Colors.white,

      body: SafeArea(

        child: Stack(

          children: [

            Column(

              crossAxisAlignment: CrossAxisAlignment.start,

              children: [

                // Title

                Padding(

                  padding: const EdgeInsets.fromLTRB(24, 24, 24, 0),

                  child: Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: const [

                      Text(

                        'Profil-elemente',

                        style: TextStyle(

                          fontSize: 24,

                          fontWeight: FontWeight.bold,

                          color: Colors.black87,

                          fontFamily: 'MyAdobeFont',

                        ),

                      ),

                      SizedBox(height: 6),

                      Text(

                        'wähle Minimum 3 fragen',

                        style: TextStyle(

                          fontSize: 16,

                          color: Colors.grey,

                          fontFamily: 'MyAdobeFont',

                        ),

                      ),

                    ],

                  ),

                ),

  

                // Q&A Cards

                Expanded(

                  child: Padding(

                    padding: const EdgeInsets.fromLTRB(24, 24, 24, 90),

                    child: Column(

                      children: [

                        Expanded(child: _buildQACard(0)),

                        const SizedBox(height: 20),

                        Expanded(child: _buildQACard(1)),

                        const SizedBox(height: 20),

                        Expanded(child: _buildQACard(2)),

                      ],

                    ),

                  ),

                ),

              ],

            ),

  

            // The button that triggers saving to the server

            Positioned(

              bottom: 20,

              right: 16,

              child: SizedBox(

                width: 60,

                height: 60,

                child: IconButton(

                  icon: SvgPicture.asset(

                    'assets/icons/arrow_icon.svg',

                    width: 50,

                    height: 50,

                  ),

                  onPressed: _saveQuestionsAndAnswers,

                ),

              ),

            ),

          ],

        ),

      ),

    );

  }

  

  Widget _buildQACard(int index) {

    final qa = qaItems[index];

    return SizedBox.expand(

      child: GestureDetector(

        onTap: () async {

          final result = await Navigator.push(

            context,

            MaterialPageRoute(

              builder: (context) => ChoseQuestionPage(

                initialQuestion: qa?.question,

                initialAnswer: qa?.answer,

              ),

            ),

          );

  

          if (result != null) {

            setState(() {

              qaItems[index] = QAItem(

                question: result['question'],

                answer: result['answer'],

              );

            });

          }

        },

        child: Container(

          width: double.infinity,

          height: double.infinity,

          padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 14.0),

          decoration: BoxDecoration(

            color: Colors.white,

            borderRadius: BorderRadius.circular(19),

            border: Border.all(color: Colors.grey.shade400, width: 2),

          ),

          child: Column(

            crossAxisAlignment: CrossAxisAlignment.start,

            mainAxisAlignment: MainAxisAlignment.center,

            children: [

              Text(

                qa?.question ?? 'wähle eine frage',

                style: TextStyle(

                  fontSize: 18,

                  fontWeight: FontWeight.bold,

                  color: qa != null ? Colors.black87 : Colors.grey,

                  fontFamily: 'MyAdobeFont',

                ),

              ),

              const SizedBox(height: 10),

              Text(

                qa?.answer ?? 'und gib deine Antwort',

                style: TextStyle(

                  fontSize: 16,

                  color: qa != null ? Colors.black54 : Colors.grey,

                  fontWeight: qa != null ? FontWeight.normal : FontWeight.w300,

                  fontFamily: 'MyAdobeFont',

                ),

              ),

            ],

          ),

        ),

      ),

    );

  }

}
```

- profile_creation/searching.dart
```
import 'package:flutter/material.dart';

import 'package:google_fonts/google_fonts.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'dart:convert';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/main_page.dart';

  

class FilterSelectionPage extends StatefulWidget {

  const FilterSelectionPage({super.key});

  

  @override

  State<FilterSelectionPage> createState() => _FilterSelectionPageState();

}

  

class _FilterSelectionPageState extends State<FilterSelectionPage> {

  // Example data

  List<String> genders = [

    "Male",

    "Female",

    "Non Binary",

    "Transgender",

    "Genderqueer",

    "Other"

  ];

  List<String> politics = [

    "Liberal",

    "Conservative",

    "Centrist",

    "Libertarian",

    "Socialist"

  ];

  List<String> religions = [

    "Christianity",

    "Islam",

    "Hinduism",

    "Buddhism",

    "Judaism",

    "Other"

  ];

  

  // Selections

  List<String> selectedGenders = [];

  List<String> selectedPolitics = [];

  List<String> selectedReligions = [];

  

  // Age range

  double _startAge = 18;

  double _endAge = 120;

  

  final storage = const FlutterSecureStorage();

  

  @override

  void initState() {

    super.initState();

    // By default, all religions and politics are selected

    selectedReligions = List.from(religions);

    selectedPolitics = List.from(politics);

  }

  

  void toggleSelection(List<String> list, List<String> selected, String item) {

    setState(() {

      if (selected.contains(item)) {

        selected.remove(item);

      } else {

        selected.add(item);

      }

    });

  }

  

  void selectAll(List<String> list, List<String> selected) {

    setState(() {

      if (selected.length == list.length) {

        selected.clear();

      } else {

        selected

          ..clear()

          ..addAll(list);

      }

    });

  }

  

  /// Saves the user's selections to your server

  Future<void> _saveSearchingFor() async {

    final token = await storage.read(key: 'jwt_token');

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('No JWT token found. Please log in.')),

      );

      return;

    }

  

    try {

      // Create the request payload

      final Map<String, dynamic> payload = {

        'genders': selectedGenders,

        'ageRange': [_startAge.round(), _endAge.round()],

        'religion': selectedReligions,

        'politics': selectedPolitics,

      };

  

      final response = await http.post(

        Uri.parse(

            '${Config.backendBaseUrl}/create_profile/update_searching_for.php'),

        headers: {

          'Authorization': 'Bearer $token',

          'Content-Type': 'application/json',

        },

        body: json.encode(payload),

      );

  

      if (response.statusCode == 200) {

        final body = json.decode(response.body);

        if (body['success'] == true) {

          ScaffoldMessenger.of(context).showSnackBar(

            SnackBar(

              content: Text(body['message'] ?? 'Filters updated successfully'),

            ),

          );

          Navigator.pushReplacement(

            context,

            MaterialPageRoute(builder: (context) => const MainPage()),

          );

  

          // For example, navigate to the next screen or close this page

          // Navigator.pushReplacement(...);

        } else {

          ScaffoldMessenger.of(context).showSnackBar(

            const SnackBar(

                content: Text('No changes made or unknown response.')),

          );

        }

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text(

                'Error ${response.statusCode} saving selections: ${response.body}'),

          ),

        );

      }

    } catch (e) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text('Error: ${e.toString()}')),

      );

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Filters')),

      body: Stack(

        children: [

          // Main content

          Positioned.fill(

            child: SingleChildScrollView(

              padding: const EdgeInsets.all(16.0),

              child: Column(

                crossAxisAlignment: CrossAxisAlignment.start,

                children: [

                  _buildBubbleSection('Gender', genders, selectedGenders,

                      allowAll: true),

                  const SizedBox(height: 20),

                  _buildAgeSlider(),

                  const SizedBox(height: 20),

                  _buildBubbleSection('Religion', religions, selectedReligions,

                      allowDontCare: true),

                  const SizedBox(height: 20),

                  _buildBubbleSection('Politics', politics, selectedPolitics,

                      allowDontCare: true),

                ],

              ),

            ),

          ),

          // Floating Next/Submit button

          Positioned(

            bottom: 20,

            right: 30,

            child: IconButton(

              icon: SvgPicture.asset(

                'assets/icons/arrow_icon.svg', // adjust path as needed

                width: 50,

                height: 50,

              ),

              onPressed: () {

                // Call the function that saves the data

                _saveSearchingFor();

              },

            ),

          ),

        ],

      ),

    );

  }

  

  Widget _buildBubbleSection(

    String title,

    List<String> options,

    List<String> selected, {

    bool allowAll = false,

    bool allowDontCare = false,

  }) {

    return Column(

      crossAxisAlignment: CrossAxisAlignment.start,

      children: [

        Text(

          title,

          style: GoogleFonts.zillaSlab(

            fontSize: 16,

            fontWeight: FontWeight.w600,

            color: Colors.grey[800],

          ),

        ),

        const SizedBox(height: 8),

        Wrap(

          spacing: 7,

          runSpacing: 5,

          children: [

            ...options.map(

              (option) => _buildBubble(

                option,

                selected,

                () => toggleSelection(options, selected, option),

              ),

            ),

            if (allowAll)

              _buildBubble(

                'All',

                selected,

                () => selectAll(options, selected),

                isSelected: selected.length == options.length,

              ),

            if (allowDontCare)

              _buildBubble(

                'Don\'t care',

                selected,

                () {

                  if (selected.length == options.length) {

                    selected.clear();

                  } else {

                    selected

                      ..clear()

                      ..addAll(options);

                  }

                  setState(() {});

                },

                isSelected: selected.length == options.length,

              ),

          ],

        ),

      ],

    );

  }

  

  Widget _buildBubble(String label, List<String> selected, VoidCallback onTap,

      {bool isSelected = false}) {

    final bool selectedOrIsSelected = isSelected || selected.contains(label);

  

    return GestureDetector(

      onTap: onTap,

      child: Chip(

        label: Text(

          label,

          style: GoogleFonts.zillaSlab(

            fontSize: 14,

            fontWeight: FontWeight.w500,

            color: selectedOrIsSelected ? Colors.white : Colors.grey[800],

          ),

        ),

        backgroundColor: selectedOrIsSelected ? Colors.blue : Colors.grey[300],

        shape: RoundedRectangleBorder(

          borderRadius: BorderRadius.circular(100),

        ),

      ),

    );

  }

  

  Widget _buildAgeSlider() {

    return Column(

      crossAxisAlignment: CrossAxisAlignment.start,

      children: [

        Text(

          'Age Range',

          style: GoogleFonts.zillaSlab(

            fontSize: 16,

            fontWeight: FontWeight.w600,

            color: Colors.grey[800],

          ),

        ),

        const SizedBox(height: 8),

        RangeSlider(

          min: 18,

          max: 120,

          values: RangeValues(_startAge, _endAge),

          onChanged: (RangeValues values) {

            setState(() {

              _startAge = values.start;

              _endAge = values.end;

            });

          },

          divisions: 102, // 18..120 inclusive

          labels: RangeLabels(

            _startAge.round().toString(),

            _endAge.round().toString(),

          ),

          activeColor: Colors.blue,

        ),

        Center(

          child: Text(

            '${_startAge.round()} - ${_endAge.round()} years',

            style: GoogleFonts.zillaSlab(

              fontSize: 14,

              fontWeight: FontWeight.w500,

              color: Colors.grey[800],

            ),

          ),

        ),

      ],

    );

  }

}
```
- profile_layout/models/profile_data.dart
```
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
```

- tabs/chat/my_chats.dart
```
import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/tabs/chat/layouts/chat.dart';

import 'dart:convert';

  

import 'package:swipe_chat_play/tabs/chat/search_user.dart';

  

class MyChatsPage extends StatefulWidget {

  const MyChatsPage({super.key});

  

  @override

  _MyChatsPageState createState() => _MyChatsPageState();

}

  

class _MyChatsPageState extends State<MyChatsPage> {

  final _secureStorage = const FlutterSecureStorage();

  List<dynamic> chats = [];

  

  @override

  void initState() {

    super.initState();

    fetchChats();

  }

  

  Future<void> fetchChats() async {

    final String? token = await _secureStorage.read(key: 'jwt_token');

  

    if (token == null) {

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(content: Text("Authorization token is missing.")),

        );

      }

      return;

    }

  

    final response = await http.get(

      Uri.parse("${Config.backendBaseUrl}/chat/myChats.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

    );

  

    if (response.statusCode == 200) {

      if (mounted) {

        setState(() {

          chats = jsonDecode(response.body);

        });

      }

    } else {

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(

            content: Text("Failed to fetch chats: ${response.reasonPhrase}"),

          ),

        );

      }

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      // No 'appBar' parameter here — we build our own custom white bar at the top.

      body: SafeArea(

        child: Column(

          children: [

            // Top "bar" with pure white background and drop shadow

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

              // 10px vertical padding around the pill-shaped search container

              padding: const EdgeInsets.symmetric(vertical: 10, horizontal: 16),

              child: Container(

                padding: const EdgeInsets.symmetric(horizontal: 8),

                decoration: BoxDecoration(

                  color: Colors.grey.shade200,

                  borderRadius: BorderRadius.circular(50), // pill shape

                ),

                child: Row(

                  children: [

                    const Icon(Icons.search, color: Colors.grey),

                    const SizedBox(width: 5),

                    Expanded(

                      child: TextField(

                        textAlignVertical: TextAlignVertical.center,

                        decoration: const InputDecoration(

                          hintText: 'Search...',

                          border: InputBorder.none,

                          contentPadding: EdgeInsets.symmetric(vertical: 10),

                        ),

                        onChanged: (value) {

                          // Implement search logic if needed

                        },

                      ),

                    ),

                  ],

                ),

              ),

            ),

            // Expanded area for the chats list

            Expanded(

              child: ListView.separated(

                itemCount: chats.length,

                separatorBuilder: (context, index) => Divider(

                  color: AppColors.dividerLine,

                  thickness: 2,

                  height: 2,

                ),

                itemBuilder: (context, index) {

                  final chat = chats[index];

                  return InkWell(

                    onTap: () {

                      Navigator.push(

                        context,

                        MaterialPageRoute(

                          builder: (context) => ChatPage(

                            chatId: chat["chat_id"],

                            profileURL: chat["profileURL"],

                            receiver: chat["receiver"],

                          ),

                        ),

                      );

                    },

                    child: Padding(

                      padding: const EdgeInsets.symmetric(

                        horizontal: 16,

                        vertical: 12,

                      ),

                      child: Row(

                        children: [

                          // Circle Avatar on the left

                          CircleAvatar(

                            backgroundImage: NetworkImage(chat["profileURL"]),

                            radius: 24,

                          ),

                          const SizedBox(width: 16),

  

                          // Name and last message/status in the middle

                          Expanded(

                            child: Column(

                              crossAxisAlignment: CrossAxisAlignment.start,

                              children: [

                                Text(

                                  chat["receiver"] ?? "Name",

                                  style: const TextStyle(

                                    fontWeight: FontWeight.bold,

                                    fontSize: 16,

                                  ),

                                ),

                                const SizedBox(height: 4),

                                Text(

                                  chat["last_message"]?.toString() ??

                                      "zb online oder zu gesehen",

                                  style: TextStyle(

                                    fontSize: 14,

                                    color: Colors.grey.shade600,

                                  ),

                                ),

                              ],

                            ),

                          ),

  

                          // Unread count badge on the right (shown only if unread > 0)

                          if ((chat["unread"] ?? 0) > 0)

                            Container(

                              width: 28,

                              height: 28,

                              decoration: BoxDecoration(

                                shape: BoxShape.circle,

                                gradient: const LinearGradient(

                                  colors: [

                                    Colors.pinkAccent,

                                    Colors.orangeAccent

                                  ],

                                  begin: Alignment.topLeft,

                                  end: Alignment.bottomRight,

                                ),

                              ),

                              alignment: Alignment.center,

                              child: Text(

                                chat["unread"].toString(),

                                style: const TextStyle(

                                  color: Colors.white,

                                  fontWeight: FontWeight.bold,

                                  fontSize: 12,

                                ),

                              ),

                            ),

                        ],

                      ),

                    ),

                  );

                },

              ),

            ),

          ],

        ),

      ),

    );

  }

}
```
- tabs/chat/search_user.dart
```
import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';

import 'dart:convert';

  

import 'layouts/chat.dart';

  

class SearchUserPage extends StatefulWidget {

  const SearchUserPage({super.key});

  

  @override

  _SearchUserPageState createState() => _SearchUserPageState();

}

  

class _SearchUserPageState extends State<SearchUserPage> {

  final TextEditingController _searchController = TextEditingController();

  final _secureStorage = FlutterSecureStorage();

  List<dynamic> searchResults = [];

  

  Future<void> searchUsers(String query) async {

    final String? token = await _secureStorage.read(key: 'jwt_token');

  

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Authorization token is missing.")),

      );

      return;

    }

  

    final response = await http.post(

      Uri.parse('${Config.backendBaseUrl}/chat/search_user.php'),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

      body: jsonEncode({"search": query}),

    );

  

    if (response.statusCode == 200) {

      setState(() {

        searchResults = jsonDecode(response.body);

      });

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(

            content: Text("Failed to fetch results: ${response.reasonPhrase}")),

      );

    }

  }

  

  Future<void> createChat(

      String receiverUsername, String profilePicture) async {

    final String? token = await _secureStorage.read(key: 'jwt_token');

  

    if (token == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Authorization token is missing.")),

      );

      return;

    }

  

    final response = await http.post(

      Uri.parse("${Config.backendBaseUrl}/chat/create_chat.php"),

      headers: {

        "Authorization": "Bearer $token",

        "Content-Type": "application/json",

      },

      body: jsonEncode({"receiver_username": receiverUsername}),

    );

  

    if (response.statusCode == 200) {

      final data = jsonDecode(response.body);

      if (data["status"] == "success") {

        Navigator.push(

          context,

          MaterialPageRoute(

            builder: (context) => ChatPage(

              chatId: data["chat_id"],

              profileURL: profilePicture,

              receiver: receiverUsername,

            ),

          ),

        );

      } else {

        ScaffoldMessenger.of(context).showSnackBar(

          SnackBar(content: Text("Failed to create chat: ${data["message"]}")),

        );

      }

    } else {

      ScaffoldMessenger.of(context).showSnackBar(

        SnackBar(content: Text("Error: ${response.reasonPhrase}")),

      );

    }

  }

  

  @override

  void initState() {

    super.initState();

    Future.delayed(Duration.zero, () {

      FocusScope.of(context).requestFocus(FocusNode());

    });

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: Text("Search Users")),

      body: Column(

        children: [

          Padding(

            padding: const EdgeInsets.all(8.0),

            child: TextField(

              controller: _searchController,

              autofocus: true,

              onChanged: (value) {

                if (value.isNotEmpty) {

                  searchUsers(value);

                } else {

                  setState(() {

                    searchResults = [];

                  });

                }

              },

              decoration: InputDecoration(

                labelText: "Search by username",

                border: OutlineInputBorder(),

                suffixIcon: IconButton(

                  onPressed: () => _searchController.clear(),

                  icon: Icon(Icons.clear),

                ),

              ),

            ),

          ),

          Expanded(

            child: ListView.builder(

              itemCount: searchResults.length,

              itemBuilder: (context, index) {

                final user = searchResults[index];

                return ListTile(

                  leading: CircleAvatar(

                    backgroundImage: user["profile_picture"] != null

                        ? NetworkImage(user["profile_picture"])

                        : null,

                    child: user["profile_picture"] == null

                        ? Icon(Icons.person)

                        : null,

                  ),

                  title: Text(user["nickname"]),

                  onTap: () async {

                    await createChat(

                      user["nickname"],

                      user["profile_picture"],

                    );

                  },

                  trailing: user["profile_picture"] == null

                      ? Icon(Icons.person)

                      : null,

                );

              },

            ),

          ),

        ],

      ),

    );

  }

}
```
- tabs/chat/layouts/attach_file_layout.dart
```
import 'dart:io';

  

import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'grid_of_cards_layout.dart';

import 'opening_messages_layout.dart';

import 'send_location_layout.dart';

import 'send_images_layout.dart';

  

class AttachFileLayout extends StatefulWidget {

  final void Function(double lat, double lng)? onLocationSelected;

  final void Function(List<File>)? onImagesSelected;

  

  const AttachFileLayout({

    Key? key,

    this.onLocationSelected,

    this.onImagesSelected,

  }) : super(key: key);

  

  @override

  _AttachFileLayoutState createState() => _AttachFileLayoutState();

}

  

class _AttachFileLayoutState extends State<AttachFileLayout> {

  int _selectedTabIndex = 0;

  late PageController _pageController;

  

  @override

  void initState() {

    super.initState();

    _pageController = PageController(initialPage: _selectedTabIndex);

  }

  

  @override

  void dispose() {

    _pageController.dispose();

    super.dispose();

  }

  

  Widget _buildTabItem({

    required String iconPath,

    required int index,

  }) {

    final bool isSelected = _selectedTabIndex == index;

    return Expanded(

      child: Container(

        decoration: BoxDecoration(

          color: isSelected ? Colors.grey[200] : Colors.transparent,

          borderRadius: BorderRadius.circular(8),

        ),

        child: IconButton(

          icon: SvgPicture.asset(iconPath, width: 24, height: 24),

          onPressed: () {

            setState(() => _selectedTabIndex = index);

            _pageController.animateToPage(

              index,

              duration: const Duration(milliseconds: 300),

              curve: Curves.easeOut,

            );

          },

        ),

      ),

    );

  }

  

  @override

  Widget build(BuildContext context) {

    return Container(

      margin: const EdgeInsets.symmetric(horizontal: 8),

      decoration: BoxDecoration(

        color: Colors.white,

        borderRadius: BorderRadius.circular(16),

        boxShadow: const [

          BoxShadow(

            color: Colors.black12,

            blurRadius: 4,

            offset: Offset(0, 2),

          ),

        ],

      ),

      padding: const EdgeInsets.all(16.0),

      child: Column(

        children: [

          // Tabs row

          Row(

            children: [

              _buildTabItem(

                iconPath: 'assets/icons/game.svg',

                index: 0,

              ),

              SizedBox(

                height: 32,

                child: VerticalDivider(

                  thickness: 1,

                  color: Colors.grey.shade300,

                ),

              ),

              _buildTabItem(

                iconPath: 'assets/icons/icebreaker.svg',

                index: 1,

              ),

              SizedBox(

                height: 32,

                child: VerticalDivider(

                  thickness: 1,

                  color: Colors.grey.shade300,

                ),

              ),

              _buildTabItem(

                iconPath: 'assets/icons/location.svg',

                index: 2,

              ),

              SizedBox(

                height: 32,

                child: VerticalDivider(

                  thickness: 1,

                  color: Colors.grey.shade300,

                ),

              ),

              _buildTabItem(

                iconPath: 'assets/icons/image.svg',

                index: 3,

              ),

            ],

          ),

          const SizedBox(height: 16),

          // PageView

          Expanded(

            child: PageView(

              controller: _pageController,

              onPageChanged: (index) {

                setState(() => _selectedTabIndex = index);

              },

              children: [

                const GridOfCardsLayout(),

                const OpeningMessagesLayout(),

                SendLocationLayout(

                  onLocationSend: (lat, lng) {

                    if (widget.onLocationSelected != null) {

                      widget.onLocationSelected!(lat, lng);

                    }

                  },

                ),

                SendImagesLayout(

                  onImagesSelected: (files) {

                    if (widget.onImagesSelected != null) {

                      widget.onImagesSelected!(files);

                    }

                  },

                ),

              ],

            ),

          ),

        ],

      ),

    );

  }

}
```

- tabs/chat/layouts/chat.dart
```
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

import 'package:swipe_chat_play/tabs/chat/new_bubbles/new_chatbubble_layouts/speech_bubble_layout.dart';

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

  //  BOTTOM SHEET: OPENS at 80% => DRAGS UP to 95% => content scrolls

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
```

- tabs/chat/layouts/chat_popup.dart
```
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

  /// Example payload coming **into** the popup:

  /// {

  ///   "type"       : "react_bubble" | "react_top_section_bubble"

  ///                    | "react_icebreaker" | "react_profile_image",

  ///   "content"    : { … },               // element‑specific data

  ///   "receiver_id": "abc123"             // profile owner

  /// }

  final Map<String, dynamic> reactionPayload;

  

  const ChatPopupDialog({Key? key, required this.reactionPayload})

      : super(key: key);

  

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

  //  UI PARTS

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

  

  //---------------------------------------------------------------------------

  //  NETWORK PART

  //---------------------------------------------------------------------------

  

  /// POSTs a reaction to `match_algorithm/sendReaction.php`

  Future<void> _sendReactionMessage() async {

    final comment = _commentController.text.trim();

    if (comment.isEmpty) return;

  

    setState(() => _isSending = true);

  

    try {

      final token = await storage.read(key: 'jwt_token');

      if (token == null) throw Exception("Missing JWT token");

  

      // ---------- NORMALISE the type the backend will see ------------------

      final rawType = widget.reactionPayload['type'];

      final apiType =

          rawType == 'react_top_section_bubble' ? 'react_bubble' : rawType;

  

      final requestBody = {

        "comment": comment,

        "react_to": {

          "type": apiType, // <-- always react_bubble now

          "content": widget.reactionPayload['content'],

        },

        "receiver_id": widget.reactionPayload['receiver_id'],

      };

      debugPrint("→ sendReaction.php payload: $requestBody");

  

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

        if (mounted) Navigator.of(context).pop(); // close popup

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

        if (mounted) Navigator.of(context).pop("NO_MATCH");

      }

  

      _commentController.clear();

    } catch (e, st) {

      debugPrint("sendReaction error: $e\n$st");

      if (mounted) {

        ScaffoldMessenger.of(context).showSnackBar(

          const SnackBar(

              content: Text("Couldn’t send reaction. Please try again.")),

        );

      }

    } finally {

      if (mounted) setState(() => _isSending = false);

    }

  }

}
```

- tabs/profile/edit_profile.dart
```
// lib/tabs/profile/edit_profile_layout.dart

  

import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:latlong2/latlong.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

  

import 'package:swipe_chat_play/config.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile_activities_page.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile_filters_page.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile_photos_page.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile_qa_page.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile_radius_page.dart';

  

class EditProfileLayout extends StatefulWidget {

  const EditProfileLayout({Key? key}) : super(key: key);

  

  @override

  State<EditProfileLayout> createState() => _EditProfileLayoutState();

}

  

class _EditProfileLayoutState extends State<EditProfileLayout> {

  // ─────────────────────────────────────────────────────────────────

  // simple text

  String? name, age, gender, spoken, religion, career, politics;

  

  // complex

  List<String?> photos = List<String?>.filled(9, null);

  double? radiusKm;

  LatLng? radiusPos;

  List<Map<String, String>>? qa;

  Set<String>? activities;

  Map<String, dynamic>? filters;

  

  // helpers

  final _storage = const FlutterSecureStorage();

  

  // ─────────────────────────────────────────────────────────────────

  // ❶ LOAD on open

  // ─────────────────────────────────────────────────────────────────

  @override

  void initState() {

    super.initState();

    _loadProfile();

  }

  

  Future<void> _loadProfile() async {

    final token = await _storage.read(key: 'jwt_token');

    if (token == null) return _snack('Please login first');

  

    final res = await http.get(

      Uri.parse('${Config.backendBaseUrl}/create_profile/get_profile.php'),

      headers: {'Authorization': 'Bearer $token'},

    );

    if (res.statusCode != 200) {

      return _snack('Load failed: HTTP ${res.statusCode}');

    }

  

    final root = json.decode(res.body) as Map<String, dynamic>;

  

    // topSection

    final top = root['topSection'] as Map<String, dynamic>;

    name = top['name']?.toString();

    age = top['age']?.toString();

    final bubbles = List<String>.from(top['infoBubbles'] ?? []);

    gender = bubbles.length > 0 ? bubbles[0] : null;

    spoken = bubbles.length > 1 ? bubbles[1] : null;

    religion = bubbles.length > 2 ? bubbles[2] : null;

    politics = bubbles.length > 3 ? bubbles[3] : null;

  

    // images

    final urls = List<String>.from(top['profileImages'] ?? []);

    photos = List<String?>.filled(9, null);

    for (int i = 0; i < urls.length && i < 9; i++) {

      photos[i] = urls[i];

    }

  

    // elements

    qa = [];

    activities = {};

    final elems = root['elements'] as Map<String, dynamic>?;

    if (elems != null) {

      for (final e in elems.values) {

        final type = e['type'];

        final c = e['content'] as Map<String, dynamic>;

        if (type == 'icebreaker') {

          qa!.add({

            'question': '${c['question']}',

            'answer': '${c['answer']}',

          });

        } else if (type == 'bubbles') {

          activities = Set<String>.from(c['bubbles'] ?? []);

        }

      }

    }

  

    setState(() {});

  }

  

  // ─────────────────────────────────────────────────────────────────

  // ❷ PARTIAL SAVE on each change

  // ─────────────────────────────────────────────────────────────────

  Future<void> _saveField(Map<String, dynamic> update) async {

    final token = await _storage.read(key: 'jwt_token');

    if (token == null) return _snack('Please login first');

  

    try {

      final res = await http.post(

        Uri.parse(

            '${Config.backendBaseUrl}/create_profile/update_userdata.php'),

        headers: {

          'Authorization': 'Bearer $token',

          'Content-Type': 'application/json',

        },

        body: json.encode(update),

      );

      final body = json.decode(res.body) as Map<String, dynamic>;

      if (res.statusCode == 200 && body['success'] == true) {

        // optional: _snack(body['message'] ?? 'Saved');

      } else {

        _snack('Save error: ${body['error'] ?? body['message']}');

      }

    } catch (e) {

      _snack('Save failed: $e');

    }

  }

  

  // ─────────────────────────────────────────────────────────────────

  // helpers for inline editing

  // ─────────────────────────────────────────────────────────────────

  void _snack(String m) =>

      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(m)));

  

  Future<void> _editTextField({

    required String title,

    required String? initial,

    required String fieldName,

    required String subKey, // e.g. 'name','age','gender',...

    TextInputType type = TextInputType.text,

  }) async {

    final c = TextEditingController(text: initial);

    await showModalBottomSheet(

      context: context,

      isScrollControlled: true,

      builder: (ctx) => Padding(

        padding: EdgeInsets.only(

          bottom: MediaQuery.of(ctx).viewInsets.bottom,

          left: 16,

          right: 16,

          top: 16,

        ),

        child: Column(mainAxisSize: MainAxisSize.min, children: [

          Text(title, style: const TextStyle(fontSize: 18)),

          TextField(controller: c, keyboardType: type, autofocus: true),

          const SizedBox(height: 12),

          ElevatedButton(

            onPressed: () {

              final v = c.text.trim();

              // update local state

              setState(() {

                switch (subKey) {

                  case 'name':

                    name = v;

                    break;

                  case 'age':

                    age = v;

                    break;

                  case 'gender':

                    gender = v;

                    break;

                  case 'spokenLanguage':

                    spoken = v;

                    break;

                  case 'religion':

                    religion = v;

                    break;

                  case 'career':

                    career = v;

                    break;

                  case 'politics':

                    politics = v;

                    break;

                }

              });

              // save just that field

              _saveField({

                'about_you': {subKey: (subKey == 'age' ? int.tryParse(v) : v)}

              });

              Navigator.pop(ctx);

            },

            child: const Text('Save'),

          ),

          const SizedBox(height: 8),

        ]),

      ),

    );

  }

  

  Widget _row(String title, String subtitle, VoidCallback onTap,

          {bool divider = true}) =>

      Material(

        color: Colors.transparent,

        child: InkWell(

          onTap: onTap,

          child: Column(children: [

            Padding(

              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 16),

              child: Row(

                mainAxisAlignment: MainAxisAlignment.spaceBetween,

                children: [

                  Expanded(

                      child: Column(

                    crossAxisAlignment: CrossAxisAlignment.start,

                    children: [

                      Text(title,

                          style: const TextStyle(

                              fontSize: 16, fontWeight: FontWeight.w600)),

                      const SizedBox(height: 4),

                      Text(subtitle,

                          style: const TextStyle(

                              fontSize: 14, color: Colors.grey)),

                    ],

                  )),

                  const Icon(Icons.chevron_right, color: Colors.grey),

                ],

              ),

            ),

            if (divider)

              const Divider(height: 1, thickness: .5, color: Colors.grey),

          ]),

        ),

      );

  

  // ───────────────── navigation to each flow ────────────────────────

  Future<void> _gotoPhotos() async {

    final res = await Navigator.push<List<String?>>(

      context,

      MaterialPageRoute(builder: (_) => EditPhotosPage(initial: photos)),

    );

    if (res != null) {

      setState(() => photos = res);

      _saveField({'images': photos.whereType<String>().toList()});

    }

  }

  

  Future<void> _gotoRadius() async {

    final res = await Navigator.push<Map<String, dynamic>>(

      context,

      MaterialPageRoute(

        builder: (_) => EditRadiusPage(

          initialRadius: radiusKm,

          initialPos: radiusPos,

        ),

      ),

    );

    if (res != null) {

      setState(() {

        radiusKm = (res['radius'] as num?)?.toDouble();

        radiusPos = res['location'] as LatLng?;

      });

      _saveField({

        'location_radius': {

          'location': radiusPos == null

              ? null

              : {

                  'type': 'Point',

                  'coordinates': [radiusPos!.longitude, radiusPos!.latitude],

                },

          'radius': radiusKm,

        }

      });

    }

  }

  

  Future<void> _gotoQA() async {

    final res = await Navigator.push<List<Map<String, String>>>(

      context,

      MaterialPageRoute(builder: (_) => EditQAPage(initial: qa)),

    );

    if (res != null) {

      setState(() => qa = res);

      _saveField({'questions_and_answers': qa});

    }

  }

  

  Future<void> _gotoActivities() async {

    final res = await Navigator.push<Set<String>>(

      context,

      MaterialPageRoute(

          builder: (_) => EditActivitiesPage(initial: activities)),

    );

    if (res != null) {

      setState(() => activities = res);

      _saveField({'open_to': activities?.toList()});

    }

  }

  

  Future<void> _gotoFilters() async {

    final res = await Navigator.push<Map<String, dynamic>>(

      context,

      MaterialPageRoute(builder: (_) => EditFiltersPage(initial: filters)),

    );

    if (res != null) {

      setState(() => filters = res);

      _saveField({'searching_for': filters});

    }

  }

  

  // ───────────────── subtitle helpers ───────────────────────────────

  String _photoSub() => photos.where((e) => e != null).isEmpty

      ? 'Add photos'

      : '${photos.where((e) => e != null).length}/9 selected';

  String _radiusSub() =>

      radiusKm == null ? 'Set radius & location' : '${radiusKm!.round()} km';

  String _qaSub() =>

      qa == null ? 'Answer 3 questions' : '${qa!.length} answered';

  String _actSub() =>

      activities == null ? 'Choose activities' : '${activities!.length} chosen';

  String _filtSub() => filters == null

      ? 'Configure filters'

      : (filters!['genders'] as List?)?.join(', ') ?? 'custom';

  

  // ─────────────────────────────────────────────────────────────────

  // UI

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Edit profile')),

      body: ListView(children: [

        const SizedBox(height: 16),

        _row(

            'Name',

            name ?? 'Add',

            () => _editTextField(

                  title: 'Name',

                  initial: name,

                  fieldName: 'about_you',

                  subKey: 'name',

                )),

        _row(

            'Age',

            age ?? 'Set',

            () => _editTextField(

                  title: 'Age',

                  initial: age,

                  fieldName: 'about_you',

                  subKey: 'age',

                  type: TextInputType.number,

                )),

        _row(

            'Gender',

            gender ?? 'Choose',

            () => _editTextField(

                  title: 'Gender',

                  initial: gender,

                  fieldName: 'about_you',

                  subKey: 'gender',

                )),

        _row(

            'Languages',

            spoken ?? 'Add',

            () => _editTextField(

                  title: 'Languages',

                  initial: spoken,

                  fieldName: 'about_you',

                  subKey: 'spokenLanguage',

                )),

        _row(

            'Religion',

            religion ?? 'Choose',

            () => _editTextField(

                  title: 'Religion',

                  initial: religion,

                  fieldName: 'about_you',

                  subKey: 'religion',

                )),

        _row(

            'Career',

            career ?? 'Add',

            () => _editTextField(

                  title: 'Career',

                  initial: career,

                  fieldName: 'about_you',

                  subKey: 'career',

                )),

        _row(

            'Politics',

            politics ?? 'Choose',

            () => _editTextField(

                  title: 'Politics',

                  initial: politics,

                  fieldName: 'about_you',

                  subKey: 'politics',

                )),

        const SizedBox(height: 12),

        _row('Profile pictures', _photoSub(), _gotoPhotos),

        _row('Dating radius', _radiusSub(), _gotoRadius),

        _row('Q & A', _qaSub(), _gotoQA),

        _row('Open to', _actSub(), _gotoActivities),

        _row('Search filters', _filtSub(), _gotoFilters, divider: false),

        const SizedBox(height: 24),

      ]),

    );

  }

}
```

- tabs/profile/edit_profile_activities_page.dart
```
import 'dart:convert';

import 'package:flutter/material.dart';

  

/// A stand-alone “Open to” activities page, refactored for the edit profile flow.

/// Returns the Set<String> of selected activities on Save.

class EditActivitiesPage extends StatefulWidget {

  /// Initial selection passed from EditProfileLayout

  final Set<String>? initial;

  const EditActivitiesPage({Key? key, this.initial}) : super(key: key);

  

  @override

  State<EditActivitiesPage> createState() => _EditActivitiesPageState();

}

  

class _EditActivitiesPageState extends State<EditActivitiesPage> {

  // Static JSON-driven data, matching original OpenToPage

  final String _json = '''

  {

    "title": "Open to",

    "subtitle": "What activities are you open to exploring together?",

    "sections": [

      {

        "categoryName": "Outdoor",

        "keywords": [

          "jogging", "hiking", "biking", "kayaking", "rock climbing", "camping", "birdwatching"

        ],

        "moreKeywords": [

          "stand-up paddleboarding", "fishing", "snowshoeing", "surfing", "horseback riding", "trail running", "scuba diving", "whitewater rafting", "beach volleyball", "zip-lining"

        ],

        "hasShowMore": true

      },

      {

        "categoryName": "Indoor Activities",

        "keywords": [

          "cooking", "yoga", "games", "movies", "Arts & Crafts", "puzzles", "board games"

        ],

        "moreKeywords": [

          "dance classes", "homebrewing", "woodworking", "pottery", "karaoke", "painting", "knitting", "mixology", "card games", "language exchange"

        ],

        "hasShowMore": true

      }

    ]

  }

  ''';

  

  late final String _title;

  late final String _subtitle;

  late final List<dynamic> _sections;

  final Map<String, bool> _expanded = {};

  late Set<String> _selected;

  

  @override

  void initState() {

    super.initState();

    final data = json.decode(_json) as Map<String, dynamic>;

    _title = data['title'] as String;

    _subtitle = data['subtitle'] as String;

    _sections = data['sections'] as List<dynamic>;

    for (var sec in _sections) {

      _expanded[sec['categoryName'] as String] = false;

    }

    _selected = Set<String>.from(widget.initial ?? {});

  }

  

  void _toggle(String keyword) {

    setState(() {

      if (_selected.contains(keyword)) {

        _selected.remove(keyword);

      } else {

        _selected.add(keyword);

      }

    });

  }

  

  Widget _buildChip(String keyword) {

    final isSel = _selected.contains(keyword);

    return ChoiceChip(

      label: Text(keyword),

      selected: isSel,

      onSelected: (_) => _toggle(keyword),

    );

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(

        title: Text(_title),

        actions: [

          TextButton(

            onPressed: () => Navigator.pop(context, _selected),

            child: const Text('Save', style: TextStyle(color: Colors.white)),

          )

        ],

      ),

      body: SingleChildScrollView(

        padding: const EdgeInsets.all(16),

        child: Column(

          crossAxisAlignment: CrossAxisAlignment.start,

          children: [

            Text(_subtitle, style: Theme.of(context).textTheme.bodyMedium),

            const SizedBox(height: 16),

            for (var sec in _sections) ...[

              Text(

                sec['categoryName'] as String,

                style:

                    const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),

              ),

              const SizedBox(height: 8),

              Wrap(

                spacing: 8,

                runSpacing: 8,

                children: [

                  ...List<String>.from(sec['keywords'] as List<dynamic>)

                      .map(_buildChip),

                  if (_expanded[sec['categoryName'] as String]!)

                    ...List<String>.from(sec['moreKeywords'] as List<dynamic>)

                        .map(_buildChip),

                ],

              ),

              if (sec['hasShowMore'] as bool)

                Align(

                  alignment: Alignment.centerLeft,

                  child: TextButton(

                    onPressed: () => setState(() {

                      final key = sec['categoryName'] as String;

                      _expanded[key] = !_expanded[key]!;

                    }),

                    child: Text(

                      _expanded[sec['categoryName'] as String]!

                          ? 'show less …'

                          : 'show more …',

                    ),

                  ),

                ),

              const SizedBox(height: 16),

            ],

          ],

        ),

      ),

    );

  }

}
```

- tabs/profile/edit_profile_filters_page.dart
```
import 'package:flutter/material.dart';

  

/// A stand-alone “Search filters” page for editing in the profile flow.

/// Returns a Map<String, dynamic> with selected genders, religions, politics, and ageRange.

class EditFiltersPage extends StatefulWidget {

  /// Initial filter values passed from EditProfileLayout

  final Map<String, dynamic>? initial;

  const EditFiltersPage({Key? key, this.initial}) : super(key: key);

  

  @override

  State<EditFiltersPage> createState() => _EditFiltersPageState();

}

  

class _EditFiltersPageState extends State<EditFiltersPage> {

  // Options

  final List<String> _genders = ['Male', 'Female', 'Non-binary'];

  final List<String> _religions = [

    'Christianity',

    'Islam',

    'Hinduism',

    'Buddhism',

    'Judaism',

    'Other'

  ];

  final List<String> _politics = [

    'Liberal',

    'Conservative',

    'Centrist',

    'Libertarian',

    'Socialist',

    'Other'

  ];

  

  // Selections

  late Set<String> _selectedGenders;

  late Set<String> _selectedReligions;

  late Set<String> _selectedPolitics;

  RangeValues _ageRange = const RangeValues(18, 120);

  

  @override

  void initState() {

    super.initState();

    // Initialize from `initial` or default

    _selectedGenders = Set<String>.from(

      widget.initial?['genders'] as List<String>? ?? _genders,

    );

    _selectedReligions = Set<String>.from(

      widget.initial?['religion'] as List<String>? ?? _religions,

    );

    _selectedPolitics = Set<String>.from(

      widget.initial?['politics'] as List<String>? ?? _politics,

    );

    if (widget.initial?['ageRange'] != null) {

      final ar = widget.initial!['ageRange'] as List<dynamic>;

      _ageRange = RangeValues(

        (ar[0] as num).toDouble(),

        (ar[1] as num).toDouble(),

      );

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(

        title: const Text('Search filters'),

        actions: [

          TextButton(

            onPressed: () => Navigator.pop(context, {

              'genders': _selectedGenders.toList(),

              'religion': _selectedReligions.toList(),

              'politics': _selectedPolitics.toList(),

              'ageRange': [

                _ageRange.start.round(),

                _ageRange.end.round(),

              ],

            }),

            child: const Text('Save', style: TextStyle(color: Colors.white)),

          ),

        ],

      ),

      body: Padding(

        padding: const EdgeInsets.all(16),

        child: Column(

          crossAxisAlignment: CrossAxisAlignment.start,

          children: [

            const Text('Gender', style: TextStyle(fontWeight: FontWeight.bold)),

            const SizedBox(height: 8),

            Wrap(

              spacing: 8,

              children: _genders.map(_buildGenderChip).toList(),

            ),

            const SizedBox(height: 24),

            const Text('Religion',

                style: TextStyle(fontWeight: FontWeight.bold)),

            const SizedBox(height: 8),

            Wrap(

              spacing: 8,

              children: _religions.map(_buildReligionChip).toList(),

            ),

            const SizedBox(height: 24),

            const Text('Politics',

                style: TextStyle(fontWeight: FontWeight.bold)),

            const SizedBox(height: 8),

            Wrap(

              spacing: 8,

              children: _politics.map(_buildPoliticsChip).toList(),

            ),

            const SizedBox(height: 24),

            const Text('Age range',

                style: TextStyle(fontWeight: FontWeight.bold)),

            RangeSlider(

              min: 18,

              max: 120,

              divisions: 102,

              labels: RangeLabels(

                _ageRange.start.round().toString(),

                _ageRange.end.round().toString(),

              ),

              values: _ageRange,

              onChanged: (values) => setState(() => _ageRange = values),

            ),

            Center(

              child: Text(

                '${_ageRange.start.round()} - ${_ageRange.end.round()} years',

                style:

                    const TextStyle(fontSize: 14, fontWeight: FontWeight.w500),

              ),

            ),

          ],

        ),

      ),

    );

  }

  

  Widget _buildGenderChip(String gender) {

    final isSelected = _selectedGenders.contains(gender);

    return FilterChip(

      label: Text(gender),

      selected: isSelected,

      onSelected: (_) {

        setState(() {

          isSelected

              ? _selectedGenders.remove(gender)

              : _selectedGenders.add(gender);

        });

      },

    );

  }

  

  Widget _buildReligionChip(String rel) {

    final isSelected = _selectedReligions.contains(rel);

    return FilterChip(

      label: Text(rel),

      selected: isSelected,

      onSelected: (_) {

        setState(() {

          isSelected

              ? _selectedReligions.remove(rel)

              : _selectedReligions.add(rel);

        });

      },

    );

  }

  

  Widget _buildPoliticsChip(String pol) {

    final isSelected = _selectedPolitics.contains(pol);

    return FilterChip(

      label: Text(pol),

      selected: isSelected,

      onSelected: (_) {

        setState(() {

          isSelected

              ? _selectedPolitics.remove(pol)

              : _selectedPolitics.add(pol);

        });

      },

    );

  }

}
```

- tabs/profile/edit_profile_photos_page.dart
```
// lib/tabs/profile/edit_profile_photos_page.dart

import 'dart:convert';

import 'dart:io';

  

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:http/http.dart' as http;

import 'package:image_cropper/image_cropper.dart';

import 'package:image_picker/image_picker.dart';

import 'package:reorderables/reorderables.dart';

  

import 'package:swipe_chat_play/config.dart';

  

class EditPhotosPage extends StatefulWidget {

  final List<String?> initial;

  const EditPhotosPage({Key? key, required this.initial}) : super(key: key);

  

  @override

  State<EditPhotosPage> createState() => _EditPhotosPageState();

}

  

class _EditPhotosPageState extends State<EditPhotosPage> {

  static const int _maxItems = 9;

  

  late List<String?> _photos; // urls OR local paths

  final _picker = ImagePicker();

  final _secureStorage = const FlutterSecureStorage();

  

  @override

  void initState() {

    super.initState();

    _photos = List<String?>.from(widget.initial);

    if (_photos.length < _maxItems) {

      _photos.addAll(List.filled(_maxItems - _photos.length, null));

    }

  }

  

  /* ───────────────────────── image pick / crop ─────────────────── */

  

  Future<void> _pick(int idx) async {

    final src = await showModalBottomSheet<ImageSource>(

      context: context,

      builder: (ctx) => SafeArea(

        child: Column(mainAxisSize: MainAxisSize.min, children: [

          ListTile(

            leading: const Icon(Icons.camera_alt),

            title: const Text('Take photo'),

            onTap: () => Navigator.pop(ctx, ImageSource.camera),

          ),

          ListTile(

            leading: const Icon(Icons.photo_library),

            title: const Text('Choose from gallery'),

            onTap: () => Navigator.pop(ctx, ImageSource.gallery),

          ),

        ]),

      ),

    );

    if (src == null) return;

  

    final x = await _picker.pickImage(source: src);

    if (x == null) return;

  

    final cropped = await Navigator.push<File?>(

      context,

      MaterialPageRoute(builder: (_) => _CropPage(file: File(x.path))),

    );

    if (cropped != null) {

      setState(() => _photos[idx] = cropped.path); // local path

    }

  }

  

  /* ───────────────────────── confirm delete ────────────────────── */

  

  Future<void> _confirmDelete(int idx) async {

    final ok = await showDialog<bool>(

      context: context,

      builder: (ctx) => AlertDialog(

        title: const Text('Delete photo?'),

        content: const Text('This picture will be removed from your profile.'),

        actions: [

          TextButton(

              onPressed: () => Navigator.pop(ctx, false),

              child: const Text('Cancel')),

          TextButton(

              onPressed: () => Navigator.pop(ctx, true),

              child: const Text('Delete', style: TextStyle(color: Colors.red))),

        ],

      ),

    );

    if (ok == true) setState(() => _photos[idx] = null);

  }

  

  /* ───────────────────────── re-order ──────────────────────────── */

  

  void _reorder(int oldIdx, int newIdx) {

    setState(() {

      if (oldIdx < newIdx) newIdx--;

      final tmp = _photos.removeAt(oldIdx);

      _photos.insert(newIdx, tmp);

    });

  }

  

  /* ───────────────────────── UPLOAD & CLOSE ────────────────────── */

  

  Future<void> _uploadAndPop() async {

    final token = await _secureStorage.read(key: 'jwt_token');

    if (token == null) {

      _snack('Login required');

      return;

    }

  

    // split into already-hosted URLs vs. local files

    final existingUrls = <String>[];

    final List<MapEntry<int, String>> localPaths = [];

  

    for (var i = 0; i < _photos.length; i++) {

      final p = _photos[i];

      if (p == null) continue;

      if (p.startsWith('http')) {

        existingUrls.add(p);

      } else {

        localPaths.add(MapEntry(i, p));

      }

    }

  

    final uri = Uri.parse(

        '${Config.backendBaseUrl}/create_profile/upload_profile_images.php');

    final req = http.MultipartRequest('POST', uri)

      ..headers['Authorization'] = 'Bearer $token'

      ..fields['images'] = json.encode(existingUrls);

  

    for (final entry in localPaths) {

      final file = File(entry.value);

      final fileName = file.path.split('/').last;

      req.files.add(await http.MultipartFile.fromPath(

          'image${entry.key}', file.path,

          filename: fileName));

    }

  

    try {

      final streamed = await req.send();

      final resp = await http.Response.fromStream(streamed);

  

      if (resp.statusCode != 200) {

        throw 'Status ${resp.statusCode}';

      }

  

      final data = json.decode(resp.body);

      if (data['success'] != true || data['images'] == null) {

        throw data['error'] ?? 'Unknown error';

      }

  

      final List<dynamic> urls = data['images'];

      // fill back into list keeping null placeholders

      var urlIdx = 0;

      for (var i = 0; i < _photos.length && urlIdx < urls.length; i++) {

        if (_photos[i] != null) {

          _photos[i] = urls[urlIdx++] as String;

        }

      }

  

      Navigator.pop(context, _photos);

    } catch (e) {

      _snack('Upload failed: $e');

    }

  }

  

  /* ───────────────────────── UI ────────────────────────────────── */

  

  @override

  Widget build(BuildContext context) {

    final boxW = MediaQuery.of(context).size.width / 3 - 24;

  

    return Scaffold(

      body: Stack(children: [

        Padding(

          padding: const EdgeInsets.all(16),

          child: Column(children: [

            const SizedBox(height: 24),

            const Text('Show yourself',

                style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),

            const SizedBox(height: 8),

            const Text("Don't worry—you will be judged but won't see it ❤️",

                style: TextStyle(fontSize: 16)),

            const SizedBox(height: 24),

            Expanded(

              child: ReorderableWrap(

                spacing: 16,

                runSpacing: 16,

                onReorder: _reorder,
                children: List.generate(
                  _maxItems,
                  (i) => _tile(i, boxW),
                ),
              ),
            ),
            const SizedBox(height: 16),

            const Center(

              child: Text('min 3 hinzufügen',

                  style: TextStyle(fontSize: 16, color: Color(0xFF7D7D7D))),

            ),

          ]),

        ),

        Positioned(

          bottom: 20,

          right: 30,

          child: IconButton(

            icon: SvgPicture.asset('assets/icons/arrow_icon.svg',

                width: 50, height: 50),

            onPressed: _uploadAndPop,

          ),

        ),

      ]),

    );

  }

  

  /* ───────────────────────── single grid tile ──────────────────── */

  

  Widget _tile(int idx, double side) {

    final src = _photos[idx];

  

    Widget pic;

    if (src == null) {

      pic = Center(

        child: SvgPicture.asset('assets/icons/image_placeholder.svg'),

      );

    } else if (src.startsWith('http')) {

      pic = Image.network(src, fit: BoxFit.cover);

    } else {

      pic = Image.file(File(src), fit: BoxFit.cover);

    }

  

    return GestureDetector(

      key: Key('$idx'),

      onTap: () => _pick(idx),

      child: Container(

        width: side,

        height: side * 1.2,

        decoration: BoxDecoration(

          color: const Color(0xFFF6F6F6),

          border: Border.all(color: const Color(0xFFBFBBBB), width: 2),

          borderRadius: BorderRadius.circular(8),

        ),

        child: Stack(children: [

          ClipRRect(

            borderRadius: BorderRadius.circular(6),

            child: SizedBox.expand(child: pic),

          ),

          if (src != null)

            Positioned(

              top: 4,

              right: 4,

              child: GestureDetector(

                onTap: () => _confirmDelete(idx),

                child: Container(

                  width: 22,

                  height: 22,

                  decoration: BoxDecoration(

                    color: Colors.black.withOpacity(.75),

                    shape: BoxShape.circle,

                  ),

                  child: const Center(

                    child: Icon(Icons.close, size: 16, color: Colors.white),

                  ),

                ),

              ),

            ),

        ]),

      ),

    );

  }

  

  /* ───────────────────────── helpers ───────────────────────────── */

  

  void _snack(String m) =>

      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(m)));

}

  

/* ───────────────────────── mini crop page ──────────────────────── */

class _CropPage extends StatefulWidget {

  final File file;

  const _CropPage({required this.file});

  

  @override

  State<_CropPage> createState() => _CropPageState();

}

  

class _CropPageState extends State<_CropPage> {

  @override

  void initState() {

    super.initState();

    _startCrop();

  }

  

  Future<void> _startCrop() async {

    final out = await ImageCropper().cropImage(

      sourcePath: widget.file.path,

      aspectRatio: const CropAspectRatio(ratioX: 8, ratioY: 12),

      compressQuality: 90,

      uiSettings: [

        AndroidUiSettings(

          toolbarTitle: 'Adjust image',

          toolbarColor: Colors.black,

          toolbarWidgetColor: Colors.white,

          lockAspectRatio: true,

        ),

        IOSUiSettings(

          title: 'Adjust image',

          aspectRatioLockEnabled: true,

        ),

      ],

    );

    Navigator.pop(context, out == null ? null : File(out.path));

  }

  

  @override

  Widget build(BuildContext context) =>

      const Scaffold(body: Center(child: CircularProgressIndicator()));

}
```

- tabs/profile/edit_profile_qa_page.dart
```
import 'package:flutter/material.dart';

  

/// Q-and-A editor page – cards now full width with 10px side margins.

class EditQAPage extends StatefulWidget {

  final List<Map<String, String>>? initial;

  const EditQAPage({Key? key, this.initial}) : super(key: key);

  

  @override

  State<EditQAPage> createState() => _EditQAPageState();

}

  

class _EditQAPageState extends State<EditQAPage> {

  late List<Map<String, String>?> _qa;

  

  @override

  void initState() {

    super.initState();

    _qa = widget.initial == null

        ? [null, null, null]

        : List<Map<String, String>?>.from(widget.initial!)

      ..length = 3;

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Profil-elemente'), actions: [

        TextButton(

            onPressed: () => Navigator.pop(

                context, _qa.whereType<Map<String, String>>().toList()),

            child: const Text('Save', style: TextStyle(color: Colors.white)))

      ]),

      body: Column(children: List.generate(3, _card)),

    );

  }

  

  Widget _card(int i) {

    final item = _qa[i];

    return Expanded(

      child: GestureDetector(

        onTap: () async {

          final res = await Navigator.push<Map<String, String>?>(

              context,

              MaterialPageRoute(

                  builder: (_) => _ChooseQuestionPage(

                        initialQuestion: item?['question'],

                        initialAnswer: item?['answer'],

                      )));

          if (res != null) setState(() => _qa[i] = res);

        },

        child: Container(

          width: double.infinity,

          margin: const EdgeInsets.fromLTRB(10, 0, 10, 16), // 10px sides

          padding: const EdgeInsets.all(16),

          decoration: BoxDecoration(

              border: Border.all(color: Colors.grey),

              borderRadius: BorderRadius.circular(19)),

          child: Column(

              mainAxisAlignment: MainAxisAlignment.center,

              crossAxisAlignment: CrossAxisAlignment.start,

              children: [

                Text(item?['question'] ?? 'wähle eine frage',

                    style: TextStyle(

                        fontSize: 18,

                        fontWeight: FontWeight.bold,

                        color: item == null ? Colors.grey : Colors.black87,

                        fontFamily: 'MyAdobeFont')),

                const SizedBox(height: 10),

                Text(item?['answer'] ?? 'und gib deine Antwort',

                    style: TextStyle(

                      fontSize: 16,

                      color: item == null ? Colors.grey : Colors.black54,

                      fontWeight:

                          item == null ? FontWeight.w300 : FontWeight.normal,

                      fontFamily: 'MyAdobeFont',

                    )),

              ]),

        ),

      ),

    );

  }

}

  

/* ── modal question picker ───────────────────────────────────── */

class _ChooseQuestionPage extends StatefulWidget {

  final String? initialQuestion;

  final String? initialAnswer;

  const _ChooseQuestionPage({this.initialQuestion, this.initialAnswer});

  

  @override

  State<_ChooseQuestionPage> createState() => _ChooseQuestionPageState();

}

  

class _ChooseQuestionPageState extends State<_ChooseQuestionPage> {

  final List<String> _questions = [

    'Wenn ich ein Tier wäre, dann wäre ich ein …',

    'Was ich erst vor kurzem herausgefunden habe...',

    'Ein Ort, an den ich immer wieder zurückkehre...',

    'Mein liebster Feiertag und warum...',

    'Das verrückteste Abenteuer, das ich je erlebt habe...',

    'Ein Buch, das mein Leben verändert hat...',

    'Wenn ich eine Superkraft haben könnte, wäre es...',

    'Mein geheimes Talent...',

    'Meine liebste Art, einen Regentag zu verbringen...',

    'Das beste Essen, das ich je gekocht habe...',

    'Ein Lied, das mich immer glücklich macht...',

    'Meine ideale Art, das Wochenende zu verbringen...',

    'Etwas, das ich gerne lernen möchte...',

    'Ein Film, der mich zum Lachen oder Weinen bringt...',

    'Das Schönste, das mir jemals passiert ist...',

    'Meine größte Herausforderung und wie ich sie gemeistert habe...',

    'Ein Zitat, nach dem ich lebe...',

    'Wenn ich einen Tag lang unsichtbar wäre, würde ich...',

    'Meine früheste Kindheitserinnerung...',

    'Ein Ort, den ich unbedingt besuchen möchte...',

    'Drei Dinge, die ich immer in meiner Tasche habe...'

  ];

  String? _sel;

  final _ctrl = TextEditingController();

  bool _stepAnswer = false;

  final _focus = FocusNode();

  

  @override

  void initState() {

    super.initState();

    _sel = widget.initialQuestion;

    _ctrl.text = widget.initialAnswer ?? '';

    if (_sel != null) _stepAnswer = true;

  }

  

  @override

  void dispose() {

    _focus.dispose();

    super.dispose();

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(

        title: const Text('Fragen & Antworten'),

        backgroundColor: Colors.white,

        elevation: 0,

        iconTheme: const IconThemeData(color: Colors.black),

        actions: _stepAnswer

            ? [

                IconButton(

                    onPressed: () {

                      if (_ctrl.text.isNotEmpty) {

                        Navigator.pop(

                            context, {'question': _sel, 'answer': _ctrl.text});

                      }

                    },

                    icon: const Icon(Icons.check, color: Colors.black))

              ]

            : null,

      ),

      backgroundColor: Colors.white,

      body: _stepAnswer ? _answer() : _list(),

    );

  }

  

  Widget _list() => ListView.builder(

        padding: const EdgeInsets.fromLTRB(10, 24, 10, 0), // 10px sides

        itemCount: _questions.length,

        itemBuilder: (c, i) => GestureDetector(

          onTap: () {

            setState(() {

              _sel = _questions[i];

              _stepAnswer = true;

            });

            Future.delayed(Duration.zero, () => _focus.requestFocus());

          },

          child: Container(

            width: double.infinity,

            margin: const EdgeInsets.only(bottom: 20),

            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),

            decoration: BoxDecoration(

              border: Border.all(color: Colors.grey.shade400, width: 2),

              borderRadius: BorderRadius.circular(19),

            ),

            child:

                Column(crossAxisAlignment: CrossAxisAlignment.start, children: [

              Text(_questions[i],

                  style: const TextStyle(

                      fontSize: 18, fontWeight: FontWeight.bold)),

              const SizedBox(height: 10),

              const Text('tippe um zu Antworten',

                  style: TextStyle(fontSize: 16, color: Colors.grey)),

            ]),

          ),

        ),

      );

  

  Widget _answer() => SingleChildScrollView(

        padding: const EdgeInsets.fromLTRB(10, 24, 10, 24), // 10px sides

        child: Container(

          width: double.infinity,

          padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 16),

          decoration: BoxDecoration(

            border: Border.all(color: Colors.grey.shade400, width: 2),

            borderRadius: BorderRadius.circular(19),

          ),

          child:

              Column(crossAxisAlignment: CrossAxisAlignment.start, children: [

            Text(_sel ?? '',

                style:

                    const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),

            const SizedBox(height: 16),

            TextField(

              controller: _ctrl,

              focusNode: _focus,

              maxLines: null,

              decoration: const InputDecoration(

                  hintText: 'Antwort eingeben...', border: InputBorder.none),

              style: const TextStyle(fontSize: 16),

            ),

          ]),

        ),

      );

}
```

- tabs/profile/edit_profile_radius_page.dart
```
import 'dart:math' as math;

import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:flutter_map/flutter_map.dart';

import 'package:geolocator/geolocator.dart';

import 'package:latlong2/latlong.dart';

  

/// Radius‑picker page adapted for the new edit profile flow.

/// Returns `{ 'radius': km, 'location': LatLng }` on save.

class EditRadiusPage extends StatefulWidget {

  final double? initialRadius;

  final LatLng? initialPos;

  const EditRadiusPage({Key? key, this.initialRadius, this.initialPos})

      : super(key: key);

  

  @override

  State<EditRadiusPage> createState() => _EditRadiusPageState();

}

  

class _EditRadiusPageState extends State<EditRadiusPage> {

  double _km = 40;

  LatLng? _pos; // use LatLng only – avoids Position ctor issues

  late final MapController _mc;

  bool _mapReady = false;

  

  @override

  void initState() {

    super.initState();

    _km = widget.initialRadius ?? 40;

    _pos = widget.initialPos;

    _mc = MapController();

  }

  

  /* ── UI helpers ────────────────────────────────────────────── */

  Widget _bubble() => Positioned(

        top: 0,

        left: _km * 2.5,

        child: Material(

          color: Colors.transparent,

          child: Container(

            padding: const EdgeInsets.all(6),

            decoration: BoxDecoration(

              color: Colors.white,

              borderRadius: BorderRadius.circular(12),

              boxShadow: const [

                BoxShadow(color: Colors.black26, blurRadius: 4)

              ],

            ),

            child: Text('${_km.round()} km'),

          ),

        ),

      );

  

  /* ── build ─────────────────────────────────────────────────── */

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      body: Stack(children: [

        Positioned.fill(

          child: SingleChildScrollView(

            padding: const EdgeInsets.all(16),

            child:

                Column(crossAxisAlignment: CrossAxisAlignment.start, children: [

              const Text(

                  'I think you get it but for layout‑consistency, here is some text.',

                  style: TextStyle(color: Colors.grey)),

              const SizedBox(height: 16),

              Container(

                height: 300,

                width: double.infinity,

                decoration: BoxDecoration(

                  border: Border.all(color: Colors.grey),

                  borderRadius: BorderRadius.circular(8),

                ),

                child: _pos == null ? _placeholder() : _buildMap(),

              ),

              const SizedBox(height: 16),

              ElevatedButton(

                onPressed: _getLocation,

                style: ElevatedButton.styleFrom(shape: const StadiumBorder()),

                child: const Text('Enable Location'),

              ),

              const SizedBox(height: 32),

              Center(

                child: SizedBox(

                  width: 300,

                  child: Stack(alignment: Alignment.center, children: [

                    Slider(

                      min: 10,

                      max: 100,

                      divisions: 90,

                      value: _km,

                      onChanged: (v) => setState(() {

                        _km = v;

                        if (_mapReady && _pos != null) _updateMap();

                      }),

                    ),

                    _bubble(),

                  ]),

                ),

              ),

            ]),

          ),

        ),

        Positioned(

          bottom: 20,

          right: 30,

          child: IconButton(

            icon: SvgPicture.asset('assets/icons/arrow_icon.svg',

                width: 50, height: 50),

            onPressed: () =>

                Navigator.pop(context, {'radius': _km, 'location': _pos}),

          ),

        ),

      ]),

    );

  }

  

  Widget _placeholder() => const Center(

      child: Text('Enable location to show map',

          style: TextStyle(color: Colors.brown)));

  

  Widget _buildMap() {

    final center = _pos!;

    return FlutterMap(

      mapController: _mc,

      options: MapOptions(

          initialCenter: center,

          initialZoom: 10,

          onMapReady: () {

            _mapReady = true;

            _updateMap();

          }),

      children: [

        TileLayer(

            urlTemplate: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',

            subdomains: ['a', 'b', 'c']),

        CircleLayer(circles: [

          CircleMarker(

              point: center,

              radius: _km * 1000,

              useRadiusInMeter: true,

              color: Colors.blue.withOpacity(.3),

              borderColor: Colors.blue,

              borderStrokeWidth: 2),

        ]),

        MarkerLayer(markers: [

          Marker(

              point: center,

              width: 40,

              height: 40,

              child:

                  const Icon(Icons.location_pin, color: Colors.red, size: 40))

        ]),

      ],

    );

  }

  

  void _updateMap() {

    if (!_mapReady || _pos == null) return;

    final zoom = _calcZoom(_pos!, _km);

    _mc.move(_pos!, zoom);

  }

  

  double _calcZoom(LatLng c, double r) {

    final latRad = c.latitude * math.pi / 180;

    final diff = (r / 111) * 2;

    final lon = (r / (111 * math.cos(latRad))) * 2;

    final maxDiff = math.max(diff, lon);

    return (math.log(360 / maxDiff) / math.log(2) - 1.2).clamp(2, 17);

  }

  

  /* ── location ​​─────── */

  Future<void> _getLocation() async {

    if (!await Geolocator.isLocationServiceEnabled()) {

      _snack('Enable location services');

      return;

    }

    var perm = await Geolocator.checkPermission();

    if (perm == LocationPermission.denied)

      perm = await Geolocator.requestPermission();

    if (perm == LocationPermission.denied ||

        perm == LocationPermission.deniedForever) {

      _snack('Location permission denied');

      return;

    }

    final p = await Geolocator.getCurrentPosition(

        desiredAccuracy: LocationAccuracy.high);

    setState(() => _pos = LatLng(p.latitude, p.longitude));

    if (_mapReady) _updateMap();

  }

  

  void _snack(String m) =>

      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(m)));

}
```
- tabs/profile/profile.dart
```
// lib/tabs/profile/profile_screen.dart

import 'package:flutter/material.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile.dart';

import 'package:swipe_chat_play/tabs/profile/view_my_profile.dart';

  

class ProfileScreen extends StatefulWidget {

  const ProfileScreen({Key? key}) : super(key: key);

  

  @override

  State<ProfileScreen> createState() => _ProfileScreenState();

}

  

class _ProfileScreenState extends State<ProfileScreen> {

  late final PageController _pageController;

  int _pageIndex = 0;

  

  @override

  void initState() {

    super.initState();

    _pageController = PageController(initialPage: _pageIndex);

  }

  

  @override

  void dispose() {

    _pageController.dispose();

    super.dispose();

  }

  

  void _onTap(int index) {

    if (_pageIndex != index) {

      _pageController.animateToPage(

        index,

        duration: const Duration(milliseconds: 300),

        curve: Curves.easeInOut,

      );

    }

  }

  

  void _onPageChanged(int index) {

    setState(() {

      _pageIndex = index;

    });

  }

  

  Widget _buildTab(String label, int index) {

    final selected = _pageIndex == index;

    return Expanded(

      child: GestureDetector(

        onTap: () => _onTap(index),

        child: Column(

          mainAxisSize: MainAxisSize.min,

          children: [

            Text(

              label,

              style: TextStyle(

                fontSize: 16,

                decoration:

                    selected ? TextDecoration.underline : TextDecoration.none,

                color: Colors.grey[800],

              ),

            ),

            const SizedBox(height: 4),

            Container(

              height: 2,

              color: selected ? Colors.blue : Colors.transparent,

            ),

          ],

        ),

      ),

    );

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      // no AppBar to maximize swipe area; safe area below

      body: SafeArea(

        child: Column(

          children: [

            // tabs row

            Padding(

              padding: const EdgeInsets.symmetric(vertical: 16.0),

              child: Row(

                children: [

                  _buildTab('profile', 0),

                  _buildTab('info', 1),

                ],

              ),

            ),

  

            // swipeable pages

            Expanded(

              child: PageView(

                controller: _pageController,

                onPageChanged: _onPageChanged,

                children: const [

                  ViewMyProfile(),

                  EditProfileLayout(),

                ],

              ),

            ),

          ],

        ),

      ),

    );

  }

}
```
- tabs/profile/profile_tab.dart
```
import 'dart:convert';

import 'dart:io';

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:image_picker/image_picker.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_svg/flutter_svg.dart';

import 'package:swipe_chat_play/colors.dart';

  

class Profiletab extends StatefulWidget {

  const Profiletab({super.key});

  

  @override

  _ProfiletabState createState() => _ProfiletabState();

}

  

class _ProfiletabState extends State<Profiletab> {

  final FlutterSecureStorage _secureStorage = const FlutterSecureStorage();

  Map<String, dynamic> profileData = {};

  bool isEditing = false;

  

  @override

  void initState() {

    super.initState();

    _loadProfileData();

  }

  

  Future<void> _loadProfileData() async {

    // Load data from local storage or server

    String? localJson = await _secureStorage.read(key: 'profile_json');

    if (localJson != null) {

      setState(() {

        profileData = jsonDecode(localJson);

      });

    } else {

      profileData = {

        "username": "John Doe",

        "age": "25",

        "description": "Add something about yourself.",

        "image": "https://example.com/profile-image.jpg"

      };

    }

  }

  

  Future<void> _pickAndUploadImage() async {

    final picker = ImagePicker();

    final XFile? pickedFile =

        await picker.pickImage(source: ImageSource.gallery);

  

    if (pickedFile != null) {

      final File imageFile = File(pickedFile.path);

      // Upload the file to the server (mock API example)

      final response = await http.post(

        Uri.parse('http://your-api-url/upload'),

        body: {"image": imageFile.path},

      );

  

      if (response.statusCode == 200) {

        setState(() {

          profileData["image"] = "https://example.com/new-uploaded-image.jpg";

        });

      }

    }

  }

  

  void _saveProfile() async {

    // Save data to server

    await _secureStorage.write(

        key: 'profile_json', value: jsonEncode(profileData));

    ScaffoldMessenger.of(context)

        .showSnackBar(SnackBar(content: Text('Profile Saved')));

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: AppColors.cardBackground,

      body: SafeArea(

        // Ensures content starts below the status bar

        child: Stack(

          // Your existing UI elements

          children: [

            // Background Gradient

            Container(

              decoration: BoxDecoration(

                gradient: LinearGradient(

                  colors: [Colors.orange, Colors.pink],

                  begin: Alignment.topCenter,

                  end: Alignment.bottomCenter,

                ),

              ),

            ),

            // White Card with MyProfileScreen Content

            Positioned.fill(

              top: 73,

              bottom: 20,

              child: Padding(

                padding: const EdgeInsets.symmetric(horizontal: 16.0),

                child: Container(

                  decoration: BoxDecoration(

                    color: Colors.white,

                    borderRadius: BorderRadius.circular(24.0),

                    boxShadow: [

                      BoxShadow(

                        color: Colors.grey.withOpacity(0.5),

                        spreadRadius: 1,

                        blurRadius: 8,

                        offset: Offset(0, 4),

                      ),

                    ],

                  ),

                  child: Padding(

                    padding: const EdgeInsets.all(16.0),

                    child: Column(

                      crossAxisAlignment: CrossAxisAlignment.start,

                      children: [

                        // Profile Image

                        Center(

                          child: GestureDetector(

                            onTap: _pickAndUploadImage,

                            child: CircleAvatar(

                              radius: 60,

                              backgroundImage: NetworkImage(

                                  profileData["image"] ??

                                      "https://via.placeholder.com/150"),

                            ),

                          ),

                        ),

                        const SizedBox(height: 16),

                        // Username

                        Text(

                          "Username:",

                          style: TextStyle(

                              fontSize: 16, fontWeight: FontWeight.bold),

                        ),

                        const SizedBox(height: 8),

                        Text(profileData["username"] ?? "Unknown"),

                        const SizedBox(height: 16),

                        // Age

                        Text(

                          "Age:",

                          style: TextStyle(

                              fontSize: 16, fontWeight: FontWeight.bold),

                        ),

                        const SizedBox(height: 8),

                        Text(profileData["age"] ?? "Unknown"),

                        const SizedBox(height: 16),

                        // Description

                        Text(

                          "Description:",

                          style: TextStyle(

                              fontSize: 16, fontWeight: FontWeight.bold),

                        ),

                        const SizedBox(height: 8),

                        Text(profileData["description"] ??

                            "Add a description..."),

                        const Spacer(),

                        // Save Button

                        Center(

                          child: ElevatedButton(

                            onPressed: _saveProfile,

                            child: Text("Save Profile"),

                          ),

                        ),

                      ],

                    ),

                  ),

                ),

              ),

            ),

            // Top Buttons

            Positioned(

              top: 20,

              left: 16,

              child: CircleButton(

                assetPath: 'assets/icons/close.svg',

                onPressed: () {

                  Navigator.of(context).pop();

                },

              ),

            ),

            Positioned(

              top: 20,

              right: 16,

              child: CircleButton(

                assetPath: 'assets/icons/check.svg',

                onPressed: _saveProfile,

              ),

            ),

          ],

        ),

      ),

    );

  }

}

  

// Reusable SVG Circle Button

class CircleButton extends StatelessWidget {

  final String assetPath;

  final VoidCallback onPressed;

  

  const CircleButton({

    super.key,

    required this.assetPath,

    required this.onPressed,

  });

  

  @override

  Widget build(BuildContext context) {

    return InkWell(

      borderRadius: BorderRadius.circular(100),

      onTap: onPressed,

      child: SvgPicture.asset(

        assetPath,

        width: 48,

        height: 48,

      ),

    );

  }

}
```
- tabs/profile/view_my_profile.dart
```
import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:http/http.dart' as http;

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/config.dart';

import 'package:google_fonts/google_fonts.dart';

  

// Import your model & widgets.

import 'package:swipe_chat_play/profile_layout/models/profile_data.dart'

    as profile;

import 'package:swipe_chat_play/profile_layout/widgets/bubble_grid_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/ice_breaker_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/image_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

  

class ViewMyProfile extends StatefulWidget {

  const ViewMyProfile({super.key});

  

  @override

  _ViewMyProfileState createState() => _ViewMyProfileState();

}

  

class _ViewMyProfileState extends State<ViewMyProfile> {

  static final storage = const FlutterSecureStorage();

  late Future<profile.ProfileData> _futureProfile;

  

  @override

  void initState() {

    super.initState();

    _futureProfile = _fetchProfile();

  }

  

  /// Fetch the user's profile from get_profile.php.

  Future<profile.ProfileData> _fetchProfile() async {

    final url = '${Config.backendBaseUrl}/create_profile/get_profile.php';

    try {

      final token = await storage.read(key: 'jwt_token');

      debugPrint('JWT token from storage: $token');

      if (token == null) {

        debugPrint('No JWT token found in storage.');

        throw Exception('No JWT token found');

      }

      debugPrint('Making request to $url...');

      final response = await http.get(

        Uri.parse(url),

        headers: {

          'Authorization': 'Bearer $token',

        },

      );

      debugPrint('Response status code: ${response.statusCode}');

      debugPrint('Response body: ${response.body}');

      if (response.statusCode == 200) {

        final Map<String, dynamic> jsonMap = json.decode(response.body);

        return profile.ProfileData.fromJson(jsonMap);

      } else {

        debugPrint('Failed to load profile. Status: ${response.statusCode}');

        throw Exception(

            'Failed to load profile. Status code: ${response.statusCode}');

      }

    } catch (e, stacktrace) {

      debugPrint('Error fetching profile: $e');

      debugPrint('Stacktrace:\n$stacktrace');

      rethrow;

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: AppColors.background,

      body: FutureBuilder<profile.ProfileData>(

        future: _futureProfile,

        builder: (context, snapshot) {

          debugPrint('FutureBuilder snapshot: ${snapshot.connectionState}');

          if (snapshot.connectionState == ConnectionState.waiting) {

            return const Center(child: CircularProgressIndicator());

          } else if (snapshot.hasError) {

            debugPrint('Snapshot error: ${snapshot.error}');

            return Center(

              child: Text(

                'Error: ${snapshot.error}',

                style: GoogleFonts.montserrat(),

              ),

            );

          } else if (!snapshot.hasData) {

            debugPrint('No profile data found in snapshot.');

            return Center(

              child: Text(

                'No profile data found.',

                style: GoogleFonts.montserrat(),

              ),

            );

          } else {

            final userProfile = snapshot.data!;

            debugPrint('Successfully loaded userProfile.');

            List<Widget> contentWidgets = [];

            // Top section widget.

            contentWidgets.add(

              TopSectionWidget(

                topSection: userProfile.topSection,

                swipedUserUsername: userProfile.topSection.username,

              ),

            );

            // Divider after top section.

            contentWidgets.add(const Divider(

              height: 2,

              thickness: 2,

              color: AppColors.dividerLine,

            ));

            // For each element.

            final elementEntries = userProfile.elements.entries.toList();

            for (int i = 0; i < elementEntries.length; i++) {

              contentWidgets.add(_buildElement(

                  elementEntries[i].value, userProfile.topSection.username));

              if (i != elementEntries.length - 1) {

                contentWidgets.add(const Divider(

                  height: 2,

                  thickness: 2,

                  color: AppColors.dividerLine,

                ));

              }

            }

            return Padding(

              padding: const EdgeInsets.all(8.0),

              child: SingleChildScrollView(

                child: Card(

                  color: AppColors.cardBackground,

                  shape: RoundedRectangleBorder(

                    borderRadius: BorderRadius.circular(16),

                  ),

                  child: Column(

                    crossAxisAlignment: CrossAxisAlignment.stretch,

                    children: contentWidgets,

                  ),

                ),

              ),

            );

          }

        },

      ),

    );

  }

  

  Widget _buildElement(

      profile.ProfileElement element, String swipedUserUsername) {

    switch (element.type) {

      case 'icebreaker':

        return IceBreakerLayout(

          content: element.content,

          swipedUserUsername: swipedUserUsername,

        );

      case 'bubbles':

        return BubbleGridLayout(

          content: element.content,

          swipedUserUsername: swipedUserUsername,

        );

      default:

        debugPrint('Unknown element type encountered: ${element.type}');

        return const SizedBox.shrink();

    }

  }

}
```

- tabs/swipe/card_stack.dart
```
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

  ///   {

  ///     "type"       : "react_bubble" or "react_icebreaker",

  ///     "content"    : { bubble_text: "...", question: "...", answer: "..." },

  ///     "receiver_id": "<Mongo _id string>"

  ///   }

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
```

- usables/inappwebview.dart
```
import 'package:flutter/material.dart';

import 'package:flutter_inappwebview/flutter_inappwebview.dart';

import 'package:flutter_inappwebview_platform_interface/flutter_inappwebview_platform_interface.dart'; // for WebUri

  

class InAppWebViewScreen extends StatelessWidget {

  final String url;

  

  InAppWebViewScreen({required this.url});

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(

        title: Text("Webview"),

      ),

      body: InAppWebView(

        initialUrlRequest: URLRequest(url: WebUri.uri(Uri.parse(url))),  // Updated to use WebUri

      ),

    );

  }

}
```
# API

## api project structure
api base url: "http://panel.krasserserver.com:8002/swipe_chatt_play_api"

woven>
|   config.php
|   upload_image.php
|
+---chat
|   |   create_chat.php
|   |   fetchChat.php
|   |   get_profile_from_chatid.php
|   |   myChats.php
|   |   search_user.php
|   |   sendMessage.php
|   |   sendReaction.php
|   |   sendVoiceMessage.php
|   |
|   \---uploads
|       \---voicememos
|               voice_67ec134b565c04.80218249_1743524681530.aac
|               ...
|
+---create_profile
|   |   add_answer.php
|   |   create_testusers.php
|   |   elements_update.php
|   |   get_profile.php
|   |   get_userdata.php
|   |   load_profile.php
|   |   retrieve_profile.php
|   |   update_loc.php
|   |   update_open_to.php
|   |   update_searching_for.php
|   |   update_userdata.php
|   |   upload_profile_images.php
|   |
|   \---uploads
|       |   67acd6d889960_image_cropper_1739380344294.jpg
|       |   ...
|       |
|       \---voicememos
+---login_signup
|       addFingerprint.php
|       fingerprintLogin.php
|       login.php
|       search_users.php
|       signup.php
|       verifycode_login.php
|       verify_code.php
|
+---logs
|       add_fingerprint_error.log
|       fetch_chat_log.log
|
+---match_algorithm
|       create_matches.php
|       curl_activities_area.php
|       debug.log
|       generate.php
|       match.php
|       my_custom_errors.log
|       react_profile.php
|       sendReaction.php
|       sendReaction_debug.log
|       submit_questionaire.php
|       swipe.php
|
\---profile
    |   create_example_users.php
    |   retrieve_profile.php
    |   save_profile.php
    |   suggest_5_profiles.php
    |   suggest_profiles.php
    |   upload_image.php
    |
    \---uploads
            6781163630dce_settings chess print.PNG
            ...




- config.php
```
<?php

// Configuration file (config.php)

$ngrokURL = "panel.krasserserver.com";  // ngrok TCP URL

$mongoPort = "19028";  // Port provided by ngrok

$mongoUser = "burgermeister";  // MongoDB username

$mongoPassword = "drdrjecky14";  // MongoDB password

$mongoAuthDB = "admin";  // Authentication database

  

//$ngrokURL = "6.tcp.eu.ngrok.io";  // ngrok TCP URL

//$mongoPort = "19028";  // Port provided by ngrok

//$mongoUser = "burgermeister";  // MongoDB username

//$mongoPassword = "drdrjecky14";  // MongoDB password

//$mongoAuthDB = "admin";  // Authentication database

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

// MongoDB connection URI

$mongoURI = "mongodb://$mongoUser:$mongoPassword@$ngrokURL:$mongoPort/$mongoAuthDB";

?>
```
- upload_image.php
```
<?php

require __DIR__ . '/../vendor/autoload.php';

require 'db_connection.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Logging configuration

$logFile = __DIR__ . '/upload_image_debug.log';

ini_set('log_errors', 1);

ini_set('error_log', $logFile);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

error_log("upload_image.php script started");

  

// Function to decode JWT and retrieve user ID

function id_jwt($authHeader, $secretKey) {

    if (!$authHeader) {

        error_log("Authorization header missing");

        http_response_code(401);

        echo json_encode(["error" => "Authorization header missing"]);

        exit;

    }

  

    $jwtToken = str_replace('Bearer ', '', $authHeader);

    error_log("Extracted JWT token: $jwtToken");

  

    try {

        $decoded = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

        $userId = $decoded->user_id ?? null;

  

        if (!$userId) {

            error_log("Token decoded but user_id not found in payload");

            http_response_code(401);

            echo json_encode(["error" => "Invalid token payload"]);

            exit;

        }

  

        error_log("Successfully decoded JWT. User ID: $userId");

        return $userId;

    } catch (Exception $e) {

        error_log("JWT decoding failed: " . $e->getMessage());

        http_response_code(401);

        echo json_encode(["error" => "Invalid or missing token"]);

        exit;

    }

}

  

// Handle the POST request

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $authHeader = $_SERVER['HTTP_AUTHORIZATION'] ?? null;

    error_log("Received Authorization header: $authHeader");

  

    $userId = id_jwt($authHeader, $secretKey);

    error_log("Decoded User ID: $userId");

  

    // Validate file upload

    if (!isset($_FILES['image']) || !isset($_POST['image_name'])) {

        error_log("Validation failed: Missing image file or image name");

        http_response_code(400);

        echo json_encode(["error" => "Image file and name are required"]);

        exit;

    }

  

    $image = $_FILES['image'];

    $imageName = basename($_POST['image_name']);

    error_log("Received image file: " . print_r($image, true));

    error_log("Received image name: $imageName");

  

    // Directory to save images

    $uploadDir = __DIR__ . "/profileImages/$userId/";

    error_log("Image upload directory: $uploadDir");

  

    if (!is_dir($uploadDir)) {

        if (mkdir($uploadDir, 0755, true)) {

            error_log("Created directory: $uploadDir");

        } else {

            error_log("Failed to create directory: $uploadDir");

            http_response_code(500);

            echo json_encode(["error" => "Failed to create image directory"]);

            exit;

        }

    }

  

    $filePath = $uploadDir . $imageName;

    error_log("Target file path: $filePath");

  

    // Move uploaded file

    if (move_uploaded_file($image['tmp_name'], $filePath)) {

        error_log("Image successfully uploaded to: $filePath");

        echo json_encode(["message" => "Image uploaded successfully", "path" => $filePath]);

    } else {

        error_log("Failed to move uploaded file to: $filePath");

        http_response_code(500);

        echo json_encode(["error" => "Failed to save the image"]);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Invalid request method"]);

}
```
- create_chat.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require '../config.php'; // MongoDB connection configuration

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["status" => "error", "message" => "Method not allowed"]);

    exit;

}

  

try {

    // Extract Authorization header

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        http_response_code(401);

        echo json_encode(["status" => "error", "message" => "Unauthorized"]);

        exit;

    }

  

    $jwtToken = $matches[1];

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $senderId = $decodedToken->user_id ?? null;

  

    if (!$senderId) {

        http_response_code(401);

        echo json_encode(["status" => "error", "message" => "Invalid token"]);

        exit;

    }

  

    // Connect to MongoDB

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // Fetch sender's username and profile image

    $senderFilter = ['_id' => new ObjectId($senderId)];

    $senderQuery = new MongoDB\Driver\Query($senderFilter);

    $senderCursor = $mongoManager->executeQuery("swipe_chat_play.users", $senderQuery);

    $sender = current($senderCursor->toArray());

  

    if (!$sender || !isset($sender->username)) {

        http_response_code(404);

        echo json_encode(["status" => "error", "message" => "Sender not found"]);

        exit;

    }

  

    $senderUsername = $sender->username;

    $senderProfileImage = '';

    if (isset($sender->profile)) {

        $senderProfile = json_decode($sender->profile, true);

        $senderProfileImage = $senderProfile['image'] ?? '';

    }

  

    // Parse request payload

    $input = json_decode(file_get_contents('php://input'), true);

    $receiverUsername = $input['receiver_username'] ?? null;

  

    if (!$receiverUsername) {

        http_response_code(400);

        echo json_encode(["status" => "error", "message" => "Receiver username is required"]);

        exit;

    }

  

    // Fetch receiver information

    $receiverFilter = ['username' => $receiverUsername];

    $receiverQuery = new MongoDB\Driver\Query($receiverFilter);

    $receiverCursor = $mongoManager->executeQuery("swipe_chat_play.users", $receiverQuery);

    $receiver = current($receiverCursor->toArray());

  

    if (!$receiver) {

        http_response_code(404);

        echo json_encode(["status" => "error", "message" => "Receiver not found"]);

        exit;

    }

  

    $receiverId = (string)$receiver->_id;

    $receiverProfileImage = '';

    if (isset($receiver->profile)) {

        $receiverProfile = json_decode($receiver->profile, true);

        $receiverProfileImage = $receiverProfile['image'] ?? '';

    }

  

    // Create chat document

    $chatData = [

        "messages" => new \stdClass(),

        "participants" => [$senderId, $receiverId]

    ];

    $bulk = new MongoDB\Driver\BulkWrite();

    $chatId = $bulk->insert($chatData);

    $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulk);

  

    $chatIdString = (string)$chatId;

  

    // Prepare chat metadata

    $chatMetaSender = [

        "profileURL" => $receiverProfileImage,

        "receiver" => $receiverUsername,

        "last_message" => "",

        "unread" => 0

    ];

  

    $chatMetaReceiver = [

        "profileURL" => $senderProfileImage,

        "receiver" => $senderUsername,

        "last_message" => "",

        "unread" => 0

    ];

  

    // Update sender's and receiver's chat lists

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

  

    echo json_encode(["status" => "success", "chat_id" => $chatIdString]);

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["status" => "error", "message" => $e->getMessage()]);

}

?>
```
- fetchChat.php
```
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
```
- get_profile_from_chatid.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

  

// NOTE: Use the same secretKey from your existing scripts:

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    // -----------------------------------------------------------

    // 1. Decode JWT from Authorization header

    // -----------------------------------------------------------

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    $jwt = $matches[1];

    $decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

    $currentUserId = $decoded->user_id ?? null;

  

    if (!$currentUserId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // -----------------------------------------------------------

    // 2. Read JSON input to get the 'chat_id'

    // -----------------------------------------------------------

    $rawData = file_get_contents('php://input');

    $postData = json_decode($rawData, true);

    $chatId = $postData['chat_id'] ?? null;

    if (!$chatId) {

        http_response_code(400);

        echo json_encode(["error" => "Missing chat_id"]);

        exit;

    }

  

    // -----------------------------------------------------------

    // 3. Connect to MongoDB

    // -----------------------------------------------------------

    // Make sure $mongoURI is defined in config.php

    $mongo = new MongoDB\Driver\Manager($mongoURI);

  

    // -----------------------------------------------------------

    // 4. Find the chat document with `_id = chatId`

    // -----------------------------------------------------------

    $filterChat = ["_id" => new MongoDB\BSON\ObjectId($chatId)];

    $optionsChat = [];

    $queryChat = new MongoDB\Driver\Query($filterChat, $optionsChat);

    // Adjust the collection name if needed (e.g. "swipe_chat_play.chats")

    $cursorChat = $mongo->executeQuery("swipe_chat_play.chats", $queryChat);

    $chatDoc = current($cursorChat->toArray());

  

    if (!$chatDoc) {

        http_response_code(404);

        echo json_encode(["error" => "Chat not found"]);

        exit;

    }

  

    // -----------------------------------------------------------

    // 5. Extract the participants array, find the 'other user'

    // -----------------------------------------------------------

    $participants = $chatDoc->participants ?? null;

    if (!$participants || !is_array($participants)) {

        http_response_code(404);

        echo json_encode(["error" => "Chat has no participants array"]);

        exit;

    }

  

    // Identify the participant that is not the current user

    $otherUserId = null;

    foreach ($participants as $participantId) {

        if ($participantId !== $currentUserId) {

            $otherUserId = $participantId;

            break;

        }

    }

  

    if (!$otherUserId) {

        http_response_code(404);

        echo json_encode(["error" => "Other user not found in participants"]);

        exit;

    }

  

    // -----------------------------------------------------------

    // 6. Query the other user’s profile

    // -----------------------------------------------------------

    $filterOther = ["_id" => new MongoDB\BSON\ObjectId($otherUserId)];

    $optionsOther = ['projection' => ['profile' => 1]];

    $queryOther = new MongoDB\Driver\Query($filterOther, $optionsOther);

    // Adjust the collection name if needed (e.g. "swipe_chat_play.users")

    $cursorOther = $mongo->executeQuery("swipe_chat_play.users", $queryOther);

    $otherUserDoc = current($cursorOther->toArray());

  

    if (!$otherUserDoc || !isset($otherUserDoc->profile)) {

        http_response_code(404);

        echo json_encode(["error" => "Other user profile not found"]);

        exit;

    }

  

    $profile = $otherUserDoc->profile;

  

    // -----------------------------------------------------------

    // 7. Format the response in the same structure as get_profile.php

    // -----------------------------------------------------------

    // Modify these fields as needed to match your data structure.

    $response = [

        'topSection' => [

            'profileImages' => $profile->images ?? [],

            'name' => $profile->about_you->name ?? 'Unknown',

            'age' => $profile->about_you->age ?? 'Unknown',

            'infoBubbles' => [

                $profile->about_you->gender ?? 'Unknown',

                $profile->about_you->spokenLanguage ?? 'Unknown',

                $profile->about_you->religion ?? 'Unknown',

                $profile->about_you->politics ?? 'Unknown',

            ],

        ],

        'elements' => [

            '1' => [

                'type' => 'icebreaker',

                'content' => [

                    'question' => 'Wenn ich ein Tier wäre, dann wäre ich ein …',

                    'answer'   => $profile->questions_and_answers[0]->answer ?? '',

                ],

            ],

            '2' => [

                'type' => 'icebreaker',

                'content' => [

                    'question' => 'Ein Ort, an den ich immer wieder zurückkehre...',

                    'answer'   => $profile->questions_and_answers[1]->answer ?? '',

                ],

            ],

            '3' => [

                'type' => 'icebreaker',

                'content' => [

                    'question' => 'Etwas, das ich gerne lernen möchte...',

                    'answer'   => $profile->questions_and_answers[2]->answer ?? '',

                ],

            ],

            '4' => [

                'type' => 'bubbles',

                'content' => [

                    'title' => 'Open to',

                    'bubbles' => $profile->open_to ?? [],

                ],

            ],

        ],

    ];

  

    echo json_encode($response, JSON_PRETTY_PRINT);

  

} catch (Exception $e) {

    error_log("Profile load error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode([

        "error" => "Server error",

        "message" => $e->getMessage()

    ]);

}

?>
```
- myChats.php
```
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

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/my_chats_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    // Check for Authorization header

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    $jwtToken = $matches[1];

  

    // Decode the JWT token to get user ID

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $userId = $decodedToken->user_id ?? null;

  

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // Connect to MongoDB

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // Query to find the user

    $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

    $query = new MongoDB\Driver\Query($filter);

  

    // Execute the query

    $cursor = $mongoManager->executeQuery("swipe_chat_play.users", $query);

    $user = $cursor->toArray();

  

    if (empty($user)) {

        http_response_code(404);

        echo json_encode(["error" => "User not found"]);

        exit;

    }

  

    // Extract chats from user data

    $user = $user[0];

    $chats = $user->chats ?? [];

  

    // Format the response

    $formattedChats = [];

    foreach ($chats as $chatId => $chatData) {

        $formattedChats[] = [

            "chat_id" => $chatId,

            "profileURL" => $chatData->profileURL ?? "",

            "receiver" => $chatData->receiver ?? "",

            "last_message" => $chatData->last_message ?? "",

            "unread" => $chatData->unread ?? 0,

        ];

    }

  

    echo json_encode($formattedChats);

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode([

        "error" => "Internal server error",

    ]);

}

?>
```
- search_user.php
```
<?php

require '../config.php'; // MongoDB connection configuration

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["success" => false, "message" => "Method not allowed"]);

    exit;

}

  

$input = file_get_contents('php://input');

$data = json_decode($input, true);

  

if (!isset($data['search']) || empty($data['search'])) {

    http_response_code(400);

    echo json_encode(["success" => false, "message" => "Missing 'search' key in request payload."]);

    exit;

}

  

$searchTerm = $data['search'];

  

try {

    $manager = new MongoDB\Driver\Manager($mongoURI);

    $namespace = "swipe_chat_play.users";

  

    $query = new MongoDB\Driver\Query([]);

    $cursor = $manager->executeQuery($namespace, $query);

    $users = $cursor->toArray();

  

    if (!$users) {

        echo json_encode([]);

        exit;

    }

  

    $results = [];

    foreach ($users as $user) {

        $username = $user->username ?? '';

        $profilePicture = $user->profile_picture ?? null;

  

        similar_text($searchTerm, $username, $percent);

        if ($percent >= 70) {

            $results[] = [

                "nickname" => $username,

                "profile_picture" => $profilePicture

            ];

        }

    }

  

    usort($results, function ($a, $b) use ($searchTerm) {

        similar_text($searchTerm, $b['nickname'], $percentB);

        similar_text($searchTerm, $a['nickname'], $percentA);

        return $percentB <=> $percentA;

    });

  

    echo json_encode($results);

} catch (MongoDB\Driver\Exception\Exception $e) {

    error_log("MongoDB error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["success" => false, "message" => "Internal server error"]);

}

?>
```
- sendMessage.php
```
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

  

    $jwtToken  = $matches[1];

    $decoded   = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $senderId  = $decoded->user_id ?? null;

  

    if (!$senderId) {

        http_response_code(401);

        echo json_encode(["status" => "error", "message" => "Invalid token"]);

        exit;

    }

  

    // 2) Parse input JSON

    $input     = json_decode(file_get_contents('php://input'), true);

    $chatId    = $input['chat_id'] ?? null;

    $timestamp = $input['timestamp'] ?? null;

    // 'type' can be "text" or "react_bubble", etc. default to "text"

    $type      = $input['type'] ?? 'text';

    // For "text" type we read 'message', else we read 'content'

    $message   = $input['message']  ?? null;

    $content   = $input['content']  ?? [];

  

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

        "sender_id"  => $senderId,

        "type"       => $type,

        "timestamp"  => $timestamp,

        "status"     => "delivered"

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
```
- sendReaction.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$errorLogFile = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/send_reaction_debug.log';

ini_set('log_errors', 1);

ini_set('error_log', $errorLogFile);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    // 1) Validate JWT

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

  

    // 2) Parse JSON

    $input = json_decode(file_get_contents('php://input'), true);

    $receiverId = $input['receiver_id'] ?? null;

    $timestamp = $input['timestamp'] ?? null;

    $type = $input['type'] ?? 'reaction';

    $message = $input['message'] ?? null;

  

    if (!$receiverId || !$timestamp || !$message) {

        http_response_code(400);

        echo json_encode(["status" => "error", "message" => "Missing required fields"]);

        exit;

    }

  

    // 3) Connect to Mongo

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // 4) Check or create match doc

    $queryFilter = [

        "users.$senderId"   => ['$exists' => true],

        "users.$receiverId" => ['$exists' => true]

    ];

    $query = new MongoDB\Driver\Query($queryFilter);

    $cursor = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

    $matchDocs = $cursor->toArray();

    $matchDoc = count($matchDocs) > 0 ? $matchDocs[0] : null;

    $bulkMatch = new MongoDB\Driver\BulkWrite();

  

    if ($matchDoc) {

        // Mark sender's swipe as yes

        $updateField = "users.$senderId";

        $bulkMatch->update(

            [ "_id" => $matchDoc->_id ],

            [ '$set' => [ $updateField => "yes" ] ],

            [ 'multi' => false, 'upsert' => false ]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatch);

  

        // If chat_id already set, error out

        if (isset($matchDoc->chat_id)) {

            echo json_encode(["status" => "error", "message" => "Chat already exists"]);

            exit;

        }

    } else {

        // No doc => create new

        $newMatchDoc = [

            "users" => [

                $senderId   => "yes",

                $receiverId => null

            ]

        ];

        $bulkMatchNew = new MongoDB\Driver\BulkWrite();

        $bulkMatchNew->insert($newMatchDoc);

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatchNew);

    }

  

    // 5) Create chat doc

    $chatData = [

        "messages" => new stdClass(),

        "participants" => [ $senderId, $receiverId ]

    ];

    $bulkChat = new MongoDB\Driver\BulkWrite();

    $chatId = $bulkChat->insert($chatData);

    $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkChat);

    $chatIdString = (string)$chatId;

  

    // 6) Insert reaction message

    $messageId = uniqid();

    $newMessage = [

        "sender_id" => $senderId,

        "type"      => $type,

        "timestamp" => $timestamp,

        "status"    => "delivered",

        "message"   => $message

    ];

    $bulkMessage = new MongoDB\Driver\BulkWrite();

    $bulkMessage->update(

        [ "_id" => new ObjectId($chatIdString) ],

        [ '$set' => [ "messages.$messageId" => $newMessage ] ],

        [ 'upsert' => false ]

    );

    $resultMessage = $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkMessage);

    if ($resultMessage->getModifiedCount() <= 0 && $resultMessage->getUpsertedCount() <= 0) {

        echo json_encode(["status" => "error", "message" => "Failed to send reaction"]);

        exit;

    }

  

    // 7) Update user docs

    // ...

    // 8) Update match doc with chat_id

    // ...

    // Return success

  

    echo json_encode([

        "status" => "sent",

        "chat_id" => $chatIdString,

        // etc.

    ]);

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["status" => "error", "message" => "Internal server error"]);

}
```
- sendVoiceMessage.php
```
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

$errorLogFile = __DIR__ . '/logs/send_voicemessage_debug.log';

ini_set('log_errors', 1);

ini_set('error_log', $errorLogFile);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    error_log("sendVoiceMessage.php started");

  

    // 1) Check Bearer token

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

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

    $chatId    = $_POST['chat_id']   ?? null;

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

    $fileName    = $_FILES['voice_file']['name'];

    $fileNewName = uniqid("voice_", true) . "_" . $fileName;

    $uploadFolder = __DIR__ . "/uploads/voicememos"; // Adjust path if needed

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

  

    // Construct the URL referencing the newly saved file:

    // Adjust $domain as needed for your environment

    $domain    = "http://node02.krasserserver.com:8002/swipe_chatt_play_api";

    $voiceUrl  = "$domain/chat/uploads/voicememos/$fileNewName";

  

    // 4) Connect to MongoDB, insert the message into the chat document

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

    $bulk        = new MongoDB\Driver\BulkWrite();

    $messageId   = uniqid();

  

    $newMessage = [

        "sender_id" => $senderId,

        "type"      => "voicemessage",

        "timestamp" => $timestamp,

        "status"    => "delivered",

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

  

    // 6) Return JSON that the Flutter app can parse immediately

    echo json_encode([

        "status"    => "sent",

        "voice_url" => $voiceUrl

    ]);

    error_log("Voice message saved, chat_id=$chatId message_id=$messageId => $voiceUrl");

  

} catch (Exception $e) {

    error_log("Exception: " . $e->getMessage());

    http_response_code(500);

    echo json_encode([

        "status"  => "error",

        "message" => "Internal server error: " . $e->getMessage()

    ]);

}
```
- add_answer.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php'; // Include MongoDB configuration

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Error logging setup

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/add_answer_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

error_log("add_answer.php script started");

  

header('Content-Type: application/json');

  

// Only allow POST requests

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

// Retrieve the JWT from the Authorization header

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

    error_log("Authorization header missing or invalid");

    http_response_code(401);

    echo json_encode(["error" => "Unauthorized"]);

    exit;

}

  

$jwtToken = $matches[1];

  

try {

    // Decode the JWT

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $userId = $decodedToken->user_id ?? null;

  

    if (!$userId) {

        error_log("Invalid token: user ID missing");

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // Get and validate request body

    $data = json_decode(file_get_contents('php://input'), true);

    if (!isset($data['about_you'])) {

        error_log("Validation failed: Missing about_you data");

        http_response_code(400);

        echo json_encode(["error" => "Missing about_you data"]);

        exit;

    }

  

    // Convert age to integer if it exists

    if (isset($data['about_you']['age'])) {

        $data['about_you']['age'] = (int)$data['about_you']['age'];

    }

  

    error_log("Connecting to MongoDB using URI: $mongoURI");

  

    // Connect to MongoDB

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // Filter for the user document by _id

    $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

  

    /*

     * The update uses MongoDB dot notation to update "profile.about_you".

     * - If the "profile" field does not exist, it is automatically created as an object.

     * - If "profile" exists and contains other fields, those remain untouched.

     */

    $update = [

        '$set' => [

            "profile.about_you" => $data['about_you']

        ]

    ];

  

    // Upsert is enabled so that if the user document doesn't exist (or doesn't have a profile), it is created.

    $options = ['multi' => false, 'upsert' => true];

  

    // Perform the update

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update($filter, $update, $options);

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

    // Return a response with a "success" field

    if ($result->getUpsertedCount() > 0) {

        echo json_encode([

            "success" => true,

            "message" => "Profile created successfully"

        ]);

    } elseif ($result->getModifiedCount() > 0) {

        echo json_encode([

            "success" => true,

            "message" => "Profile updated successfully"

        ]);

    } else {

        echo json_encode([

            "success" => true,

            "message" => "No changes needed"

        ]);

    }

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

}

  

error_log("add_answer.php script finished");
```
- create_testusers.php
```
<?php

/**

 * generate_random_users.php

 *

 * This script generates 10,000 random user documents with a structure

 * similar to the example you provided and inserts them into the

 * `swipe_chat_play.users` collection in MongoDB.

 */

  

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php'; // Include MongoDB configuration

  

// Connect to MongoDB using the $mongoURI from config.php

$manager = new MongoDB\Driver\Manager($mongoURI);

  

// Define lists of potential values

$names = ["Alice", "Bob", "Charlie", "Diana", "Eve", "Frank", "Greta", "Henry", "Iris", "Jack", "Katrina", "Leo", "Mona", "Nina", "Oscar", "Petra", "Quinn", "Rafael", "Sara", "Thomas", "Uma", "Vince", "Wendy", "Xander", "Yara", "Zane"];

$genders = ["Male", "Female", "Non Binary", "Transgender", "Genderqueer", "Other"];

$languages = ["German", "English", "French", "Italian"];

$religions = ["Christianity", "Islam", "Hinduism", "Buddhism", "Judaism", "Other"];

$politics = ["Liberal", "Conservative", "Centrist", "Libertarian", "Socialist"];

  

// Possible images

$possibleImages = [

    "http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c4093a0_image_cropper_1740063386927.jpg",

    "http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b23f_image_cropper_1740063403624.jpg",

    "http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b3d8_image_cropper_1740063416124.jpg",

    "http://node02.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/67b742c40b52c_image_cropper_1740063423203.jpg"

];

  

// Pool of activities for "open_to"

$openToActivities = [

    "jogging", "snowshoeing", "stand-up paddleboarding", "trail running", "scuba diving",

    "whitewater rafting", "games", "cooking", "puzzles", "Arts & Crafts", "board games",

    "horseback riding", "beach volleyball", "zip-lining", "surfing", "fishing", "camping",

    "kayaking", "rock climbing", "birdwatching", "hiking", "biking", "movies", "yoga",

    "language exchange", "mixology", "card games", "knitting", "painting", "karaoke",

    "pottery", "woodworking", "homebrewing", "dance classes"

];

  

// Sample Q&A

$questionPool = [

    "My favorite holiday and why...",

    "A place I return to often...",

    "If I were an animal, I would be...",

    "What is your favorite food?",

    "What is your dream job?",

    "Describe your perfect day."

];

$answerPool = [

    "I love celebrating every day!",

    "I always love going back to my hometown.",

    "I'd probably be an eagle, so I could fly freely.",

    "Pizza! I could eat it every day.",

    "I've always wanted to be an astronaut.",

    "My perfect day starts with sunshine and ends with a good book."

];

  

// Define criteria for "searching_for"

$searchGenders = ["Non Binary", "Transgender", "Genderqueer", "Female", "Other"];

$searchAgeRange = [18, 66];

  

/**

 * Generates a random location within a certain radius (in km) from a given center point.

 *

 * @param float $centerLat Latitude of the center

 * @param float $centerLon Longitude of the center

 * @param float $maxDistanceKm Maximum distance from the center in kilometers

 * @return array [longitude, latitude] for GeoJSON

 */

function generateRandomLocation($centerLat, $centerLon, $maxDistanceKm = 50) {

    // Random distance in [0, $maxDistanceKm)

    $distance = lcg_value() * $maxDistanceKm;

    // Random bearing in [0, 2*pi)

    $bearing = lcg_value() * 2 * M_PI;

  

    // Approximate degrees per km

    $latDegreesPerKm = 1 / 111;

    $lonDegreesPerKm = 1 / (111 * cos(deg2rad($centerLat)));

  

    // Convert distance to degrees

    $latOffset = $distance * $latDegreesPerKm;

    $lonOffset = $distance * $lonDegreesPerKm;

  

    // Apply bearing

    $deltaLat = $latOffset * cos($bearing);

    $deltaLon = $lonOffset * sin($bearing);

  

    // New latitude and longitude

    $newLat = $centerLat + $deltaLat;

    $newLon = $centerLon + $deltaLon;

  

    return [$newLon, $newLat]; // Return in [longitude, latitude] format for GeoJSON

}

  

// Prepare a bulk write operation

$bulk = new MongoDB\Driver\BulkWrite();

  

// Generate 10,000 users

for ($i = 0; $i < 100; $i++) {

    $randomName = $names[array_rand($names)];

    $randomAge = rand(18, 50);

    $randomGender = $genders[array_rand($genders)];

    $randomLang = $languages[array_rand($languages)];

    $randomReligion = $religions[array_rand($religions)];

    $randomPolitics = $politics[array_rand($politics)];

  

    // Create username, firstname, etc.

    $username = strtolower($randomName) . "_" . $i;

    $firstname = $randomName;

    $lastname = "Doe_" . $i;

    $email = strtolower($randomName) . "_" . $i . "@example.com";

    $passwordHash = password_hash("password123", PASSWORD_BCRYPT);

  

    // Pick random images

    $numImages = rand(1, count($possibleImages));

    shuffle($possibleImages);

    $chosenImages = array_slice($possibleImages, 0, $numImages);

  

    // Generate random Q&A

    $qaCount = rand(1, 3);

    shuffle($questionPool);

    shuffle($answerPool);

    $questionsAndAnswers = [];

    for ($q = 0; $q < $qaCount; $q++) {

        $questionsAndAnswers[] = [

            "question" => $questionPool[$q],

            "answer" => $answerPool[$q]

        ];

    }

  

    // Generate a random location around Munich (48.135482, 11.573893) within 50 km

    $randomCoordinates = generateRandomLocation(48.134266, 11.581092, 200);

  

    $locationRadius = [

        "location" => [

            "type" => "Point",

            "coordinates" => $randomCoordinates

        ],

        "radius" => rand(300, 2000) // random radius in meters

    ];

  

    // Select random activities (5 picks)

    shuffle($openToActivities);

    $chosenOpenTo = array_slice($openToActivities, 0, 5);

  

    // Generate searching_for criteria (ensuring at least 1 gender)

    shuffle($searchGenders);

    // Slice a random subset of genders, ensuring at least 1

    $chosenSearchGenders = array_slice($searchGenders, 0, rand(1, count($searchGenders)));

  

    // Build the user document

    $userDocument = [

        "username" => $username,

        "firstname" => $firstname,

        "lastname" => $lastname,

        "email" => $email,

        "password_hash" => $passwordHash,

        "verification_code" => null,

        "verification_code_expiry" => null,

        "is_verified" => true,

        "fingerprint_hash" => "",

        "profile" => [

            "about_you" => [

                "name" => $randomName,

                "age" => $randomAge,

                "gender" => $randomGender,

                "spokenLanguage" => $randomLang,

                "religion" => $randomReligion,

                "career" => "Example Career $i",

                "politics" => $randomPolitics

            ],

            "images" => $chosenImages,

            "questions_and_answers" => $questionsAndAnswers,

            "location_radius" => $locationRadius,

            "open_to" => $chosenOpenTo,

            "searching_for" => [

                "genders" => $chosenSearchGenders,

                "ageRange" => $searchAgeRange,

                "religion" => $religions,

                "politics" => $politics

            ]

        ]

    ];

  

    // Add to bulk operation

    $bulk->insert($userDocument);

}

  

// Execute bulk write

try {

    $result = $manager->executeBulkWrite("swipe_chat_play.users", $bulk);

    echo "Inserted " . $result->getInsertedCount() . " user documents.\n";

} catch (MongoDB\Driver\Exception\Exception $e) {

    echo "Error inserting documents: " . $e->getMessage() . "\n";

}
```
- elements_update.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// You can set a custom error log file if desired

$error_log_file = __DIR__ . '/elements_update_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

// Replace with your actual secret key

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

// Parse Authorization header for Bearer token

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

    http_response_code(401);

    echo json_encode(["error" => "Unauthorized"]);

    exit;

}

  

$jwtToken = $matches[1];

  

try {

    // Decode JWT

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $userId = $decodedToken->user_id ?? null;

  

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // Parse incoming JSON

    $data = json_decode(file_get_contents('php://input'), true);

    if (!isset($data['questions_and_answers']) || !is_array($data['questions_and_answers'])) {

        http_response_code(400);

        echo json_encode(["error" => "Invalid request format"]);

        exit;

    }

  

    // Connect to Mongo

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // Build filter for user

    $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

  

    // The data to be updated

    $update = [

        '$set' => [

            "profile.questions_and_answers" => $data['questions_and_answers']

        ]

    ];

  

    // Execute update

    $options = ['multi' => false, 'upsert' => true];

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update($filter, $update, $options);

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

    echo json_encode([

        "success" => true,

        "message" => "Questions and answers updated successfully"

    ]);

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

}
```
- get_profile.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    // Get JWT token from headers

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    $jwt = $matches[1];

    $decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

    $userId = $decoded->user_id ?? null;

  

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // Connect to MongoDB

    $mongo = new MongoDB\Driver\Manager($mongoURI);

    $filter = ['_id' => new MongoDB\BSON\ObjectId($userId)];

    $options = ['projection' => ['profile' => 1]];

  

    $query = new MongoDB\Driver\Query($filter, $options);

    $cursor = $mongo->executeQuery('swipe_chat_play.users', $query);

    $user = current($cursor->toArray());

  

    if (!$user || !isset($user->profile)) {

        http_response_code(404);

        echo json_encode(["error" => "User profile not found"]);

        exit;

    }

  

    $profile = $user->profile;

  

    // Transform MongoDB profile data into the expected JSON structure

    // NOTE: The keys are now `topSection` and `elements` so it matches your Dart model.

    $response = [

        'topSection' => [

            'profileImages' => $profile->images ?? [],

            'name' => $profile->about_you->name ?? 'Unknown',

            'age' => $profile->about_you->age ?? 'Unknown',

            'infoBubbles' => [

                $profile->about_you->gender ?? 'Unknown',

                $profile->about_you->spokenLanguage ?? 'Unknown',

                $profile->about_you->religion ?? 'Unknown',

                $profile->about_you->politics ?? 'Unknown',

            ],

        ],

        'elements' => [

            '1' => [

                'type' => 'icebreaker',

                'content' => [

                    'question' => 'Wenn ich ein Tier wäre, dann wäre ich ein …',

                    'answer' => $profile->questions_and_answers[0]->answer ?? 'No answer provided',

                ],

            ],

            '2' => [

                'type' => 'icebreaker',

                'content' => [

                    'question' => 'Ein Ort, an den ich immer wieder zurückkehre...',

                    'answer' => $profile->questions_and_answers[1]->answer ?? 'No answer provided',

                ],

            ],

            '3' => [

                'type' => 'icebreaker',

                'content' => [

                    'question' => 'Etwas, das ich gerne lernen möchte...',

                    'answer' => $profile->questions_and_answers[2]->answer ?? 'No answer provided',

                ],

            ],

            '4' => [

                'type' => 'bubbles',

                'content' => [

                    'title' => 'Open to',

                    'bubbles' => $profile->open_to ?? [],

                ],

            ],

        ],

    ];

  

    echo json_encode($response, JSON_PRETTY_PRINT);

  

} catch (Exception $e) {

    error_log("Profile load error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Server error", "message" => $e->getMessage()]);

}
```
- get_userdata.php
```
<?php

// get_userdata.php  – returns the **full editable profile**

// JWT in Authorization header (“Bearer <token>”)

// response: 200 + JSON   or   401/500

  

require '../vendor/autoload.php';           // composer libs (firebase/php-jwt, mongodb)

require_once '../config/database.php';      // -> $mongoClient

require_once '../helpers/jwt.php';          // verifyJwt($token) => userId|false

  

header('Content-Type: application/json; charset=utf-8');

  

try {

    $headers = getallheaders();

    if (empty($headers['Authorization'])) throw new Exception('No token', 401);

  

    [$type, $token] = explode(' ', $headers['Authorization'], 2);

    if (strcasecmp($type,'Bearer') !== 0)  throw new Exception('Bad header', 401);

  

    $uid = verifyJwt($token);               // throws on failure

    if (!$uid) throw new Exception('Bad token', 401);

  

    $users = (new MongoDB\Client($MONGO_DSN))->swipe_chat_play->users;

    $user  = $users->findOne(

      ['_id' => new MongoDB\BSON\ObjectId($uid)],

      ['projection' => ['profile' => 1]]

    );

  

    if (!$user)  throw new Exception('User not found', 404);

  

    echo json_encode($user['profile'], JSON_UNESCAPED_SLASHES);

    exit;

  

} catch(Exception $e) {

    http_response_code($e->getCode() ?: 500);

    echo json_encode(['error'=>$e->getMessage()]);

    exit;

}

?>
```
- load_profile.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    $jwt = $matches[1];

    $decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

    $userId = $decoded->user_id ?? null;

  

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    $mongo = new MongoDB\Driver\Manager($mongoURI);

    $filter = ['_id' => new MongoDB\BSON\ObjectId($userId)];

    $options = ['projection' => ['profile' => 1]];

  

    $query = new MongoDB\Driver\Query($filter, $options);

    $cursor = $mongo->executeQuery('swipe_chat_play.users', $query);

    $user = current($cursor->toArray());

  

    if (!$user || !isset($user->profile)) {

        http_response_code(404);

        echo json_encode(["error" => "User profile not found"]);

        exit;

    }

  

    $profile = $user->profile;

    $response = [

        'profile' => [

            'about_you' => $profile->about_you ?? [],

            'images' => $profile->images ?? [],

            'open_to' => $profile->open_to ?? [],

            'location_radius' => [

                'location' => [

                    'type' => 'Point',

                    'coordinates' => $profile->location_radius->location->coordinates ?? [0, 0]

                ],

                'radius' => $profile->location_radius->radius ?? 0

            ],

            'questions_and_answers' => $profile->questions_and_answers ?? []

        ]

    ];

  

    $response['profile']['location_radius']['location']['coordinates'] = array_map(

        fn($coord) => (float)$coord,

        $response['profile']['location_radius']['location']['coordinates']

    );

  

    echo json_encode($response, JSON_PRETTY_PRINT);

  

} catch (Exception $e) {

    error_log("Profile load error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Server error", "message" => $e->getMessage()]);

}
```
- retrieve_profile.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php'; // MongoDB configuration

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Error logging setup

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/retrieve_profile_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

error_log("retrieve_profile.php script started");

  

// We expect a GET request to retrieve the profile

if ($_SERVER['REQUEST_METHOD'] !== 'GET') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

// Set JSON header

header('Content-Type: application/json');

  

// Retrieve the JWT from the Authorization header

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

    error_log("Authorization header missing or invalid");

    http_response_code(401);

    echo json_encode(["error" => "Unauthorized"]);

    exit;

}

  

$jwtToken = $matches[1];

  

try {

    // Decode the JWT (must match the algorithm you used in add_answer.php)

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    // Same field name you used when encoding the token in your login process:

    $userId = $decodedToken->user_id ?? null;

  

    if (!$userId) {

        error_log("Invalid token: user ID missing");

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // Connect to MongoDB

    error_log("Connecting to MongoDB using URI: $mongoURI");

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // Find the user by _id

    $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

    $options = [];

    $query = new MongoDB\Driver\Query($filter, $options);

    $cursor = $mongoManager->executeQuery("swipe_chat_play.users", $query);

    $userDocument = current($cursor->toArray());

  

    if (!$userDocument) {

        error_log("User not found with _id: $userId");

        http_response_code(404);

        echo json_encode(["error" => "User not found"]);

        exit;

    }

  

    // If the user has no 'profile', return a minimal structure or error

    if (!isset($userDocument->profile)) {

        error_log("User has no profile field");

        http_response_code(404);

        echo json_encode(["error" => "No profile found"]);

        exit;

    }

  

    $profile = $userDocument->profile;

    // --- Build your final JSON response based on user data ---

    // For example:

    $profileImages = $profile->images ?? [];

    $username      = $profile->about_you->name ?? "| add username";

    $age           = $profile->about_you->age ?? "how old are you?";

    // 'open_to' is an array => concatenate

    $searching     = isset($profile->open_to) && is_array($profile->open_to)

        ? implode(', ', $profile->open_to)

        : "what are you looking for?";

  

    // Prepare the main JSON structure

    $response = [

        "profile_images" => $profileImages,

        "username"       => $username,

        "age"            => $age,

        "searching"      => $searching,

        "element"        => []

    ];

  

    // If 'questions_and_answers' exist, map them to "icebreaker" elements

    $elementIndex = 1;

    if (isset($profile->questions_and_answers) && is_array($profile->questions_and_answers)) {

        foreach ($profile->questions_and_answers as $qa) {

            $question = $qa->question ?? "| add subtitle";

            $answer   = $qa->answer ?? "Click here to add some text about yourself.";

  

            $response["element"][(string) $elementIndex] = [

                "type"     => "icebreaker",

                // match your Flutter layout: 'question' vs 'answer', or 'content' vs 'subtitle'

                "content"  => $answer,

                "subtitle" => $question

            ];

            $elementIndex++;

        }

    }

  

    // Output the final JSON

    echo json_encode($response, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE);

    error_log("retrieve_profile.php script finished");

    exit;

  

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

    exit;

}
```
- update_loc.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

  

// Only allow POST

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

  http_response_code(405);

  echo json_encode(["error" => "Method not allowed"]);

  exit;

}

  

// Get JWT from header

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

  http_response_code(401);

  echo json_encode(["error" => "Unauthorized"]);

  exit;

}

  

$jwtToken = $matches[1];

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

try {

  // Decode token

  $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

  $userId = $decodedToken->user_id ?? null;

  if (!$userId) {

    throw new Exception("Invalid token");

  }

  

  // Get input data

  $data = json_decode(file_get_contents('php://input'), true);

  // Validate required fields

  if (!isset($data['latitude'], $data['longitude'], $data['dating_radius'])) {

    http_response_code(400);

    echo json_encode(["error" => "Missing location data"]);

    exit;

  }

  

  // Connect to MongoDB

  $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

  // Prepare update data

  $updateData = [

    'location' => [

      'type' => 'Point',

      'coordinates' => [

        (float)$data['longitude'],

        (float)$data['latitude']

      ]

    ],

    'radius' => (float)$data['dating_radius']

  ];

  

  // Update user document

  $bulk = new MongoDB\Driver\BulkWrite();

  $bulk->update(

    ['_id' => new MongoDB\BSON\ObjectId($userId)],

    ['$set' => ["profile.location_radius" => $updateData]],

    ['multi' => false, 'upsert' => true]

  );

  

  $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

  echo json_encode([

    "success" => true,

    "message" => "Location and radius updated"

  ]);

  

} catch (Exception $e) {

  http_response_code(401);

  echo json_encode(["error" => $e->getMessage()]);

}
```
- update_open_to.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$error_log_file = __DIR__ . '/update_open_to_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

    http_response_code(401);

    echo json_encode(["error" => "Unauthorized"]);

    exit;

}

  

$jwtToken = $matches[1];

  

try {

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $userId = $decodedToken->user_id ?? null;

  

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    $data = json_decode(file_get_contents('php://input'), true);

    if (!isset($data['open_to']) || !is_array($data['open_to'])) {

        http_response_code(400);

        echo json_encode(["error" => "Invalid request format"]);

        exit;

    }

  

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

    $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

    $update = [

        '$set' => [

            "profile.open_to" => $data['open_to']

        ]

    ];

  

    $options = ['multi' => false, 'upsert' => true];

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update($filter, $update, $options);

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

    echo json_encode([

        "success" => true,

        "message" => "Open to preferences updated successfully"

    ]);

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

}
```
- update_searching_for.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$error_log_file = __DIR__ . '/update_searching_for_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

// IMPORTANT: use your actual secret key

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

// Extract Bearer token from Authorization header

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

if (!$authHeader || !preg_match('/Bearer\\s(\\S+)/', $authHeader, $matches)) {

    http_response_code(401);

    echo json_encode(["error" => "Unauthorized"]);

    exit;

}

$jwtToken = $matches[1];

  

try {

    // Decode the token

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $userId = $decodedToken->user_id ?? null;

  

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token"]);

        exit;

    }

  

    // Get JSON from request body

    $data = json_decode(file_get_contents('php://input'), true);

    if (!isset($data['genders']) || !isset($data['ageRange'])

        || !isset($data['religion']) || !isset($data['politics'])) {

        http_response_code(400);

        echo json_encode(["error" => "Missing required fields"]);

        exit;

    }

  

    // Connect to Mongo

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    // Define filter to find user by _id

    $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

  

    // Prepare the update

    $update = [

        '$set' => [

            "profile.searching_for" => [

                "genders"   => $data['genders'],

                "ageRange"  => $data['ageRange'],

                "religion"  => $data['religion'],

                "politics"  => $data['politics']

            ]

        ]

    ];

  

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update($filter, $update, ['multi' => false, 'upsert' => true]);

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

    echo json_encode([

        "success" => true,

        "message" => "Searching-for filters updated successfully"

    ]);

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

}
```
- update_userdata.php
```
<?php

declare(strict_types=1);

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json; charset=utf-8');

  

/*─────────────────────────────────────────────────────*

 * 1.  Method check

 *─────────────────────────────────────────────────────*/

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(['success' => false, 'error' => 'Method not allowed']);

    exit;

}

  

/*─────────────────────────────────────────────────────*

 * 2.  JWT auth  (Bearer <token>, HS256, claim user_id)

 *─────────────────────────────────────────────────────*/

$authHeader = $_SERVER['HTTP_AUTHORIZATION']

           ?? ($_SERVER['REDIRECT_HTTP_AUTHORIZATION'] ?? '');

  

if (!preg_match('/Bearer\s(\S+)/', $authHeader, $m)) {

    http_response_code(401);

    echo json_encode(['success' => false, 'error' => 'Missing token']);

    exit;

}

  

try {

    $decoded = JWT::decode($m[1], new Key($secretKey, 'HS256'));

    $userId  = $decoded->user_id ?? null;

    if (!$userId) throw new Exception('Token has no user_id');

} catch (Throwable $e) {

    http_response_code(401);

    echo json_encode(['success' => false, 'error' => 'Invalid token']);

    exit;

}

  

/*─────────────────────────────────────────────────────*

 * 3.  Parse JSON body

 *─────────────────────────────────────────────────────*/

$payload = json_decode(file_get_contents('php://input'), true);

if (!is_array($payload)) {

    http_response_code(400);

    echo json_encode(['success' => false, 'error' => 'Invalid JSON']);

    exit;

}

  

/*─────────────────────────────────────────────────────*

 * 4.  Build $set only for supplied keys

 *     – nested objects merged field-by-field

 *─────────────────────────────────────────────────────*/

$set = [];

  

/* helper for nested objects */

$mergeNested = function (string $prefix, array $data) use (&$set) {

    foreach ($data as $k => $v) {

        $set["$prefix.$k"] = $v;

    }

};

  

/* about_you ─ nested object */

if (isset($payload['about_you']) && is_array($payload['about_you'])) {

    $mergeNested('profile.about_you', $payload['about_you']);

}

  

/* images – full array replace */

if (array_key_exists('images', $payload) && is_array($payload['images'])) {

    $set['profile.images'] = $payload['images'];

}

  

/* questions_and_answers – full array replace */

if (array_key_exists('questions_and_answers', $payload)

    && is_array($payload['questions_and_answers'])) {

    $set['profile.questions_and_answers'] = $payload['questions_and_answers'];

}

  

/* location_radius – nested object */

if (isset($payload['location_radius']) && is_array($payload['location_radius'])) {

    $mergeNested('profile.location_radius', $payload['location_radius']);

}

  

/* open_to – full array replace */

if (array_key_exists('open_to', $payload) && is_array($payload['open_to'])) {

    $set['profile.open_to'] = $payload['open_to'];

}

  

/* searching_for – nested object */

if (isset($payload['searching_for']) && is_array($payload['searching_for'])) {

    $mergeNested('profile.searching_for', $payload['searching_for']);

}

  

if (!$set) {

    echo json_encode(['success' => false, 'error' => 'Nothing to update']);

    exit;

}

  

/*─────────────────────────────────────────────────────*

 * 5.  MongoDB update

 *─────────────────────────────────────────────────────*/

try {

    $manager = new MongoDB\Driver\Manager($mongoURI);

  

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update(

        ['_id' => new MongoDB\BSON\ObjectId($userId)],

        ['$set' => $set],

        ['multi' => false, 'upsert' => false]

    );

  

    $result = $manager->executeBulkWrite('swipe_chat_play.users', $bulk);

  

    echo json_encode([

        'success' => true,

        'message' => $result->getModifiedCount() > 0

            ? 'Updated successfully'

            : 'No changes made'

    ]);

} catch (Throwable $e) {

    http_response_code(500);

    echo json_encode([

        'success' => false,

        'error'   => 'Database error: ' . $e->getMessage()

    ]);

}
```
- upload_profile_images.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php'; // Include MongoDB configuration

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

// Enable error reporting (remove in production if needed)

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Directory where images will be uploaded

$uploadDir = __DIR__ . '/uploads/';

if (!file_exists($uploadDir)) {

    mkdir($uploadDir, 0777, true);

}

  

// JWT secret key (must match what your mobile app / other endpoints use)

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA=="; // Replace with your actual secret key

  

header('Content-Type: application/json');

  

// Only accept POST requests

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(['error' => 'Method not allowed. Use POST.']);

    exit;

}

  

// Retrieve the JWT from the Authorization header

$headers = getallheaders();

$authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

    http_response_code(401);

    echo json_encode(["error" => "Unauthorized: Missing or invalid token"]);

    exit;

}

  

$jwtToken = $matches[1];

  

// Decode the JWT

try {

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    $userId = $decodedToken->user_id ?? null;

    if (!$userId) {

        http_response_code(401);

        echo json_encode(["error" => "Invalid token: user ID missing"]);

        exit;

    }

} catch (Exception $e) {

    http_response_code(401);

    echo json_encode(["error" => "Invalid token: " . $e->getMessage()]);

    exit;

}

  

// Connect to MongoDB

try {

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => "DB connection failed: " . $e->getMessage()]);

    exit;

}

  

// Fetch the existing user profile

$filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

$query = new MongoDB\Driver\Query($filter);

$existingProfile = $mongoManager->executeQuery("swipe_chat_play.users", $query)->toArray();

  

// Ensure `profile` is an object, not an array

$profile = isset($existingProfile[0]->profile) && is_object($existingProfile[0]->profile)

    ? (array) $existingProfile[0]->profile

    : [];

  

// Read the existing images field (if present)

$existingImages = isset($profile['images']) && is_array($profile['images'])

    ? $profile['images']

    : [];

  

// Read images JSON field from the request (optional existing images)

$existingImagesJson = $_POST['images'] ?? '[]';

$newImagesFromRequest = json_decode($existingImagesJson, true);

if (!is_array($newImagesFromRequest)) {

    $newImagesFromRequest = [];

}

  

// Process uploaded files

$uploadedImages = [];

  

// If you send images via "image0", "image1", etc.

foreach ($_FILES as $fileKey => $fileInfo) {

    if ($fileInfo['error'] === UPLOAD_ERR_OK) {

        $tmpPath  = $fileInfo['tmp_name'];

        $origName = basename($fileInfo['name']);

        $uniqueName = uniqid() . '_' . $origName;

        $targetPath = $uploadDir . $uniqueName;

  

        if (move_uploaded_file($tmpPath, $targetPath)) {

            // Build a publicly accessible URL

            $protocol = isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] === 'on' ? 'https://' : 'http://';

            $host = $_SERVER['HTTP_HOST'];

            $scriptDir = rtrim(dirname($_SERVER['SCRIPT_NAME']), '/\\');

            $imageUrl = $protocol . $host . $scriptDir . '/uploads/' . $uniqueName;

  

            $uploadedImages[] = $imageUrl;

        }

    }

}

  

// Merge all images: previously stored + sent via request + newly uploaded

$finalImages = array_unique(array_merge($existingImages, $newImagesFromRequest, $uploadedImages));

  

// Ensure `profile` is an object and assign the images array

$profile['images'] = $finalImages;

  

// Update the user's profile in MongoDB

$update = [

    '$set' => [

        "profile" => $profile, // Ensuring profile is an object

    ],

];

  

$options = ['multi' => false, 'upsert' => true];

  

try {

    $bulk = new MongoDB\Driver\BulkWrite();

    $bulk->update($filter, $update, $options);

    $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

    // Return the final images array

    echo json_encode([

        "success" => true,

        "images" => $finalImages

    ]);

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => "DB update failed: " . $e->getMessage()]);

    exit;

}
```
- addFingerprint.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require '../config.php'; // MongoDB connection configuration

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

// Logging setup

$logFile = '/home/container/webroot/swipe_chatt_play_api/logs/add_fingerprint_error.log';

ini_set('log_errors', 1);

ini_set('error_log', $logFile);

  

function logMessage($message) {

    error_log(date('Y-m-d H:i:s') . " - " . $message);

}

  

logMessage("addFingerprint.php script started");

  

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    // Get the Authorization header

    $authHeader = $_SERVER['HTTP_AUTHORIZATION'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        logMessage("Error: Missing or invalid Authorization header.");

        http_response_code(400);

        echo json_encode(["error" => "Authorization header with Bearer token is required"]);

        exit;

    }

  

    $jwt = $matches[1]; // Extract the token from the header

  

    $fingerprint = $_POST['fingerprint'] ?? null;

    if (!$fingerprint) {

        logMessage("Error: Missing fingerprint.");

        http_response_code(400);

        echo json_encode(["error" => "Fingerprint is required"]);

        exit;

    }

  

    try {

        // Decode the JWT token

        $decoded = JWT::decode($jwt, new Key($secretKey, 'HS256'));

        $userId = $decoded->user_id;

        logMessage("Decoded user ID from JWT: $userId");

  

        // Hash the fingerprint

        $fingerprintHash = password_hash($fingerprint, PASSWORD_BCRYPT);

  

        // Connect to MongoDB

        $manager = new MongoDB\Driver\Manager($mongoURI);

  

        // Prepare the MongoDB update query

        $bulk = new MongoDB\Driver\BulkWrite();

        $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

        $update = ['$set' => ["fingerprint_hash" => $fingerprintHash]];

  

        $bulk->update($filter, $update);

  

        $result = $manager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

        if ($result->getModifiedCount() > 0) {

            logMessage("Fingerprint successfully added for user ID: $userId");

            http_response_code(200);

            echo json_encode(["message" => "Fingerprint added successfully"]);

        } else {

            logMessage("Failed to add fingerprint for user ID: $userId");

            http_response_code(400);

            echo json_encode(["error" => "Failed to add fingerprint"]);

        }

    } catch (Firebase\JWT\ExpiredException $e) {

        logMessage("JWT expired: " . $e->getMessage());

        http_response_code(401);

        echo json_encode(["error" => "Token expired. Please login again."]);

    } catch (MongoDB\Driver\Exception\Exception $e) {

        logMessage("MongoDB error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    } catch (Exception $e) {

        logMessage("General error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    logMessage("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}
```
- fingerprintLogin.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require __DIR__ . '/../config.php'; // MongoDB configuration

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

// Logging setup

$logFile = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/fingerprint_login_error.log';

ini_set('log_errors', 1);

ini_set('error_log', $logFile);

  

error_log("fingerprintLogin.php script started");

  

function logMessage($message) {

    $logFile = '/home/container/webroot/logfile.log';

    file_put_contents($logFile, date('Y-m-d H:i:s') . " - " . $message . PHP_EOL, FILE_APPEND);

    error_log($message);

}

  

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $fingerprint = $_POST['fingerprint'] ?? null;

  

    if (!$fingerprint) {

        $errorMsg = "Fingerprint is required";

        logMessage("Error: $errorMsg");

        http_response_code(400);

        echo json_encode(["error" => $errorMsg]);

        exit;

    }

  

    try {

        logMessage("Received fingerprint: $fingerprint");

  

        // Connect to MongoDB

        $manager = new MongoDB\Driver\Manager($mongoURI);

        $namespace = "swipe_chat_play.users";

  

        // Fetch all users and their fingerprint hashes

        $query = new MongoDB\Driver\Query([]);

        $cursor = $manager->executeQuery($namespace, $query);

        $users = $cursor->toArray();

  

        $userId = null;

        foreach ($users as $user) {

            if (empty($user->fingerprint_hash)) {

                // Skip users with no fingerprint hash

                logMessage("Skipping user ID: {$user->_id} due to null/empty fingerprint hash");

                continue;

            }

  

            if (password_verify($fingerprint, $user->fingerprint_hash)) {

                $userId = (string)$user->_id; // Use the ObjectId as the user ID

                break;

            }

        }

  

        if (!$userId) {

            $errorMsg = "Invalid fingerprint";

            logMessage("Error: $errorMsg");

            http_response_code(401);

            echo json_encode(["error" => $errorMsg]);

            exit;

        }

  

        logMessage("Valid fingerprint for user ID: $userId");

  

        // Generate JWT

        $payload = [

            "iss" => "http://panel.krasserserver.com",

            "aud" => "http://panel.krasserserver.com",

            "iat" => time(),

            "exp" => time() + (60 * 60), // Token expires in 1 hour

            "user_id" => $userId

        ];

        $jwt = JWT::encode($payload, $secretKey, 'HS256');

  

        logMessage("JWT generated successfully for user ID: $userId");

  

        // Respond with token

        http_response_code(200);

        echo json_encode(["token" => $jwt]);

    } catch (MongoDB\Driver\Exception\Exception $e) {

        $errorMsg = "MongoDB error: " . $e->getMessage();

        logMessage("Error: $errorMsg");

        http_response_code(500);

        echo json_encode(["error" => $errorMsg]);

    } catch (Exception $e) {

        $errorMsg = "Internal server error: " . $e->getMessage();

        logMessage("Error: $errorMsg");

        http_response_code(500);

        echo json_encode(["error" => $errorMsg]);

    }

} else {

    $errorMsg = "Method not allowed";

    logMessage("Error: $errorMsg");

    http_response_code(405);

    echo json_encode(["error" => $errorMsg]);

}
```
- login.php
```
<?php

use PHPMailer\PHPMailer\PHPMailer;

use PHPMailer\PHPMailer\Exception;

use MongoDB\BSON\UTCDateTime;

use MongoDB\Driver\Exception\Exception as MongoDBException;

  

require __DIR__ . '/../../vendor/autoload.php';

require '../config.php'; // MongoDB configuration

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set the error log file path

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/login_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

error_log("login.php script started");

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    $email = $_POST['email'] ?? null;

    $password = $_POST['password'] ?? null;

  

    if (!$email || !$password) {

        error_log("Missing email or password");

        http_response_code(400);

        echo json_encode(["error" => "Email and password are required"]);

        exit;

    }

  

    try {

        // Connect to MongoDB

        $manager = new MongoDB\Driver\Manager($mongoURI);

        $namespace = "swipe_chat_play.users";

  

        // Fetch user by email

        $filter = ["email" => $email];

        $query = new MongoDB\Driver\Query($filter);

        $cursor = $manager->executeQuery($namespace, $query);

        $user = current($cursor->toArray());

  

        if ($user && password_verify($password, $user->password_hash)) {

            // Generate verification code and expiry time

            $verificationCode = str_pad(random_int(0, 9999), 4, '0', STR_PAD_LEFT); // 4-digit code

            $expiry = new UTCDateTime((time() + 900) * 1000); // 15 minutes from now

  

            // Update the user document with the verification code and expiry

            $bulk = new MongoDB\Driver\BulkWrite();

            $bulk->update(

                ["_id" => $user->_id],

                ['$set' => ["verification_code" => $verificationCode, "verification_code_expiry" => $expiry]]

            );

            $manager->executeBulkWrite($namespace, $bulk);

  

            // Send verification email using PHPMailer

            $mail = new PHPMailer(true);

  

            try {

                // SMTP server configuration

                $mail->isSMTP();

                $mail->Host = 'smtp.strato.de'; // Replace with your SMTP host

                $mail->SMTPAuth = true;

                $mail->Username = 'owee@fahrschule-peppermint.com'; // Replace with your email

                $mail->Password = '(0+1)+(0+1)'; // Replace with your email password

                $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;

                $mail->Port = 587; // Replace with your SMTP port

  

                // Email settings

                $mail->setFrom('owee@fahrschule-peppermint.com', 'Luna'); // Replace with sender name

                $mail->addAddress($email); // Recipient email

                $mail->Subject = 'Your Login Verification Code';

                $mail->Body = "Hello,\n\nYour verification code is: $verificationCode\n\nThis code is valid for 15 minutes.";

  

                // Send the email

                if ($mail->send()) {

                    error_log("Verification code sent to $email");

                    http_response_code(200);

                    echo json_encode(["message" => "Verification code sent"]);

                }

            } catch (Exception $e) {

                error_log("Email send failed: " . $mail->ErrorInfo);

                http_response_code(500);

                echo json_encode(["error" => "Could not send verification code. Error: " . $mail->ErrorInfo]);

            }

        } else {

            error_log("Invalid credentials for email: $email");

            http_response_code(401);

            echo json_encode(["error" => "Invalid credentials"]);

        }

    } catch (MongoDBException $e) {

        error_log("MongoDB error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    } catch (Exception $e) {

        error_log("General error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}
```
- search_users.php
```
<?php

require '../config.php'; // MongoDB connection configuration

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["success" => false, "message" => "Method not allowed"]);

    exit;

}

  

$input = file_get_contents('php://input');

$data = json_decode($input, true);

  

if (!isset($data['search']) || empty($data['search'])) {

    http_response_code(400);

    echo json_encode(["success" => false, "message" => "Missing 'search' key in request payload."]);

    exit;

}

  

$searchTerm = $data['search'];

  

try {

    // Connect to MongoDB

    $manager = new MongoDB\Driver\Manager($mongoURI);

    $namespace = "swipe_chat_play.users";

  

    // Fetch all users from the MongoDB collection

    $query = new MongoDB\Driver\Query([]);

    $cursor = $manager->executeQuery($namespace, $query);

    $users = $cursor->toArray();

  

    if (!$users) {

        echo json_encode([]);

        exit;

    }

  

    // Perform fuzzy search using similar_text

    $results = [];

    foreach ($users as $user) {

        $username = $user->username ?? '';

        $profilePicture = $user->profile_picture ?? null;

  

        similar_text($searchTerm, $username, $percent);

        if ($percent >= 70) { // Only include matches with 70% or higher similarity

            $results[] = [

                "nickname" => $username,

                "profile_picture" => $profilePicture

            ];

        }

    }

  

    // Sort results by similarity (higher matches first)

    usort($results, function ($a, $b) use ($searchTerm) {

        similar_text($searchTerm, $b['nickname'], $percentB);

        similar_text($searchTerm, $a['nickname'], $percentA);

        return $percentB <=> $percentA;

    });

  

    echo json_encode($results);

} catch (MongoDB\Driver\Exception\Exception $e) {

    error_log("MongoDB error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["success" => false, "message" => "Internal server error"]);

    exit;

} catch (Exception $e) {

    error_log("General error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["success" => false, "message" => "Internal server error"]);

    exit;

}
```
- signup.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // Include configuration for MongoDB connection

  

use PHPMailer\PHPMailer\PHPMailer; // PHPMailer for sending emails

use PHPMailer\PHPMailer\Exception; // Exception handling for PHPMailer

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set the error log file path

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/signup_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

error_log("signup.php script started");

  

header('Content-Type: application/json');

  

// Handle POST requests

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $username = $_POST['username'] ?? null;

    $email = $_POST['email'] ?? null;

    $password = $_POST['password'] ?? null;

  

    if (!$username || !$email || !$password) {

        error_log("Validation failed: Missing required fields");

        http_response_code(400);

        echo json_encode(["error" => "Missing required fields"]);

        exit;

    }

  

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {

        error_log("Validation failed: Invalid email format - $email");

        http_response_code(400);

        echo json_encode(["error" => "Invalid email format"]);

        exit;

    }

  

    if (strlen($password) < 8) {

        error_log("Validation failed: Password too short");

        http_response_code(400);

        echo json_encode(["error" => "Password must be at least 8 characters long"]);

        exit;

    }

  

    try {

        error_log("Connecting to MongoDB using URI: $mongoURI");

  

        // Connect to MongoDB

        $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

        // Check if a user with the same email already exists

        $filter = ['email' => $email];

        $options = [];

        $query = new MongoDB\Driver\Query($filter, $options);

        $existingUsers = $mongoManager->executeQuery("swipe_chat_play.users", $query);

        $existingUsersArray = $existingUsers->toArray();

  

        // Generate common values for both new insertions and updates

        $passwordHash = password_hash($password, PASSWORD_BCRYPT);

        $verificationCode = str_pad(random_int(0, 9999), 4, '0', STR_PAD_LEFT);

        $expiry = new MongoDB\BSON\UTCDateTime((time() + 900) * 1000); // 15 minutes from now

  

        if (!empty($existingUsersArray)) {

            // User exists, check verification status

            if (isset($existingUsersArray[0]->is_verified) && $existingUsersArray[0]->is_verified == 1) {

                error_log("User already exists and is verified: $email");

                http_response_code(200);

                echo json_encode(["message" => "is verified"]);

                exit;

            } else {

                // User exists but is not verified. Overwrite the existing record.

                error_log("User already exists but not verified. Overwriting record for: $email");

                $updateDoc = [

                    '$set' => [

                        "username" => $username,

                        "firstname" => "example",

                        "lastname" => "mample",

                        "password_hash" => $passwordHash,

                        "verification_code" => $verificationCode,

                        "verification_code_expiry" => $expiry,

                        "is_verified" => 0

                    ]

                ];

  

                $bulk = new MongoDB\Driver\BulkWrite();

                $bulk->update(['email' => $email], $updateDoc, ['multi' => false, 'upsert' => false]);

                $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

                // Send verification email using PHPMailer

                $mail = new PHPMailer(true);

                try {

                    $mail->isSMTP();

                    $mail->Host = 'smtp.strato.de';

                    $mail->SMTPAuth = true;

                    $mail->Username = 'owee@fahrschule-peppermint.com';

                    $mail->Password = '(0+1)+(0+1)';

                    $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;

                    $mail->Port = 587;

  

                    $mail->setFrom('owee@fahrschule-peppermint.com', 'Luna');

                    $mail->addAddress($email);

                    $mail->Subject = 'Your Verification Code';

                    $mail->Body = "Hello $username,\n\nYour verification code is: $verificationCode\n\nThis code is valid for 15 minutes.";

  

                    if ($mail->send()) {

                        error_log("Verification email sent to $email after updating record");

                        http_response_code(201);

                        echo json_encode(["message" => "Success"]);

                    }

                } catch (Exception $e) {

                    error_log("Email send failed: " . $mail->ErrorInfo);

                    http_response_code(500);

                    echo json_encode(["error" => "Signup successful, but failed to send verification code."]);

                }

                exit;

            }

        }

  

        // No existing user found. Insert new document.

        $document = [

            "username" => $username,

            "firstname" => "example",

            "lastname" => "mample",

            "email" => $email,

            "password_hash" => $passwordHash,

            "verification_code" => $verificationCode,

            "verification_code_expiry" => $expiry,

            "is_verified" => 0

        ];

  

        error_log("Inserting document into the database");

        $bulk = new MongoDB\Driver\BulkWrite();

        $bulk->insert($document);

        $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

        error_log("Document inserted successfully. Sending verification email.");

  

        // Send verification email using PHPMailer

        $mail = new PHPMailer(true);

        try {

            $mail->isSMTP();

            $mail->Host = 'smtp.strato.de';

            $mail->SMTPAuth = true;

            $mail->Username = 'owee@fahrschule-peppermint.com';

            $mail->Password = '(0+1)+(0+1)';

            $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;

            $mail->Port = 587;

  

            $mail->setFrom('owee@fahrschule-peppermint.com', 'Luna');

            $mail->addAddress($email);

            $mail->Subject = 'Your Verification Code';

            $mail->Body = "Hello $username,\n\nYour verification code is: $verificationCode\n\nThis code is valid for 15 minutes.";

  

            if ($mail->send()) {

                error_log("Verification email sent to $email");

                http_response_code(201);

                echo json_encode(["message" => "Success"]);

            }

        } catch (Exception $e) {

            error_log("Email send failed: " . $mail->ErrorInfo);

            http_response_code(500);

            echo json_encode(["error" => "Signup successful, but failed to send verification code."]);

        }

    } catch (MongoDB\Driver\Exception\Exception $e) {

        error_log("Database error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}
```
- verifycode_login.php
```
<?php

use Firebase\JWT\JWT;

use MongoDB\BSON\UTCDateTime;

use MongoDB\BSON\ObjectId;

  

require __DIR__ . '/../../vendor/autoload.php';

require '../config.php'; // MongoDB configuration

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set the error log file path

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/verify_code_login_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

error_log("verify_code_login.php script started");

  

header('Content-Type: application/json');

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA=="; // Secure key for JWT

  

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $email = $_POST['email'] ?? null;

    $code = $_POST['code'] ?? null;

  

    if (!$email || !$code) {

        error_log("Missing email or verification code");

        http_response_code(400);

        echo json_encode(["error" => "Email and code are required"]);

        exit;

    }

  

    try {

        // Connect to MongoDB

        $manager = new MongoDB\Driver\Manager($mongoURI);

        $namespace = "swipe_chat_play.users";

  

        error_log("Looking up user by email: $email");

  

        // Query to fetch user by email

        $filter = ["email" => $email];

        $query = new MongoDB\Driver\Query($filter);

        $cursor = $manager->executeQuery($namespace, $query);

        $user = current($cursor->toArray());

  

        if (!$user) {

            error_log("User not found for email: $email");

            http_response_code(404);

            echo json_encode(["error" => "User not found"]);

            exit;

        }

  

        // Check if the verification code is correct and not expired

        $currentTime = new UTCDateTime(time() * 1000);

        if (

            $user->verification_code !== $code ||

            $user->verification_code_expiry < $currentTime

        ) {

            error_log("Invalid or expired verification code for email: $email");

            http_response_code(401);

            echo json_encode(["error" => "Wrong code"]);

            exit;

        }

  

        // Generate JWT token

        $payload = [

            "iss" => "http://krasserserver.com",

            "aud" => "http://krasserserver.com",

            "iat" => time(),

            "exp" => time() + (60 * 60), // Token expires in 1 hour

            "user_id" => (string)$user->_id // Store ObjectId as a string

        ];

        $jwt = JWT::encode($payload, $secretKey, 'HS256');

  

        // Update the user document to set "is_verified" to true and clear the verification code

        $bulk = new MongoDB\Driver\BulkWrite();

        $bulk->update(

            ["_id" => new ObjectId($user->_id)],

            ['$set' => ["is_verified" => true, "verification_code" => null, "verification_code_expiry" => null]]

        );

        $manager->executeBulkWrite($namespace, $bulk);

  

        error_log("User verified successfully. JWT generated.");

  

        http_response_code(200);

        echo json_encode(["message" => "Success", "token" => $jwt]);

    } catch (MongoDB\Driver\Exception\Exception $e) {

        error_log("MongoDB error in verify_code_login.php: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    } catch (Exception $e) {

        error_log("General error in verify_code_login.php: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}
```
- verify_code.php
```
<?php

use Firebase\JWT\JWT;

  

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // Include configuration for MongoDB connection

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Logging configuration

$logFile = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/verify_code_error.log';

ini_set('log_errors', 1);

ini_set('error_log', $logFile);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

header('Content-Type: application/json');

  

error_log("[INFO] verify_code.php script started");

  

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $email = $_POST['email'] ?? null;

    $code = $_POST['code'] ?? null;

  

    if (!$email || !$code) {

        error_log("[ERROR] Validation failed: Missing email or verification code. Email: $email, Code: $code");

        http_response_code(400);

        echo json_encode(["error" => "Missing email or verification code"]);

        exit;

    }

  

    try {

        // Connect to MongoDB

        error_log("[INFO] Connecting to MongoDB using URI: $mongoURI");

        $mongoManager = new MongoDB\Driver\Manager($mongoURI);

        error_log("[INFO] MongoDB connection established.");

  

        // Query the user by email

        error_log("[INFO] Looking up user by email: $email");

        $filter = ["email" => $email];

        $query = new MongoDB\Driver\Query($filter);

        $cursor = $mongoManager->executeQuery("swipe_chat_play.users", $query);

        $user = current($cursor->toArray());

  

        if (!$user) {

            error_log("[ERROR] No user found for email: $email");

            http_response_code(404);

            echo json_encode(["error" => "User not found"]);

            exit;

        }

  

        error_log("[INFO] User found: " . json_encode($user));

  

        if ($user->is_verified) {

            error_log("[WARNING] User is already verified: " . $user->_id);

            http_response_code(400);

            echo json_encode(["error" => "User is already verified"]);

            exit;

        }

  

        if ($user->verification_code !== $code) {

            error_log("[ERROR] Invalid code. Provided: $code, Expected: " . $user->verification_code);

            http_response_code(400);

            echo json_encode(["error" => "Invalid verification code"]);

            exit;

        }

  

        if ((new DateTime()) > $user->verification_code_expiry->toDateTime()) {

            error_log("[ERROR] Verification code expired. Expiry: " . $user->verification_code_expiry->toDateTime()->format('Y-m-d H:i:s'));

            http_response_code(400);

            echo json_encode(["error" => "Verification code has expired"]);

            exit;

        }

  

        // Mark user as verified

        error_log("[INFO] Marking user as verified: " . $user->_id);

        $bulk = new MongoDB\Driver\BulkWrite();

        $bulk->update(

            ['_id' => $user->_id],

            ['$set' => ['is_verified' => true, 'verification_code' => null, 'verification_code_expiry' => null]]

        );

        $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

        if ($result->getModifiedCount() === 0) {

            error_log("[ERROR] Failed to mark user as verified: " . $user->_id);

            http_response_code(500);

            echo json_encode(["error" => "Failed to verify user"]);

            exit;

        }

  

        // Generate JWT

        error_log("[INFO] Generating JWT for user ID: " . $user->_id);

        $payload = [

            "iss" => "http://krasserserver.com",

            "aud" => "http://krasserserver.com",

            "iat" => time(),

            "exp" => time() + (60 * 60), // Token expires in 1 hour

            "user_id" => (string)$user->_id

        ];

        $jwt = JWT::encode($payload, $secretKey, 'HS256');

  

        error_log("[INFO] JWT generated successfully for user ID: " . $user->_id);

        http_response_code(200);

        echo json_encode(["message" => "Success", "token" => $jwt]);

    } catch (MongoDB\Driver\Exception\ConnectionTimeoutException $e) {

        error_log("[CRITICAL] MongoDB connection timed out: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "MongoDB connection timeout"]);

    } catch (MongoDB\Driver\Exception\Exception $e) {

        error_log("[CRITICAL] MongoDB Exception: " . $e->getMessage() . " | File: " . $e->getFile() . " | Line: " . $e->getLine());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    } catch (Exception $e) {

        error_log("[CRITICAL] General Exception: " . $e->getMessage() . " | File: " . $e->getFile() . " | Line: " . $e->getLine());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    error_log("[ERROR] Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}
```
- create_matches.php
```
<?php

// seed_matches.php

  

require __DIR__ . '/vendor/autoload.php';

require_once __DIR__ . '/config.php'; // Ensure this file defines $mongoURI

  

// Create MongoDB manager.

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

// Define the current user's username.

$currentUsername = "luna";

  

// Query for all users except the current user.

$filter = ['username' => ['$ne' => $currentUsername]];

$query = new MongoDB\Driver\Query($filter);

$usersCursor = $mongoManager->executeQuery("swipe_chat_play.users", $query);

  

$matchesCreated = 0;

  

// Loop through each user.

foreach ($usersCursor as $user) {

    // Randomly decide with 30% chance.

    if (mt_rand(0, 100) / 100 <= 0.3) {

        // Get the other user's username.

        $otherUsername = $user->username;

  

        // Check if a match already exists between "luna" and $otherUsername.

        $matchFilter = [

            "users.$currentUsername" => "yes",

            "users.$otherUsername"   => "yes"

        ];

        $matchQuery = new MongoDB\Driver\Query($matchFilter);

        $existingMatchCursor = $mongoManager->executeQuery("swipe_chat_play.matches", $matchQuery);

        $existingMatch = current($existingMatchCursor->toArray());

        if ($existingMatch) {

            // Skip if match already exists.

            continue;

        }

  

        // Build a new match document.

        $newMatch = [

            "users" => [

                $currentUsername => "yes",

                $otherUsername   => "yes"

            ]

        ];

  

        // Insert the new match document.

        $bulk = new MongoDB\Driver\BulkWrite();

        $bulk->insert($newMatch);

        try {

            $result = $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

            $matchesCreated++;

            echo "Match created with user: $otherUsername\n";

        } catch (Exception $e) {

            echo "Error creating match with user: $otherUsername. " . $e->getMessage() . "\n";

        }

    }

}

  

echo "Total matches created: $matchesCreated\n";

?>
```
- curl_activities_area.php
```
<?php

header('Content-Type: application/json');

  

// Retrieve parameters from GET (lat, lon, radius in km)

$lat = isset($_GET['lat']) ? floatval($_GET['lat']) : null;

$lon = isset($_GET['lon']) ? floatval($_GET['lon']) : null;

$radius_km = isset($_GET['radius']) ? floatval($_GET['radius']) : null;

  

if (!$lat || !$lon || !$radius_km) {

    echo json_encode(["error" => "Missing parameters. Please provide lat, lon, and radius (in km)."]);

    exit;

}

  

// Calculate bounding box based on the given radius

$offsetLat = $radius_km / 111.0;

$offsetLon = $radius_km / (111.0 * cos(deg2rad($lat)));

  

$left = $lon - $offsetLon;

$right = $lon + $offsetLon;

$top = $lat + $offsetLat;

$bottom = $lat - $offsetLat;

  

// List of categories to search for

$categories = [

    'Restaurant',

    'Cafe',

    'Bar',

    'Park',

    'Cinema'

];

  

$results = [];

  

foreach ($categories as $category) {

    $query = urlencode($category);

    $viewbox = $left . ',' . $top . ',' . $right . ',' . $bottom;

    $url = "https://nominatim.openstreetmap.org/search?format=json&q={$query}&viewbox={$viewbox}&bounded=1&limit=10";

  

    // Set a user agent as required by Nominatim's usage policy.

    $opts = [

        "http" => [

            "header" => "User-Agent: YourAppName/1.0\r\n"

        ]

    ];

    $context = stream_context_create($opts);

    $response = file_get_contents($url, false, $context);

    if ($response === FALSE) {

        continue; // Skip this category if the request fails

    }

  

    $data = json_decode($response, true);

    if (!empty($data)) {

        $category_results = [];

        foreach ($data as $item) {

            $maps_url = "https://www.google.com/maps/dir/?api=1&destination=" . $item['lat'] . "," . $item['lon'];

            $category_results[] = [

                'name' => $item['display_name'],

                'lat' => $item['lat'],

                'lon' => $item['lon'],

                'maps_url' => $maps_url

            ];

        }

  

        $results[$category] = $category_results;

    }

}

  

// Output structured JSON with categories as keys

echo json_encode($results, JSON_PRETTY_PRINT);

?>
```
- generate.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // Include configuration for MongoDB connection

use MongoDB\Driver\Manager;

use MongoDB\BSON\ObjectId;

use Faker\Factory;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set the error log file path

$error_log_file = __DIR__ . '/generate_users_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

error_log("generate_users.php script started");

  

// Initialize Faker for generating fake data

$faker = Factory::create();

  

// Function to generate random hobbies

function generateRandomHobbies() {

    $hobbies = [

        'Diving/Snorkeling', 'Jogging', 'Hiking', 'Swimming', 'Cycling', 'Tennis', 'Golf', 'Skiing', 'Climbing', 'Kayaking',

        'Surfing', 'Archery', 'Visiting museums', 'Theater', 'Cinema', 'Concerts', 'Painting/Drawing', 'Reading books',

        'Photography', 'Writing', 'Art galleries', 'Opera', 'Learning dance styles', 'Antique hunting', 'Cooking', 'Baking',

        'Wine tasting', 'Craft beer breweries', 'Street food markets', 'Trying new restaurants', 'Barista workshops',

        'Mixing cocktails', 'Food tastings', 'Picnicking', 'Exploring vegetarian/vegan cuisine', 'Cooking competitions'

    ];

    return array_rand(array_flip($hobbies), rand(1, 5)); // Randomly pick 1 to 5 hobbies

}

  

// Function to generate random openTo categories

function generateRandomOpenTo() {

    $categories = [

        "Sports & Outdoor Activities" => ['Jogging', 'Hiking', 'Swimming', 'Cycling', 'Tennis', 'Golf', 'Skiing', 'Climbing', 'Kayaking', 'Surfing', 'Archery', 'Diving/Snorkeling'],

        "Arts & Culture" => ['Visiting museums', 'Theater', 'Cinema', 'Concerts', 'Painting/Drawing', 'Reading books', 'Photography', 'Writing', 'Art galleries', 'Opera', 'Learning dance styles', 'Antique hunting'],

        "Food & Drinks" => ['Cooking', 'Baking', 'Wine tasting', 'Craft beer breweries', 'Street food markets', 'Trying new restaurants', 'Barista workshops', 'Mixing cocktails', 'Food tastings', 'Picnicking', 'Exploring vegetarian/vegan cuisine', 'Cooking competitions']

    ];

    $openTo = [];

    foreach ($categories as $category => $options) {

        if (rand(0, 1)) { // Randomly decide whether to include the category

            $openTo[$category] = array_rand(array_flip($options), rand(1, count($options))); // Randomly pick 1 to all options

        }

    }

    return $openTo;

}

  

try {

    error_log("Connecting to MongoDB using URI: $mongoURI");

  

    // Connect to MongoDB

    $mongoManager = new Manager($mongoURI);

  

    // Generate 100 random users

    for ($i = 0; $i < 100; $i++) {

        $user = [

            '_id' => new ObjectId(),

            'username' => $faker->userName,

            'firstname' => $faker->firstName,

            'lastname' => $faker->lastName,

            'email' => $faker->email,

            'password_hash' => password_hash($faker->password, PASSWORD_BCRYPT),

            'verification_code' => null,

            'verification_code_expiry' => null,

            'is_verified' => true,

            'fingerprint_hash' => password_hash($faker->uuid, PASSWORD_BCRYPT),

            'profile' => json_encode([

                'image' => $faker->imageUrl(),

                'username' => $faker->userName,

                'age' => $faker->numberBetween(18, 65) . ' year old',

                'searching' => $faker->randomElement(['relationship', 'friendship', 'casual']),

                'element' => [['type' => 'text', 'content' => $faker->sentence]],

                'last' => '+ add element'

            ]),

            'chats' => [],

            'questionaire_answers' => [

                'gender' => $faker->randomElement(['Woman', 'Man', 'Non-Binary', 'Trans-woman', 'Trans-man', 'Demi-Girl', 'Demi-Boy', 'Agender', 'Genderfluid', 'Genderqueer', 'Prefer Not to Say', 'Custom']),

                'maxDistance' => $faker->randomFloat(2, 1, 200),

                'hobbies' => generateRandomHobbies(),

                'openTo' => generateRandomOpenTo(),

                'location' => [

                    'latitude' => $faker->latitude,

                    'longitude' => $faker->longitude

                ],

                'heightRange' => $faker->optional(0.5)->randomElements([null, [$faker->numberBetween(150, 190), $faker->numberBetween(150, 190)]]), // 50% chance of being null

                'ageRange' => [$faker->numberBetween(18, 30), $faker->numberBetween(31, 100)]

            ]

        ];

  

        // Insert the user into the MongoDB collection

        $bulk = new MongoDB\Driver\BulkWrite();

        $bulk->insert($user);

        $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

        error_log("User created: " . $user['username']);

    }

  

    error_log("100 random users have been created successfully!");

    echo json_encode(["message" => "100 random users have been created successfully!"]);

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

}

  

error_log("generate_users.php script finished");

?>
```
- match.php
```
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
```
- react_profile.php
```
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

  

error_log("[DEBUG] Entered reaction.php script");

  

/**

 * Helper function to send response with logging.

 */

function sendResponse($response, $httpCode = 200) {

    http_response_code($httpCode);

    error_log("[DEBUG] Sending response: " . print_r($response, true));

    echo json_encode($response);

    exit;

}

  

// Only allow POST

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    error_log("[DEBUG] Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    sendResponse(["status" => "error", "message" => "Method not allowed"], 405);

}

  

// Grab JWT from Authorization header

$headers = getallheaders();

error_log("[DEBUG] Headers: " . print_r($headers, true));

  

if (!isset($headers['Authorization'])) {

    error_log("[DEBUG] Missing Authorization header");

    sendResponse(["status" => "error", "message" => "Missing Authorization header"], 401);

}

  

$authHeader = $headers['Authorization'];

if (strpos($authHeader, "Bearer ") !== 0) {

    error_log("[DEBUG] Invalid Authorization header format");

    sendResponse(["status" => "error", "message" => "Invalid Authorization header"], 401);

}

  

$jwtToken  = trim(str_replace("Bearer ", "", $authHeader));

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

// Decode JWT => get swiper (sender) info

try {

    $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

    error_log("[DEBUG] Decoded JWT: " . print_r($decodedToken, true));

    $senderId     = $decodedToken->user_id ?? null;

} catch(Exception $e) {

    error_log("[DEBUG] JWT decode error: " . $e->getMessage());

    sendResponse(["status" => "error", "message" => "Invalid token"], 401);

}

  

if (!$senderId) {

    error_log("[DEBUG] Missing user_id in token");

    sendResponse(["status" => "error", "message" => "Invalid token"], 401);

}

  

// Connect to MongoDB

error_log("[DEBUG] Connecting to MongoDB with URI: $mongoURI");

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

// Get the swiper doc

try {

    $senderFilter = ['_id' => new ObjectId($senderId)];

    $senderQuery  = new MongoDB\Driver\Query($senderFilter);

    $senderCursor = $mongoManager->executeQuery("swipe_chat_play.users", $senderQuery);

    $sender       = current($senderCursor->toArray());

} catch(Exception $e) {

    error_log("[DEBUG] Error fetching sender doc: " . $e->getMessage());

    sendResponse(["status" => "error", "message" => "Error fetching sender"], 500);

}

  

if (!$sender || !isset($sender->username)) {

    error_log("[DEBUG] Sender not found or missing username in DB");

    sendResponse(["status" => "error", "message" => "Sender not found"], 404);

}

$senderUsername = $sender->username;

error_log("[DEBUG] Sender username: $senderUsername");

  

// Parse input

$input = json_decode(file_get_contents('php://input'), true);

error_log("[DEBUG] Input: " . print_r($input, true));

$profileUser  = $input['profile_username'] ?? null;  // e.g. "alex_25"

$reactionType = $input['type'] ?? 'react_bubble';

$timestamp    = $input['timestamp'] ?? date('c');

$content      = $input['content'] ?? [];

  

$parts = explode(' ', $profileUser);

$firstName = $parts[0] ?? ''; // basic safety check

error_log("[DEBUG] Extracted firstName from profileUser: $firstName");

  

// If you prefer for each comment to imply "yes" (like a swipe-right):

$swipeValue = "yes";

  

// Basic validation

if (!$profileUser) {

    error_log("[DEBUG] Missing profile_username in input");

    sendResponse(["status" => "error", "message" => "Missing profile_username"], 400);

}

  

// Find the swiped user doc

try {

    $receiverFilter = ['username' => $firstName];

    $receiverQuery  = new MongoDB\Driver\Query($receiverFilter);

    $receiverCursor = $mongoManager->executeQuery("swipe_chat_play.users", $receiverQuery);

    $receiver       = current($receiverCursor->toArray());

} catch(Exception $e) {

    error_log("[DEBUG] Error fetching receiver doc: " . $e->getMessage());

    sendResponse(["status" => "error", "message" => "Error fetching swiped user"], 500);

}

  

if (!$receiver) {

    error_log("[DEBUG] Swiped user (receiver) not found in DB");

    sendResponse(["status" => "error", "message" => "Swiped user not found"], 404);

}

$receiverUsername = $receiver->username;

$receiverId       = (string)$receiver->_id;

  

error_log("[DEBUG] Receiver found: $receiverUsername (ID: $receiverId)");

  

// 1) Insert or update doc in 'matches'

$queryFilter = [

    "users.$senderUsername"   => ['$exists' => true],

    "users.$receiverUsername" => ['$exists' => true]

];

$query = new MongoDB\Driver\Query($queryFilter);

  

try {

    $cursor   = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

    $matchDoc = current($cursor->toArray());

} catch(Exception $e) {

    error_log("[DEBUG] Error querying matches: " . $e->getMessage());

    sendResponse(["status" => "error", "message" => "Error checking matches"], 500);

}

  

$bulk = new MongoDB\Driver\BulkWrite();

  

if ($matchDoc) {

    error_log("[DEBUG] Match doc found. Updating existing doc. _id: " . (string)$matchDoc->_id);

  

    // Update the current user's (swiper's) swipe

    $updateField = "users.$senderUsername";

    $bulk->update(

        ['_id' => $matchDoc->_id],

        ['$set' => [$updateField => $swipeValue]],

        ['multi' => false, 'upsert' => false]

    );

  

    try {

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

    } catch(Exception $e) {

        error_log("[DEBUG] Error updating match doc: " . $e->getMessage());

        sendResponse(["status" => "error", "message" => "Error updating match"], 500);

    }

  

    // Re-read the doc

    try {

        $query      = new MongoDB\Driver\Query(['_id' => $matchDoc->_id]);

        $cursor     = $mongoManager->executeQuery("swipe_chat_play.matches", $query);

        $updatedDoc = current($cursor->toArray());

    } catch(Exception $e) {

        error_log("[DEBUG] Error re-reading match doc: " . $e->getMessage());

        sendResponse(["status" => "error", "message" => "Error retrieving updated match"], 500);

    }

  

    $users        = $updatedDoc->users;

    $swiperSwipe  = $users->$senderUsername   ?? null;

    $swipedSwipe  = $users->$receiverUsername ?? null;

    error_log("[DEBUG] Swiper swipe: $swiperSwipe, Swiped swipe: $swipedSwipe");

  

    if ($swiperSwipe === "yes" && $swipedSwipe === "yes") {

        // => It's a match

        error_log("[DEBUG] It's a match! Checking for existing chat_id");

  

        // Check if chat_id is set

        $chatIdString = $updatedDoc->chat_id ?? null;

  

        if (!$chatIdString) {

            error_log("[DEBUG] No chat_id yet, creating a new chat doc...");

  

            // Attempt to fetch a profile image for the users

            $senderProfileImage   = "";

            $receiverProfileImage = "";

  

            if (isset($sender->profile)) {

                $senderProfile = is_string($sender->profile)

                    ? json_decode($sender->profile, true)

                    : (array)$sender->profile;

                $senderProfileImage = $senderProfile['images'][0] ?? "";

            }

            if (isset($receiver->profile)) {

                $receiverProfile = is_string($receiver->profile)

                    ? json_decode($receiver->profile, true)

                    : (array)$receiver->profile;

                $receiverProfileImage = $receiverProfile['images'][0] ?? "";

            }

  

            // Create the chat doc

            $bulkChat = new MongoDB\Driver\BulkWrite();

            $chatData = [

                "messages"     => new \stdClass(),

                "participants" => [ $senderId, $receiverId ]

            ];

            $chatId = $bulkChat->insert($chatData);

  

            try {

                $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkChat);

            } catch(Exception $e) {

                error_log("[DEBUG] Error creating chat doc: " . $e->getMessage());

                sendResponse(["status" => "error", "message" => "Error creating chat"], 500);

            }

  

            $chatIdString = (string)$chatId;

            error_log("[DEBUG] New chat created with _id: $chatIdString");

  

            // Update both user docs with chat meta

            $chatMetaSender = [

                "profileURL"   => $receiverProfileImage,

                "receiver"     => $receiverUsername,

                "last_message" => "",

                "unread"       => 0

            ];

            $chatMetaReceiver = [

                "profileURL"   => $senderProfileImage,

                "receiver"     => $senderUsername,

                "last_message" => "",

                "unread"       => 0

            ];

  

            $bulkUpdate = new MongoDB\Driver\BulkWrite();

            $bulkUpdate->update(

                ["_id" => new ObjectId($senderId)],

                ['$set' => ["chats.$chatIdString" => $chatMetaSender]]

            );

            $bulkUpdate->update(

                ["_id" => new ObjectId($receiverId)],

                ['$set' => ["chats.$chatIdString" => $chatMetaReceiver]]

            );

  

            try {

                $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUpdate);

            } catch(Exception $e) {

                error_log("[DEBUG] Error updating users with chat meta: " . $e->getMessage());

                sendResponse(["status" => "error", "message" => "Error updating users"], 500);

            }

  

            // Update the match doc with chat_id

            $bulkMatchUpdate = new MongoDB\Driver\BulkWrite();

            $bulkMatchUpdate->update(

                ['_id' => $updatedDoc->_id],

                ['$set' => [

                    "chat_id"             => $chatIdString,

                    "receiver_profileURL" => $receiverProfileImage

                ]]

            );

  

            try {

                $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatchUpdate);

            } catch(Exception $e) {

                error_log("[DEBUG] Error updating match with chat_id: " . $e->getMessage());

                sendResponse(["status" => "error", "message" => "Error updating match doc"], 500);

            }

        }

  

        // Now we have $chatIdString => Insert the reaction message

        $messageId   = uniqid();

        $messageData = [

            "sender_id" => $senderId,

            "type"      => $reactionType, // e.g. 'react_bubble'

            "timestamp" => $timestamp,

            "status"    => "delivered",

            "content"   => $content

        ];

  

        $bulkMsg = new MongoDB\Driver\BulkWrite();

        $bulkMsg->update(

            ["_id" => new ObjectId($chatIdString)],

            ['$set' => [ "messages.$messageId" => $messageData ]],

            ['upsert' => false]

        );

  

        try {

            $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkMsg);

            error_log("[DEBUG] Reaction message stored in chat $chatIdString, messageId: $messageId");

        } catch(Exception $e) {

            error_log("[DEBUG] Error inserting reaction message: " . $e->getMessage());

            sendResponse(["status" => "error", "message" => "Error saving reaction message"], 500);

        }

  

        // Optionally update last_message for both

        $lastMessage = "($reactionType)";

        if ($reactionType === 'text' && isset($input['message'])) {

            $lastMessage = $input['message'];

        }

  

        $bulkUser = new MongoDB\Driver\BulkWrite();

        $bulkUser->update(

            ["chats.$chatIdString" => ['$exists' => true]],

            ['$set' => ["chats.$chatIdString.last_message" => $lastMessage]],

            ['multi' => true]

        );

  

        try {

            $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUser);

            error_log("[DEBUG] Updated last_message in users' chat meta to: $lastMessage");

        } catch(Exception $e) {

            error_log("[DEBUG] Error updating last_message: " . $e->getMessage());

        }

  

        // Return match & chat_id

        sendResponse([

            "status"      => "match",

            "chat_id"     => $chatIdString,

            "profileURL"  => $updatedDoc->receiver_profileURL ?? "",

            "receiver"    => $receiverUsername

        ]);

    } else {

        // Not both yes => store in "requested_chats" with the new message

        error_log("[DEBUG] Not a match yet. Storing reaction in requested_chats");

        _storeInRequestedChats($mongoManager, $senderId, $receiverId, $reactionType, $timestamp, $content);

        sendResponse(["status" => "no match"]);

    }

} else {

    error_log("[DEBUG] No existing match doc => creating one. Setting $senderUsername => $swipeValue");

    // No doc => create one => set [sender => yes, receiver => null]

    $newDoc = [

        "users" => [

            $senderUsername   => $swipeValue,

            $receiverUsername => null

        ]

    ];

    $bulk->insert($newDoc);

  

    try {

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

    } catch(Exception $e) {

        error_log("[DEBUG] Error creating new match doc: " . $e->getMessage());

        sendResponse(["status" => "error", "message" => "Error creating new match"], 500);

    }

  

    // Also store the reaction in "requested_chats"

    error_log("[DEBUG] Also storing reaction in requested_chats");

    _storeInRequestedChats($mongoManager, $senderId, $receiverId, $reactionType, $timestamp, $content);

  

    sendResponse(["status" => "no match"]);

}

  

/**

 * Helper to store the message in "requested_chats".

 * If doc doesn't exist, create it. Otherwise, append a new message.

 */

function _storeInRequestedChats($mongoManager, $senderId, $receiverId, $type, $timestamp, $content) {

    error_log("[DEBUG] _storeInRequestedChats() called with senderId=$senderId, receiverId=$receiverId");

    $messageId = uniqid();

    $messageData = [

        "sender_id" => $senderId,

        "type"      => $type,

        "timestamp" => $timestamp,

        "status"    => "delivered",

        "content"   => $content

    ];

  

    $filter = [

        "participants" => [

            '$all' => [ $senderId, $receiverId ]

        ]

    ];

  

    try {

        $query  = new MongoDB\Driver\Query($filter);

        $cursor = $mongoManager->executeQuery("swipe_chat_play.requested_chats", $query);

        $doc    = current($cursor->toArray());

    } catch(Exception $e) {

        error_log("[DEBUG] Error querying requested_chats: " . $e->getMessage());

        return;

    }

  

    $bulk = new MongoDB\Driver\BulkWrite();

  

    if ($doc) {

        error_log("[DEBUG] requested_chats doc found. _id: " . (string)$doc->_id);

        // Update existing doc

        $bulk->update(

            ["_id" => $doc->_id],

            ['$set' => ["messages.$messageId" => $messageData]]

        );

    } else {

        error_log("[DEBUG] No requested_chats doc found. Creating a new one");

        // Insert new doc

        $docData = [

            "participants" => [$senderId, $receiverId],

            "messages"     => [

                $messageId => $messageData

            ]

        ];

        $bulk->insert($docData);

    }

  

    try {

        $mongoManager->executeBulkWrite("swipe_chat_play.requested_chats", $bulk);

    } catch(Exception $e) {

        error_log("[DEBUG] Error inserting/updating requested_chats doc: " . $e->getMessage());

    }

}
```
- sendReaction.php
```
<?php

// ─────────────────────────────────────────────────────────────────────────────

//  match_algorithm/sendReaction.php   – Create / update swipe‑docs and

//                                       log a **reaction** message

// ─────────────────────────────────────────────────────────────────────────────

  

require __DIR__.'/../../vendor/autoload.php';

require_once __DIR__.'/../config.php';          // →  $mongoURI  +  $secretKey

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

/* ─────────────────────────  HTTP / error‑handling  ──────────────────────── */

header('Content-Type: application/json');

ini_set('display_errors',        '0');

ini_set('display_startup_errors','0');

ini_set('log_errors',            '1');

ini_set('error_log',  __DIR__.'/my_custom_errors.log');

error_reporting(E_ALL);

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["status"=>"error","message"=>"Method not allowed"]);

    exit;

}

  

/* ───────────────────────────────  JWT auth  ─────────────────────────────── */

$auth = getallheaders()['Authorization'] ?? '';

if (strpos($auth,'Bearer ') !== 0) {

    http_response_code(401);

    echo json_encode(["status"=>"error","message"=>"Missing / invalid Authorization header"]);

    exit;

}

  

try {

    $tokenData = JWT::decode(substr($auth,7), new Key($secretKey,'HS256'));

    $senderId  = $tokenData->user_id ?? null;

} catch (Exception $e) {

    http_response_code(401);

    echo json_encode(["status"=>"error","message"=>"Invalid token"]);

    exit;

}

if (!$senderId) {

    http_response_code(401);

    echo json_encode(["status"=>"error","message"=>"Invalid token"]);

    exit;

}

  

/* ──────────────────────────────  Request body  ─────────────────────────── */

$body         = json_decode(file_get_contents('php://input'), true);

$comment      = trim($body['comment']        ?? '');

$reactTo      =       $body['react_to']      ?? null;     // { type, content }

$receiverIdIn =       $body['receiver_id']   ?? null;

  

if (!$reactTo || !$receiverIdIn ||

    !isset($reactTo['type'], $reactTo['content'])) {

    http_response_code(400);

    echo json_encode(["status"=>"error","message"=>"Malformed request body"]);

    exit;

}

  

/* ---------- normalise type (top‑section bubbles become simple bubbles) ---- */

$reactionType = ($reactTo['type'] === 'react_top_section_bubble')

                ? 'react_bubble'

                : (string)$reactTo['type'];

  

$content      = (array)$reactTo['content'];

if ($comment !== '') $content['comment'] = $comment;

  

/* ────────────────────────────  MongoDB helpers  ────────────────────────── */

$mongo = new MongoDB\Driver\Manager($mongoURI);

$findOne = function(string $collection, array $filter) use ($mongo) {

    $cursor = $mongo->executeQuery($collection, new MongoDB\Driver\Query($filter));

    return current($cursor->toArray()) ?: null;

};

  

/* users */

$sender   = $findOne("swipe_chat_play.users", ['_id'=>new ObjectId($senderId)]);

$receiver = $findOne("swipe_chat_play.users", ['_id'=>new ObjectId($receiverIdIn)]);

if (!$sender || !$receiver) {

    http_response_code(404);

    echo json_encode(["status"=>"error","message"=>"User not found"]);

    exit;

}

$receiverId = (string)$receiver->_id;

  

/* ─────────────────────  MATCHES collection (swipes)  ───────────────────── */

$matchFilter = [

    "users.$senderId"   => ['$exists'=>true],

    "users.$receiverId" => ['$exists'=>true],

];

$matchDoc = $findOne("swipe_chat_play.matches", $matchFilter);

$bulk     = new MongoDB\Driver\BulkWrite();

$swipeVal = "yes";  // reaction = right‑swipe

  

if ($matchDoc) {

    // update this user's swipe

    $bulk->update(

        ['_id'=>$matchDoc->_id],

        ['$set'=>["users.$senderId"=>$swipeVal]]

    );

    $mongo->executeBulkWrite("swipe_chat_play.matches", $bulk);

  

    // reload to inspect both swipes

    $matchDoc = $findOne("swipe_chat_play.matches", ['_id'=>$matchDoc->_id]);

    $u       = $matchDoc->users;

    $my      = $u->$senderId   ?? null;

    $peer    = $u->$receiverId ?? null;

  

    /* ─────────── MATCH! ─────────── */

    if ($my === "yes" && $peer === "yes") {

        // 1) Check for an existing chat whose participants array already

        //    contains BOTH user IDs stored as *strings*

        $chatFilter = [

            'participants' => ['$all' => [$senderId, $receiverId]]

        ];

        $existingChat = $findOne("swipe_chat_play.chats", $chatFilter);

  

        if ($existingChat) {

            // reuse it

            $chatId = (string)$existingChat->_id;

        } else {

            // create a new chat

            $chatId = createChat($mongo, $sender, $receiver);

        }

  

        // ensure matches doc points at this chat

        if (!isset($matchDoc->chat_id) || $matchDoc->chat_id !== $chatId) {

            $bw = new MongoDB\Driver\BulkWrite();

            $bw->update(

                ['_id'=>$matchDoc->_id],

                ['$set'=>["chat_id"=>$chatId]]

            );

            $mongo->executeBulkWrite("swipe_chat_play.matches", $bw);

        }

  

        // add the reaction message (bubble/emoji/comment/etc.)

        insertChatMessage($mongo, $chatId, $senderId, $reactionType, $content);

  

        echo json_encode([

            "status"      => "match",

            "chat_id"     => $chatId,

            "profileURL"  => "",

            "receiver_id" => $receiverId,

            "receiver"    => $receiver->username

        ]);

        exit;

    }

  

    // not a mutual yes yet

    echo json_encode(["status"=>"no match"]);

    exit;

}

  

// first swipe ever between these two → create match doc

$bulk->insert([

    "users" => [

        $senderId   => $swipeVal,

        $receiverId => null

    ]

]);

$mongo->executeBulkWrite("swipe_chat_play.matches", $bulk);

echo json_encode(["status"=>"no match"]);

exit;

  

/* ─────────────────────────────  Helper funcs  ──────────────────────────── */

function createChat($mgr, $sender, $receiver) {

    $sid = (string)$sender->_id;

    $rid = (string)$receiver->_id;

  

    // insert chat doc with both user IDs as strings

    $bw  = new MongoDB\Driver\BulkWrite();

    $chatId = $bw->insert([

        "messages"     => new stdClass(),

        "participants" => [$sid, $rid]

    ]);

    $mgr->executeBulkWrite("swipe_chat_play.chats", $bw);

    $cid = (string)$chatId;

  

    // link to each user's chat list

    $link = function($uid,$otherId,$otherName) use ($mgr,$cid) {

        $b = new MongoDB\Driver\BulkWrite();

        $b->update(

            ["_id"=>new ObjectId($uid)],

            ['$set'=>["chats.$cid"=>[

                "profileURL"   => "",

                "receiver_id"  => $otherId,

                "receiver"     => $otherName,

                "last_message" => "",

                "unread"       => 0

            ]]]

        );

        $mgr->executeBulkWrite("swipe_chat_play.users", $b);

    };

    $link($sid, $rid, $receiver->username);

    $link($rid, $sid, $sender->username);

  

    return $cid;

}

  

function newMessageId(): string {

    return bin2hex(random_bytes(16));

}

  

function insertChatMessage($mgr, $chatId, $senderId, $type, $content) {

    $msgId = newMessageId();

    $ts    = (new DateTimeImmutable('now', new DateTimeZone('UTC')))

             ->format('Y-m-d\TH:i:s.u\Z');

  

    $bw = new MongoDB\Driver\BulkWrite();

    $bw->update(

        ['_id'=>new ObjectId($chatId)],

        ['$set'=>["messages.$msgId"=>[

            "sender_id"=>$senderId,

            "type"     =>$type,

            "timestamp"=>$ts,

            "status"   =>"delivered",

            "content"  =>$content

        ]]],

        ['upsert'=>true]

    );

    $mgr->executeBulkWrite("swipe_chat_play.chats", $bw);

  

    // update chat preview for both participants if there's a comment

    if (!empty($content['comment'])) {

        $bu = new MongoDB\Driver\BulkWrite();

        $bu->update(

            ["chats.$chatId"=>['$exists'=>true]],

            ['$set'=>["chats.$chatId.last_message"=>$content['comment']]],

            ['multi'=>true]

        );

        $mgr->executeBulkWrite("swipe_chat_play.users", $bu);

    }

}
```
- submit_questionaire.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require __DIR__ . '/../config.php'; // Include MongoDB connection configuration

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

// Set the error log file path

$error_log_file = __DIR__ . '/submit_questionaire_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

header('Content-Type: application/json');

  

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    // Log the start of the request

    error_log("Received POST request to submit_questionaire.php");

  

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        error_log("Authorization header missing or invalid");

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    $jwtToken = $matches[1];

    error_log("JWT Token extracted: $jwtToken");

  

    $secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

    try {

        // Decode the JWT

        error_log("Attempting to decode JWT token");

        $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

        $userId = $decodedToken->user_id ?? null;

  

        if (!$userId) {

            error_log("User ID not found in JWT");

            http_response_code(401);

            echo json_encode(["error" => "Invalid token"]);

            exit;

        }

  

        error_log("Decoded user ID: $userId");

  

        // Get the JSON payload

        error_log("Reading JSON payload from request body");

        $payload = json_decode(file_get_contents('php://input'), true);

        if (!$payload) {

            error_log("Invalid payload received");

            http_response_code(400);

            echo json_encode(["error" => "Invalid payload"]);

            exit;

        }

  

        error_log("Payload received: " . print_r($payload, true));

  

        // MongoDB connection

        error_log("Connecting to MongoDB");

        $manager = new MongoDB\Driver\Manager($mongoURI);

        $namespace = "swipe_chat_play.users";

  

        // Update the user's profile with the questionnaire answers

        error_log("Preparing MongoDB update query");

        $bulk = new MongoDB\Driver\BulkWrite;

        $bulk->update(

            ['_id' => new MongoDB\BSON\ObjectId($userId)],

            ['$set' => ['questionaire_answers' => $payload]]

        );

  

        error_log("Executing MongoDB update query");

        $result = $manager->executeBulkWrite($namespace, $bulk);

  

        error_log("MongoDB update successful");

        echo json_encode(["success" => true]);

    } catch (Exception $e) {

        error_log("Error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}

?>
```
- swipe.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once __DIR__ . '/../config.php';

  

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

use MongoDB\BSON\ObjectId;

  

ini_set('display_errors', '0');

ini_set('display_startup_errors', '0');

ini_set('log_errors', '1');

ini_set('error_log', __DIR__ . '/my_custom_errors.log');

error_reporting(E_ALL);

  

/* ─────────────────────────  Allow POST only  ─────────────────────────── */

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["status" => "error", "message" => "Method not allowed"]);

    exit;

}

  

/* ─────────────────────────────  JWT auth  ────────────────────────────── */

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

$jwtToken = trim(substr($authHeader, 7));

  

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

  

/* ───────────────────────  Mongo & sender lookup  ─────────────────────── */

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

$sender = current(

    $mongoManager

        ->executeQuery(

            "swipe_chat_play.users",

            new MongoDB\Driver\Query(['_id' => new ObjectId($senderId)])

        )

        ->toArray()

);

if (!$sender) {

    http_response_code(404);

    echo json_encode(["status" => "error", "message" => "Sender not found"]);

    exit;

}

  

/* ────────────────────────────  Request body  ─────────────────────────── */

$input        = json_decode(file_get_contents('php://input'), true);

$inputReceiverId = $input['receiver_id'] ?? null;   // receiver’s _id

$swipeValue      = $input['swipe']       ?? null;   // "yes" | "no"

  

if (!$inputReceiverId || !$swipeValue) {

    http_response_code(400);

    echo json_encode(["status" => "error", "message" => "receiver_id and swipe are required"]);

    exit;

}

  

/* ───────────────────────  Receiver lookup  ───────────────────────────── */

$receiver = current(

    $mongoManager

        ->executeQuery(

            "swipe_chat_play.users",

            new MongoDB\Driver\Query(['_id' => new ObjectId($inputReceiverId)])

        )

        ->toArray()

);

if (!$receiver) {

    http_response_code(404);

    echo json_encode(["status" => "error", "message" => "Receiver not found"]);

    exit;

}

$receiverId = (string)$receiver->_id;

  

/* ───────────────────────  MATCHES collection  ─────────────────────────── */

$queryFilter = [

    "users.$senderId"   => ['$exists' => true],

    "users.$receiverId" => ['$exists' => true]

];

$matchDoc = current(

    $mongoManager

        ->executeQuery("swipe_chat_play.matches", new MongoDB\Driver\Query($queryFilter))

        ->toArray()

);

$bulk = new MongoDB\Driver\BulkWrite();

  

if ($matchDoc) {

    /* -------- update my swipe ---------- */

    $bulk->update(

        ['_id' => $matchDoc->_id],

        ['$set' => ["users.$senderId" => $swipeValue]],

    );

    $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

  

    /* -------- reload doc to inspect swipes ---------- */

    $updatedDoc = current(

        $mongoManager

            ->executeQuery("swipe_chat_play.matches", new MongoDB\Driver\Query(['_id' => $matchDoc->_id]))

            ->toArray()

    );

    $users = $updatedDoc->users;

    $swiperSwipe = $users->$senderId   ?? null;

    $swipedSwipe = $users->$receiverId ?? null;

  

    /* ─────────────────────────  MATCH!  ──────────────────────────────── */

    if ($swiperSwipe === "yes" && $swipedSwipe === "yes") {

  

        /* --- 1. If match doc already references a chat, return it -------- */

        if (isset($updatedDoc->chat_id)) {

            echo json_encode([

                "status"      => "match",

                "chat_id"     => $updatedDoc->chat_id,

                "profileURL"  => $updatedDoc->receiver_profileURL ?? "",

                "receiver_id" => $receiverId,

                "receiver"    => $receiver->username

            ]);

            exit;

        }

  

        /* --- 2. Otherwise see if a chat between these two users exists --- */

        $existingChat = current(

            $mongoManager

                ->executeQuery(

                    "swipe_chat_play.chats",

                    new MongoDB\Driver\Query([

                        'participants' => ['$all' => [$senderId, $receiverId]]

                    ])

                )

                ->toArray()

        );

  

        if ($existingChat) {

            /* Link chat to match doc (so we never create dupes again) */

            $bulkMatchUpdate = new MongoDB\Driver\BulkWrite();

            $bulkMatchUpdate->update(

                ['_id' => $updatedDoc->_id],

                ['$set' => ["chat_id" => (string)$existingChat->_id]]

            );

            $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatchUpdate);

  

            echo json_encode([

                "status"      => "match",

                "chat_id"     => (string)$existingChat->_id,

                "profileURL"  => $updatedDoc->receiver_profileURL ?? "",

                "receiver_id" => $receiverId,

                "receiver"    => $receiver->username

            ]);

            exit;

        }

  

        /* --- 3. No chat yet → create one --------------------------------- */

  

        /* 3a. Fetch profile images (optional) */

        $receiverProfileImage = '';

        if (isset($receiver->profile)) {

            $receiverProfile = is_string($receiver->profile)

                ? json_decode($receiver->profile, true)

                : (array)$receiver->profile;

            $receiverProfileImage = $receiverProfile['images'][0] ?? '';

        }

  

        $senderProfileImage = '';

        if (isset($sender->profile)) {

            $senderProfile = is_string($sender->profile)

                ? json_decode($sender->profile, true)

                : (array)$sender->profile;

            $senderProfileImage = $senderProfile['images'][0] ?? '';

        }

  

        /* 3b. Insert chat */

        $bulkChat = new MongoDB\Driver\BulkWrite();

        $chatId = $bulkChat->insert([

            "messages"     => new stdClass(),

            "participants" => [$senderId, $receiverId]

        ]);

        $mongoManager->executeBulkWrite("swipe_chat_play.chats", $bulkChat);

        $chatIdString = (string)$chatId;

  

        /* 3c. Update both users’ chat lists */

        $chatMetaSender = [

            "profileURL"   => $receiverProfileImage,

            "receiver_id"  => $receiverId,

            "receiver"     => $receiver->username,

            "last_message" => "",

            "unread"       => 0

        ];

        $chatMetaReceiver = [

            "profileURL"   => $senderProfileImage,

            "receiver_id"  => $senderId,

            "receiver"     => $sender->username,

            "last_message" => "",

            "unread"       => 0

        ];

  

        $bulkUsers = new MongoDB\Driver\BulkWrite();

        $bulkUsers->update(

            ['_id' => new ObjectId($senderId)],

            ['$set' => ["chats.$chatIdString" => $chatMetaSender]]

        );

        $bulkUsers->update(

            ['_id' => new ObjectId($receiverId)],

            ['$set' => ["chats.$chatIdString" => $chatMetaReceiver]]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulkUsers);

  

        /* 3d. Link chat to match doc */

        $bulkMatchUpdate = new MongoDB\Driver\BulkWrite();

        $bulkMatchUpdate->update(

            ['_id' => $updatedDoc->_id],

            ['$set' => [

                "chat_id"            => $chatIdString,

                "receiver_profileURL" => $receiverProfileImage

            ]]

        );

        $mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulkMatchUpdate);

  

        echo json_encode([

            "status"      => "match",

            "chat_id"     => $chatIdString,

            "profileURL"  => $receiverProfileImage,

            "receiver_id" => $receiverId,

            "receiver"    => $receiver->username

        ]);

        exit;

    }

  

    /* ─────────────────────────  NOT a match  ─────────────────────────── */

    echo json_encode(["status" => "no match"]);

    exit;

}

  

/* ─────────────────────  First interaction → create doc  ───────────────── */

$bulk->insert([

    "users" => [

        $senderId   => $swipeValue,

        $receiverId => null

    ]

]);

$mongoManager->executeBulkWrite("swipe_chat_play.matches", $bulk);

echo json_encode(["status" => "no match"]);
```
- create_example_users.php
```
<?php

require_once '../config.php'; // Include configuration for MongoDB connection

  

// Function to generate random users

function generateRandomUser($id) {

    $heights = range(150, 200);

    $ages = range(18, 45);

    $seeks = ["relationship", "fun", "friends"];

  

    return [

        "id" => "user_$id",

        "attributes" => [

            "height" => (string)$heights[array_rand($heights)],

            "age" => (string)$ages[array_rand($ages)],

            "seek" => $seeks[array_rand($seeks)]

        ],

        "searching_for" => [

            "height" => [

                "min" => (string)rand(150, 190),

                "max" => (string)rand(191, 220)

            ],

            "age" => [

                "min" => (string)rand(18, 30),

                "max" => (string)rand(31, 50)

            ],

            "should_seek" => $seeks[array_rand($seeks)]

        ]

    ];

}

  

// Generate 30 users

$exampleUsers = [];

for ($i = 1; $i <= 30; $i++) {

    $exampleUsers[] = generateRandomUser($i);

}

  

// Connect to MongoDB using config.php's $mongoURI

$mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

// Insert users into MongoDB

$bulk = new MongoDB\Driver\BulkWrite();

foreach ($exampleUsers as $user) {

    $bulk->insert($user);

}

  

// Execute the bulk write

try {

    $mongoManager->executeBulkWrite("swipe_chat_play.example_users", $bulk);

    echo json_encode(["message" => "30 example users created successfully"]);

} catch (Exception $e) {

    echo json_encode(["error" => $e->getMessage()]);

}

?>
```
- retrieve_profile.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php';

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

// Configure error logging

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/retrieve_profile_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

header('Content-Type: application/json');

  

try {

    if ($_SERVER['REQUEST_METHOD'] !== 'GET') {

        http_response_code(405);

        echo json_encode(["error" => "Method not allowed"]);

        exit;

    }

  

    // Authenticate user

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        error_log("Authorization header missing or invalid");

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    // Get requested user ID from query parameters

    $requestedUserId = $_GET['user_id'] ?? null;

    if (!$requestedUserId) {

        error_log("Missing user_id parameter");

        http_response_code(400);

        echo json_encode(["error" => "User ID parameter is required"]);

        exit;

    }

  

    // Validate JWT

    $jwtToken = $matches[1];

    JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

  

    // Connect to MongoDB

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

    $filter = ['_id' => new MongoDB\BSON\ObjectId($requestedUserId)];

    $query = new MongoDB\Driver\Query($filter);

  

    // Execute query

    $result = $mongoManager->executeQuery("swipe_chat_play.users", $query)->toArray();

  

    if (empty($result)) {

        error_log("No user found with ID: $requestedUserId");

        http_response_code(404);

        echo json_encode(["error" => "User not found"]);

        exit;

    }

  

    // Extract user data

    $user = $result[0];

    $profileJson = $user->profile ?? '{}';

    $profileData = json_decode($profileJson, true) ?? [];

  

    // Build response matching your document structure

    $response = [

        'user_id' => $requestedUserId,

        'username' => $user->username ?? '',

        'firstname' => $user->firstname ?? '',

        'lastname' => $user->lastname ?? '',

        'email' => $user->email ?? '',

        'profileURL' => $profileData['image'] ?? '',

        'age' => $profileData['age'] ?? '',

        'searching' => $profileData['searching'] ?? '',

        'profile_images' => $profileData['profile_images'] ?? [],

        'elements' => $profileData['element'] ?? [],

        'last_element' => $profileData['last'] ?? ''

    ];

  

    error_log("Successfully retrieved profile for: $requestedUserId");

    echo json_encode($response);

  

} catch (MongoDB\Driver\Exception\InvalidArgumentException $e) {

    error_log("Invalid user ID format: " . $e->getMessage());

    http_response_code(400);

    echo json_encode(["error" => "Invalid user ID format"]);

} catch (Exception $e) {

    error_log("Error: " . $e->getMessage());

    http_response_code(500);

    echo json_encode(["error" => "Internal server error"]);

}
```
- save_profile.php
```
<?php

require __DIR__ . '/../../vendor/autoload.php';

require_once '../config.php'; // Include configuration for MongoDB connection

use Firebase\JWT\JWT;

use Firebase\JWT\Key;

  

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Set the error log file path

$error_log_file = __DIR__ . '/home/container/webroot/swipe_chatt_play_api/logs/save_profile_log.log';

ini_set('log_errors', 1);

ini_set('error_log', $error_log_file);

  

$secretKey = "hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZSA==";

  

error_log("save_profile.php script started");

  

header('Content-Type: application/json');

  

// Handle only POST requests

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $headers = getallheaders();

    $authHeader = $headers['Authorization'] ?? $headers['authorization'] ?? null;

  

    if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {

        error_log("Authorization header missing or invalid");

        http_response_code(401);

        echo json_encode(["error" => "Unauthorized"]);

        exit;

    }

  

    $jwtToken = $matches[1];

  

    try {

        // Decode the JWT

        $decodedToken = JWT::decode($jwtToken, new Key($secretKey, 'HS256'));

        $userId = $decodedToken->user_id ?? null;

  

        if (!$userId) {

            error_log("Invalid token: user ID missing");

            http_response_code(401);

            echo json_encode(["error" => "Invalid token"]);

            exit;

        }

  

        // Get the raw POST body and decode as JSON

        $data = json_decode(file_get_contents('php://input'), true);

        $profileJson = $data ?? null;

  

        if (!$profileJson) {

            error_log("Validation failed: Missing profile data");

            http_response_code(400);

            echo json_encode(["error" => "Missing profile data"]);

            exit;

        }

  

        error_log("Connecting to MongoDB using URI: $mongoURI");

  

        // Connect to MongoDB

        $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

        // Create filter and update

        $filter = ["_id" => new MongoDB\BSON\ObjectId($userId)];

        $update = [

            '$set' => [

                "profile" => json_encode($profileJson), // Store as a JSON string for efficiency

            ],

        ];

  

        $options = ['multi' => false, 'upsert' => false]; // Only update the first matching record

  

        // Perform update

        $bulk = new MongoDB\Driver\BulkWrite();

        $bulk->update($filter, $update, $options);

        $result = $mongoManager->executeBulkWrite("swipe_chat_play.users", $bulk);

  

        if ($result->getMatchedCount() > 0) {

            if ($result->getModifiedCount() > 0) {

                error_log("Profile updated successfully for user ID: $userId");

                http_response_code(200);

                echo json_encode(["message" => "Profile updated successfully"]);

            } else {

                error_log("No changes made, but user ID $userId exists.");

                http_response_code(200);

                echo json_encode(["message" => "No changes made. Profile is already up to date."]);

            }

        } else {

            error_log("No matching record found for user ID: $userId");

            http_response_code(404);

            echo json_encode(["error" => "User not found"]);

        }

    } catch (Exception $e) {

        error_log("Error: " . $e->getMessage());

        http_response_code(500);

        echo json_encode(["error" => "Internal server error"]);

    }

} else {

    error_log("Invalid request method: " . $_SERVER['REQUEST_METHOD']);

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

}

  

error_log("save_profile.php script finished");
```
- suggest_5_profiles.php
```
<?php

require_once '../config.php'; // Include configuration for MongoDB connection

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

$inputData = json_decode(file_get_contents('php://input'), true);

  

if (!isset($inputData['userdata'])) {

    http_response_code(400);

    echo json_encode(["error" => "Missing userdata in payload"]);

    exit;

}

  

$userData = $inputData['userdata'];

if (!isset($userData['searching_for'])) {

    http_response_code(400);

    echo json_encode(["error" => "Missing searching_for in userdata"]);

    exit;

}

  

$searchCriteria = $userData['searching_for'];

$heightMin = $searchCriteria['height']['min'] ?? null;

$heightMax = $searchCriteria['height']['max'] ?? null;

$ageMin = $searchCriteria['age']['min'] ?? null;

$ageMax = $searchCriteria['age']['max'] ?? null;

$shouldSeek = $searchCriteria['should_seek'] ?? null;

  

if (!$heightMin || !$heightMax || !$ageMin || !$ageMax || !$shouldSeek) {

    http_response_code(400);

    echo json_encode(["error" => "Incomplete searching_for criteria"]);

    exit;

}

  

try {

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    $query = new MongoDB\Driver\Query([

        "attributes.height" => [

            '$gte' => (int)$heightMin,

            '$lte' => (int)$heightMax

        ],

        "attributes.age" => [

            '$gte' => (int)$ageMin,

            '$lte' => (int)$ageMax

        ],

        "attributes.seek" => $shouldSeek

    ]);

  

    $cursor = $mongoManager->executeQuery("swipe_chat_play.users", $query);

  

    $matchingProfiles = [];

    foreach ($cursor as $document) {

        $matchingProfiles[] = $document;

    }

  

    http_response_code(200);

    echo json_encode(["profiles" => $matchingProfiles]);

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => $e->getMessage()]);

}

?>
```
- suggest_profiles.php
```
<?php

require_once '../config.php'; // Include configuration for MongoDB connection

  

header('Content-Type: application/json');

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(["error" => "Method not allowed"]);

    exit;

}

  

$inputData = json_decode(file_get_contents('php://input'), true);

  

if (!isset($inputData['userdata'])) {

    http_response_code(400);

    echo json_encode(["error" => "Missing userdata in payload"]);

    exit;

}

  

$userData = $inputData['userdata'];

if (!isset($userData['searching_for'])) {

    http_response_code(400);

    echo json_encode(["error" => "Missing searching_for in userdata"]);

    exit;

}

  

$searchCriteria = $userData['searching_for'];

$heightMin = $searchCriteria['height']['min'] ?? null;

$heightMax = $searchCriteria['height']['max'] ?? null;

$ageMin = $searchCriteria['age']['min'] ?? null;

$ageMax = $searchCriteria['age']['max'] ?? null;

$shouldSeek = $searchCriteria['should_seek'] ?? null;

  

if (!$heightMin || !$heightMax || !$ageMin || !$ageMax || !$shouldSeek) {

    http_response_code(400);

    echo json_encode(["error" => "Incomplete searching_for criteria"]);

    exit;

}

  

try {

    $mongoManager = new MongoDB\Driver\Manager($mongoURI);

  

    $query = new MongoDB\Driver\Query([

        "attributes.height" => [

            '$gte' => (int)$heightMin,

            '$lte' => (int)$heightMax

        ],

        "attributes.age" => [

            '$gte' => (int)$ageMin,

            '$lte' => (int)$ageMax

        ],

        "attributes.seek" => $shouldSeek

    ]);

  

    $cursor = $mongoManager->executeQuery("swipe_chat_play.users", $query);

  

    $matchingProfiles = [];

    foreach ($cursor as $document) {

        $matchingProfiles[] = $document;

    }

  

    http_response_code(200);

    echo json_encode(["profiles" => $matchingProfiles]);

} catch (Exception $e) {

    http_response_code(500);

    echo json_encode(["error" => $e->getMessage()]);

}

?>
```
- upload_image.php
```
<?php

// Enable error reporting for debugging

ini_set('display_errors', 1);

ini_set('display_startup_errors', 1);

error_reporting(E_ALL);

  

// Define the uploads directory (make sure it exists and is writable)

$uploadDir = __DIR__ . '/uploads/';

if (!file_exists($uploadDir)) {

    mkdir($uploadDir, 0777, true); // Create the directory if it doesn't exist

}

  

// Check if the request is a POST request

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    // Check if an image file was uploaded

    if (isset($_FILES['image']) && $_FILES['image']['error'] === UPLOAD_ERR_OK) {

        $fileTmpPath = $_FILES['image']['tmp_name'];

        $fileName = basename($_FILES['image']['name']);

        $uniqueFileName = uniqid() . '_' . $fileName; // Add a unique prefix to avoid overwriting

        $filePath = $uploadDir . $uniqueFileName;

  

        // Move the uploaded file to the uploads directory

        if (move_uploaded_file($fileTmpPath, $filePath)) {

            $imageUrl = '/uploads/' . $uniqueFileName; // URL to access the uploaded image

            $fullUrl = (isset($_SERVER['HTTPS']) ? 'https://' : 'http://') . $_SERVER['HTTP_HOST'] . dirname($_SERVER['REQUEST_URI']) . $imageUrl;

  

            // Send back the URL as JSON response

            header('Content-Type: application/json');

            echo json_encode(['message' => 'Image uploaded successfully', 'url' => $fullUrl]);

        } else {

            http_response_code(500);

            echo json_encode(['error' => 'Failed to move the uploaded file']);

        }

    } else {

        http_response_code(400);

        echo json_encode(['error' => 'No image file received or upload error']);

    }

} else {

    http_response_code(405);

    echo json_encode(['error' => 'Method not allowed. Use POST.']);

}
```