# Flutter CRUD operation using RTBD

MVVM (Model-View-ViewModel) is an architectural pattern commonly used in software development to separate concerns and improve the maintainability of code. Flutter, being a cross-platform framework, can also adopt the MVVM pattern for structuring the codebase and implementing CRUD (Create, Read, Update, Delete) operations on a Real-Time Database (RTDB).

Here's a high-level overview of how you can implement MVVM with CRUD operations on an RTDB in Flutter:

1. Model: The model represents the data structure or entities in your application. In this case, it could be the data objects you want to store and retrieve from the RTDB. You can define a Dart class that represents your data model.

```dart
class MyDataModel {
  String id;
  String name;
  int age;

  MyDataModel({required this.id, required this.name, required this.age});
}
```

2. View: The view represents the user interface (UI) of your application. In Flutter, it could be a widget tree that displays the data and provides user interaction. Create the necessary widgets to display the data and capture user input for CRUD operations.

3. ViewModel: The view model acts as a bridge between the view and the model. It contains the business logic for retrieving, manipulating, and persisting data. In this case, it will interact with the RTDB. Create a Dart class for the view model.

```dart
import 'package:firebase_database/firebase_database.dart';

class MyViewModel {
  final DatabaseReference _databaseReference =
      FirebaseDatabase.instance.reference().child('myData');

  Future<List<MyDataModel>> fetchData() async {
    DataSnapshot dataSnapshot = await _databaseReference.once();
    List<MyDataModel> dataList = [];

    if (dataSnapshot.value != null) {
      dataSnapshot.value.forEach((key, value) {
        dataList.add(
          MyDataModel(
            id: key,
            name: value['name'],
            age: value['age'],
          ),
        );
      });
    }

    return dataList;
  }

  Future<void> createData(MyDataModel data) async {
    await _databaseReference.push().set({
      'name': data.name,
      'age': data.age,
    });
  }

  Future<void> updateData(MyDataModel data) async {
    await _databaseReference.child(data.id).update({
      'name': data.name,
      'age': data.age,
    });
  }

  Future<void> deleteData(String id) async {
    await _databaseReference.child(id).remove();
  }
}
```

4. Connect the ViewModel to the View: In the Flutter UI layer, you can use a state management solution like Provider or Riverpod to connect the view model to the view. These packages provide a way to manage and propagate state changes.

Here's a simplified example using Provider:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class MyView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final myViewModel = Provider.of<MyViewModel>(context, listen: false);

    return Scaffold(
      appBar: AppBar(
        title: Text('My View'),
      ),
      body: FutureBuilder<List<MyDataModel>>(
        future: myViewModel.fetchData(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }

          if (snapshot.hasError) {
            return Center(child: Text('Error occurred'));
          }

          List<MyDataModel> dataList = snapshot.data ?? [];

          return ListView.builder(
            itemCount: dataList.length,
            itemBuilder: (context, index) {
              MyDataModel data = dataList[index];

              return ListTile(
                title: Text

(data.name),
                subtitle: Text(data.age.toString()),
              );
            },
          );
        },
      ),
    );
  }
}
```

5. Dependency Injection: To provide an instance of the view model to the view, you need to set up dependency injection. You can use the `Provider` class from the `provider` package to achieve this. Wrap the root widget of your app with the `ChangeNotifierProvider` and provide an instance of your view model.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => MyViewModel(),
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App',
      home: MyView(),
    );
  }
}
```

That's a basic implementation of MVVM with CRUD operations on an RTDB in Flutter. Remember to set up your Firebase project, including authentication and Real-Time Database configuration, before using it in your Flutter app.


## Unit Test of View Model

Sure! Here's an example of how you can write unit tests for each CRUD operation in the `MyViewModel` class using the `flutter_test` package:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/my_view_model.dart';
import 'package:my_app/my_data_model.dart';

