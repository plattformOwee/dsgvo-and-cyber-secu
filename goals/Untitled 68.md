// topic_suggestions.dart

  

import 'package:flutter/material.dart';

  

class TopicSuggestionsPage extends StatelessWidget {

  const TopicSuggestionsPage({Key? key}) : super(key: key);

  

  @override

  Widget build(BuildContext context) {

    final borderColor = Colors.grey.shade300;

    final titleStyle = Theme.of(context)

        .textTheme

        .headlineLarge

        ?.copyWith(fontWeight: FontWeight.w600, color: Colors.black87);

    final subtitleStyle =

        Theme.of(context).textTheme.bodyMedium?.copyWith(color: Colors.black54);

  

    return Scaffold(

      body: SafeArea(

        minimum: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),

        child: Column(

          crossAxisAlignment: CrossAxisAlignment.start,

          children: [

            Text('Suggestions', style: titleStyle),

            const SizedBox(height: 4),

            Text(

              'suggest some Topics to get\nthe conversation started',

              style: subtitleStyle,

            ),

            const SizedBox(height: 24),

  

            // Caption selector

            Container(

              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),

              decoration: BoxDecoration(

                borderRadius: BorderRadius.circular(12),

                border: Border.all(color: borderColor, width: 1.2),

              ),

              child: Row(

                children: const [

                  Expanded(

                    child: Text(

                      'gewählte caption',

                      style: TextStyle(fontSize: 16),

                    ),

                  ),

                  Icon(Icons.edit, size: 20, color: Colors.grey),

                ],

              ),

            ),

            const SizedBox(height: 24),

  

            // Option boxes

            _OptionBox(label: '| type option 1'),

            const SizedBox(height: 16),

            _OptionBox(label: '| type option 2'),

            const SizedBox(height: 16),

            _OptionBox(label: '| type option 3'),

            const Spacer(),

  

            // Next button

            Row(

              children: [

                const Spacer(),

                GestureDetector(

                  onTap: () {

                    // TODO: handle next

                  },

                  child: Container(

                    width: 48,

                    height: 48,

                    decoration: BoxDecoration(

                      color: Colors.grey.shade400,

                      shape: BoxShape.circle,

                    ),

                    alignment: Alignment.center,

                    child: const Icon(

                      Icons.arrow_forward_ios_rounded,

                      size: 22,

                      color: Colors.white,

                    ),

                  ),

                ),

              ],

            ),

          ],

        ),

      ),

    );

  }

}

  

class _OptionBox extends StatelessWidget {

  final String label;

  

  const _OptionBox({Key? key, required this.label}) : super(key: key);

  

  @override

  Widget build(BuildContext context) {

    final borderColor = Colors.grey.shade300;

    return Container(

      width: double.infinity,

      height: 56,

      padding: const EdgeInsets.symmetric(horizontal: 16),

      alignment: Alignment.centerLeft,

      decoration: BoxDecoration(

        borderRadius: BorderRadius.circular(12),

        border: Border.all(color: borderColor, width: 1.2),

      ),

      child: Text(

        label,

        style: const TextStyle(fontSize: 16, color: Colors.black54),

      ),

    );

  }

}
"
task1: modify flutter layout above to make work with server
1. make such that if user clicks on an option, onlcik:
	1. call "[http://node02.krasserserver.com:8002/swipe_chatt_play_api/update_topic_suggestion.php](http://node02.krasserserver.com:8002/swipe_chatt_play_api/update_question.php)" with jwt as auth bearer and as payload:  
	   content: {  
    prompt: 'the prompt',  
    options: [ 'musik', 'kunst', 'geschichte' ]  
    }
    } 2. if response is "success" > navigate to DatingRadiusPage

