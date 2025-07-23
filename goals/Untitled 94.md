Below is the minimal change to fix that error: we’ve made the `State` class public so you can type your `GlobalKey` against it.

---
### 1) `lib/tabs/profile/edit_profile.dart`

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:latlong2/latlong.dart';
import 'package:http/http.dart' as http;
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:swipe_chat_play/config.dart';
import 'package:swipe_chat_play/tabs/profile/security_privacy_page.dart';
import 'package:swipe_chat_play/tabs/profile/edit_profile_photos_page.dart';
import 'package:swipe_chat_play/tabs/profile/edit_profile_radius_page.dart';
import 'package:swipe_chat_play/tabs/profile/edit_profile_qa_page.dart';
import 'package:swipe_chat_play/tabs/profile/edit_profile_activities_page.dart';
import 'package:swipe_chat_play/tabs/profile/edit_profile_filters_page.dart';

class EditProfileLayout extends StatefulWidget {
  const EditProfileLayout({Key? key}) : super(key: key);

  @override
  EditProfileLayoutState createState() => EditProfileLayoutState();
}

/// Made public so we can key off it from ProfileScreen
class EditProfileLayoutState extends State<EditProfileLayout> {
  String? name, age, gender, spoken, religion, career, politics;
  List<String?> photos = List<String?>.filled(9, null);
  double? radiusKm;
  LatLng? radiusPos;
  List<Map<String, String>>? qa;
  Set<String>? activities;
  Map<String, dynamic>? filters;

  final _storage = const FlutterSecureStorage();

  /// Called by ProfileScreen after successful re-auth
  Future<void> reloadProfile() => _loadProfile();

  @override
  void initState() {
    super.initState();
    _loadProfile();
  }

  Future<void> _loadProfile() async {
    final token = await _storage.read(key: 'jwt_token');
    if (token == null) return _snack('Please login first');

    final res = await http.get(
      Uri.parse('${Config.backendBaseUrl}/new_signup/get_profile.php'),
      headers: {'Authorization': 'Bearer $token'},
    );
    if (res.statusCode != 200) {
      return _snack('Load failed: HTTP ${res.statusCode}');
    }
    final root = json.decode(res.body) as Map<String, dynamic>;

    final top = root['topSection'] as Map<String, dynamic>? ?? {};
    name    = top['name']?.toString();
    age     = top['age']?.toString();
    final info = top['infoBubbles'] as Map<String, dynamic>? ?? {};
    gender   = info['gender']?.toString();
    religion = info['religion']?.toString();
    politics = info['politics']?.toString();
    career   = info['career']?.toString();
    spoken   = (List<String>.from(info['languages'] ?? [])).join(', ');

    final urls = List<String>.from(top['profileImages'] ?? []);
    photos = List<String?>.filled(9, null);
    for (var i = 0; i < urls.length && i < 9; i++) {
      photos[i] = urls[i];
    }

    final rawElems = root['elements'];
    final loadedQA = <Map<String, String>>[];
    var loadedActivities = <String>{};
    if (rawElems is List) {
      for (final eRaw in rawElems) {
        final e = eRaw as Map<String, dynamic>;
        final type    = e['type'] as String? ?? '';
        final content = (e['content'] as Map<String, dynamic>?) ?? {};
        if (type == 'icebreaker' || type == 'question_and_inputfield') {
          loadedQA.add({
            'question': content['question']?.toString() ?? '',
            'answer'  : content['answer']?.toString() ?? '',
          });
        } else if (type == 'bubbles') {
          loadedActivities = Set<String>.from(content['bubbles'] ?? []);
        }
      }
    }

    final sf    = root['search_filter'] as Map<String, dynamic>? ?? {};
    filters     = sf['searching_for'] as Map<String, dynamic>?;
    final locRad= sf['location_radius'] as Map<String, dynamic>?;
    if (locRad != null) {
      radiusKm = (locRad['radius'] as num?)?.toDouble();
      final loc = locRad['location'] as Map<String, dynamic>?;
      if (loc != null && loc['coordinates'] is List) {
        final c = List<num>.from(loc['coordinates']);
        radiusPos = LatLng(c[1].toDouble(), c[0].toDouble());
      }
    }

    setState(() {
      qa         = loadedQA;
      activities = loadedActivities;
    });
  }

  Future<void> _saveField(Map<String, dynamic> update) async {
    final token = await _storage.read(key: 'jwt_token');
    if (token == null) return _snack('Please login first');

    try {
      final res = await http.post(
        Uri.parse('${Config.backendBaseUrl}/create_profile/update_userdata.php'),
        headers: {
          'Authorization': 'Bearer $token',
          'Content-Type' : 'application/json',
        },
        body: json.encode(update),
      );
      final b = json.decode(res.body) as Map<String, dynamic>;
      if (res.statusCode != 200 || b['success'] != true) {
        _snack('Save error: ${b['error'] ?? b['message']}');
      }
    } catch (e) {
      _snack('Save failed: $e');
    }
  }

