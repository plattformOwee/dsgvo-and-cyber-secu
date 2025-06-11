so turn these two:

 profile: {
    about_you: {
      name: 'luna',
      age: 23,
      gender: 'Non-binary',
      spokenLanguage: 'Arabic, Amharic',
      religion: 'Hinduism',
      politics: 'Socialist'
    },
    images: [
      'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg'
    ],
    questions_and_answers: [
      {
        question: 'Das verrückteste Abenteuer, das ich je erlebt habe...',
        answer: 'evw'
      }
    ],
    location_radius: {
      location: { type: 'Point', coordinates: [ 11.5526917, 48.10666 ] },
      radius: 100
    },
    open_to: [
      'birdwatching',
      'kayaking',
      'biking',
      'puzzles',
      'Arts & Crafts'
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
  profile_structured: {
    top_section: {
      profileImages: [
        'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg'
      ],
      name: 'luna',
      username: '',
      id: '',
      age: '23',
      infoBubbles: [ 'Non-binary', 'Arabic, Amharic', 'Hinduism', 'Socialist' ]
    },
    elements: {
      '1': {
        type: 'icebreaker',
        content: {
          question: 'Das verrückteste Abenteuer, das ich je erlebt habe...',
          answer: 'evw'
        }
      },
      '2': {
        type: 'bubbles',
        content: {
          title: 'Open to',
          bubbles: [
            'birdwatching',
            'kayaking',
            'biking',
            'puzzles',
            'Arts & Crafts'
          ]
        }
      }
    }
  },
  "
  
  profile_structured: {
    top_section: {
      profileImages: [
        'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg'
      ],
      name: 'luna',
      username: '',
      id: '',
      age: '23',
      infoBubbles: [ 'Non-binary', 'Arabic, Amharic', 'Hinduism', 'Socialist' ]
    },
    elements: {
      '1': {
        type: 'icebreaker',
        content: {
          question: 'Das verrückteste Abenteuer, das ich je erlebt habe...',
          answer: 'evw'
        }
      },
      '2': {
        type: 'bubbles',
        content: {
          title: 'Open to',
          bubbles: [
            'birdwatching',
            'kayaking',
            'biking',
            'puzzles',
            'Arts & Crafts'
          ]
        }
      }
    }
  },"
  
  
into one:

profile_structured: {
	"non_visible_data":{
		location_radius: {
	      location: { type: 'Point', coordinates: [ 11.5526917, 48.10666 ] },
	      radius: 100
	    },
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
	}
    top_section: {
      profileImages: [
        'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg'
      ],
      name: 'luna',
      username: '',
      id: '',
      age: '23',
      infoBubbles: {
	      "gender":"non-binary",
	      "languages":["language1", "language2"],
	      "politics":"socialist for example",
      },
    },
    elements: {
      '1': {
        type: 'icebreaker',
        content: {
          question: 'Das verrückteste Abenteuer, das ich je erlebt habe...',
          answer: 'evw'
        }
      },
      '2': {
        type: 'bubbles',
        content: {
          title: 'Open to',
          bubbles: [ // let match algorithm now read that list from here
            'birdwatching',
            'kayaking',
            'biking',
            'puzzles',
            'Arts & Crafts'
          ]
        }
      }
    }
  },
1. write a script that turns the former into the latter 
2. adapt the match algorithm to this
