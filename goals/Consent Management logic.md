"**1. Is Your Consent "Informed"?**
When a user is about to flip a toggle from false to true, they must be informed about what they are agreeing to.

- **Action:** For each consent toggle in your app's settings, provide a short, clear explanation. You can use the wording directly from the Privacy Policy we just created."
Task1: Please edit the permissions page, to make it so:
 "
```
import 'dart:convert';

import 'dart:async';

import 'package:flutter/gestures.dart';

import 'package:flutter/material.dart';

import 'package:google_fonts/google_fonts.dart';

import 'package:permission_handler/permission_handler.dart';

import 'package:firebase_messaging/firebase_messaging.dart';

import 'package:geolocator/geolocator.dart';

  

import 'package:swipe_chat_play/services/consent_service.dart';

import 'package:swipe_chat_play/usables/inappwebview.dart';

import 'package:swipe_chat_play/usables/primary_button.dart';

  

class PermissionsPage extends StatefulWidget {

  const PermissionsPage({super.key});

  

  @override

  State<PermissionsPage> createState() => _PermissionsPageState();

}

  

class _PermissionsPageState extends State<PermissionsPage> {

  static const _stripe = Color(0xFFFFEDE6);

  static const _highlight = Color(0xFFFA938E);

  

  bool agreeTos = false;

  bool newsOn = false;

  bool sensOn = false;

  bool pushOn = false;

  bool locOn = false;

  bool micOn = false;

  bool _busy = false;

  

  StreamSubscription<Position>? _locStream;

  

  @override

  void initState() {

    super.initState();

    _sync();

  }

  

  Future<void> _sync() async {

    setState(() => _busy = true);

    await ConsentService.instance.init();

    final c = ConsentService.instance.current;

    setState(() {

      agreeTos = c.tos;

      newsOn = c.newsletter;

      sensOn = c.profileSensitives;

      pushOn = c.notifications;

      locOn = c.location;

      micOn = c.microphone;

      _busy = false;

    });

  }

  

  /* ───── Push toggle ─────────────────────────────────────────── */

  Future<void> _togglePush(bool target) async {

    // optimistic UI – switch moves instantly

    setState(() => pushOn = target);

  

    if (!target) {

      await FirebaseMessaging.instance.deleteToken();

    } else {

      // 1) OS‑level permission (Android 13+ or iOS)

      final permStatus = await Permission.notification.request();

      if (!permStatus.isGranted) {

        setState(() => pushOn = false); // undo

        return;

      }

  

      // 2) FCM permission (iOS) – returns settings object

      final settings = await FirebaseMessaging.instance.requestPermission();

      final authorised =

          settings.authorizationStatus == AuthorizationStatus.authorized ||

              settings.authorizationStatus == AuthorizationStatus.provisional;

  

      if (!authorised) {

        setState(() => pushOn = false); // undo

        return;

      }

  

      // 3) Ensure the device has a token (creates one if needed)

      await FirebaseMessaging.instance.getToken();

    }

  }

  

  /* ───── Location toggle ─────────────────────────────────────── */

  Future<void> _toggleLoc(bool target) async {

    if (target) {

      final perm = await Permission.locationWhenInUse.request();

      if (!perm.isGranted) return;

      _locStream = Geolocator.getPositionStream().listen((_) {});

    } else {

      await _locStream?.cancel();

      _locStream = null;

    }

    setState(() => locOn = target);

  }

  

  /* ───── Microphone toggle ───────────────────────────────────── */

  Future<void> _toggleMic(bool target) async {

    if (target) {

      final perm = await Permission.microphone.request();

      if (!perm.isGranted) return;

    }

    setState(() => micOn = target);

  }

  

  /* ───── Persist all consents ────────────────────────────────── */

  Future<void> _save() async {

    if (!agreeTos) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text('Please accept the Terms first')),

      );

      return;

    }

    setState(() => _busy = true);

    final next = Consent(

      tos: agreeTos,

      newsletter: newsOn,

      profileSensitives: sensOn,

      location: locOn,

      notifications: pushOn,

      microphone: micOn,

    );

    await ConsentService.instance.update(next);

    if (mounted) Navigator.pop(context, true);

  }

  

  /* ───── UI ──────────────────────────────────────────────────── */

  @override

  Widget build(BuildContext context) {

    return Scaffold(

      backgroundColor: Colors.grey[100],

      body: SafeArea(

        child: Column(

          children: [

            Container(height: 8, color: _stripe),

            const SizedBox(height: 24),

            Text('Permissions',

                style: GoogleFonts.zillaSlab(

                  fontSize: 24,

                  fontWeight: FontWeight.w600,

                  color: Colors.grey[800],

                )),

            const SizedBox(height: 16),

            Expanded(

              child: _busy

                  ? const Center(child: CircularProgressIndicator())

                  : ListView(

                      padding: const EdgeInsets.symmetric(horizontal: 16),

                      children: [

                        SwitchListTile(

                          value: newsOn,

                          onChanged: (v) => setState(() => newsOn = v),

                          title: const Text(

                              'Receive newsletter / product updates'),

                        ),

                        SwitchListTile(

                          value: sensOn,

                          onChanged: (v) => setState(() => sensOn = v),

                          title:

                              const Text('Show religion / politics in profile'),

                        ),

                        SwitchListTile(

                          value: pushOn,

                          onChanged: _togglePush,

                          title: const Text('Allow push notifications'),

                        ),

                        SwitchListTile(

                          value: locOn,

                          onChanged: _toggleLoc,

                          title: const Text('Enable location‑based matching'),

                        ),

                        SwitchListTile(

                          value: micOn,

                          onChanged: _toggleMic,

                          title: const Text('Allow microphone access'),

                        ),

                      ],

                    ),

            ),

            Padding(

              padding: const EdgeInsets.fromLTRB(24, 0, 24, 24),

              child: PrimaryButton(

                text: 'Save',

                isLoading: _busy,

                onPressed: _busy ? null : _save,

              ),

            ),

          ],

        ),

      ),

    );

  }

  

  @override

  void dispose() {

    _locStream?.cancel();

    super.dispose();

  }

}
```