  void _snack(String msg) =>
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  // ... rest of your editing UI/helpers unchanged ...
  // (inline _editTextField, _row, navigation to Photos/Radius/Q&A etc.)

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        children: [
          const SizedBox(height: 16),
          // your _row calls for Name, Age, Gender, ...
          _row('Sicherheit & Datenschutz', 'Details anzeigen', () {
            Navigator.push(context, MaterialPageRoute(builder: (_) => const SecurityPrivacyPage()));
          }),
          // … and the rest of your rows …
        ],
      ),
    );
  }
}
```

---

### 2) `lib/tabs/profile/profile_screen.dart`

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:local_auth/local_auth.dart';
import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart';
import 'package:swipe_chat_play/tabs/profile/view_my_profile.dart';
import 'package:swipe_chat_play/tabs/profile/edit_profile.dart';

class ProfileScreen extends StatefulWidget {
  const ProfileScreen({Key? key}) : super(key: key);
  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  final _pageController = PageController(initialPage: 0);
  // Now we can type against the public state class:
  final _editKey = GlobalKey<EditProfileLayoutState>();

  int  _pageIndex     = 0;
  bool _editMode      = false;
  bool _authInProgress= false;

  final _storage   = const FlutterSecureStorage();
  final _localAuth = LocalAuthentication();

  void _snack(String msg) {
    if (!mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));
  }

  Future<bool> _authenticateForEdit() async {
    if (_authInProgress) return false;
    setState(() => _authInProgress = true);

    final jwt = await _storage.read(key: 'jwt_token');
    if (jwt == null) {
      _snack('Please log in first');
      setState(() => _authInProgress = false);
      return false;
    }

    try {
      final r = await http.get(
        Uri.parse('${Config.backendBaseUrl}/dsgvo/method.php'),
        headers: {'Authorization': 'Bearer $jwt'},
      );
      if (r.statusCode != 200) {
        _snack('Failed to fetch auth methods');
        return false;
      }
      final body = json.decode(r.body) as Map<String, dynamic>;
      final methods = (body['methods'] as List?)?.cast<String>() ?? ['password'];

      if (methods.contains('device_auth') &&
          (await _localAuth.canCheckBiometrics ||
           await _localAuth.isDeviceSupported())) {
        final did = await _localAuth.authenticate(
          localizedReason: 'Authenticate to edit your profile',
          options: const AuthenticationOptions(biometricOnly: false),
        );
        if (did) {
          setState(() => _authInProgress = false);
          return true;
        }
      }

      final pwdCtrl = TextEditingController();
      final ok = await showDialog<bool>(
        context: context,
        barrierDismissible: false,
        builder: (_) => AlertDialog(
          title: const Text('Confirm with password'),
          content: TextField(
            controller: pwdCtrl,
            obscureText: true,
            decoration: const InputDecoration(labelText: 'Password'),
          ),
          actions: [
            TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('Cancel')),
            TextButton(onPressed: () => Navigator.pop(context, true),  child: const Text('OK')),
          ],
        ),
      );
      if (ok != true) return false;

      final v = await http.post(
        Uri.parse('${Config.backendBaseUrl}/dsgvo/verify.php'),
        headers: {
          'Authorization': 'Bearer $jwt',
          'Content-Type' : 'application/json',
        },
        body: json.encode({'password': pwdCtrl.text.trim()}),
      );
      return v.statusCode == 200;
    } catch (e) {
      _snack('Auth error: $e');
      return false;
    } finally {
      setState(() => _authInProgress = false);
    }
  }

  Future<void> _goToPage(int index) async {
    if (index == 1 && !_editMode) {
      final ok = await _authenticateForEdit();
      if (!ok) return;
      _editKey.currentState?.reloadProfile();
      setState(() => _editMode = true);
    }
    if (index == 0 && _editMode) {
      setState(() => _editMode = false);
    }
    setState(() => _pageIndex = index);
    _pageController.animateToPage(
      index,
      duration: const Duration(milliseconds: 300),
      curve: Curves.easeInOut,
    );
  }

  Widget _buildTab(String label, int index) {
    final selected = _pageIndex == index;
    return Expanded(
      child: GestureDetector(
        onTap: () => _goToPage(index),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const SizedBox(height: 12),
            Text(
              label,
              style: TextStyle(
                fontSize: 16,
                color: selected ? Colors.grey[800] : Colors.grey[400],
                fontWeight: selected ? FontWeight.w700 : FontWeight.w400,
              ),
            ),
            const SizedBox(height: 8),
            Container(
              height: selected ? 4 : 1,
              color: selected ? Colors.grey[800] : Colors.grey[300],
              margin: const EdgeInsets.symmetric(horizontal: 24),
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      body: SafeArea(
        child: Column(
          children: [
            Row(children: [
              _buildTab('Profile', 0),
              _buildTab('Edit',    1),
            ]),
            Expanded(
              child: GestureDetector(
                behavior: HitTestBehavior.translucent,
                onHorizontalDragEnd: (d) {
                  final v = d.velocity.pixelsPerSecond.dx;
                  if (v < -200 && _pageIndex == 0) {
                    _goToPage(1);
                  } else if (v > 200 && _pageIndex == 1) {
                    _goToPage(0);
                  }
                },
                child: PageView(
                  controller: _pageController,
                  physics: _editMode
                      ? const BouncingScrollPhysics()
                      : const NeverScrollableScrollPhysics(),
                  onPageChanged: (i) => _goToPage(i),
                  children: [
                    const ViewMyProfile(),
                    EditProfileLayout(key: _editKey),
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
```

**What changed?**

- **Renamed** the private `_EditProfileLayoutState` to a **public** `EditProfileLayoutState` in `edit_profile.dart`.
    
- Updated the `createState()` override to return that public class.
    
- In `profile_screen.dart` we now use `GlobalKey<EditProfileLayoutState>()` instead of the private type.
    

That fixes the “isn't a type” compilation error and restores your re-auth–then–edit flow.