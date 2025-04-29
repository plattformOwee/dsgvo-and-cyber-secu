// EDIT‑PROFILE FLOW – single file implementation

// ------------------------------------------------------------

// This file bundles an _EditProfileLayout_ (the main screen) and

// lightweight **edit‑screens** for every complex section that was

// previously spread across the sign‑up wizard. Each edit‑screen

// works in _modal_ mode: the user changes a value, taps **Save**, and

// the screen pops – returning the updated data to the caller.

//

// ⚠️ For brevity the networking code is reduced to TODO markers – you

// can simply paste the full POST logic you already have in the

// corresponding places. All UI/UX behaviour is preserved.

// ------------------------------------------------------------

import 'dart:convert';

import 'dart:io';

import 'package:flutter/material.dart';

import 'package:flutter_svg/svg.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:image_cropper/image_cropper.dart';

import 'package:image_picker/image_picker.dart';

import 'package:latlong2/latlong.dart';

import 'package:flutter_map/flutter_map.dart';

import 'package:reorderables/reorderables.dart';

import 'package:swipe_chat_play/config.dart';

/* ────────────────────────────────────────────────────────────

* MAIN LAYOUT – looks like AboutYouLayout + extra rows

* ───────────────────────────────────────────────────────────*/

class EditProfileLayout extends StatefulWidget {

const EditProfileLayout({super.key});

@override

State<EditProfileLayout> createState() => _EditProfileLayoutState();

}

class _EditProfileLayoutState extends State<EditProfileLayout> {

// Simple string fields

String? name;

String? age;

String? gender;

String? spokenLanguage;

String? religion;

String? career;

String? politics;

// Complex fields – we only keep HUMAN‑READABLE summaries here

List<File?> profileImages = List.filled(9, null);

double? datingRadius; // km

LatLng? datingLocation;

List<Map<String, String>>? qaItems;

Set<String>? openToActivities;

Map<String, dynamic>? filters; // gender/age etc.

final _storage = const FlutterSecureStorage();

/* ---------------------- helpers ------------------------ */

Future<void> _showTextInput({

required String title,

required String? current,

TextInputType type = TextInputType.text,

required void Function(String) onDone,

}) async {

final controller = TextEditingController(text: current);

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

const SizedBox(height: 10),

TextField(controller: controller, keyboardType: type),

const SizedBox(height: 10),

ElevatedButton(

onPressed: () {

onDone(controller.text);

Navigator.pop(ctx);

},

child: const Text('Save'),

),

const SizedBox(height: 10),

]),

),

);

}

Widget _row(String title, String subtitle, VoidCallback onTap,

{bool showDivider = true}) {

return Material(

color: Colors.transparent,

child: InkWell(

onTap: onTap,

child: Column(

children: [

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

),

),

const Icon(Icons.chevron_right, color: Colors.grey),

],

),

),

if (showDivider)

const Divider(height: 1, thickness: 0.5, color: Colors.grey),

],

),

),

);

}

/* ---------------- navigation targets ------------------- */

Future<void> _editImages() async {

final result = await Navigator.push<List<File?>>(

context,

MaterialPageRoute(

builder: (_) => EditProfilbilderLayout(initial: profileImages)));

if (result != null) setState(() => profileImages = result);

}

Future<void> _editDatingRadius() async {

final res = await Navigator.push<Map<String, dynamic>>(

context,

MaterialPageRoute(

builder: (_) => EditDatingRadiusPage(

initialRadius: datingRadius, initialPos: datingLocation)));

if (res != null) {

setState(() {

datingRadius = res['radius'];

datingLocation = res['location'];

});

}

}

Future<void> _editQA() async {

final res = await Navigator.push<List<Map<String, String>>>(

context,

MaterialPageRoute(

builder: (_) => EditProfileElementsPage(initial: qaItems)));

if (res != null) setState(() => qaItems = res);

}

Future<void> _editOpenTo() async {

final res = await Navigator.push<Set<String>>(

context,

MaterialPageRoute(

builder: (_) => EditOpenToPage(initial: openToActivities)));

if (res != null) setState(() => openToActivities = res);

}

