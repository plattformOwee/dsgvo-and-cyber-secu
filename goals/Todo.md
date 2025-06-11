#### 7.5
- [x] app name : woven [[name ideas dating app ]]
- [x] alle bisherigen bubble design files nice designed in flutter
- [x] schuldschein prüfung in auftrag gegeben
- [x] block and report user funktional gemacht
- [x] das ganze mit dem getrennten chat.dart vereint
- [ ] unifying the structured json and search algorithm to not have to update everything twice by copying the entire folder and renaming it "name of app"
- [ ] write out the json of each 
	- [ ] profile element json
		- [ ] {
		      "object id":{
			      "type":"question and inputfield"
			      "content":{
				      "question":"was hat dich heute gefreut?",
				      "answer":"what user typed into inputfield"
			      }
		      },
		      "object id":{
			      "type":"voicememo",
			      "content":{
				      "prompt":"the prompt", // can be null
				      "link_voicemessage":"http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f877edb34a88.05803339_1744336870367.aac"  // the link generated after saving the thing i will provide test link for now
			      }
		      },
		      "object id":{ 
			      "type":"topic_suggestions"
			      "content":{
				      "prompt":"the prompt" // example: "ask me about..."
				      "choice":"the chosen topic" // example: "musik"
				      "answer":"the answer from the swiper" // example: "welche interpreten magst du?"
			      }
		      }
		      "object id":{
			      "type":"image"
			      "content":{
				      "prompt":"the prompt", // can be null and then just image is shown
				      "image_linke":"http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg", // will come from menu later but now use this 
			      }
		      }
		    }
	- [ ] assosiated chat bubble json
- [ ] create list of elements and jobs 
	- [ ] q and a
		- [ ] profile layout file
	- [ ] q and inputfield

for each element
- [ ] add in profile elements
	- [ ] in db
	- [ ] as layout file
	- [ ] creation menu
		- [ ] in creation flow
		- [ ] in edit profile page
- [ ] add reaction bubble of each missing profiel type to chat 
	- [ ] in db
	- [ ] as layout file
	- [ ] as consequence of reacting to and element
- [ ] maybe fix reorder of profile
#### 06.05.25
- [x] chat design in prototype fertig all bubbles
- [ ] switch each design file
- [ ] add element types
- [ ] write gerd make an appointment for next week
- [ ] plan in one day time intervals the next weeks and then parts into it


- [ ] preiskalkulator fertig
	- [ ] auto
		- [x] skalierend 
		- [x] tabelle richtig
		- [ ] slider worked
			- [ ] richtig
			- [x] worked
	- [ ] motorrad
		- [ ] skalierend
		- [ ] tabelle richtig
	- [ ] roller
		- [ ] skalierend
		- [ ] tabelle richtig
	- [ ] mofa
		- [ ] skalierend
		- [ ] tabelle richtig
	- [ ] anhänger
		- [ ] skalierend
		- [ ] tabelle richtig
- [ ] Login and Signup Layouts