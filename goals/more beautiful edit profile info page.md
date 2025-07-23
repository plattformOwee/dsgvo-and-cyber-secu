```
// lib/tabs/profile/edit_profile_layout.dart

  

import 'dart:convert';

import 'package:flutter/material.dart';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:latlong2/latlong.dart';

  

import 'package:swipe_chat_play/config.dart';

  

class EditProfileLayout extends StatefulWidget {

  const EditProfileLayout({Key? key}) : super(key: key);

  

  @override

  State<EditProfileLayout> createState() => _EditProfileLayoutState();

}

  

class _EditProfileLayoutState extends State<EditProfileLayout> {

  final _storage = const FlutterSecureStorage();

  Map<String, dynamic>? _profile;

  bool _loading = true;

  bool _saving = false;

  

  @override

  void initState() {

    super.initState();

    _loadProfile();

  }

  

  Future<void> _loadProfile() async {

    final jwt = await _storage.read(key: 'jwt_token');

    if (jwt == null) {

      _showSnack('Please log in first');

      return;

    }

    final res = await http.get(

      Uri.parse('${Config.backendBaseUrl}/new_signup/get_profile.php'),

      headers: {'Authorization': 'Bearer $jwt'},

    );

    if (res.statusCode != 200) {

      _showSnack('Failed to load profile: ${res.statusCode}');

      return;

    }

    setState(() {

      _profile = json.decode(res.body) as Map<String, dynamic>;

      _loading = false;

    });

  }

  

  void _showSnack(String msg) {

    if (!mounted) return;

    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  }

  

  /// Inline editor for any string field under topSection → (about_you or infoBubbles)

  Future<void> _editTopLevel(String title, List<String> path) async {

    // path e.g. ['topSection','name'] or ['topSection','infoBubbles','gender']

    final initial = _dig(_profile!, path) as String? ?? '';

    final ctrl = TextEditingController(text: initial);

    final result = await showModalBottomSheet<String?>(

      context: context,

      isScrollControlled: true,

      builder: (ctx) => Padding(

        padding: EdgeInsets.only(

          bottom: MediaQuery.of(ctx).viewInsets.bottom + 16,

          left: 16,

          right: 16,

          top: 16,

        ),

        child: Column(mainAxisSize: MainAxisSize.min, children: [

          Text(title, style: const TextStyle(fontSize: 18)),

          TextField(controller: ctrl, autofocus: true),

          const SizedBox(height: 12),

          ElevatedButton(

            onPressed: () => Navigator.pop(ctx, ctrl.text.trim()),

            child: const Text('OK'),

          ),

          const SizedBox(height: 8),

        ]),

      ),

    );

    if (result != null) {

      setState(() => _poke(_profile!, path, result));

    }

  }

  

  /// Helper to dig into nested Map

  dynamic _dig(Map m, List<String> path) {

    dynamic cur = m;

    for (final p in path) {

      if (cur is Map && cur.containsKey(p)) {

        cur = cur[p];

      } else {

        return null;

      }

    }

    return cur;

  }

  

  /// Helper to poke into nested Map and set a value

  void _poke(Map m, List<String> path, dynamic value) {

    if (path.length == 1) {

      m[path.first] = value;

      return;

    }

    final head = path.first;

    final rest = path.sublist(1);

    if (m[head] is Map<String, dynamic>) {

      _poke(m[head], rest, value);

    }

  }

  

  /// Send the entire profile JSON back to server

  Future<void> _saveAll() async {

    final jwt = await _storage.read(key: 'jwt_token');

    if (jwt == null) {

      _showSnack('Please log in first');

      return;

    }

    setState(() => _saving = true);

    try {

      final res = await http.post(

        Uri.parse(

            '${Config.backendBaseUrl}/create_profile/update_userdata.php'),

        headers: {

          'Authorization': 'Bearer $jwt',

          'Content-Type': 'application/json',

        },

        body: json.encode(

            _profile!['topSection']), // adjust according to your API contract

      );

      final body = json.decode(res.body) as Map<String, dynamic>;

      if (res.statusCode == 200 && body['success'] == true) {

        Navigator.pop(context, true); // signal "saved"

      } else {

        _showSnack('Save failed: ${body['error'] ?? body['message']}');

      }

    } catch (e) {

      _showSnack('Save error: $e');

    } finally {

      if (mounted) setState(() => _saving = false);

    }

  }

  

  Future<bool> _confirmDiscard() async {

    final ans = await showDialog<bool>(

      context: context,

      builder: (_) => AlertDialog(

        title: const Text('Discard changes?'),

        content: const Text('Are you sure you want to discard all changes?'),

        actions: [

          TextButton(

              onPressed: () => Navigator.pop(_, false),

              child: const Text('Cancel')),

          TextButton(

              onPressed: () => Navigator.pop(_, true),

              child: const Text('Discard')),

        ],

      ),

    );

    return ans == true;

  }

  

  Widget _row(String label, String value, VoidCallback onTap) {

    return ListTile(

      title: Text(label),

      subtitle: Text(value.isEmpty ? '—' : value),

      trailing: const Icon(Icons.edit),

      onTap: onTap,

    );

  }

  

  @override

  Widget build(BuildContext context) {

    if (_loading) {

      return const Scaffold(body: Center(child: CircularProgressIndicator()));

    }

    final top = _profile!['topSection'] as Map<String, dynamic>;

    final info = (top['infoBubbles'] ?? {}) as Map<String, dynamic>;

  

    return Scaffold(

      body: Column(children: [

        Expanded(

          child: ListView(children: [

            const SizedBox(height: 8),

            _row('Name', top['name']?.toString() ?? '', () {

              _editTopLevel('Name', ['topSection', 'name']);

            }),

            _row('Age', top['age']?.toString() ?? '', () {

              _editTopLevel('Age', ['topSection', 'age']);

            }),

            _row('Gender', info['gender']?.toString() ?? '', () {

              _editTopLevel('Gender', ['topSection', 'infoBubbles', 'gender']);

            }),

            _row('Languages', (info['languages'] as List? ?? []).join(', '),

                () {

              _editTopLevel(

                  'Languages', ['topSection', 'infoBubbles', 'languages']);

            }),

            _row('Religion', info['religion']?.toString() ?? '', () {

              _editTopLevel(

                  'Religion', ['topSection', 'infoBubbles', 'religion']);

            }),

            _row('Career', info['career']?.toString() ?? '', () {

              _editTopLevel('Career', ['topSection', 'infoBubbles', 'career']);

            }),

            _row('Politics', info['politics']?.toString() ?? '', () {

              _editTopLevel(

                  'Politics', ['topSection', 'infoBubbles', 'politics']);

            }),

            // …add more personal‐data rows here as needed…

          ]),

        ),

  

        // Cancel / Save row

        Padding(

          padding: const EdgeInsets.all(16),

          child: Row(children: [

            Expanded(

              child: OutlinedButton.icon(

                icon: const Icon(Icons.close),

                label: const Text('Cancel Changes'),

                style: OutlinedButton.styleFrom(

                  shape: const StadiumBorder(),

                ),

                onPressed: () async {

                  if (await _confirmDiscard()) {

                    Navigator.pop(context, false);

                  }

                },

              ),

            ),

            const SizedBox(width: 12),

            Expanded(

              child: ElevatedButton.icon(

                icon: _saving

                    ? const SizedBox(

                        width: 16,

                        height: 16,

                        child: CircularProgressIndicator(

                            strokeWidth: 2, color: Colors.white),

                      )

                    : const Icon(Icons.check),

                label: const Text('Save Changes'),

                style: ElevatedButton.styleFrom(

                  shape: const StadiumBorder(),

                ),

                onPressed: _saving ? null : _saveAll,

              ),

            ),

          ]),

        ),

      ]),

    );

  }

}
```

Instead of just displaying the values in rows, i want to show the profile but with editable inputfields.

Task1: rewrite this card layout file and make each text a clickable box which opens an inputfield

Task2: in edit profile page, show the profile data in this new type of layout but with white squares for images. (and no need to build the elements list at bottom since we only care about profile info)