Future<void> _editFilters() async {

final res = await Navigator.push<Map<String, dynamic>>(

context,

MaterialPageRoute(

builder: (_) => EditFilterSelectionPage(initial: filters)));

if (res != null) setState(() => filters = res);

}

/* ------------------------------------------------------- */

@override

Widget build(BuildContext context) {

return Scaffold(

appBar: AppBar(title: const Text('Edit profile')),

body: ListView(

children: [

const SizedBox(height: 16),

// simple fields

_row(

'Name',

name ?? 'Add your name',

() => _showTextInput(

title: 'Your name',

current: name,

onDone: (v) => setState(() => name = v))),

_row(

'Age',

age ?? 'Set your age',

() => _showTextInput(

title: 'Your age',

current: age,

type: TextInputType.number,

onDone: (v) => setState(() => age = v))),

_row('Gender', gender ?? 'Choose your gender', () async {

const opts = ['Male', 'Female', 'Non‑binary', 'Other'];

final sel = await showModalBottomSheet<String>(

context: context,

builder: (ctx) => ListView(

children: opts

.map((e) => ListTile(

title: Text(e), onTap: () => Navigator.pop(ctx, e)))

.toList(),

),

);

if (sel != null) setState(() => gender = sel);

}),

_row(

'Languages',

spokenLanguage ?? 'Add languages',

() => _showTextInput(

title: 'Languages (comma separated)',

current: spokenLanguage,

onDone: (v) => setState(() => spokenLanguage = v))),

_row('Religion', religion ?? 'Choose religion', () async {

const opts = [

'Christianity',

'Islam',

'Hinduism',

'Buddhism',

'Judaism',

'Other'

];

final sel = await showModalBottomSheet<String>(

context: context,

builder: (ctx) => ListView(

children: opts

.map((e) => ListTile(

title: Text(e), onTap: () => Navigator.pop(ctx, e)))

.toList(),

),

);

if (sel != null) setState(() => religion = sel);

}),

_row(

'Career',

career ?? 'Add your job / studies',

() => _showTextInput(

title: 'Career',

current: career,

onDone: (v) => setState(() => career = v))),

_row('Politics', politics ?? 'Political view', () async {

const opts = [

'Liberal',

'Conservative',

'Centrist',

'Libertarian',

'Socialist',

'Other'

];

final sel = await showModalBottomSheet<String>(

context: context,

builder: (ctx) => ListView(

children: opts

.map((e) => ListTile(

title: Text(e), onTap: () => Navigator.pop(ctx, e)))

.toList(),

),

);

if (sel != null) setState(() => politics = sel);

}),

// complex sections

const SizedBox(height: 16),

_row('Profile pictures', _imagesSubtitle(), _editImages),

_row('Dating radius', _radiusSubtitle(), _editDatingRadius),

_row('Q & A', _qaSubtitle(), _editQA),

_row('Open to', _openToSubtitle(), _editOpenTo),

_row('Search filters', _filtersSubtitle(), _editFilters,

showDivider: false),

const SizedBox(height: 24),

],

),

floatingActionButton: FloatingActionButton(

child: const Icon(Icons.save),

onPressed: _saveEverything,

),

);

}

/* ---------------- summaries for subtitles -------------- */

String _imagesSubtitle() {

final cnt = profileImages.where((e) => e != null).length;

return cnt == 0 ? 'Add some photos' : '$cnt / 9 photos selected';

}

String _radiusSubtitle() {

if (datingRadius == null) return 'Set radius & location';

return '${datingRadius!.toStringAsFixed(0)} km around you';

}

String _qaSubtitle() {

final c = qaItems?.length ?? 0;

return c == 0 ? 'Pick & answer 3 questions' : '$c questions answered';

}

String _openToSubtitle() {

final c = openToActivities?.length ?? 0;

return c == 0 ? 'Select activities' : '$c activities chosen';

}

