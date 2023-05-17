# mvvm-flutter

In Flutter, MVVM (Model-View-ViewModel) is a design pattern that separates the presentation logic (View) from the business logic (Model) using a mediator (ViewModel). Here's an example of how to create a simple MVVM structure in Flutter:

1. Create the `Model` class:

   ```dart
   class Person {
     final String name;
     final int age;

     Person({required this.name, required this.age});
   }
   ```

2. Create the `ViewModel` class:

   ```dart
   import 'package:flutter/material.dart';
   import 'package:flutter_mvvm_example/models/person.dart';

   class PersonViewModel extends ChangeNotifier {
     Person? _person;

     void setPerson(Person person) {
       _person = person;
       notifyListeners();
     }

     String get name => _person?.name ?? '';
     int get age => _person?.age ?? 0;
   }
   ```

   The `PersonViewModel` class is responsible for managing the `Person` object. It exposes the `name` and `age` properties through getters and uses the `notifyListeners` method to notify the View when the data changes.

3. Create the `View`:

   ```dart
   import 'package:flutter/material.dart';
   import 'package:flutter_mvvm_example/viewmodels/person_viewmodel.dart';
   import 'package:provider/provider.dart';

   class PersonPage extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return Scaffold(
         appBar: AppBar(
           title: Text('Person'),
         ),
         body: Center(
           child: Column(
             mainAxisAlignment: MainAxisAlignment.center,
             children: [
               Consumer<PersonViewModel>(
                 builder: (context, personViewModel, child) => Text(
                   personViewModel.name,
                   style: TextStyle(fontSize: 24),
                 ),
               ),
               Consumer<PersonViewModel>(
                 builder: (context, personViewModel, child) => Text(
                   '${personViewModel.age}',
                   style: TextStyle(fontSize: 24),
                 ),
               ),
             ],
           ),
         ),
       );
     }
   }
   ```

   The `PersonPage` class is responsible for rendering the View. It uses the `Consumer` widget from the `provider` package to listen to changes in the `PersonViewModel` and update the UI accordingly.

4. Set up the `ViewModel`:

   ```dart
   import 'package:flutter/material.dart';
   import 'package:flutter_mvvm_example/models/person.dart';
   import 'package:flutter_mvvm_example/viewmodels/person_viewmodel.dart';
   import 'package:flutter_mvvm_example/views/person_page.dart';
   import 'package:provider/provider.dart';

   void main() {
     runApp(MyApp());
   }

   class MyApp extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return ChangeNotifierProvider(
         create: (context) => PersonViewModel(),
         child: MaterialApp(
           title: 'Flutter MVVM Example',
           home: PersonPage(),
         ),
       );
     }
   }
   ```

   In the `main` function, we set up the `PersonViewModel` using the `ChangeNotifierProvider` widget from the `provider` package. This widget allows us to provide the `PersonViewModel` to the `PersonPage` and its child widgets.

With this structure, we can easily manage the `Person` data using the `PersonViewModel` and update the UI using the `PersonPage`. This separation of concerns makes our code more modular and easier to maintain.

# mvvm-flutter-firebase

MVVM (Model-View-ViewModel) is an architectural pattern that separates an application into three layers: the Model layer (data), the View layer (UI), and the ViewModel layer (business logic). In Flutter, we can implement MVVM architecture to make our code more modular, testable, and maintainable. Here's an example of a simple MVVM structure in Flutter using Firebase:

1. Create a `model` directory to store all the data-related classes:

```dart
class User {
  final String id;
  final String email;

  User({required this.id, required this.email});
}
```

2. Create a `view` directory to store all the UI-related classes:

```dart
class HomeView extends StatelessWidget {
  final UserViewModel viewModel;

  HomeView({required this.viewModel});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Welcome ${viewModel.user.email}'),
            ElevatedButton(
              onPressed: viewModel.signOut,
              child: Text('Sign Out'),
            ),
          ],
        ),
      ),
    );
  }
}
```

3. Create a `viewmodel` directory to store all the business logic classes:

```dart
class UserViewModel {
  final FirebaseAuth _auth = FirebaseAuth.instance;
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;

  User? _user;
  User? get user => _user;

  Future<void> signIn() async {
    // sign in with Firebase Authentication
    final UserCredential userCredential = await _auth.signInAnonymously();

    // create a new user in Firestore
    final User newUser = User(
      id: userCredential.user!.uid,
      email: userCredential.user!.email!,
    );
    await _firestore.collection('users').doc(newUser.id).set(newUser);

    _user = newUser;
  }

  Future<void> signOut() async {
    // sign out from Firebase Authentication
    await _auth.signOut();

    // delete the user from Firestore
    await _firestore.collection('users').doc(_user!.id).delete();

    _user = null;
  }
}
```

