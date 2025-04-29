import 'package:flutter/material.dart';  
  
class ChatBubbleLayout extends StatelessWidget {  
  final String message;  
  final String timestamp;  
  final bool isLeft;  
  
  const ChatBubbleLayout({  
    Key? key,  
    required this.message,  
    required this.timestamp,  
    required this.isLeft,  
  }) : super(key: key);  
  
  @override  
  Widget build(BuildContext context) {  
    return Container(  
      margin: EdgeInsets.only(  
        top: 8.0,  
        bottom: 8.0,  
        left: isLeft ? 8.0 : 50.0,  
        right: isLeft ? 50.0 : 8.0,  
      ),  
      padding: const EdgeInsets.all(12.0),  
      decoration: BoxDecoration(  
        color: Colors.grey[800],  
        borderRadius: BorderRadius.only(  
          topLeft: Radius.circular(15.0),  
          topRight: Radius.circular(15.0),  
          bottomLeft: isLeft ? Radius.circular(0.0) : Radius.circular(15.0),  
          bottomRight: isLeft ? Radius.circular(15.0) : Radius.circular(0.0),  
        ),  
        border: Border.all(color: Colors.grey),  
      ),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.end,  
        children: [  
          Text(  
            message,  
            style: TextStyle(  
              color: Colors.white,  
              fontSize: 16.0,  
            ),  
          ),  
          const SizedBox(height: 4.0),  
          Text(  
            timestamp,  
            style: TextStyle(  
              color: Colors.grey[400],  
              fontSize: 12.0,  
            ),  
          ),  
        ],  
      ),  
    );  
  }  
}