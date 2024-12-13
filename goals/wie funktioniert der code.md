api.php nimmt zwei ids als input und frägt dann die daten von databse ab

api_newfromtemplate_changejson_update_start_givedeliveryURL.php

1.  newfromtemplate() > jsonformat for contract
	1. get document id from that
2. edit the json with changejson()
3. update the document in scrive POST update 
4. start document POST start -> get json and extract deliveryURL where user can sign > give that to app
5. flutter app receives answer link opens in webview nutzer kann unterschreiben


#### Flutter Bloc structure:
###### **What is BLoC Architecture?**

BLoC (Business Logic Component) is a design pattern for Flutter that separates **business logic** from the **UI layer**, making the app more testable, reusable, and maintainable.

- **UI Layer**: Widgets that display the interface and listen to changes in the state.
- **BLoC Layer**: Manages the application logic and state changes.
- **Repository Layer**: Handles data access (e.g., API calls).
- **Data Layer**: Communicates with the database or APIs.

---

###### **Project File Structure**

Here’s the suggested file structure for your **Flutter project**:
lib/
├── bloc/
│   ├── login_bloc.dart
│   ├── login_event.dart
│   ├── login_state.dart
│   ├── signup_bloc.dart
│   ├── signup_event.dart
│   ├── signup_state.dart
├── data/
│   ├── api/
│   │   ├── auth_api.dart   # Handles API calls (login/signup)
│   └── models/
│       ├── user_model.dart # User data model
├── repository/
│   ├── auth_repository.dart # Interacts with the auth_api.dart
├── screens/
│   ├── login_screen.dart
│   ├── signup_screen.dart
├── widgets/
│   ├── custom_input_field.dart
│   ├── custom_button.dart
├── main.dart


---

###### **Flutter File Details**

####### 1. **BLoC Layer**

- **`login_bloc.dart`**  
    Handles the login logic, such as validating inputs and calling the repository to interact with the API.
    
- **`signup_bloc.dart`**  
    Handles signup logic similarly.
    
- **`login_event.dart`**  
    Defines events like `LoginSubmitted`.
    
- **`login_state.dart`**  
    Defines states like `LoginInitial`, `LoginLoading`, `LoginSuccess`, and `LoginFailure`.
    

####### 2. **Data Layer**

- **`auth_api.dart`**  
    Handles actual API requests (e.g., POST for login/signup).
    
- **`user_model.dart`**  
    Defines the structure of user data.
    

####### 3. **Repository Layer**

- **`auth_repository.dart`**  
    Abstracts the API calls to the BLoC.

####### 4. **UI Layer**

- **`login_screen.dart` & `signup_screen.dart`**  
    Flutter screens that use the BLoC to handle user interactions.