String _filtersSubtitle() {

if (filters == null) return 'Set who you search for';

final g = (filters!['genders'] as List?)?.join(', ') ?? 'custom';

return 'Genders: $g';

}

/* ---------------- save to backend ---------------------- */

Future<void> _saveEverything() async {

// TODO: merge the request bodies you already use for each section

// and POST them to a dedicated "update_profile" endpoint.

ScaffoldMessenger.of(context).showSnackBar(

const SnackBar(content: Text('Saving… (implement your API call here)')),

);

}

}

/* ────────────────────────────────────────────────────────────

* EDIT – PROFILE IMAGES (simplified from original)

* ───────────────────────────────────────────────────────────*/

class EditProfilbilderLayout extends StatefulWidget {

final List<File?> initial;

const EditProfilbilderLayout({super.key, required this.initial});

@override

State<EditProfilbilderLayout> createState() => _EditProfilbilderLayoutState();

}

class _EditProfilbilderLayoutState extends State<EditProfilbilderLayout> {

late List<File?> _images;

final _picker = ImagePicker();

@override

void initState() {

super.initState();

_images = List<File?>.from(widget.initial);

}

Future<void> _pick(int idx) async {

final source = await showModalBottomSheet<ImageSource>(

context: context,

builder: (ctx) => Column(mainAxisSize: MainAxisSize.min, children: [

ListTile(

leading: const Icon(Icons.camera_alt),

title: const Text('Camera'),

onTap: () => Navigator.pop(ctx, ImageSource.camera)),

ListTile(

leading: const Icon(Icons.photo_library),

title: const Text('Gallery'),

onTap: () => Navigator.pop(ctx, ImageSource.gallery)),

]),

);

if (source == null) return;

final file = await _picker.pickImage(source: source);

if (file == null) return;

// Optionally crop here …

setState(() => _images[idx] = File(file.path));

}

void _reorder(int oldIndex, int newIndex) {

setState(() {

if (oldIndex < newIndex) newIndex--;

final item = _images.removeAt(oldIndex);

_images.insert(newIndex, item);

});

}

@override

Widget build(BuildContext context) {

final size = MediaQuery.of(context).size.width / 3 - 24;

return Scaffold(

appBar: AppBar(actions: [

TextButton(

onPressed: () => Navigator.pop(context, _images),

child: const Text('Save', style: TextStyle(color: Colors.white)))

], title: const Text('Photos')),

body: Padding(

padding: const EdgeInsets.all(16),

child: ReorderableWrap(

spacing: 16,

runSpacing: 16,

onReorder: _reorder,

children: List.generate(

9,

(i) => GestureDetector(

key: Key('$i'),

onTap: () => _pick(i),

child: Container(

width: size,

height: size * 1.2,

decoration: BoxDecoration(

color: Colors.grey[200],

border: Border.all(color: Colors.grey),

borderRadius: BorderRadius.circular(8),

),

child: _images[i] == null

? const Icon(Icons.add_a_photo)

: ClipRRect(

borderRadius: BorderRadius.circular(6),

child:

Image.file(_images[i]!, fit: BoxFit.cover)),

),

)),

),

),

);

}

}

/* ────────────────────────────────────────────────────────────

* EDIT – DATING RADIUS (trimmed version)

* ───────────────────────────────────────────────────────────*/

class EditDatingRadiusPage extends StatefulWidget {

final double? initialRadius;

final LatLng? initialPos;

const EditDatingRadiusPage({super.key, this.initialRadius, this.initialPos});

@override

State<EditDatingRadiusPage> createState() => _EditDatingRadiusPageState();

}

