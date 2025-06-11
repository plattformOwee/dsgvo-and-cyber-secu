{
    _id: ObjectId('6839d7de2be764913f0e12f7'),
    username: 'luna',
    firstname: 'example',
    lastname: 'mample',
    email: 'luna.vogl.35@gmail.com',
    password_hash: '$2y$10$QssxRLLvl9H5Q5an.XBe2udquukygXHIcBsA.Ks8u2Y0eLkI/txiu',
    verification_code: null,
    verification_code_expiry: null,
    is_verified: true,
    userdata: {
      lender :{
	      NextPayment: { amount_to_pay: '200$', date_to_pay: '15.10' },
	      upcomingPayments: {
	        '1': {
	          amount: '200',
	          payment_method: 'debit via SEPA',
	          monthly_payment_date: '26',
	          comprising_contracts: {
	            '1': '100$ flexible rate',
	            '2': '80$ flexible rate',
	            '3': '80$ flexible rate'
	          }
	        },
	        '2': {
	          amount: '200',
	          payment_method: 'debit via SEPA',
	          monthly_payment_date: '26',
	          comprising_contracts: {
	            '1': '100$ flexible rate',
	            '2': '80$ flexible rate',
	            '3': '80$ flexible rate'
	          }
	        },
	        '3': {
	          amount: '200',
	          payment_method: 'debit via SEPA',
	          monthly_payment_date: '26',
	          comprising_contracts: {
	            '1': '100$ flexible rate',
	            '2': '80$ flexible rate',
	            '3': '80$ flexible rate'
	          }
	        }
	      },
	      history: {
	        '1': {
	          description: '200$ payed out to paypal',
	          subtitle: 'borrowed 20.09',
	          date: '26',
	          'link to symbol': 'www.link.de'
	        },
	        '2': {
	          description: '400$ payed out to paypal',
	          subtitle: 'borrowed 20.09',
	          date: '26',
	          'link to symbol': 'www.link.de'
	        },
	        '3': {
	          description: '600$ payed out to paypal',
	          subtitle: 'borrowed 20.09',
	          date: '26',
	          'link to symbol': 'www.link.de'
	        }
	      },
	      contracts: {
	        '1': {
	          amount: '200$',
	          'from / to': '@nickname',
	          timeframe: '26.11.22 - 26.4.23',
	          raten: '4 Monate je 50€'
	        },
	        '2': {
	          amount: '200$',
	          'from / to': '@nickname',
	          timeframe: '26.11.22 - 26.4.23',
	          raten: '4 Monate je 50€'
	        },
	        '3': {
	          amount: '200$',
	          'from / to': '@nickname',
	          timeframe: '26.11.22 - 26.4.23',
	          raten: '4 Monate je 50€'
	        }
	      }
      }
      borrower :{
	      account_balance: "2500",
	      balance_over_time: {
	        (store here however this graph should be stored)
	      },
	      history: {
	        '1': {
	          description: '200$ payed out to paypal',
	          subtitle: 'borrowed 20.09',
	          date: '26',
	          'link to symbol': 'www.link.de'
	        },
	        '2': {
	          description: '400$ payed out to paypal',
	          subtitle: 'borrowed 20.09',
	          date: '26',
	          'link to symbol': 'www.link.de'
	        },
	        '3': {
	          description: '600$ payed out to paypal',
	          subtitle: 'borrowed 20.09',
	          date: '26',
	          'link to symbol': 'www.link.de'
	        }
	      },
	      contracts: {
	        '1': {
	          amount: '200$',
	          'from / to': '@nickname',
	          timeframe: '26.11.22 - 26.4.23',
	          raten: '4 Monate je 50€'
	        },
	        '2': {
	          amount: '200$',
	          'from / to': '@nickname',
	          timeframe: '26.11.22 - 26.4.23',
	          raten: '4 Monate je 50€'
	        },
	        '3': {
	          amount: '200$',
	          'from / to': '@nickname',
	          timeframe: '26.11.22 - 26.4.23',
	          raten: '4 Monate je 50€'
	        }
	      }
      }
    },
    fingerprint_hash: '$2y$10$Rc1q0gV0G2m2eDMknY82LeBAhxdfuXcBPLyQFaQeJPqzqCl3wdfGq'
  }