{
  _id: ObjectId('681ce7ce9ea65af1560087b9'),
  profile: {
    topSection: {
      profileImages: [
        'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/681cb62220a04_image_cropper_1746712091332.jpg'
      ],
      name: 'luna2',
      age: 23,
      infoBubbles: {
        gender: 'Genderqueer',
        languages: [ 'German', 'English' ],
        religion: 'Judaism',
        politics: 'Socialist'
      }
    },
    elements: [
      {
        _id: ObjectId('681ce83c75e68e8ba54eeb86'),
        type: 'icebreaker',
        content: {
          question: 'Ein Ort, an den ich immer wieder zurückkehre...',
          answer: 'susi'
        }
      },
      {
        _id: ObjectId('681ce83c75e68e8ba54eeb87'),
        type: 'bubbles',
        content: { title: 'Open to', bubbles: [ 'biking', 'puzzles', 'games' ] }
      },
      {
        _id: ObjectId('681ce83c75e68e8ba54eeb88'),
        type: 'question_and_inputfield',
        content: {
          question: 'was hat dich heute gefreut?',
          answer: 'what user typed into inputfield'
        }
      },
      {
        _id: ObjectId('681ce83c75e68e8ba54eeb89'),
        type: 'voicememo',
        content: {
          prompt: 'the prompt',
          link_voicemessage: 'http://node02.krasserserver.com:8002/swipe_chatt_play_api/chat/uploads/voicememos/voice_67f877edb34a88.05803339_1744336870367.aac'
        }
      },
      {
        _id: ObjectId('681ce83c75e68e8ba54eeb8a'),
        type: 'topic_suggestions',
        content: {
          prompt: 'ask me about...',
          choice: 'musik',
          answer: 'welche interpreten magst du?'
        }
      },
      {
        _id: ObjectId('681ce83c75e68e8ba54eeb8b'),
        type: 'image_and_prompt',
        content: {
          prompt: 'the prompt',
          image_link: 'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/6813d870a43b1_image_cropper_1746131052193.jpg'
        }
      }
    ]
  },
  search_filter: {
    location_radius: {
      location: { type: 'Point', coordinates: [ 11.5801204, 48.1235802 ] },
      radius: 87
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
    },
    open_to: [ 'biking', 'puzzles', 'games' ]
  },
  userdata: {
    email: 'luna.vogl.35@gmail.com',
    password_hash: '$2y$10$YYyEA8ocGk9iU9dZUjLBaeEj9iToWfgcy0oECE5UrA3NIhik2Cm5e',
    verification_code: null,
    verification_code_expiry: null,
    is_verified: true,
    fingerprint_hash: '$2y$10$eoEopqwq3ZtyePlBmpP8e.yHN2QQrsl2VGqqt9TrztElBT6aFXS5.'
  }
}