task2: write the backend script
1. write update_topic_suggestion.php to save the payload as such in the profiledata:  
    "  
    [  
    {  
    _id: ObjectId('681e16a5e8126a01fc04ff56'),  
    profile: {  
    topSection: {  
    profileImages: [  
    '[http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/681e16dd46f02_image_cropper_1746802394567.jpg](http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/681e16dd46f02_image_cropper_1746802394567.jpg)'  
    ],  
    name: 'luna',  
    age: 23,  
    infoBubbles: {  
    career: 'bsbw',  
    gender: 'Female',  
    languages: [ 'German', 'English' ],  
    politics: 'Socialist',  
    religion: 'Other'  
    }  
    },  
    elements: [  
    {  
    _id: ObjectId('681e16e02be764913f0e0d4f'),  
    type: 'icebreaker',  
    content: {  
    question: 'Das verrückteste Abenteuer, das ich je erlebt habe...',  
    answer: 'j'  
    }  
    },  
    {  
    _id: ObjectId('681ccfa0ed13220a4c0b6243'),  
    type: 'icebreaker',  
    content: {  
    question: 'Ein Ort, an den ich immer wieder zurückkehre...',  
    answer: 'susi'  
    }  
    },  
    {  
    _id: ObjectId('681ccfa0ed13220a4c0b6244'),  
    type: 'bubbles',  
    content: {  
    title: 'Open to',  
    bubbles: [ 'biking', 'puzzles', 'games' ]  
    }  
    },  
    {  
    _id: ObjectId('681ccfa0ed13220a4c0b6245'),  
    type: 'question_and_inputfield',  
    content: { question: 'was hat dich heute gefreut?' }  
    },  
    {  
    _id: ObjectId('681ccfa0ed13220a4c0b6246'),  
    type: 'voicememo',  
    content: {  
    prompt: 'the prompt',  
    link_voicemessage: '[http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f17f96138772.67969131_1743880080766.aac](http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f17f96138772.67969131_1743880080766.aac)'  
    }  
    },  
    {  
    _id: ObjectId('681ccfa0ed13220a4c0b6247'),  
    type: 'topic_suggestions',  
    content: {  
    prompt: 'the prompt',  
    options: [ 'musik', 'kunst', 'geschichte' ]  
    }  
    },  
    {  
    _id: ObjectId('681ccfa0ed13220a4c0b6248'),  
    type: 'image_and_prompt',  
    content: {  
    prompt: 'the prompt',  
    image_link: '[http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg](http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg)'  
    }  
    }  
    ]  
    },  
    search_filter: {  
    location_radius: {  
    location: { type: 'Point', coordinates: [ 11.5801653, 48.1235981 ] },  
    radius: 90  
    },  
    open_to: [  
    'kayaking',  
    'yoga',  
    'birdwatching',  
    'rock climbing',  
    'movies',  
    'games',  
    'puzzles'  
    ],  
    searching_for: {  
    genders: [  
    'Male',  
    'Female',  
    'Non Binary',  
    'Transgender',  
    'Genderqueer',  
    'Other'  
    ],  
    ageRange: [ 18, 120 ],  
    religion: [  
    'Christianity',  
    'Islam',  
    'Hinduism',  
    'Buddhism',  
    'Judaism',  
    'Other'  
    ],  
    politics: [  
    'Liberal',  
    'Conservative',  
    'Centrist',  
    'Libertarian',  
    'Socialist'  
    ]  
    }  
    },  
    userdata: {  
    email: '[luna.vogl.35@gmail.com](mailto:luna.vogl.35@gmail.com)',  
    password_hash: '$2y$10$E/rpAq21ragiL7RyLxduV.1X8.7ly/otk1grNMzWO1rpRZQ2TrX7a',  
    verification_code: null,  
    verification_code_expiry: null,  
    is_verified: true,  
    fingerprint_hash: '$2y$10$38adZ8gTcN.CTzRNCbw7S.Ys0nI9W1H1ZKCmnvuMJsPqce4VP/uJO'  
    },  
    chats: {  
    '6824c063dfcc0109a90db00c': {  
    profileURL: '',  
    receiver_id: '681e20c02be764913f0e0ff6',  
    receiver: null,  
    last_message: '(voicemessage)',  
    unread: 0  
    }  
    }  
    },  
    ]"