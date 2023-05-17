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
