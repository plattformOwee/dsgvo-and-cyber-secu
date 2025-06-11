	The Goal of this chat is to make my Profile layout coherent.

Currently there are different paddings top and bottom for the different elements and also left and right padding is not consistant.

Task1: get to know the current state:

"

import 'dart:convert';

import 'dart:math' as math;

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

import 'package:swipe_chat_play/profile_layout/widgets/image_and_prompt_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/question_input_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/top_section.dart';

import 'package:swipe_chat_play/profile_layout/widgets/topic_suggestions_layout.dart';

import 'package:swipe_chat_play/profile_layout/widgets/voice_memo_layout.dart';

// Import edit pages

import 'package:swipe_chat_play/tabs/profile/edit_profile_qa_page.dart';

import 'package:swipe_chat_play/tabs/profile/edit_profile_activities_page.dart';

/// View the current user's profile.

/// Top section is fixed; elements can be tapped to edit or reordered by long-pressing.

class ViewMyProfile extends StatefulWidget {

const ViewMyProfile({super.key});

@override

State<ViewMyProfile> createState() => _ViewMyProfileState();

}

class _ViewMyProfileState extends State<ViewMyProfile> {

static const storage = FlutterSecureStorage();

late Future<profile.ProfileData> _futureProfile;

List<MapEntry<String, profile.ProfileElement>>? _entries;

profile.ProfileData? _fullProfile;

@override

void initState() {

super.initState();

_futureProfile = _fetchProfile();

}

Future<profile.ProfileData> _fetchProfile() async {

final url = '${Config.backendBaseUrl}/create_profile/get_profile.php';

final token = await storage.read(key: 'jwt_token');

if (token == null) throw Exception('No JWT token found');

final response = await http.get(

Uri.parse(url),

headers: {'Authorization': 'Bearer $token'},

);

debugPrint('[fetchProfile] ${response.body}');

if (response.statusCode == 200) {

final map = json.decode(response.body) as Map<String, dynamic>;

// Ensure `elements` is a Map<String, dynamic> for fromJson()

final rawElements = map['elements'];

if (rawElements is List) {

final asMap = <String, dynamic>{};

for (var i = 0; i < rawElements.length; i++) {

asMap['${i + 1}'] = rawElements[i];

}

map['elements'] = asMap;

} else if (rawElements is! Map<String, dynamic>) {

map['elements'] = <String, dynamic>{};

}

return profile.ProfileData.fromJson(map);

}

throw Exception('Failed to load profile. Status ${response.statusCode}');

}

/// Save the reordered structure to the server.

Future<void> uploadChange() async {

final url = '${Config.backendBaseUrl}/create_profile/saveProfile.php';

final token = await storage.read(key: 'jwt_token');

if (token == null) return;

// Build a List of element-maps { _id, type, content }

final elementsList = _entries!.map((e) {

final json = e.value.toJson(); // { "type": "...", "content": {...} }

json['_id'] = e.key; // insert the ObjectId string

return json;

}).toList();

// Grab the rest of the profile

final map = _fullProfile!.toJson();

map['elements'] = elementsList; // overwrite 'elements' with the List

final payload = jsonEncode(map);

debugPrint('[uploadChange] Payload: $payload');

final res = await http.post(

Uri.parse(url),

headers: {

'Authorization': 'Bearer $token',

'Content-Type': 'application/json'

},

body: payload,

);

debugPrint('[uploadChange] ⇣ ${res.statusCode} ${res.body}');

}

/// Save edits to a single element via update_userdata endpoint.

Future<void> _saveElementChange(Map<String, dynamic> update) async {

final token = await storage.read(key: 'jwt_token');

if (token == null) return;

try {

final res = await http.post(

Uri.parse(

'${Config.backendBaseUrl}/create_profile/update_userdata.php'),

headers: {

'Authorization': 'Bearer $token',

'Content-Type': 'application/json'

},

body: jsonEncode(update),

);

debugPrint('[saveElement] ${res.statusCode} ${res.body}');

} catch (e) {

debugPrint('[saveElement] error: $e');

}

}

/// Handle taps on elements to route to the appropriate editor.

Future<void> _editElement(

int index, String key, profile.ProfileElement element) async {

switch (element.type) {

case 'icebreaker':

// Single Q&A

final initial = <Map<String, String>>[

{

'question': element.content['question']?.toString() ?? '',

'answer': element.content['answer']?.toString() ?? ''

}

];

final result = await Navigator.push<List<Map<String, String>>>(

context,

MaterialPageRoute(builder: (_) => EditQAPage(initial: initial)),

);

if (result != null && result.isNotEmpty) {

final qa = result[0];

setState(() {

_entries![index] = MapEntry(

key,

profile.ProfileElement(

type: 'icebreaker',

content: {

'question': qa['question']!,

'answer': qa['answer']!

},

));

});

await _saveElementChange({'questions_and_answers': result});

}

break;

case 'bubbles':

// Activities

final initialBubbles =

List<String>.from(element.content['bubbles'] ?? []);

final result = await Navigator.push<Set<String>>(

context,

MaterialPageRoute(

builder: (_) =>

EditActivitiesPage(initial: initialBubbles.toSet())),

);

if (result != null) {

final newList = result.toList();

setState(() {

_entries![index] = MapEntry(

key,

profile.ProfileElement(

type: 'bubbles',

content: {

'title': element.content['title'] ?? 'Open to',

'bubbles': newList

},

));

});

await _saveElementChange({'open_to': newList});

}

break;

default:

// No editor

break;

}

}

@override

Widget build(BuildContext context) {

return Scaffold(

backgroundColor: AppColors.background,

body: FutureBuilder<profile.ProfileData>(

future: _futureProfile,

builder: (context, snapshot) {

if (snapshot.connectionState == ConnectionState.waiting) {

return const Center(child: CircularProgressIndicator());

} else if (snapshot.hasError) {

return _buildError(snapshot.error);

} else if (!snapshot.hasData) {

return _buildError('No profile data found.');

}

final userProfile = snapshot.data!;

_fullProfile ??= userProfile;

_entries ??= userProfile.elements.entries.toList(growable: true);

return Padding(

padding: const EdgeInsets.all(8),

child: Card(

color: AppColors.cardBackground,

shape: RoundedRectangleBorder(

borderRadius: BorderRadius.circular(16)),

child: SingleChildScrollView(

physics: const BouncingScrollPhysics(),

child: Column(

crossAxisAlignment: CrossAxisAlignment.stretch,

children: [

TopSectionWidget(topSection: userProfile.topSection),

const Divider(

height: 2, thickness: 2, color: AppColors.dividerLine),

ReorderableListView.builder(

shrinkWrap: true,

physics: const NeverScrollableScrollPhysics(),

padding: const EdgeInsets.symmetric(vertical: 8),

itemCount: _entries!.length,

buildDefaultDragHandles: false,

onReorder: (oldIndex, newIndex) async {

setState(() {

if (newIndex > oldIndex) newIndex -= 1;

final moved = _entries!.removeAt(oldIndex);

_entries!.insert(newIndex, moved);

});

await uploadChange();

},

itemBuilder: (context, index) {

final entry = _entries![index];

final element = entry.value;

return ReorderableDelayedDragStartListener(

index: index,

key: ValueKey('${entry.key}-$index'),

child: GestureDetector(

behavior: HitTestBehavior.opaque,

onTap: () =>

_editElement(index, entry.key, element),

child: Container(

width: double.infinity,

color: Colors.white,

padding: const EdgeInsets.symmetric(

vertical: 8, horizontal: 16),

child: Column(

crossAxisAlignment: CrossAxisAlignment.stretch,

children: [

_buildElement(

element, userProfile.topSection.name),

if (index != _entries!.length - 1)

const Divider(

height: 2,

thickness: 2,

color: AppColors.dividerLine,

),

],

),

),

),

);

},

),

],

),

),

),

);

},

),

);

}

Widget _buildError(Object? error) => Center(

child: Text('Error: $error', style: GoogleFonts.montserrat()),

);

Widget _buildElement(

profile.ProfileElement element,

String swipedUserUsername,

) {

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

case 'question_and_inputfield':

return QuestionInputLayout(

question: element.content['question'] as String? ?? '',

initialAnswer: element.content['answer'] as String? ?? '',

onSubmitted: (String newAnswer) {

// send updated answer to server

_saveElementChange({

'questions_and_answers': [

{

'question': element.content['question'],

'answer': newAnswer,

}

]

});

// optionally you could setState here to update local UI

},

);

// inside your ProfileCard switch:

case 'voicememo':

return VoiceMemoLayout(

prompt: element.content['prompt'] as String? ?? '',

audioUrl: element.content['link_voicemessage'] as String? ?? '',

isSender: false, // or true if needed

onPlayPressed: () {

// optional: analytics, toast, etc.

},

);

case 'topic_suggestions':

return TopicSuggestionsLayout(

content: element.content,

onOptionTap: (String choice) {

// send selected topic to server

_saveElementChange({

'topic_suggestions': {

'prompt': element.content['prompt'],

'selected': choice,

}

});

// optionally update local UI

},

);

case 'image_and_prompt':

return ImageAndPromptLayout(

content: element.content,

onImageTap: () {

// TODO: full-screen preview or editing

},

);

default:

return const SizedBox.shrink();

}

}

}

