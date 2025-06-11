{
  _id: ObjectId('681d1c19e8126a01fc04ff50'),
  username: 'lunaneu',
  firstname: 'example',
  lastname: 'mample',
  email: 'luna@ji5.de',
  password_hash: '$2y$10$9P8MGw.6o.X.tPY9K.BTKu5NzjFGyQ1bnNXJ6em5MO/SeTI4qvs0a',
  verification_code: null,
  verification_code_expiry: null,
  is_verified: true,
  fingerprint_hash: '$2y$10$Oe/v8PxRt2e9xa71/uIQ1.ihbPrREZYL9YoXK95SLRT3YoYgjlkWC',
  profile: {
    topSection: {
      age: 23,
      infoBubbles: {
        career: 'sgv',
        languages: [ 'Afrikaans', 'Amharic' ],
        politics: 'Socialist'
      },
      name: 'lunaneu2',
      profileImages: [
        'http://panel.krasserserver.com:8002/swipe_chatt_play_api/create_profile/uploads/681d1c3be912d_image_cropper_1746738232621.jpg'
      ]
    },
    elements: [
      {
        _id: ObjectId('681d1c402be764913f0e07d3'),
        type: 'question_and_inputfield',
        content: {
          question: 'Wenn ich ein Tier wäre, dann wäre ich ein …',
          answer: 'hebw'
        }
      }
    ]
  },
  search_filter: {
    location_radius: {
      location: { type: 'Point', coordinates: [ 11.5801406, 48.1235834 ] },
      radius: 95
    },
    open_to: [ 'kayaking', 'movies', 'Arts & Crafts' ],
    searching_for: {
      genders: [ 'Other', 'Non Binary', 'Female', 'Transgender', 'Genderqueer' ],
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
}