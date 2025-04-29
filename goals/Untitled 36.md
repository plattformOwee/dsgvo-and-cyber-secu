import 'package:flutter/material.dart';

import 'package:flutter_svg/flutter_svg.dart';

import 'package:swipe_chat_play/colors.dart';

  

class CustomNavigationBar extends StatefulWidget {

  final int selectedIndex;

  final Function(int) onTap;

  

  const CustomNavigationBar({

    super.key,

    required this.selectedIndex,

    required this.onTap,

  });

  

  @override

  _CustomNavigationBarState createState() => _CustomNavigationBarState();

}

  

class _CustomNavigationBarState extends State<CustomNavigationBar> {

  bool _isNavigating = false;

  

  @override

  Widget build(BuildContext context) {

    // List of icon paths for each tab

    final List<String> iconPathsInactive = [

      'assets/icons/profile_0.svg', // Inactive profile

      'assets/icons/swipe_0.svg', // Inactive swipe

      'assets/icons/chat_0.svg', // Inactive chat

    ];

    final List<String> iconPathsActive = [

      'assets/icons/profile_1.svg', // Active profile

      'assets/icons/swipe_1.svg', // Active swipe

      'assets/icons/chat_1.svg', // Active chat

    ];

  

    // Labels for tabs

    final List<String> labels = ['Profile', 'Swipe', 'Chat'];

  

    return Container(

      // White background with a shadow above the bar

      decoration: BoxDecoration(

        color: Colors.white,

        boxShadow: [

          BoxShadow(

            color: Colors.black12, // Slightly darker for visible shadow

            offset: const Offset(0, -2),

            blurRadius: 4,

          ),

        ],

      ),

      child: BottomNavigationBar(

        backgroundColor: Colors.white, // Also ensure the bar itself is white

        currentIndex: widget.selectedIndex,

        onTap: (index) async {

          if (_isNavigating) return;

          _isNavigating = true;

  

          if (mounted) {

            await widget.onTap(index);

          }

  

          _isNavigating = false;

        },

        type: BottomNavigationBarType.fixed,

        selectedItemColor: AppColors.primaryColor,

        unselectedItemColor: AppColors.textOnWhite,

        items: List.generate(3, (index) {

          return BottomNavigationBarItem(

            icon: SvgPicture.asset(

              widget.selectedIndex == index

                  ? iconPathsActive[index]

                  : iconPathsInactive[index],

              width: 24,

              height: 24,

            ),

            label: labels[index],

          );

        }),

      ),

    );

  }

}