class _EditDatingRadiusPageState extends State<EditDatingRadiusPage> {

double _radius = 40;

LatLng? _pos;

late final MapController _mc;

@override

void initState() {

super.initState();

_radius = widget.initialRadius ?? 40;

_pos = widget.initialPos;

_mc = MapController();

}

@override

Widget build(BuildContext context) {

return Scaffold(

appBar: AppBar(title: const Text('Dating radius'), actions: [

TextButton(

onPressed: () =>

Navigator.pop(context, {'radius': _radius, 'location': _pos}),

child: const Text('Save', style: TextStyle(color: Colors.white)))

]),

body: Column(children: [

Expanded(

child: _pos == null

? Center(

child: ElevatedButton(

onPressed: _setLocation,

child: const Text('Use current location')))

: FlutterMap(

mapController: _mc,

options: MapOptions(initialCenter: _pos!, initialZoom: 10),

children: [

TileLayer(

urlTemplate:

'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',

subdomains: ['a', 'b', 'c']),

CircleLayer(circles: [

CircleMarker(

point: _pos!,

radius: _radius * 1000,

color: Colors.blue.withOpacity(0.3),

borderColor: Colors.blue,

borderStrokeWidth: 2,

useRadiusInMeter: true),

]),

MarkerLayer(markers: [

Marker(

point: _pos!,

width: 40,

height: 40,

child: const Icon(Icons.location_pin,

color: Colors.red, size: 40)),

]),

],

),

),

Slider(

min: 10,

max: 100,

divisions: 90,

value: _radius,

onChanged: (v) => setState(() => _radius = v),

),

const SizedBox(height: 16),

]),

);

}

Future<void> _setLocation() async {

// TODO: request location permission; use Geolocator to fetch current position

// For demo we put Berlin centre

setState(() => _pos = const LatLng(52.52, 13.405));

}

}

/* ────────────────────────────────────────────────────────────

* EDIT – PROFILE Q&A (simplified)

* ───────────────────────────────────────────────────────────*/

class EditProfileElementsPage extends StatefulWidget {

final List<Map<String, String>>? initial;

const EditProfileElementsPage({super.key, this.initial});

@override

State<EditProfileElementsPage> createState() =>

_EditProfileElementsPageState();

}

class _EditProfileElementsPageState extends State<EditProfileElementsPage> {

late List<Map<String, String>?> _qa; // allow null slots

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

appBar: AppBar(actions: [

TextButton(

onPressed: () => Navigator.pop(

context, _qa.whereType<Map<String, String>>().toList()),

child: const Text('Save', style: TextStyle(color: Colors.white)))

], title: const Text('Q & A')),

body: Padding(

padding: const EdgeInsets.all(16),

child: Column(children: List.generate(3, (i) => _card(i))),

),

);

}

Widget _card(int i) {

final qa = _qa[i];

return Expanded(

child: GestureDetector(

onTap: () async {

final res = await _editCard(qa);

if (res != null) setState(() => _qa[i] = res);

},

child: Container(

margin: const EdgeInsets.only(bottom: 16),

padding: const EdgeInsets.all(16),

decoration: BoxDecoration(

border: Border.all(color: Colors.grey),

borderRadius: BorderRadius.circular(16)),

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

mainAxisAlignment: MainAxisAlignment.center,

children: [

Text(qa?['question'] ?? 'Pick a question',

style: const TextStyle(

fontSize: 16, fontWeight: FontWeight.bold)),

const SizedBox(height: 8),

Text(qa?['answer'] ?? 'Tap to answer',

style: const TextStyle(fontSize: 14, color: Colors.grey)),

]),

),

),

);

}

Future<Map<String, String>?> _editCard(Map<String, String>? qa) async {

String? q = qa?['question'];

String? a = qa?['answer'];

return await showDialog<Map<String, String>>(

context: context,

builder: (ctx) {

final qCtrl = TextEditingController(text: q);

final aCtrl = TextEditingController(text: a);

return AlertDialog(

title: const Text('Edit Q & A'),

content: Column(mainAxisSize: MainAxisSize.min, children: [

TextField(

controller: qCtrl,

decoration: const InputDecoration(labelText: 'Question')),

TextField(

controller: aCtrl,

decoration: const InputDecoration(labelText: 'Answer')),

]),

actions: [

TextButton(

onPressed: () => Navigator.pop(ctx),

child: const Text('Cancel')),

ElevatedButton(

onPressed: () => Navigator.pop(

ctx, {'question': qCtrl.text, 'answer': aCtrl.text}),

child: const Text('Save')),

],

);

},

);

}

}

