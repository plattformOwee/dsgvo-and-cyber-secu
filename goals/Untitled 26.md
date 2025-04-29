Also each subpayment can be moved maximally 3 times so we should track how often one has been moved in the json and also display that.

1. change signup.dart to save profilejson > upcoming payments differently:
   "
   upcomingPayments: {
          '1': {
            amount: '200',
            payment_method: 'debit via SEPA',
            monthly_payment_date: '26',
            comprising_contracts: {
              '1': {name: '100$ flexible rate', reschedules_ _left:"3"},
              '2': {name: '80$ flexible rate', reschedules_ _left:"3"},
              '3': {name: '80$ flexible rate', reschedules_ _left:"3"}
            }
          },
          '2': {
            amount: '200',
            payment_method: 'debit via SEPA',
            monthly_payment_date: '26',
            comprising_contracts: {
              '1':{name: '100$ flexible rate', reschedules_ _left:"3"}, 
              '2': {name: '80$ flexible rate', reschedules_ _left:"3"}, 
              '3': {name: '80$ flexible rate', reschedules_ _left:"3"},
            }
          },
          '3': {
            amount: '200',
            payment_method: 'debit via SEPA',
            monthly_payment_date: '26',
            comprising_contracts: {
              '1':{name: '100$ flexible rate', reschedules_ _left:"3"}, 
              '2': {name: '80$ flexible rate', reschedules_ _left:"3"},
              '3': {name: '80$ flexible rate', reschedules_left:"3"},
            }
          }
        },
   "

2. change reschedule payment so it additionally marks how often a given subpament has been moved by checking how often it has been "moved": 
   'if ()"moved" <= 3): make minus 1 and save new value.
   else: return status "limit reached" and current profile json. (minus one if moved down and plus one if moved up.)
3. change UpcomingPayments to 
	3.1 also accept "limit reached" and current profile json as response. 
	3.2 make the string in button "reschedule" to be "reschedule (§reschedules_left)"