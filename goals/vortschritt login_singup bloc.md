Great!
-MongoDB is installed, admin account created, logged in with password and created "myDatabase" and have a user there for the api:
"
myDatabase> db.getUsers();
{witched to db myDatabase
  users: [
    {
      _id: 'myDatabase.myPhpUser',
      userId: UUID('72ba4ed7-da73-4584-94a9-4a1aa462383e'),
      user: 'myPhpUser',
      db: 'myDatabase',
      roles: [ { role: 'readWrite', db: 'myDatabase' } ],
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
    }
  ],
  ok: 1
}
" and seems to be running on localhost:
"Connecting to:          mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=admin&appName=mongosh+2.3.3
"

- PHP mongoDB extension is enabled (shown on "http://localhost/phpinfo.php")

- i have created the same folderstructure as you have in my lib in my flutter project:
"
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
"
- pasted these files:
	- auth_api.dart (../lib/data/api/auth_api.dart
	- login.php (../htdocs/login.php)
	- signup.php (../htdocs/signup.php)



pvUGRCdGiEqOloo+2DyTt2Bdlp5KFiyy0yoJAjDzaQ2/5hBQKlm9E6L7PVDWUW0r
kpBxIMlAjCyL9H2ZhBlq+g==