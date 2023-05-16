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

#mvvm-flutter-firebase

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
