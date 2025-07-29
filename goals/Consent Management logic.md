# Consent Management logic
- in settings, we have a "permissions" page, where the user can toggle on and off each permission and bellow each one, the purpose is shown using language based on the privacy notice and AGBs  > informed consent
- we have a consent service:
  "
  import 'dart:convert';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import 'package:http/http.dart' as http;

import 'package:firebase_messaging/firebase_messaging.dart';

import 'package:geolocator/geolocator.dart';

import '../config.dart';

  

class Consent {

  Consent({

    required this.tos,

    required this.newsletter,

    required this.profileSensitives,

    required this.location,

    required this.notifications,

    required this.microphone,

  });

  

  bool tos;

  bool newsletter;

  bool profileSensitives;

  bool location;

  bool notifications;

  bool microphone;

  

  factory Consent.empty() => Consent(

        tos: false,

        newsletter: false,

        profileSensitives: false,

        location: false,

        notifications: false,

        microphone: false,

      );

  

  factory Consent.fromJson(Map<String, dynamic> j) => Consent(

        tos: j['tos'] ?? false,

        newsletter: j['newsletter'] ?? false,

        profileSensitives: j['profile_sensitives'] ?? false,

        location: j['location'] ?? false,

        notifications: j['notifications'] ?? false,

        microphone: j['microphone'] ?? false,

      );

  

  Map<String, dynamic> toJson() => {

        'tos': tos,

        'newsletter': newsletter,

        'profile_sensitives': profileSensitives,

        'location': location,

        'notifications': notifications,

        'microphone': microphone,

      };

}

  

class ConsentService {

  ConsentService._internal();

  static final instance = ConsentService._internal();

  

  final _local = const FlutterSecureStorage();

  Consent _current = Consent.empty();

  

  Consent get current => _current;

  

  /* ── initialise at app start ─────────────────────────────────── */

  Future<void> init() async {

    final jwt = await _local.read(key: 'jwt_token');

    final localRaw = await _local.read(key: 'consents_json');

    if (localRaw != null) {

      _current = Consent.fromJson(json.decode(localRaw));

    }

    if (jwt != null) {

      try {

        final res = await http.get(

          Uri.parse('${Config.backendBaseUrl}/dsgvo/get_consents.php'),

          headers: {'Authorization': 'Bearer $jwt'},

        );

        if (res.statusCode == 200) {

          _current = Consent.fromJson(json.decode(res.body));

        }

      } catch (_) {/* ignore – stay on local */}

    }

  }

  

  /* ── update consent and apply side-effects ───────────────────── */

  Future<void> update(Consent next) async {

    final old = _current;

    _current = next;

    await _local.write(key: 'consents_json', value: json.encode(next.toJson()));

  

    // Push token management

    if (old.notifications && !next.notifications) {

      await FirebaseMessaging.instance.deleteToken();

    } else if (!old.notifications && next.notifications) {

      await FirebaseMessaging.instance.requestPermission();

      await FirebaseMessaging.instance

          .getToken(); // triggers register-endpoint in your code

    }

  

    // Location stream management

    if (!next.location) {

      Geolocator.getPositionStream().listen(null).cancel();

    }

  

    // Microphone – nothing to revoke programmatically; just gate UI.

  

    // Persist remotely

    final jwt = await _local.read(key: 'jwt_token');

    if (jwt != null) {

      await http.post(

        Uri.parse('${Config.backendBaseUrl}/dsgvo/consent.php'),

        headers: {

          'Authorization': 'Bearer $jwt',

          'Content-Type': 'application/json',

        },

        body: json.encode(next.toJson()),

      );

    }

  }

}
  "
  > [please check if this is extensive and is making the consents truly effective on the phone]