"

// lib/profile_layout/widgets/image_and_prompt_layout.dart

import 'package:flutter/material.dart';

import 'package:cached_network_image/cached_network_image.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

/// Renders a prompt above a single image, all wrapped in

/// a 2.5px #BAB0AB rounded(13) outline. The prompt area

/// now only rounds its top corners (11px), so you’ll still

/// see the outer border peeking at 13px.

class ImageAndPromptLayout extends StatelessWidget {

final Map<String, dynamic> content;

final VoidCallback? onImageTap;

const ImageAndPromptLayout({

Key? key,

required this.content,

this.onImageTap,

}) : super(key: key);

@override

Widget build(BuildContext context) {

final prompt = content['prompt'] as String? ?? '';

final imageUrl = content['image_link'] as String? ?? '';

return Padding(

padding: const EdgeInsets.all(13),

child: Container(

// Outer: 2.5px border @ 13px radius

decoration: BoxDecoration(

border: Border.all(color: const Color(0xFFBAB0AB), width: 2.5),

borderRadius: BorderRadius.circular(13),

),

child: Column(

crossAxisAlignment: CrossAxisAlignment.stretch,

children: [

// Prompt area: white, only top corners rounded (11px)

Container(

decoration: const BoxDecoration(

color: Colors.white,

borderRadius: BorderRadius.only(

topLeft: Radius.circular(11),

topRight: Radius.circular(11),

),

),

padding: const EdgeInsets.symmetric(

horizontal: 12,

vertical: 8,

),

child: Text(

prompt,

style: AppFonts.secondaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w400,

color: AppColors.textOnWhite,

),

),

),

// Image: only bottom corners rounded (11px)

if (imageUrl.isNotEmpty)

ClipRRect(

borderRadius: const BorderRadius.only(

bottomLeft: Radius.circular(11),

bottomRight: Radius.circular(11),

),

child: GestureDetector(

onTap: onImageTap,

child: CachedNetworkImage(

imageUrl: imageUrl,

height: 200,

width: double.infinity,

fit: BoxFit.cover,

placeholder: (_, __) =>

Container(color: Colors.grey.shade200, height: 200),

errorWidget: (_, __, ___) =>

Container(color: Colors.grey.shade400, height: 200),

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

"

// lib/profile_layout/widgets/topic_suggestions_layout.dart

import 'package:flutter/material.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

/// Renders a prompt and a vertical list of options for "topic_suggestions".

///

/// content = {

/// 'prompt': 'Frag mich etwas über:',

/// 'options': ['musik', 'kunst', 'geschichte']

/// }

class TopicSuggestionsLayout extends StatelessWidget {

final Map<String, dynamic> content;

/// Called when the user taps one of the options.

final void Function(String option)? onOptionTap;

const TopicSuggestionsLayout({

Key? key,

required this.content,

this.onOptionTap,

}) : super(key: key);

@override

Widget build(BuildContext context) {

final prompt = content['prompt'] as String? ?? '';

final options =

(content['options'] as List<dynamic>?)?.cast<String>() ?? [];

return Padding(

padding: const EdgeInsets.all(12),

child: Column(

crossAxisAlignment: CrossAxisAlignment.start,

children: [

// Prompt text, white on card background

if (prompt.isNotEmpty) ...[

Text(

prompt,

style: AppFonts.secondaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w400,

color: AppColors.textOnWhite,

),

),

const SizedBox(height: 8),

],

// Each option styled like the question‐input field

...options.map((opt) => Padding(

padding: const EdgeInsets.symmetric(vertical: 4),

child: GestureDetector(

onTap: () => onOptionTap?.call(opt),

child: Container(

width: double.infinity,

padding: const EdgeInsets.symmetric(

horizontal: 17, vertical: 13),

decoration: BoxDecoration(

color: const Color(0xFFDED6D1),

border: Border.all(

color: const Color.fromARGB(0, 135, 113, 113),

width: 1,

),

borderRadius: BorderRadius.circular(12),

),

child: Text(

opt,

style: AppFonts.tertiaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w600,

color: AppColors.textOnWhite,

),

),

),

),

)),

const SizedBox(height: 12),

],

),

);

}

}

"

"

// lib/profile_layout/widgets/voice_memo_layout.dart

import 'package:flutter/material.dart';

import 'package:flutter_sound/flutter_sound.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:swipe_chat_play/colors.dart';

import 'package:swipe_chat_play/fonts.dart';

/// Slider track that spans full width

class _TightSliderTrackShape extends RoundedRectSliderTrackShape {

const _TightSliderTrackShape();

@override

Rect getPreferredRect({

required RenderBox parentBox,

Offset offset = Offset.zero,

required SliderThemeData sliderTheme,

bool isEnabled = false,

bool isDiscrete = false,

}) {

final trackHeight = sliderTheme.trackHeight ?? 8;

final trackTop = offset.dy + (parentBox.size.height - trackHeight) / 2;

return Rect.fromLTWH(

offset.dx,

trackTop,

parentBox.size.width,

trackHeight,

);

}

}

/// Renders a voice memo prompt and full-width slider with a play/pause button,

/// *without* any timestamp under it, but with nice bottom padding.

class VoiceMemoLayout extends StatefulWidget {

final String prompt;

final String audioUrl;

final bool isSender;

final String playAsset;

final String stopAsset;

final VoidCallback? onPlayPressed;

const VoiceMemoLayout({

Key? key,

required this.prompt,

required this.audioUrl,

this.isSender = false,

this.playAsset = 'assets/icons/play.svg',

this.stopAsset = 'assets/icons/pause.svg',

this.onPlayPressed,

}) : super(key: key);

@override

_VoiceMemoLayoutState createState() => _VoiceMemoLayoutState();

}

class _VoiceMemoLayoutState extends State<VoiceMemoLayout> {

late final FlutterSoundPlayer _player;

bool _isPlaying = false;

double _position = 0;

double _duration = 1;

@override

void initState() {

super.initState();

_player = FlutterSoundPlayer()

..openPlayer().then((_) {

_player.setSubscriptionDuration(const Duration(milliseconds: 100));

_player.onProgress?.listen((e) {

if (_player.isPlaying && e.duration != null && e.position != null) {

setState(() {

_duration = e.duration!.inMilliseconds.toDouble();

_position = e.position!.inMilliseconds.toDouble();

});

}

});

});

}

@override

void dispose() {

_player.closePlayer();

super.dispose();

}

Future<void> _togglePlay() async {

if (_player.isPlaying) {

await _player.pausePlayer();

setState(() => _isPlaying = false);

} else {

if (_position >= _duration) _position = 0;

await _player.startPlayer(

fromURI: widget.audioUrl,

whenFinished: () {

setState(() {

_isPlaying = false;

_position = 0;

});

},

);

setState(() => _isPlaying = true);

}

}

Future<void> _seek(double v) async {

if (_player.isPlaying) {

await _player.seekToPlayer(Duration(milliseconds: v.round()));

} else {

setState(() => _position = v);

}

}

@override

Widget build(BuildContext context) {

return Column(

crossAxisAlignment:

widget.isSender ? CrossAxisAlignment.end : CrossAxisAlignment.start,

children: [

if (widget.prompt.isNotEmpty)

Padding(

padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),

child: Text(

widget.prompt,

style: AppFonts.secondaryFont.copyWith(

fontSize: 16,

fontWeight: FontWeight.w400,

color: AppColors.textOnWhite,

),

),

),

Container(

constraints:

BoxConstraints(maxWidth: MediaQuery.of(context).size.width * .85),

padding: const EdgeInsets.fromLTRB(6, 0, 14, 0),

decoration: BoxDecoration(

color: Colors.white,

borderRadius: BorderRadius.circular(30),

),

child: Row(

children: [

IconButton(

padding: EdgeInsets.zero,

constraints: const BoxConstraints(),

iconSize: 32,

onPressed: () async {

widget.onPlayPressed?.call();

await _togglePlay();

},

icon: SvgPicture.asset(

_isPlaying ? widget.stopAsset : widget.playAsset,

width: 32,

height: 32,

),

),

const SizedBox(width: 1),

Expanded(

child: SliderTheme(

data: SliderTheme.of(context).copyWith(

trackHeight: 8,

thumbShape:

const RoundSliderThumbShape(enabledThumbRadius: 0),

trackShape: const _TightSliderTrackShape(),

overlayShape:

const RoundSliderOverlayShape(overlayRadius: 0),

),

child: Slider(

activeColor: Colors.redAccent,

inactiveColor: Colors.redAccent.withOpacity(0.3),

value: _position.clamp(0, _duration),

max: _duration,

onChanged: _seek,

),

),

),

],

),

),

// add a little breathing room under the slider

const SizedBox(height: 12),

],

);

}

}

"

Task2: rewrite ViewMyProfile to keep giving the devider lines but make all padding for each element 0 from all sides