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