void main() {
  group('MyViewModel', () {
    late MyViewModel myViewModel;

    setUp(() {
      myViewModel = MyViewModel();
    });

    test('Fetch data should return a list of MyDataModel', () async {
      List<MyDataModel> dataList = await myViewModel.fetchData();
      expect(dataList, isA<List<MyDataModel>>());
    });

    test('Create data should add a new data entry', () async {
      MyDataModel newData = MyDataModel(id: '1', name: 'John', age: 25);
      await myViewModel.createData(newData);

      List<MyDataModel> dataList = await myViewModel.fetchData();
      expect(dataList.length, 1);
      expect(dataList[0].id, '1');
      expect(dataList[0].name, 'John');
      expect(dataList[0].age, 25);
    });

    test('Update data should modify an existing data entry', () async {
      MyDataModel existingData = MyDataModel(id: '1', name: 'John', age: 25);
      await myViewModel.createData(existingData);

      MyDataModel updatedData = MyDataModel(id: '1', name: 'Jane', age: 30);
      await myViewModel.updateData(updatedData);

      List<MyDataModel> dataList = await myViewModel.fetchData();
      expect(dataList.length, 1);
      expect(dataList[0].id, '1');
      expect(dataList[0].name, 'Jane');
      expect(dataList[0].age, 30);
    });

    test('Delete data should remove an existing data entry', () async {
      MyDataModel existingData = MyDataModel(id: '1', name: 'John', age: 25);
      await myViewModel.createData(existingData);

      await myViewModel.deleteData('1');

      List<MyDataModel> dataList = await myViewModel.fetchData();
      expect(dataList.length, 0);
    });
  });
}
```

In this example, we use the `group` function to group related tests together. The `setUp` function is used to set up the necessary dependencies before each test case. Each test case is written using the `test` function, which asserts the expected behavior of the corresponding CRUD operation.

Make sure to import the necessary dependencies (`flutter_test`, `my_view_model.dart`, and `my_data_model.dart`) and adjust the imports based on your project structure.

You can run the unit tests using the `flutter test` command or by running the tests from your IDE.


## Unit Test of View

Certainly! Here's an example of how you can write a unit test for the `MyView` widget in Flutter using the `flutter_test` package:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/my_view.dart';
import 'package:my_app/my_view_model.dart';
import 'package:provider/provider.dart';

void main() {
  group('MyView', () {
    testWidgets('Should display data from view model', (WidgetTester tester) async {
      // Create a mock view model with sample data
      MyViewModel mockViewModel = MockViewModel();
      mockViewModel.mockDataList = [
        MyDataModel(id: '1', name: 'John', age: 25),
        MyDataModel(id: '2', name: 'Jane', age: 30),
      ];

      // Build the MyView widget and provide the mock view model
      await tester.pumpWidget(
        ChangeNotifierProvider<MyViewModel>.value(
          value: mockViewModel,
          child: MaterialApp(
            home: MyView(),
          ),
        ),
      );

      // Verify that the data is displayed correctly
      expect(find.text('John'), findsOneWidget);
      expect(find.text('Jane'), findsOneWidget);
      expect(find.text('25'), findsOneWidget);
      expect(find.text('30'), findsOneWidget);
    });
  });
}

// MockViewModel class for testing
class MockViewModel extends MyViewModel {
  List<MyDataModel> mockDataList = [];

  @override
  Future<List<MyDataModel>> fetchData() async {
    return mockDataList;
  }
}
```

In this example, we use the `testWidgets` function to write a widget test case. Inside the test case, we create a mock view model (`MockViewModel`) that extends the `MyViewModel` class and overrides the `fetchData` method to return a predefined list of `MyDataModel` objects.

We then use the `pumpWidget` function to build the `MyView` widget and provide the `MockViewModel` as the dependency using `ChangeNotifierProvider`. Finally, we use the `expect` function to verify that the data from the view model is correctly displayed in the widget.

Make sure to import the necessary dependencies (`flutter/material.dart`, `flutter_test/flutter_test.dart`, `my_view.dart`, `my_view_model.dart`, and `provider/provider.dart`) and adjust the imports based on your project structure.

You can run the unit test using the `flutter test` command or by running the tests from your IDE.
