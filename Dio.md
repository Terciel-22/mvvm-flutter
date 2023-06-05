## MVVM DIO

MVVM (Model-View-ViewModel) is an architectural pattern that helps separate concerns and improve the maintainability of code. When working with Flutter, you can use the DIO package for making HTTP requests and implement MVVM to structure your code. Here's an example of implementing MVVM using DIO in Flutter:

1. Model: The model represents the data structure or entities in your application. It could be a Dart class that defines the structure of the data you'll be working with.

```dart
class MyDataModel {
  String id;
  String name;
  int age;

  MyDataModel({required this.id, required this.name, required this.age});
}
```

2. View: The view represents the user interface (UI) of your application. It could be a widget tree that displays the data and captures user input. Create the necessary widgets to present the data to the user.

3. ViewModel: The view model acts as a bridge between the view and the model. It contains the business logic for retrieving and processing data from an API using DIO. Create a Dart class for the view model.

```dart
import 'package:dio/dio.dart';
import 'package:your_app/my_data_model.dart';

class MyViewModel {
  final Dio _dio = Dio();

  Future<List<MyDataModel>> fetchData() async {
    try {
      final response = await _dio.get('https://api.example.com/data');
      List<MyDataModel> dataList = [];

      if (response.statusCode == 200) {
        final data = response.data;

        data.forEach((item) {
          dataList.add(
            MyDataModel(
              id: item['id'],
              name: item['name'],
              age: item['age'],
            ),
          );
        });
      }

      return dataList;
    } catch (e) {
      // Handle error
      return [];
    }
  }
}
```

In this example, we use the `Dio` package for making HTTP requests. The `fetchData` method sends a GET request to the specified API endpoint and processes the response to create a list of `MyDataModel` objects.

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
                title: Text(data.name),
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

5. Dependency Injection: To provide an instance of the view model to the view, you need to set up dependency injection. Wrap the root widget of your app with the `ChangeNotifierProvider` and provide an instance of your view model.

```dart
import 'package:flutter/material.dart';
import

 'package:provider/provider.dart';

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

That's a basic implementation of MVVM using DIO in Flutter. Remember to add the necessary dependencies (`dio` and `provider`) to your `pubspec.yaml` file and run `flutter pub get` to download them.

Feel free to adapt the code based on your specific requirements and adjust the API endpoints and response structures accordingly.


## CRUD

Certainly! Here's an example of how you can extend the previous MVVM implementation with DIO in Flutter to include CRUD (Create, Read, Update, Delete) operations:

1. Modify the `MyViewModel` class to include CRUD operations:

```dart
import 'package:dio/dio.dart';
import 'package:your_app/my_data_model.dart';

class MyViewModel {
  final Dio _dio = Dio();
  final String baseUrl = 'https://api.example.com/data';

  Future<List<MyDataModel>> fetchData() async {
    try {
      final response = await _dio.get(baseUrl);
      List<MyDataModel> dataList = [];

      if (response.statusCode == 200) {
        final data = response.data;

        data.forEach((item) {
          dataList.add(
            MyDataModel(
              id: item['id'],
              name: item['name'],
              age: item['age'],
            ),
          );
        });
      }

      return dataList;
    } catch (e) {
      // Handle error
      return [];
    }
  }

  Future<void> createData(MyDataModel data) async {
    try {
      await _dio.post(baseUrl, data: {
        'name': data.name,
        'age': data.age,
      });
    } catch (e) {
      // Handle error
    }
  }

  Future<void> updateData(MyDataModel data) async {
    try {
      await _dio.put('$baseUrl/${data.id}', data: {
        'name': data.name,
        'age': data.age,
      });
    } catch (e) {
      // Handle error
    }
  }

  Future<void> deleteData(String id) async {
    try {
      await _dio.delete('$baseUrl/$id');
    } catch (e) {
      // Handle error
    }
  }
}
```

2. Update the `MyView` widget to include buttons for creating, updating, and deleting data:

```dart
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
                title: Text(data.name),
                subtitle: Text(data.age.toString()),
              );
            },
          );
        },
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: () {
              MyDataModel newData = MyDataModel(
                id: 'newId',
                name: 'New Data',
                age: 25,
              );
              myViewModel.createData(newData);
            },
            child: Icon(Icons.add),
          ),
          SizedBox(height: 16.0),
          FloatingActionButton(
            onPressed: () {
              MyDataModel updatedData = MyDataModel(
                id: 'existingId',
                name: 'Updated Data',
                age: 30,
              );
              myViewModel.updateData(updatedData);
            },
            child: Icon(Icons.edit),
          ),
          SizedBox(height: 16.0),
          FloatingActionButton(
            onPressed: () {
              String dataId = 'existingId';
              myViewModel.deleteData

(dataId);
            },
            child: Icon(Icons.delete),
          ),
        ],
      ),
    );
  }
}
```

In this updated example, we've added `createData`, `updateData`, and `deleteData` methods to the `MyViewModel` class. These methods use the DIO package to send HTTP requests for creating, updating, and deleting data.

We've also added floating action buttons to the `MyView` widget that trigger the corresponding CRUD operations when pressed.

Make sure to import the necessary dependencies (`dio` and `provider`) in your file.

Remember to adjust the `baseUrl` and the data structures (`MyDataModel`) to match your API and application requirements.

These additions allow you to perform CRUD operations using DIO in your MVVM architecture in Flutter.
