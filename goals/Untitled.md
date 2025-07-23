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

  Future<void> _showRestrictionConfirmation() async {
    if (!mounted) return;

    final confirm = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Restrict Processing?'),
        content: SingleChildScrollView(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('By restricting processing:'),
              SizedBox(height: 8),
              Text('• Your profile will be hidden from other users'),
              Text('• You won\'t appear in any searches or matches'),
              Text('• Existing matches can still message you'),
              Text('• You can unrestrict processing at any time'),
              SizedBox(height: 16),
              Text('Do you want to continue?'),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Abort'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Continue'),
          ),
        ],
      ),
    );

    if (confirm == true) {
      await _restrictMe();
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
      _snack('Processing restricted');
      if (mounted) setState(() {});
    } else {
      _snack('Restriction failed: HTTP ${res.statusCode}');
    }
  }

  Future<void> _unrestrictMe() async {
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
      Uri.parse('${Config.backendBaseUrl}/dsgvo/unrestrict_me.php'),
      headers: headers,
      body: body,
    );

    if (res.statusCode == 200) {
      await _storage.delete(key: 'restriction_date');
      _snack('Processing unrestricted');
      if (mounted) setState(() {});
    } else {
      _snack('Unrestriction failed: HTTP ${res.statusCode}');
    }
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
                return Column(
                  children: [
                    Text(
                      'Processing restricted $days day${days == 1 ? '' : 's'} ago',
                      style: const TextStyle(color: Colors.orangeAccent),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'Your profile is hidden from other users',
                      style: TextStyle(color: Colors.grey[600]),
                    ),
                  ],
                );
              },
            ),
            const SizedBox(height: 16),
            FutureBuilder<String?>(
              future: _restrictionDate(),
              builder: (_, snap) {
                final isRestricted = snap.data != null;
                return ElevatedButton(
                  onPressed: isRestricted
                      ? _unrestrictMe
                      : _showRestrictionConfirmation,
                  style: ElevatedButton.styleFrom(
                    backgroundColor:
                        isRestricted ? Colors.green : Colors.orangeAccent,
                    minimumSize: const Size.fromHeight(48),
                  ),
                  child: Text(
                    isRestricted
                        ? 'Unrestrict Processing'
                        : 'Restrict Processing',
                    style: const TextStyle(color: Colors.white),
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            FutureBuilder<String?>(
              future: _deletionDate(),
              builder: (_, snap) {
                final val = snap.data;
                if (val == null) return const SizedBox.shrink();
                final deletionDate = DateTime.parse(val);
                final daysLeft = deletionDate.difference(DateTime.now()).inDays;
                return Column(
                  children: [
                    Text(
                      'Your profile will be deleted in $daysLeft day${daysLeft == 1 ? '' : 's'}',
                      style: const TextStyle(color: Colors.redAccent),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'You can cancel by logging in again',
                      style: TextStyle(color: Colors.grey[600]),
                    ),
                  ],
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
}"
- make it check if profile is currently due to be deleted and also if its restricted currently
- write endpoint for it aswell