4. In the `main` function, create instances of the `UserViewModel` and `HomeView` classes and pass the `viewModel` to the `HomeView`:

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final UserViewModel viewModel = UserViewModel();

    return MaterialApp(
      title: 'MVVM with Firebase',
      home: HomeView(viewModel: viewModel),
    );
  }
}
```

In this example, we use Firebase Authentication and Firestore to authenticate and store user data. The `User` class represents a user object with an `id` and an `email`. The `UserViewModel` class contains the logic for signing in and out, as well as storing the current user. The `HomeView` class displays the user's email and a sign-out button. When the user signs out, the `HomeView` updates automatically to show a message to the user.

# mvvm-flutter-with-api-call-using-http-package

Sure! Here's an example of a simple MVVM structure in Flutter with API CRUD operations using the `http` package for making HTTP requests:

1. Model:
```dart
class User {
  final int id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}
```

2. ViewModel:
```dart
import 'package:http/http.dart' as http;

class UserViewModel {
  final String apiUrl = 'https://example.com/api/users';

  Future<List<User>> getUsers() async {
    final response = await http.get(Uri.parse(apiUrl));

    if (response.statusCode == 200) {
      final List<dynamic> jsonResponse = jsonDecode(response.body);
      return jsonResponse.map((json) => User.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load users');
    }
  }

  Future<User> createUser(User user) async {
    final response = await http.post(
      Uri.parse(apiUrl),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(user.toJson()),
    );

    if (response.statusCode == 201) {
      return User.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to create user');
    }
  }

  Future<void> updateUser(User user) async {
    final response = await http.put(
      Uri.parse('$apiUrl/${user.id}'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(user.toJson()),
    );

    if (response.statusCode != 200) {
      throw Exception('Failed to update user');
    }
  }

  Future<void> deleteUser(int id) async {
    final response = await http.delete(Uri.parse('$apiUrl/$id'));

    if (response.statusCode != 204) {
      throw Exception('Failed to delete user');
    }
  }
}
```

3. View:
```dart
import 'package:flutter/material.dart';
import 'package:your_app_name/view_model.dart';

class UserListPage extends StatefulWidget {
  @override
  _UserListPageState createState() => _UserListPageState();
}

class _UserListPageState extends State<UserListPage> {
  final UserViewModel _userViewModel = UserViewModel();
  List<User> _users = [];

  @override
  void initState() {
    super.initState();
    _loadUsers();
  }

  Future<void> _loadUsers() async {
    final users = await _userViewModel.getUsers();
    setState(() {
      _users = users;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('User List'),
      ),
      body: ListView.builder(
        itemCount: _users.length,
        itemBuilder: (context, index) {
          final user = _users[index];
          return ListTile(
            title: Text(user.name),
            subtitle: Text(user.email),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          final newUser = User(id: 0, name: 'New User', email: 'newuser@example.com');
          final createdUser = await _userViewModel.createUser(newUser);
          setState(() {
            _users.add(createdUser);
          });
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

In this example, we have a `User` model that represents a user with an `id`, `name`, and `email` property. The `UserViewModel` is responsible for making HTTP requests to the API and returning data to the view. It has methods for `getUsers()`, `createUser()`, `updateUser()`, and `deleteUser()` that make requests to the API. The `UserListPage` is the view that displays a list of users and allows the user to add new users. It uses the `UserViewModel` to load the list of users and to create new users.

This example is just one way of implementing MVVM in Flutter, and there are many other ways to structure your code depending on your specific needs and preferences.

# mvvm-flutter-with-api-call-using-http-package-2

Sure, here's a simple MVVM structure in Flutter with API calls and CRUD operations using the `http` package.

First, let's define the model class `User`:

```dart
class User {
  final int id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }

  Map<String, dynamic> toJson() => {
        'id': id,
        'name': name,
        'email': email,
      };
}
```

This is a simple `User` model with an `id`, `name`, and `email` properties. We also define a `fromJson` factory method to convert a JSON object to a `User` object, and a `toJson` method to convert a `User` object to a JSON object.

Next, let's define the `UserService` class, which handles the API calls:

```dart
class UserService {
  static const String baseUrl = 'https://jsonplaceholder.typicode.com';

  Future<List<User>> getUsers() async {
    final response = await http.get(Uri.parse('$baseUrl/users'));

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body) as List;
      return data.map((json) => User.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load users');
    }
  }

  Future<User> createUser(User user) async {
    final response = await http.post(
      Uri.parse('$baseUrl/users'),
      body: jsonEncode(user.toJson()),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode == 201) {
      final data = jsonDecode(response.body);
      return User.fromJson(data);
    } else {
      throw Exception('Failed to create user');
    }
  }

  Future<void> updateUser(User user) async {
    final response = await http.put(
      Uri.parse('$baseUrl/users/${user.id}'),
      body: jsonEncode(user.toJson()),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode != 200) {
      throw Exception('Failed to update user');
    }
  }

  Future<void> deleteUser(int userId) async {
    final response = await http.delete(Uri.parse('$baseUrl/users/$userId'));

    if (response.statusCode != 200) {
      throw Exception('Failed to delete user');
    }
  }
}
```

The `UserService` class has four methods for CRUD operations: `getUsers`, `createUser`, `updateUser`, and `deleteUser`. Each method makes an HTTP request using the `http` package and returns a `Future` with the result.

Now, let's define the `UserViewModel` class, which is responsible for fetching and updating the user data:

```dart
class UserViewModel extends ChangeNotifier {
  final UserService _userService = UserService();
  List<User> _users = [];
  bool _loading = false;

  List<User> get users => _users;
  bool get loading => _loading;

  Future<void> getUsers() async {
    _loading = true;
    notifyListeners();

    try {
      _users = await _userService.getUsers();
      notifyListeners();
    } catch (e) {
      print(e);
    } finally {
      _loading = false;
      notifyListeners();
    }
  }

  Future<void> createUser(User user) async {
    try {
      final newUser = await _userService.createUser(user);
      _users.add(newUser);
      notifyListeners();
    } catch (e) {
      print(e);
    }


Similarly, we can define the `updateUser` and `deleteUser` methods in the `UserViewModel` class:

```dart
  Future<void> updateUser(User user) async {
    try {
      await _userService.updateUser(user);
      final index = _users.indexWhere((u) => u.id == user.id);
      _users[index] = user;
      notifyListeners();
    } catch (e) {
      print(e);
    }
  }

  Future<void> deleteUser(int userId) async {
    try {
      await _userService.deleteUser(userId);
      _users.removeWhere((user) => user.id == userId);
      notifyListeners();
    } catch (e) {
      print(e);
    }
  }
}
```

Finally, we can use the `UserViewModel` class in the `UserScreen` widget:

```dart
class UserScreen extends StatefulWidget {
  const UserScreen({Key? key}) : super(key: key);

  @override
  _UserScreenState createState() => _UserScreenState();
}

class _UserScreenState extends State<UserScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  late final UserViewModel _viewModel;

  @override
  void initState() {
    super.initState();
    _viewModel = UserViewModel();
    _viewModel.getUsers();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('User Management'),
      ),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          const SizedBox(height: 16),
          if (_viewModel.loading) const CircularProgressIndicator(),
          Expanded(
            child: ListView.builder(
              itemCount: _viewModel.users.length,
              itemBuilder: (context, index) {
                final user = _viewModel.users[index];
                return ListTile(
                  title: Text(user.name),
                  subtitle: Text(user.email),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete),
                    onPressed: () => _viewModel.deleteUser(user.id),
                  ),
                );
              },
            ),
          ),
          const SizedBox(height: 16),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Form(
              key: _formKey,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  TextFormField(
                    controller: _nameController,
                    decoration: const InputDecoration(
                      labelText: 'Name',
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please enter a name';
                      }
                      return null;
                    },
                  ),
                  TextFormField(
                    controller: _emailController,
                    decoration: const InputDecoration(
                      labelText: 'Email',
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please enter an email';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      if (_formKey.currentState?.validate() ?? false) {
                        final user = User(
                          id: _viewModel.users.length + 1,
                          name: _nameController.text,
                          email: _emailController.text,
                        );
                        _viewModel.createUser(user);
                        _nameController.clear();
                        _emailController.clear();
                      }
                    },
                    child: const Text('Add User'),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

This `UserScreen` widget displays a list of users fetched from the API using the `UserViewModel` class. It also allows the user to add a new user and delete an existing user. The form validation is done using the `Form`.

Please note that in the code I provided, I assumed that the API methods of the `UserService` class (`createUser`, `getUsers`, `updateUser`, and `deleteUser`) are implemented using the `http` package. You will need to define these methods according to the API you are using.


# Here's an example of a simple MVVM (Model-View-ViewModel) implementation using FlutterFire, which is a set of Flutter plugins for Firebase integration:

```dart
// lib/model/user.dart

class User {
  final String id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});
}
```

```dart
// lib/viewmodel/user_viewmodel.dart

import 'package:flutter/foundation.dart';
import 'package:your_app_name/model/user.dart';
import 'package:firebase_auth/firebase_auth.dart';

class UserViewModel extends ChangeNotifier {
  User? _currentUser;

  User? get currentUser => _currentUser;

  final FirebaseAuth _firebaseAuth = FirebaseAuth.instance;

  Future<void> login(String email, String password) async {
    try {
      final userCredential = await _firebaseAuth.signInWithEmailAndPassword(
        email: email,
        password: password,
      );
      final firebaseUser = userCredential.user;

      if (firebaseUser != null) {
        _currentUser = User(
          id: firebaseUser.uid,
          name: firebaseUser.displayName ?? '',
          email: firebaseUser.email ?? '',
        );
      }

      notifyListeners();
    } catch (e) {
      // Handle login error
    }
  }

  Future<void> logout() async {
    await _firebaseAuth.signOut();
    _currentUser = null;
    notifyListeners();
  }
}
```

```dart
// lib/main.dart

import 'package:flutter/material.dart';
import 'package:your_app_name/viewmodel/user_viewmodel.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => UserViewModel(),
      child: MaterialApp(
        title: 'MVVM FlutterFire Example',
        home: LoginPage(),
      ),
    );
  }
}

class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userViewModel = Provider.of<UserViewModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Login Page'),
      ),
      body: Center(
        child: ElevatedButton(
          child: Text('Login'),
          onPressed: () {
            userViewModel.login('example@example.com', 'password');
          },
        ),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userViewModel = Provider.of<UserViewModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Home Page'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Welcome, ${userViewModel.currentUser?.name ?? 'Guest'}!'),
            ElevatedButton(
              child: Text('Logout'),
              onPressed: () {
                userViewModel.logout();
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

In this example, we have a `User` class representing a user's data. The `UserViewModel` class is responsible for managing the user state, including login and logout functionality. It uses the `FirebaseAuth` class from FlutterFire to handle authentication.

In the `main.dart` file, we use the `ChangeNotifierProvider` from the `provider` package to provide an instance of the `UserViewModel` to the widget tree. The `LoginPage` widget shows a login button, which triggers the `login` method in the `UserViewModel` when pressed. Upon successful login, the `currentUser` property is updated, and the UI is notified.

If the user is logged in, the `HomePage` widget is displayed, showing a welcome

# Here's an example of how you can write unit tests for the `User` model class:

```dart
// test/user_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/model/user.dart';

void main() {
  test('User constructor should set the provided values', () {
    final user = User(
      id: '1',
      name: 'John Doe',
      email: 'john.doe@example.com',
    );

    expect(user.id, '1');
    expect(user.name, 'John Doe');
    expect(user.email, 'john.doe@example.com');
  });

  test('User toString() should return a formatted string', () {
    final user = User(
      id: '1',
      name: 'John Doe',
      email: 'john.doe@example.com',
    );

    final expectedString = 'User{id: 1, name: John Doe, email: john.doe@example.com}';
    expect(user.toString(), expectedString);
  });
}
```

In this example, we have two unit tests for the `User` model class. The first test verifies that the constructor correctly sets the provided values. It creates an instance of `User` with specific values and then asserts that the properties (`id`, `name`, and `email`) are set correctly.

The second test validates the `toString()` method of the `User` class. It creates a `User` instance and compares the result of calling `toString()` with the expected formatted string.

To run these tests, you can use the `flutter test` command in the terminal or run the tests from your IDE. If all the tests pass, it confirms that the `User` model class behaves as expected. If any test fails, it indicates that there might be an issue with the `User` class implementation.

# Here's an example of how you can write unit tests for the `UserViewModel` class:

```dart
// test/user_viewmodel_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/model/user.dart';
import 'package:your_app_name/viewmodel/user_viewmodel.dart';

class MockFirebaseAuth extends Mock implements FirebaseAuth {}

void main() {
  group('UserViewModel', () {
    late UserViewModel userViewModel;
    late MockFirebaseAuth mockFirebaseAuth;

    setUp(() {
      mockFirebaseAuth = MockFirebaseAuth();
      userViewModel = UserViewModel(firebaseAuth: mockFirebaseAuth);
    });

    test('Login should set the current user when successful', () async {
      // Arrange
      final email = 'test@example.com';
      final password = 'password';
      final firebaseUser = MockFirebaseUser();
      when(mockFirebaseAuth.signInWithEmailAndPassword(
        email: email,
        password: password,
      )).thenAnswer((_) => Future.value(UserCredential(user: firebaseUser)));

      // Act
      await userViewModel.login(email, password);

      // Assert
      expect(userViewModel.currentUser, User(
        id: firebaseUser.uid,
        name: firebaseUser.displayName ?? '',
        email: firebaseUser.email ?? '',
      ));
    });

    test('Logout should reset the current user', () async {
      // Arrange
      userViewModel._currentUser = User(
        id: '1',
        name: 'John Doe',
        email: 'john.doe@example.com',
      );

      // Act
      await userViewModel.logout();

      // Assert
      expect(userViewModel.currentUser, isNull);
    });
  });
}
```

In this example, we have two unit tests for the `UserViewModel` class. We use the `MockFirebaseAuth` class to create a mock implementation of the `FirebaseAuth` class to simulate Firebase authentication behavior.

In the first test, we simulate a successful login scenario. We set up the necessary dependencies, including the email and password, and create a mock `FirebaseUser` instance. We use the `when` function from the `mockito` package to define the behavior of the mock `FirebaseAuth` instance. We instruct it to return a `UserCredential` containing the mock `FirebaseUser` when the `signInWithEmailAndPassword` method is called with the expected email and password.

We then call the `login` method of the `userViewModel` instance and verify that the `currentUser` property is correctly set to a `User` instance with the expected values.

The second test focuses on the logout functionality. We set a sample user in the `currentUser` property of the `userViewModel` instance and then call the `logout` method. After the logout, we assert that the `currentUser` property is reset to `null`.

To run these tests, you can use the `flutter test` command in the terminal or run the tests from your IDE. If all the tests pass, it confirms that the `UserViewModel` class behaves as expected. If any test fails, it indicates that there might be an issue with the `UserViewModel` class implementation.

# To unit test a view in Flutter, you can follow these steps:

1. Create a new test file for your view. The convention is to place test files in the `test/` directory, following your project's directory structure.

2. Import the necessary packages, including `flutter_test.dart` and the files that contain the view and its associated view model.

3. Define a test group using the `group` function. This helps organize and group related test cases together.

4. Within the test group, use the `testWidgets` function to define individual test cases that verify the behavior of your view.

5. Use the `pumpWidget` function to build and render the view in the test case.

6. Use various `expect` functions to assert the expected behavior and appearance of the rendered view.

Here's an example of how you can write unit tests for a view in Flutter:

```dart
// test/my_view_test.dart

import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/view/my_view.dart';
import 'package:your_app_name/viewmodel/my_view_model.dart';

void main() {
  group('MyView', () {
    testWidgets('UI should display the correct text', (WidgetTester tester) async {
      final viewModel = MyViewModel();
      
      await tester.pumpWidget(
        MaterialApp(
          home: MyView(viewModel: viewModel),
        ),
      );

      final textFinder = find.text('Hello, World!');
      expect(textFinder, findsOneWidget);
    });

    testWidgets('Button should trigger the correct action', (WidgetTester tester) async {
      final viewModel = MyViewModel();
      
      await tester.pumpWidget(
        MaterialApp(
          home: MyView(viewModel: viewModel),
        ),
      );

      final buttonFinder = find.byType(ElevatedButton);
      expect(buttonFinder, findsOneWidget);

      await tester.tap(buttonFinder);
      await tester.pump();

      expect(viewModel.counter, 1);
    });
  });
}
```

In this example, we have two unit tests for a `MyView` widget. The first test, `'UI should display the correct text'`, verifies that the view correctly displays the expected text, "Hello, World!". It creates an instance of the associated view model, `MyViewModel`, and uses the `pumpWidget` function to build and render the view. The `find.text` function is then used to locate the text widget with the expected text, and the `expect` function is used to ensure it is found.

The second test, `'Button should trigger the correct action'`, tests the behavior of a button in the view. It again creates an instance of the `MyViewModel` and uses `pumpWidget` to build and render the view. The `find.byType` function is used to locate the button widget, and `tester.tap` is called to simulate tapping the button. After pumping the widget again, the `expect` function verifies that the `counter` property of the view model has been incremented.

To run these tests, you can use the `flutter test` command in the terminal or run the tests from your IDE. If all the tests pass, it confirms that the view behaves as expected. If any test fails, it indicates that there might be an issue with the view's UI or interaction with the associated view model.


# CRUD MVVM FLUTTER WITH API CALL

Sure! Here's an example of how you can create a simple CRUD (Create, Read, Update, Delete) implementation in the MVVM (Model-View-ViewModel) architecture using the `http` package in Flutter:

**lib/model/user.dart**
```dart
class User {
  final int id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}
```

**lib/viewmodel/user_viewmodel.dart**
```dart
import 'dart:convert';
import 'package:flutter/foundation.dart';
import 'package:http/http.dart' as http;
import 'package:your_app_name/model/user.dart';

class UserViewModel extends ChangeNotifier {
  List<User> _users = [];
  List<User> get users => _users;

  Future<void> fetchUsers() async {
    final response = await http.get(Uri.parse('https://api.example.com/users'));
    if (response.statusCode == 200) {
      final List<dynamic> jsonResponse = json.decode(response.body);
      _users = jsonResponse.map((data) => User.fromJson(data)).toList();
      notifyListeners();
    } else {
      throw Exception('Failed to fetch users');
    }
  }

  Future<void> addUser(User user) async {
    final response = await http.post(
      Uri.parse('https://api.example.com/users'),
      body: json.encode(user.toJson()),
      headers: {'Content-Type': 'application/json'},
    );
    if (response.statusCode == 201) {
      final newUser = User.fromJson(json.decode(response.body));
      _users.add(newUser);
      notifyListeners();
    } else {
      throw Exception('Failed to add user');
    }
  }

  Future<void> updateUser(User user) async {
    final response = await http.put(
      Uri.parse('https://api.example.com/users/${user.id}'),
      body: json.encode(user.toJson()),
      headers: {'Content-Type': 'application/json'},
    );
    if (response.statusCode == 200) {
      final updatedUser = User.fromJson(json.decode(response.body));
      final index = _users.indexWhere((u) => u.id == updatedUser.id);
      if (index != -1) {
        _users[index] = updatedUser;
        notifyListeners();
      }
    } else {
      throw Exception('Failed to update user');
    }
  }

  Future<void> deleteUser(int userId) async {
    final response = await http.delete(
      Uri.parse('https://api.example.com/users/$userId'),
    );
    if (response.statusCode == 200) {
      _users.removeWhere((user) => user.id == userId);
      notifyListeners();
    } else {
      throw Exception('Failed to delete user');
    }
  }
}
```

In this example, we have a `User` model class representing user data. The `UserViewModel` class handles the CRUD operations by making HTTP requests to an API.

The `fetchUsers` method fetches the list of users from the API using an HTTP GET request. It parses the JSON response into a list of `User` objects and updates the `_users` list in the view model.

The `addUser` method adds a new user to the API by sending an HTTP POST request. It encodes the user data as JSON and sends it in the request body. If the request is successful (status code 201), the newly created user

 is added to the `_users` list in the view model.

The `updateUser` method updates an existing user in the API by sending an HTTP PUT request. It encodes the updated user data as JSON and sends it in the request body. If the request is successful (status code 200), the corresponding user in the `_users` list is updated.

The `deleteUser` method deletes a user from the API by sending an HTTP DELETE request. If the request is successful (status code 200), the corresponding user is removed from the `_users` list.

Note: Replace `'https://api.example.com/users'` with the actual API endpoint for your CRUD operations.

You can use the `UserViewModel` class in your Flutter views to perform CRUD operations on users and update the UI accordingly.

Certainly! Here's an example of how you can create a view in Flutter that interacts with the `UserViewModel` to perform CRUD operations:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:your_app_name/model/user.dart';
import 'package:your_app_name/viewmodel/user_viewmodel.dart';

class UserListView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userViewModel = Provider.of<UserViewModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('User List'),
      ),
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () {
              _showAddUserDialog(context, userViewModel);
            },
            child: Text('Add User'),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: userViewModel.users.length,
              itemBuilder: (context, index) {
                final user = userViewModel.users[index];
                return ListTile(
                  title: Text(user.name),
                  subtitle: Text(user.email),
                  trailing: IconButton(
                    icon: Icon(Icons.delete),
                    onPressed: () {
                      userViewModel.deleteUser(user.id);
                    },
                  ),
                  onTap: () {
                    _showEditUserDialog(context, userViewModel, user);
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  void _showAddUserDialog(BuildContext context, UserViewModel userViewModel) {
    final nameController = TextEditingController();
    final emailController = TextEditingController();

    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text('Add User'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: nameController,
                decoration: InputDecoration(labelText: 'Name'),
              ),
              TextField(
                controller: emailController,
                decoration: InputDecoration(labelText: 'Email'),
              ),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: Text('Cancel'),
            ),
            TextButton(
              onPressed: () {
                final newUser = User(
                  id: DateTime.now().millisecondsSinceEpoch,
                  name: nameController.text,
                  email: emailController.text,
                );
                userViewModel.addUser(newUser);
                Navigator.pop(context);
              },
              child: Text('Add'),
            ),
          ],
        );
      },
    );
  }

  void _showEditUserDialog(
    BuildContext context,
    UserViewModel userViewModel,
    User user,
  ) {
    final nameController = TextEditingController(text: user.name);
    final emailController = TextEditingController(text: user.email);

    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text('Edit User'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: nameController,
                decoration: InputDecoration(labelText: 'Name'),
              ),
              TextField(
                controller: emailController,
                decoration: InputDecoration(labelText: 'Email'),
              ),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: Text('Cancel'),
            ),
            TextButton(
              onPressed: () {
                final updatedUser = User(
                  id: user.id,
                  name: nameController.text,
                  email: emailController.text,
                );
                userViewModel.updateUser(updatedUser);
                Navigator.pop(context);
              },
              child: Text('Save'),
            ),
          ],
        );
      },
    );
  }
}
```

In this example, the `UserListView` is a `StatelessWidget` that displays a list of users retrieved from

 the `UserViewModel`. It also provides options to add a new user, delete a user, and edit existing user details.

The `ElevatedButton` at the top triggers the `_showAddUserDialog` function, which shows an `AlertDialog` for adding a new user. The user input is collected using `TextEditingController`, and when the "Add" button is pressed, a new `User` object is created with the entered details and added to the `UserViewModel` using `userViewModel.addUser`.

Each `User` in the `ListView` is displayed as a `ListTile`, including the name, email, and a delete button. When the delete button is pressed, the corresponding user is deleted using `userViewModel.deleteUser`. Tapping on a user's tile triggers the `_showEditUserDialog` function, which shows an `AlertDialog` pre-filled with the existing user details. When the "Save" button is pressed, the updated user details are collected, and the `UserViewModel` is updated using `userViewModel.updateUser`.

Remember to wrap your application's root widget with a `ChangeNotifierProvider` for the `UserViewModel` to make it accessible to the view:

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => UserViewModel(),
      child: MyApp(),
    ),
  );
}
```

This example demonstrates a basic implementation of the CRUD operations in a Flutter view using the MVVM architecture with the `http` package. You can modify and extend it based on your specific requirements.

- UNIT TEST FOR USER MODEL 

Certainly! Here's an example of how you can write a unit test for the `User` model class:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/model/user.dart';

void main() {
  test('User.fromJson should parse JSON correctly', () {
    final json = {
      'id': 1,
      'name': 'John Doe',
      'email': 'john.doe@example.com',
    };

    final user = User.fromJson(json);

    expect(user.id, 1);
    expect(user.name, 'John Doe');
    expect(user.email, 'john.doe@example.com');
  });

  test('User.toJson should convert to JSON correctly', () {
    final user = User(
      id: 1,
      name: 'John Doe',
      email: 'john.doe@example.com',
    );

    final json = user.toJson();

    expect(json['id'], 1);
    expect(json['name'], 'John Doe');
    expect(json['email'], 'john.doe@example.com');
  });
}
```

In this unit test, we have two test cases. The first test case, `'User.fromJson should parse JSON correctly'`, ensures that the `User.fromJson` method correctly parses the provided JSON data into a `User` object. We create a sample JSON object, call `User.fromJson` with it, and assert that the resulting `User` object has the expected values.

The second test case, `'User.toJson should convert to JSON correctly'`, verifies that the `User.toJson` method correctly converts a `User` object into a JSON representation. We create a sample `User` object, call `toJson` on it, and assert that the resulting JSON object has the expected key-value pairs.

To run this unit test, you can use the `flutter test` command in the terminal or run the test from your IDE. If the test passes, it confirms that the `User` model class behaves as expected. If the test fails, it indicates that there might be an issue with the `User` model's `fromJson` or `toJson` implementation.

- UNIT TEST FOR USER VIEW MODEL
Certainly! Here's an example of how you can write a unit test for the `UserViewModel` class:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:mockito/mockito.dart';
import 'package:your_app_name/model/user.dart';
import 'package:your_app_name/viewmodel/user_viewmodel.dart';

class MockHttpClient extends Mock implements http.Client {}

void main() {
  group('UserViewModel', () {
    late UserViewModel userViewModel;
    late MockHttpClient mockHttpClient;

    setUp(() {
      mockHttpClient = MockHttpClient();
      userViewModel = UserViewModel(httpClient: mockHttpClient);
    });

    test('fetchUsers should populate users list', () async {
      final jsonResponse = [
        {'id': 1, 'name': 'John Doe', 'email': 'john.doe@example.com'},
        {'id': 2, 'name': 'Jane Smith', 'email': 'jane.smith@example.com'},
      ];

      when(mockHttpClient.get(Uri.parse('https://api.example.com/users')))
          .thenAnswer((_) async => http.Response(jsonResponse.toString(), 200));

      await userViewModel.fetchUsers();

      expect(userViewModel.users.length, 2);
      expect(userViewModel.users[0].id, 1);
      expect(userViewModel.users[0].name, 'John Doe');
      expect(userViewModel.users[0].email, 'john.doe@example.com');
      expect(userViewModel.users[1].id, 2);
      expect(userViewModel.users[1].name, 'Jane Smith');
      expect(userViewModel.users[1].email, 'jane.smith@example.com');
    });

    test('addUser should add user to users list', () async {
      final newUser = User(id: 1, name: 'John Doe', email: 'john.doe@example.com');

      when(mockHttpClient.post(
        Uri.parse('https://api.example.com/users'),
        body: anyNamed('body'),
        headers: anyNamed('headers'),
      )).thenAnswer((_) async => http.Response(newUser.toJson().toString(), 201));

      await userViewModel.addUser(newUser);

      expect(userViewModel.users.length, 1);
      expect(userViewModel.users[0].id, 1);
      expect(userViewModel.users[0].name, 'John Doe');
      expect(userViewModel.users[0].email, 'john.doe@example.com');
    });

    test('updateUser should update user in users list', () async {
      final existingUser = User(id: 1, name: 'John Doe', email: 'john.doe@example.com');
      final updatedUser = User(id: 1, name: 'John Smith', email: 'john.smith@example.com');

      userViewModel.users = [existingUser];

      when(mockHttpClient.put(
        Uri.parse('https://api.example.com/users/${existingUser.id}'),
        body: anyNamed('body'),
        headers: anyNamed('headers'),
      )).thenAnswer((_) async => http.Response(updatedUser.toJson().toString(), 200));

      await userViewModel.updateUser(updatedUser);

      expect(userViewModel.users.length, 1);
      expect(userViewModel.users[0].id, 1);
      expect(userViewModel.users[0].name, 'John Smith');
      expect(userViewModel.users[0].email, 'john.smith@example.com');
    });

    test('deleteUser should remove user from users list', () async {
      final userId = 1;
      final existingUser = User(id: userId, name: 'John Doe', email: 'john.doe@example.com');

      userViewModel.users = [existingUser];

      when(mockHttpClient.delete(
        Uri

.parse('https://api.example.com/users/$userId'),
        headers: anyNamed('headers'),
      )).thenAnswer((_) async => http.Response('', 200));

      await userViewModel.deleteUser(userId);

      expect(userViewModel.users.length, 0);
    });
  });
}
```

In this unit test, we use the `flutter_test` package and `mockito` package for mocking the `http.Client` dependency. We create a `MockHttpClient` class that extends `Mock` from `mockito` to simulate the behavior of the HTTP client.

The unit test focuses on testing the different methods of the `UserViewModel` class:

- In the `'fetchUsers should populate users list'` test case, we mock an HTTP GET request to return a JSON response containing a list of users. We expect the `fetchUsers` method of the `UserViewModel` to populate the `users` list correctly based on the received data.

- The `'addUser should add user to users list'` test case simulates an HTTP POST request to add a new user. We mock the POST request and expect the `addUser` method to add the user to the `users` list correctly.

- The `'updateUser should update user in users list'` test case simulates an HTTP PUT request to update an existing user. We provide an initial user in the `users` list, mock the PUT request, and expect the `updateUser` method to update the user in the `users` list correctly.

- The `'deleteUser should remove user from users list'` test case simulates an HTTP DELETE request to remove a user. We provide an initial user in the `users` list, mock the DELETE request, and expect the `deleteUser` method to remove the user from the `users` list correctly.

By running this unit test, you can ensure that the `UserViewModel` class performs the CRUD operations as expected and interacts with the `http.Client` correctly.

- UNIT TEST FOR VIEW 

When it comes to testing Flutter views, the most common approach is to use widget testing. Widget testing allows you to verify the behavior and rendering of your UI components. Here's an example of how you can write a unit test for a Flutter view:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/view/user_list_view.dart';
import 'package:your_app_name/viewmodel/user_viewmodel.dart';
import 'package:provider/provider.dart';

void main() {
  testWidgets('UserListView should display users correctly', (WidgetTester tester) async {
    final userViewModel = UserViewModel();
    userViewModel.users = [
      User(id: 1, name: 'John Doe', email: 'john.doe@example.com'),
      User(id: 2, name: 'Jane Smith', email: 'jane.smith@example.com'),
    ];

    await tester.pumpWidget(
      ChangeNotifierProvider<UserViewModel>(
        create: (context) => userViewModel,
        child: MaterialApp(
          home: UserListView(),
        ),
      ),
    );

    // Verify the AppBar title
    expect(find.text('User List'), findsOneWidget);

    // Verify the "Add User" button
    expect(find.text('Add User'), findsOneWidget);

    // Verify the user list items
    expect(find.text('John Doe'), findsOneWidget);
    expect(find.text('john.doe@example.com'), findsOneWidget);
    expect(find.text('Jane Smith'), findsOneWidget);
    expect(find.text('jane.smith@example.com'), findsOneWidget);
  });
}
```

In this unit test, we use the `flutter_test` package to perform widget testing. The test case `'UserListView should display users correctly'` verifies the correct display of users in the `UserListView`. 

We create a `UserViewModel` instance and populate the `users` list with sample user data. Then, we use the `tester.pumpWidget` method to build the `UserListView` within a `ChangeNotifierProvider` that provides the `UserViewModel` instance.

Finally, we use various `expect` statements to verify the presence of UI elements such as the AppBar title, "Add User" button, and the list of user names and emails.

Running this unit test will validate that the `UserListView` renders the users correctly based on the provided `UserViewModel`. It helps ensure that the UI components are structured and displayed as expected.
