old working version:

"

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

import 'package:swipe_chat_play/tabs/profile/security_privacy_page.dart';

class EditProfileLayout extends StatefulWidget {

const EditProfileLayout({Key? key}) : super(key: key);

@override

State<EditProfileLayout> createState() => _EditProfileLayoutState();

}

class _EditProfileLayoutState extends State<EditProfileLayout> {

// ─────────────────────────────────────────────────────────────────

// simple text fields

String? name, age, gender, spoken, religion, career, politics;

// complex data

List<String?> photos = List<String?>.filled(9, null);

double? radiusKm;

LatLng? radiusPos;

List<Map<String, String>>? qa;

Set<String>? activities;

Map<String, dynamic>? filters;

// storage & helpers

final _storage = const FlutterSecureStorage();

void _openSecurityPrivacy() {

Navigator.push(

context,

MaterialPageRoute(builder: (_) => const SecurityPrivacyPage()),

);

}

Future<void> _logout() async {

await _storage.deleteAll();

Navigator.pushReplacementNamed(context, '/login');

}

// ─────────────────────────────────────────────────────────────────

// ❶ LOAD on init

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

// ─── topSection ───────────────────────────────────────────

final top = root['topSection'] as Map<String, dynamic>;

name = top['name']?.toString();

age = top['age']?.toString();

final info = top['infoBubbles'] as Map<String, dynamic>? ?? {};

gender = info['gender']?.toString();

religion = info['religion']?.toString();

politics = info['politics']?.toString();

career = info['career']?.toString();

final langs = List<String>.from(info['languages'] ?? []);

spoken = langs.join(', ');

final urls = List<String>.from(top['profileImages'] ?? []);

photos = List<String?>.filled(9, null);

for (int i = 0; i < urls.length && i < 9; i++) {

photos[i] = urls[i];

}

// ─── elements ─────────────────────────────────────────────

final rawElems = root['elements'];

final loadedQA = <Map<String, String>>[];

var loadedActivities = <String>{};

if (rawElems is List) {

for (final eRaw in rawElems) {

final e = eRaw as Map<String, dynamic>;

final type = e['type'] as String? ?? '';

final content = (e['content'] as Map<String, dynamic>?) ?? {};

if (type == 'icebreaker' || type == 'question_and_inputfield') {

loadedQA.add({

'question': content['question']?.toString() ?? '',

'answer': content['answer']?.toString() ?? '',

});

} else if (type == 'bubbles') {

loadedActivities = Set<String>.from(content['bubbles'] ?? []);

}

}

} else if (rawElems is Map<String, dynamic>) {

for (final e in rawElems.values) {

final elem = e as Map<String, dynamic>;

final type = elem['type'] as String? ?? '';

final content = (elem['content'] as Map<String, dynamic>?) ?? {};

if (type == 'icebreaker' || type == 'question_and_inputfield') {

loadedQA.add({

'question': content['question']?.toString() ?? '',

'answer': content['answer']?.toString() ?? '',

});

} else if (type == 'bubbles') {

loadedActivities = Set<String>.from(content['bubbles'] ?? []);

}

}

}

// ─── search_filter ────────────────────────────────────────

final searchFilter = root['search_filter'] as Map<String, dynamic>? ?? {};

filters = searchFilter['searching_for'] as Map<String, dynamic>?;

final locRad = searchFilter['location_radius'] as Map<String, dynamic>?;

if (locRad != null) {

radiusKm = (locRad['radius'] as num?)?.toDouble();

final loc = locRad['location'] as Map<String, dynamic>?;

if (loc != null && loc['coordinates'] is List) {

final c = List<num>.from(loc['coordinates']);

radiusPos = LatLng(c[1].toDouble(), c[0].toDouble());

}

}

setState(() {

qa = loadedQA;

activities = loadedActivities;

});

}

// ─────────────────────────────────────────────────────────────────

// ❷ PARTIAL SAVE helper

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

if (!(res.statusCode == 200 && body['success'] == true)) {

_snack('Save error: ${body['error'] ?? body['message']}');

}

} catch (e) {

_snack('Save failed: $e');

}

}

