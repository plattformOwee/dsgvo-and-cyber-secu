import 'package:flutter/material.dart';

class ChoseCaptionPic extends StatefulWidget {  
const ChoseCaptionPic({Key? key}) : super(key: key);

@override  
_SearchCaptionPageState createState() => _SearchCaptionPageState();  
}

class _SearchCaptionPageState extends State {  
final TextEditingController _controller = TextEditingController();

final List _allSuggestions = [  
'hier steht das was er Nutzer eingegeben hat und kann geklickt werden zum hinzufügen',  
'hier steht die Option zwei für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
'hier steht die Option x für caption von Bild',  
];

late List _filteredSuggestions = List.from(_allSuggestions);

@override  
void initState() {  
super.initState();  
_controller.addListener(_onSearchChanged);  
}

void _onSearchChanged() {  
final query = _controller.text.toLowerCase();  
setState(() {  
_filteredSuggestions = _allSuggestions  
.where((item) => item.toLowerCase().contains(query))  
.toList(growable: false);  
});  
}

@override  
void dispose() {  
_controller.dispose();  
super.dispose();  
}

@override  
Widget build(BuildContext context) {  
return Scaffold(  
backgroundColor: Colors.white,  
body: SafeArea(  
child: Padding(  
padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),  
child: Column(  
children: [  
Container(  
height: 44,  
decoration: BoxDecoration(  
borderRadius: BorderRadius.circular(30),  
border: Border.all(color: Colors.grey.shade400),  
),  
child: Row(  
children: [  
const SizedBox(width: 12),  
const Icon(Icons.search, size: 20),  
const SizedBox(width: 8),  
Expanded(  
child: TextField(  
controller: _controller,  
decoration: const InputDecoration(  
hintText: 'search',  
border: InputBorder.none,  
isCollapsed: true,  
),  
style: const TextStyle(fontSize: 14),  
),  
),  
IconButton(  
icon: const Icon(Icons.send, size: 20),  
onPressed: () {},  
),  
],  
),  
),  
const SizedBox(height: 12),  
Expanded(  
child: ListView.separated(  
itemCount: _filteredSuggestions.length,  
separatorBuilder: (_, __) => Divider(  
height: 1,  
thickness: 0.7,  
color: Colors.grey.shade400,  
),  
itemBuilder: (context, index) {  
final caption = _filteredSuggestions[index];  
return InkWell(  
onTap: () {  
// Handle tap on caption option  
},  
child: Padding(  
padding: const EdgeInsets.symmetric(  
vertical: 14,  
horizontal: 4,  
),  
child: Text(  
caption,  
style: const TextStyle(fontSize: 14),  
),  
),  
);  
},  
),  
),  
],  
),  
),  
),  
);  
}  
}"

task1: change the above script:
1. Add-image-field should onclick open the image picker and then with chosen image, lead into image cutting and then from image cutting back to this layout with the image in the middle in this card
    
2. "gewählte caption" button should lead to "ChoseCaptionPic"
    
3. next button should:
    

- if no image selected > show message "please select image"
    
- if image selected >
    

1. call "[http://node02.krasserserver.com:8002/swipe_chatt_play_api/update_captionpic.php](http://node02.krasserserver.com:8002/swipe_chatt_play_api/update_captionpic.php)" with jwt as auth bearer and as payload:  
    content: {  
    prompt: 'the prompt',  
    image_link: '[http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg](http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg)'  
    } 2. if response is "success" > navigate to DatingRadiusPage

task2:
1. write update_captionpic.php to save the payload as such in the profiledata:  
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
    {  
    _id: ObjectId('6825078f2be764913f0e1011'),  
    profile: {  
    topSection: { profileImages: [], name: 'luna', age: null, infoBubbles: {} },  
    elements: []  
    },  
    search_filter: {},  
    userdata: {  
    email: '[ogl@cracki.com](mailto:ogl@cracki.com)',  
    password_hash: '$2y$10$Vl602byl8L58hBVkxEVpEejlBDfRRP1PVgr3BhOoPzBxte7KbfY/q',  
    verification_code: '6412',  
    verification_code_expiry: ISODate('2025-05-14T21:28:51.000Z'),  
    is_verified: 0,  
    fingerprint_hash: null  
    }  
    },  
    {  
    _id: ObjectId('6825dc4adfcc0109a90db00d'),  
    profile: {  
    topSection: {  
    profileImages: [  
    '[http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6825dc83bf2a1_image_cropper_1747311734793.jpg](http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6825dc83bf2a1_image_cropper_1747311734793.jpg)',  
    '[http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6825dc83bf8e1_image_cropper_1747311742714.jpg](http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6825dc83bf8e1_image_cropper_1747311742714.jpg)'  
    ],  
    name: 'luna',  
    age: 23,  
    infoBubbles: {  
    gender: 'Female',  
    languages: [ 'German', 'English' ],  
    politics: 'Socialist',  
    religion: 'Other'  
    }  
    },  
    elements: [  
    {  
    _id: ObjectId('6825dc87e8126a01fc04ff5b'),  
    type: 'icebreaker',  
    content: {  
    question: 'Wenn ich ein Tier wäre, dann wäre ich ein …',  
    answer: 'zbab'  
    }  
    }  
    ]  
    },  
    search_filter: {},  
    userdata: {  
    email: '[lunalala@ji5.de](mailto:lunalala@ji5.de)',  
    password_hash: '$2y$10$PuYiFz.IWUKFk8ehm5R3ceXzsmLfhhyADpvgZHhSRtKbFZ16GIpYK',  
    verification_code: null,  
    verification_code_expiry: null,  
    is_verified: true,  
    fingerprint_hash: '$2y$10$0IXvrbVJvMVBXctVkjeMQuRHhvoUX5/TR8LoIm2/E2KBLcUpFTJj.'  
    }  
    },  
    {  
    _id: ObjectId('68260815dfcc0109a90db014'),  
    profile: {  
    topSection: { profileImages: [], name: 'luna', age: null, infoBubbles: {} },  
    elements: []  
    },  
    search_filter: {},  
    userdata: {  
    email: '[lunalarry3@ji5.de](mailto:lunalarry3@ji5.de)',  
    password_hash: '$2y$10$RNBOdRF2jbmGmwU7m6Efsum06XP7NyAwaRS1b/n6Fyt9/f48y7eyS',  
    verification_code: null,  
    verification_code_expiry: null,  
    is_verified: true,  
    fingerprint_hash: '$2y$10$kQhet0Aaqvu1H..G3GIMLue.7fozkRZ4k7O.JvgjMdB8UbkajprHi'  
    }  
    }  
    ]"