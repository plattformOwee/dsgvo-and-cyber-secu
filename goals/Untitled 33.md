import 'package:flutter/material.dart';

import 'package:swipe_chat_play/profile_creation/profile.dart';

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

    ViewMyProfile(), // profile

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