// ─────────────────────────────────────────────────────────────────

void _snack(String msg) =>

ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

// ─────────────────────────────────────────────────────────────────

Future<void> _editTextField({

required String title,

required String? initial,

required String fieldName,

required String subKey,

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

_saveField({

fieldName: {

subKey: (subKey == 'age' ? int.tryParse(v) ?? v : v)

}

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

Future<void> _gotoPhotos() async {

final result = await Navigator.push<List<String?>>(

context,

MaterialPageRoute(builder: (_) => EditPhotosPage(initial: photos)),

);

if (result != null) {

setState(() => photos = result);

_saveField({'images': photos.whereType<String>().toList()});

}

}

Future<void> _gotoRadius() async {

final result = await Navigator.push<Map<String, dynamic>>(

context,

MaterialPageRoute(

builder: (_) => EditRadiusPage(

initialRadius: radiusKm,

initialPos: radiusPos,

),

),

);

if (result != null) {

setState(() {

radiusKm = (result['radius'] as num?)?.toDouble();

radiusPos = result['location'] as LatLng?;

});

_saveField({

'location_radius': {

'location': radiusPos == null

? null

: {

'type': 'Point',

'coordinates': [radiusPos!.longitude, radiusPos!.latitude]

},

'radius': radiusKm,

}

});

}

}

Future<void> _gotoQA() async {

final result = await Navigator.push<List<Map<String, String>>>(

context,

MaterialPageRoute(builder: (_) => EditQAPage(initial: qa)),

);

if (result != null) {

setState(() => qa = result);

_saveField({'questions_and_answers': qa});

}

}

Future<void> _gotoActivities() async {

final result = await Navigator.push<Set<String>>(

context,

MaterialPageRoute(

builder: (_) => EditActivitiesPage(initial: activities)),

);

if (result != null) {

setState(() => activities = result);

_saveField({'open_to': activities?.toList()});

}

}

Future<void> _gotoFilters() async {

final result = await Navigator.push<Map<String, dynamic>>(

context,

MaterialPageRoute(builder: (_) => EditFiltersPage(initial: filters)),

);

if (result != null) {

setState(() => filters = result);

_saveField({'searching_for': filters});

}

}

String _photoSub() => photos.where((e) => e != null).isEmpty

? 'Add photos'

: '${photos.where((e) => e != null).length}/9 selected';

String _radiusSub() =>

radiusKm == null ? 'Set radius & location' : '${radiusKm!.round()} km';

String _qaSub() => qa == null ? 'Answer Q&A' : '${qa!.length} answered';

String _actSub() =>

activities == null ? 'Choose activities' : '${activities!.length} chosen';

String _filtSub() => filters == null

? 'Configure filters'

: (filters?['genders'] as List?)?.join(', ') ?? 'Custom filters';

@override

Widget build(BuildContext context) {

return Scaffold(

body: ListView(children: [

const SizedBox(height: 16),

_row(

'Name',

name ?? 'Add',

() => _editTextField(

title: 'Name',

initial: name,

fieldName: 'about_you',

subKey: 'name')),

_row(

'Age',

age ?? 'Set',

() => _editTextField(

title: 'Age',

initial: age,

fieldName: 'about_you',

subKey: 'age',

type: TextInputType.number)),

_row(

'Gender',

gender ?? 'Choose',

() => _editTextField(

title: 'Gender',

initial: gender,

fieldName: 'about_you',

subKey: 'gender')),

_row(

'Languages',

spoken ?? 'Add',

() => _editTextField(

title: 'Languages',

initial: spoken,

fieldName: 'about_you',

subKey: 'spokenLanguage')),

_row(

'Religion',

religion ?? 'Choose',

() => _editTextField(

title: 'Religion',

initial: religion,

fieldName: 'about_you',

subKey: 'religion')),

_row(

'Career',

career ?? 'Add',

() => _editTextField(

title: 'Career',

initial: career,

fieldName: 'about_you',

subKey: 'career')),

_row(

'Politics',

politics ?? 'Choose',

() => _editTextField(

title: 'Politics',

initial: politics,

fieldName: 'about_you',

subKey: 'politics')),

_row('Sicherheit & Datenschutz', 'Details anzeigen',

_openSecurityPrivacy),

const SizedBox(height: 12),

_row('Profile pictures', _photoSub(), _gotoPhotos),

_row('Dating radius', _radiusSub(), _gotoRadius),

_row('Q & A', _qaSub(), _gotoQA),

_row('Open to', _actSub(), _gotoActivities),

_row('Filters', _filtSub(), _gotoFilters),

_row('Logout', '', _logout, divider: false),

const SizedBox(height: 24),

]),

);

}

}"

"

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

"

Task1: edit ProfileScreen:
1. "profile" and "edit" at the top should both be text on white background with the selected one being dark grey and thick and not selected one being normal thickness and light grey
2. when user clicks edit please do same flow as in this example to verify the user before switching to the edit tab:
   "
   import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter/services.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:local_auth/local_auth.dart';

import 'package:http/http.dart' as http;

import 'package:swipe_chat_play/config.dart'; // contains backendBaseUrl

  

class SecurityPrivacyPage extends StatefulWidget {

  const SecurityPrivacyPage({Key? key}) : super(key: key);

  

  @override

  State<SecurityPrivacyPage> createState() => _SecurityPrivacyPageState();

}

  

class _SecurityPrivacyPageState extends State<SecurityPrivacyPage> {

  final _storage = const FlutterSecureStorage();

  final _auth = LocalAuthentication();

  

  Future<String?> _token() => _storage.read(key: 'jwt_token');

  Future<String?> _deletionDate() => _storage.read(key: 'deletion_date');

  Future<String?> _restrictionDate() => _storage.read(key: 'restriction_date');

  

  void _snack(String msg) {

    if (!mounted) return;

    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  }

  

  Future<Map<String, String>?> _authenticate() async {

    final jwt = await _token();

    if (jwt == null) {

      _snack('Please login first');

      return null;

    }

  

    final methodsRes = await http.get(

      Uri.parse('${Config.backendBaseUrl}/dsgvo/method.php'),

      headers: {'Authorization': 'Bearer $jwt'},

    );

    if (methodsRes.statusCode != 200 || methodsRes.body.trim().isEmpty) {

      _snack('Failed to fetch auth methods');

      return null;

    }

  

    late final List<String> methods;

    try {

      final body = json.decode(methodsRes.body) as Map<String, dynamic>;

      methods = (body['methods'] as List).cast<String>();

    } on FormatException {

      _snack('Invalid auth-methods response');

      return null;

    }

  

    if (methods.contains('device_auth') && await _auth.canCheckBiometrics) {

      final didAuth = await _auth.authenticate(

        localizedReason: 'Authenticate to continue',

      );

      if (didAuth) {

        return {

          'X-Auth-Method': 'device_auth',

          'X-Device-Auth': '1',

        };

      }

    }

  

    if (!mounted) return null;

    final pwdCtrl = TextEditingController();

    final ok = await showDialog<bool>(

      context: context,

      builder: (_) => AlertDialog(

        title: const Text('Confirm with password'),

        content: TextField(

          controller: pwdCtrl,

          obscureText: true,

          decoration: const InputDecoration(labelText: 'Password'),

        ),

        actions: [

          TextButton(

              onPressed: () => Navigator.pop(context, false),

              child: const Text('Cancel')),

          TextButton(

              onPressed: () => Navigator.pop(context, true),

              child: const Text('OK')),

        ],

      ),

    );

    if (ok != true) return null;

  

    return {

      'X-Auth-Method': 'password',

      'Content-Type': 'application/json',

      'password': pwdCtrl.text,

    };

  }

  

  Future<void> _exportData() async {

    final jwt = await _token();

    if (jwt == null) return _snack('Please login first');

  

    final auth = await _authenticate();

    if (auth == null) return;

  

    final headers = {

      'Authorization': 'Bearer $jwt',

      'X-Auth-Method': auth['X-Auth-Method']!,

    };

    String? body;

    if (auth['X-Auth-Method'] == 'device_auth') {

      headers['X-Device-Auth'] = auth['X-Device-Auth']!;

    } else {

      headers['Content-Type'] = auth['Content-Type']!;

      body = json.encode({'password': auth['password']});

    }

  

    final res = await http.post(

      Uri.parse('${Config.backendBaseUrl}/dsgvo/request_me.php'),

      headers: headers,

      body: body,

    );

  

    if (res.statusCode == 200) {

      late final Map<String, dynamic> resp;

      try {

        resp = json.decode(res.body) as Map<String, dynamic>;

      } on FormatException {

        return _snack('Server sent non-JSON response');

      }

      final url = resp['url'] as String?;

      if (url == null) return _snack('Unexpected response format');

  

      _snack('Download ready');

      if (!mounted) return;

      showDialog(

        context: context,

        builder: (_) => AlertDialog(

          title: const Text('Your data export'),

          content: SelectableText('Download link (24 h valid):\n$url'),

          actions: [

            TextButton(

              onPressed: () {

                Clipboard.setData(ClipboardData(text: url));

                Navigator.pop(context);

                _snack('Link copied to clipboard');

              },

              child: const Text('Copy'),

            ),

            TextButton(

                onPressed: () => Navigator.pop(context),

                child: const Text('Close')),

          ],

        ),

      );

    } else {

      _snack('Export failed: HTTP ${res.statusCode}');

    }

  }

  

  Future<void> _restrictMe() async {

    final jwt = await _token();

    if (jwt == null) return _snack('Please login first');

  

    final auth = await _authenticate();

    if (auth == null) return;

  

    final headers = {

      'Authorization': 'Bearer $jwt',

      'X-Auth-Method': auth['X-Auth-Method']!,

    };

    String? body;

    if (auth['X-Auth-Method'] == 'device_auth') {

      headers['X-Device-Auth'] = auth['X-Device-Auth']!;

    } else {

      headers['Content-Type'] = auth['Content-Type']!;

      body = json.encode({'password': auth['password']});

    }

  

    final res = await http.post(

      Uri.parse('${Config.backendBaseUrl}/dsgvo/restrict_me.php'),

      headers: headers,

      body: body,

    );

  

    if (res.statusCode == 200) {

      late final Map<String, dynamic> resp;

      try {

        resp = json.decode(res.body) as Map<String, dynamic>;

      } on FormatException {

        return _snack('Server sent non-JSON response');

      }

      final restrictionDate = resp['restriction_date'] as String?;

      if (restrictionDate != null) {

        await _storage.write(key: 'restriction_date', value: restrictionDate);

      }

      _snack('Processing restricted as of ${restrictionDate ?? 'now'}');

      if (mounted) setState(() {});

    } else {

      _snack('Restriction failed: HTTP ${res.statusCode}');

    }

  }

  

  Future<void> _deleteMe() async {

    final jwt = await _token();

    if (jwt == null) return _snack('Please login first');

  

    if (!mounted) return;

    final confirm = await showDialog<bool>(

      context: context,

      builder: (_) => AlertDialog(

        title: const Text('Delete account?'),

        content: const Text(

            'Your profile will be permanently removed after 30 days. Continue?'),

        actions: [

          TextButton(

              onPressed: () => Navigator.pop(context, false),

              child: const Text('Cancel')),

          TextButton(

              onPressed: () => Navigator.pop(context, true),

              child: const Text('Delete')),

        ],

      ),

    );

    if (confirm != true) return;

  

    final auth = await _authenticate();

    if (auth == null) return;

  

    final headers = {

      'Authorization': 'Bearer $jwt',

      'X-Auth-Method': auth['X-Auth-Method']!,

    };

    String? body;

    if (auth['X-Auth-Method'] == 'device_auth') {

      headers['X-Device-Auth'] = auth['X-Device-Auth']!;

    } else {

      headers['Content-Type'] = auth['Content-Type']!;

      body = json.encode({'password': auth['password']});

    }

  

    final res = await http.post(

      Uri.parse('${Config.backendBaseUrl}/dsgvo/delete_me.php'),

      headers: headers,

      body: body,

    );

  

    if (res.statusCode == 202) {

      late final Map<String, dynamic> resp;

      try {

        resp = json.decode(res.body) as Map<String, dynamic>;

      } on FormatException {

        return _snack('Server sent non-JSON response');

      }

      final deletionDate = resp['deletion_date'] as String?;

      if (deletionDate != null) {

        await _storage.write(key: 'deletion_date', value: deletionDate);

      }

      _snack('Deletion scheduled – you will be logged out.');

      if (mounted) setState(() {});

    } else {

      _snack('Delete failed: HTTP ${res.statusCode}');

    }

  }

  

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      appBar: AppBar(title: const Text('Sicherheit & Datenschutz')),

      body: Padding(

        padding: const EdgeInsets.all(16.0),

        child: Column(

          children: [

            FutureBuilder<String?>(

              future: _restrictionDate(),

              builder: (_, snap) {

                final val = snap.data;

                if (val == null) return const SizedBox.shrink();

                final date = DateTime.parse(val);

                final days = DateTime.now().difference(date).inDays;

                return Text(

                  'Processing restricted $days day${days == 1 ? '' : 's'} ago',

                  style: const TextStyle(color: Colors.orangeAccent),

                );

              },

            ),

            const SizedBox(height: 16),

            ElevatedButton(

              onPressed: _restrictMe,

              style: ElevatedButton.styleFrom(

                backgroundColor: Colors.orangeAccent,

                minimumSize: const Size.fromHeight(48),

              ),

              child: const Text('Restrict Processing',

                  style: TextStyle(color: Colors.white)),

            ),

            const SizedBox(height: 16),

            FutureBuilder<String?>(

              future: _deletionDate(),

              builder: (_, snap) {

                final val = snap.data;

                if (val == null) return const SizedBox.shrink();

                final deletionDate = DateTime.parse(val);

                final daysLeft = deletionDate.difference(DateTime.now()).inDays;

                return Text(

                  'Your profile will be deleted in $daysLeft day${daysLeft == 1 ? '' : 's'}',

                  style: const TextStyle(color: Colors.redAccent),

                );

              },

            ),

            const SizedBox(height: 16),

            ElevatedButton(

              onPressed: _deleteMe,

              style: ElevatedButton.styleFrom(

                backgroundColor: Colors.redAccent,

                minimumSize: const Size.fromHeight(48),

              ),

              child: const Text('Delete Me',

                  style: TextStyle(color: Colors.white)),

            ),

            const SizedBox(height: 16),

            ElevatedButton(

              onPressed: _exportData,

              style: ElevatedButton.styleFrom(

                  minimumSize: const Size.fromHeight(48)),

              child: const Text('Request My Data',

                  style: TextStyle(color: Colors.black)),

            ),

          ],

        ),

      ),

    );

  }

}
   "
   
   "
   <?php

declare(strict_types=1);

  

// 0 · Error reporting

ini_set('display_errors', '1');

ini_set('display_startup_errors', '1');

ini_set('log_errors', '1');

error_reporting(E_ALL);

  

// 1 · Logging setup

$logDir = dirname(__DIR__, 2) . '/logs';

if (!is_dir($logDir) && !mkdir($logDir, 0775, true) && !is_dir($logDir)) {

    error_log("[method.php] could not create log directory");

}

$logFile = $logDir . '/method.log';

function logDebug(array $lines): void {

    global $logFile;

    $ts = date('c');

    $out = '';

    foreach ($lines as $l) {

        $out .= "[{$ts}] {$l}\n";

    }

    file_put_contents($logFile, $out, FILE_APPEND|LOCK_EX);

}

logDebug(['--- method.php start ---']);

  

// 2 · Bootstrap + config

require_once dirname(__DIR__,2) . '/bootstrap.php';  // auth(), dbg(), autoload

require_once dirname(__DIR__,2) . '/config.php';     // $mongoURI, $mongoDb

  

header('Content-Type: application/json; charset=utf-8');

header('Cache-Control: no-store');

  

// 3 · Authenticate

try {

    logDebug(['Calling auth()']);

    $claims = auth();

    $userId = $claims->sub;

    logDebug(["Authenticated userId={$userId}"]);

} catch (Throwable $e) {

    logDebug(['Auth failed: '.$e->getMessage()]);

    http_response_code(401);

    echo json_encode(['error'=>'Unauthorized']);

    logDebug(['--- method.php end (401) ---']);

    exit;

}

  

// 4 · Fetch user document

try {

    logDebug(["Connecting to MongoDB: {$mongoURI}"]);

    $mgr = new MongoDB\Driver\Manager($mongoURI);

    $qry = new MongoDB\Driver\Query(

        ['_id'=> new MongoDB\BSON\ObjectId($userId)],

        ['projection'=> ['userdata.device_auth'=>1]]

    );

    $rows = $mgr->executeQuery("{$mongoDb}.users", $qry)->toArray();

    if (count($rows) === 0) {

        logDebug(['User not found']);

        http_response_code(404);

        echo json_encode(['error'=>'User not found']);

        logDebug(['--- method.php end (404) ---']);

        exit;

    }

    $user = $rows[0];

    logDebug(['Fetched user document']);

    // 5 · Build methods

    $methods = [];

    if (!empty($user->userdata->device_auth->enabled)) {

        $methods[] = 'device_auth';

    }

    $methods[] = 'password';

    echo json_encode(['methods' => $methods]);

    logDebug(['Responded with methods: ' . implode(',', $methods)]);

    logDebug(['--- method.php end (200) ---']);

    exit;

} catch (Throwable $e) {

    logDebug(['Exception: '.$e->getMessage()]);

    http_response_code(500);

    echo json_encode(['error'=>'Server error']);

    logDebug(['--- method.php end (500) ---']);

    exit;

}
   "
"
<?php

declare(strict_types=1);

  

// 0 · Error/reporting

ini_set('display_errors', '1');

ini_set('display_startup_errors','1');

ini_set('log_errors','1');

error_reporting(E_ALL);

  

// 1 · Logging setup

$logDir = dirname(__DIR__, 2) . '/logs';

if (!is_dir($logDir) && !mkdir($logDir, 0775, true)) {

    error_log("[request_me.php] could not create log directory");

}

$logFile = $logDir . '/request_me.log';

function logDebug(array $lines): void {

    global $logFile;

    $ts = date('c');

    foreach ($lines as $l) {

        file_put_contents($logFile, "[{$ts}] {$l}\n", FILE_APPEND|LOCK_EX);

    }

}

logDebug(['--- request_me.php start ---']);

  

// 2 · Bootstrap + headers

require_once dirname(__DIR__,2) . '/bootstrap.php';  // auth(), etc.

require_once dirname(__DIR__,2) . '/config.php';     // $mongoURI, $mongoDb

  

header('Content-Type: application/json; charset=utf-8');

header('Cache-Control: no-store');

  

// 3 · Authenticate user

try {

    logDebug(['Calling auth()']);

    $claims = auth();

    $userId = $claims->sub;

    logDebug(["Authenticated userId={$userId}"]);

} catch (Throwable $e) {

    logDebug(['Auth failed: '.$e->getMessage()]);

    http_response_code(401);

    echo json_encode(['error'=>'Unauthorized']);

    exit;

}

  

try {

    // 4 · Fetch user document

    logDebug(["Connecting to MongoDB: {$mongoURI}"]);

    $mgr = new MongoDB\Driver\Manager($mongoURI);

    $qry = new MongoDB\Driver\Query(

        ['_id'=>new MongoDB\BSON\ObjectId($userId)],

        ['projection'=>['profile'=>1,'search_filter'=>1,'userdata'=>1,'chats'=>1]]

    );

    $rows = $mgr->executeQuery("{$mongoDb}.users", $qry)->toArray();

    if (count($rows) === 0) {

        logDebug(['User not found']);

        http_response_code(404);

        echo json_encode(['error'=>'User not found']);

        exit;

    }

    $user = $rows[0];

    logDebug(['Fetched user document']);

  

    // 4.1 · Credential check

    $method = $_SERVER['HTTP_X_AUTH_METHOD'] ?? '';

    if ($method === 'password') {

        $input = json_decode(file_get_contents('php://input'), true);

        $pwd = $input['password'] ?? '';

        if (!password_verify($pwd, $user->userdata->password_hash ?? '')) {

            http_response_code(401);

            echo json_encode(['error'=>'Unauthorized']);

            exit;

        }

        logDebug(['Password verified']);

    } elseif ($method === 'device_auth') {

        if (($_SERVER['HTTP_X_DEVICE_AUTH'] ?? '') !== '1') {

            http_response_code(401);

            echo json_encode(['error'=>'Unauthorized']);

            exit;

        }

        logDebug(['Device auth header OK']);

    } else {

        http_response_code(400);

        echo json_encode(['error'=>'Missing auth method']);

        exit;

    }

  

    // 5 · Build personal-data package (Art 15 GDPR)

    $package = [

        'profile'        => $user->profile        ?? new stdClass,

        'search_filter'  => $user->search_filter  ?? new stdClass,

        'userdata'       => $user->userdata       ?? new stdClass,

        'chats'          => $user->chats          ?? new stdClass,

        'exported_at'    => gmdate('c'),

        'schema_version' => 'v1',

    ];

    logDebug(['Built export package']);

  

    // 5.1 · Article 15 contextual metadata

    $meta = [

        'purposes'         => ['profile management','matching & chat'],           // :contentReference[oaicite:11]{index=11}

        'data_categories'  => ['profile','search_filter','userdata','chats'],     // :contentReference[oaicite:12]{index=12}

        'recipients'       => ['internal_team','mailer_service'],                // :contentReference[oaicite:13]{index=13}

        'retention'        => '30 days after deletion request',                  // :contentReference[oaicite:14]{index=14}

        'rights'           => ['rectification','erasure','restriction','objection'], // :contentReference[oaicite:15]{index=15}

        'complaint'        => 'https://your-domain.tld/supervisory-authority',     // :contentReference[oaicite:16]{index=16}

        'source'           => 'direct user input and system timestamps',         // :contentReference[oaicite:17]{index=17}

        'automated_decision'=> 'none',                                           // :contentReference[oaicite:18]{index=18}

        'safeguards'       => ['SCC for third-country transfers'],               // :contentReference[oaicite:19]{index=19}

    ];

  

    // 6 · Write JSON file to public/exports

    $exportDir = dirname(__DIR__,2).'/public/exports';

    if (!is_dir($exportDir) && !mkdir($exportDir,0755,true)) {

        throw new \RuntimeException("Cannot create exportDir {$exportDir}");

    }

    $filename = "{$userId}_" . time() . ".json";

    $fullpath = "{$exportDir}/{$filename}";

    $out = json_encode(['package'=>$package, 'meta'=>$meta], JSON_PRETTY_PRINT);

    if (false === file_put_contents($fullpath, $out)) {

        throw new \RuntimeException("Failed writing export to {$fullpath}");

    }

    logDebug(["Wrote export to: {$fullpath}"]);

  

    // 7 · Return HTTP URL to the export

    $scheme = (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS']!=='off') ? 'https' : 'http';

    $host   = $_SERVER['HTTP_HOST'] ?? 'your-domain.tld';

    $url    = "{$scheme}://{$host}/exports/{$filename}";

    echo json_encode(['url'=>$url, 'meta'=>$meta]);

    logDebug(["Responded with URL and meta"]);

    logDebug(['--- request_me.php end (200) ---']);

    exit;

  

} catch (Throwable $e) {

    logDebug(['Exception: '.$e->getMessage(), $e->getTraceAsString()]);

    http_response_code(500);

    echo json_encode(['error'=>'Server error','message'=>$e->getMessage()]);

    exit;

}" 