"**2. Is Your Consent "Granular" and "Unbundled? ** " > yes it is

"**3. Is Consent Easy to Withdraw?**"
> i have a page where user can withdraw any consents so i guess yes


"**4. Are You Keeping Records of Consent? (Gold Standard)**"
Task2: please make it also save the timestamps:
"
<?php

declare(strict_types=1);

/* ─── 0 · Logging setup ─────────────────────────────────────────── */

$logDir = dirname(__DIR__, 2) . '/logs';

if (!is_dir($logDir) && !mkdir($logDir, 0775, true) && !is_dir($logDir)) {

    error_log('[consent] could not create log directory');

}

ini_set('log_errors', 1);

ini_set('error_log', $logDir . '/consent.log');

error_log('[consent] request start');

  

/* ─── 1 · Bootstrap + config ────────────────────────────────────── */

require_once dirname(__DIR__, 2) . '/bootstrap.php';

require_once dirname(__DIR__, 2) . '/config.php';

  

use MongoDB\BSON\ObjectId;

use MongoDB\Driver\{Manager, BulkWrite};

  

dbg('consent: begin');

  

/* ─── 2 · Only POST ──────────────────────────────────────────────── */

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {

    http_response_code(405);

    echo json_encode(['error' => 'Only POST']);

    exit;

}

  

$claims = auth();               // will 401 & exit on failure

$userId = $claims->sub;

  

$raw    = file_get_contents('php://input');

$data   = json_decode($raw, true);

  

if (json_last_error() !== JSON_ERROR_NONE) {

    http_response_code(400);

    echo json_encode(['error' => 'Invalid JSON']);

    exit;

}

  

$bulk = new BulkWrite();

$bulk->update(

    ['_id' => new ObjectId($userId)],

    ['$set' => ['userdata.consents' => $data]],

    ['multi' => false]

);

  

$manager = new Manager($mongoURI);

$manager->executeBulkWrite("$mongoDb.users", $bulk);

  

echo json_encode(['status' => 'ok']);
"