/* ────────────────────────────────────────────────────────────

* EDIT – OPEN TO ACTIVITIES (simplified)

* ───────────────────────────────────────────────────────────*/

class EditOpenToPage extends StatefulWidget {

final Set<String>? initial;

const EditOpenToPage({super.key, this.initial});

@override

State<EditOpenToPage> createState() => _EditOpenToPageState();

}

class _EditOpenToPageState extends State<EditOpenToPage> {

final List<String> _all = [

'jogging',

'hiking',

'cooking',

'games',

'movies',

'surfing',

'dance',

// … add the rest

];

late Set<String> _sel;

@override

void initState() {

super.initState();

_sel = Set<String>.from(widget.initial ?? {});

}

@override

Widget build(BuildContext context) {

return Scaffold(

appBar: AppBar(actions: [

TextButton(

onPressed: () => Navigator.pop(context, _sel),

child: const Text('Save', style: TextStyle(color: Colors.white)))

], title: const Text('Open to')),

body: Padding(

padding: const EdgeInsets.all(16),

child:

Wrap(spacing: 8, runSpacing: 8, children: _all.map(_chip).toList()),

),

);

}

Widget _chip(String k) {

final sel = _sel.contains(k);

return ChoiceChip(

label: Text(k),

selected: sel,

onSelected: (_) => setState(() => sel ? _sel.remove(k) : _sel.add(k)),

);

}

}

/* ────────────────────────────────────────────────────────────

* EDIT – FILTERS (simplified)

* ───────────────────────────────────────────────────────────*/

class EditFilterSelectionPage extends StatefulWidget {

final Map<String, dynamic>? initial;

const EditFilterSelectionPage({super.key, this.initial});

@override

State<EditFilterSelectionPage> createState() =>

_EditFilterSelectionPageState();

}

class _EditFilterSelectionPageState extends State<EditFilterSelectionPage> {

final List<String> genders = ['Male', 'Female', 'Non‑binary'];

late Set<String> _selGender;

RangeValues _age = const RangeValues(18, 120);

@override

void initState() {

super.initState();

_selGender = Set<String>.from(widget.initial?['genders'] ?? genders);

if (widget.initial?['ageRange'] != null) {

final ar = widget.initial!['ageRange'];

_age = RangeValues((ar[0] as num).toDouble(), (ar[1] as num).toDouble());

}

}

@override

Widget build(BuildContext context) {

return Scaffold(

appBar: AppBar(actions: [

TextButton(

onPressed: () => Navigator.pop(context, {

'genders': _selGender.toList(),

'ageRange': [_age.start.round(), _age.end.round()],

}),

child: const Text('Save', style: TextStyle(color: Colors.white)))

], title: const Text('Search filters')),

body: Padding(

padding: const EdgeInsets.all(16),

child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [

const Text('Gender', style: TextStyle(fontWeight: FontWeight.bold)),

Wrap(spacing: 8, children: genders.map(_gChip).toList()),

const SizedBox(height: 24),

const Text('Age range',

style: TextStyle(fontWeight: FontWeight.bold)),

RangeSlider(

min: 18,

max: 120,

divisions: 102,

labels: RangeLabels('${_age.start.round()}', '${_age.end.round()}'),

values: _age,

onChanged: (v) => setState(() => _age = v),

),

]),

),

);

}

Widget _gChip(String g) => FilterChip(

label: Text(g),

selected: _selGender.contains(g),

onSelected: (_) => setState(() =>

_selGender.contains(g) ? _selGender.remove(g) : _selGender.add(g)),

);

}" // rewrite to not include all these special layouts (profile picstures, dating radius, q&a, open to, search filters.

And instead rewrite each of those layouts as individual files but refactored such that they link back to this page

"

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

"

"

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

"

"

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

"

"

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

"

"

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

"