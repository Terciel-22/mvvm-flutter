To create a `MultipleChangeNotifier` class in Flutter, you can extend the `ChangeNotifier` class and add multiple properties that extend `ChangeNotifier` as well. Here's an example:

```dart

import 'package:flutter/foundation.dart';

class Counter extends ChangeNotifier {

  int _count = 0;

  

  int get count => _count;

  

  void increment() {

    _count++;

    notifyListeners();

  }

}

class Message extends ChangeNotifier {

  String _text = '';

  

  String get text => _text;

  

  void updateText(String newText) {

    _text = newText;

    notifyListeners();

  }

}

class MultipleChangeNotifier extends ChangeNotifier {

  Counter _counter = Counter();

  Message _message = Message();

  

  Counter get counter => _counter;

  Message get message => _message;

}

```

In the example above, we have a `Counter` class and a `Message` class, both extending `ChangeNotifier`. Each class has its own properties and methods specific to its functionality. 

The `MultipleChangeNotifier` class is the main class that aggregates the `Counter` and `Message` classes as properties. It acts as a container for multiple `ChangeNotifier` instances. 

By extending `ChangeNotifier`, the `MultipleChangeNotifier` class gains the ability to listen for changes and notify its listeners when any of its properties (the `Counter` or `Message` instances) change.

To use the `MultipleChangeNotifier` in a widget, you can wrap it with a `ChangeNotifierProvider` or `Provider` from the `provider` package. Here's an example:

```dart

import 'package:flutter/material.dart';

import 'package:provider/provider.dart';

class MyWidget extends StatelessWidget {

  @override

  Widget build(BuildContext context) {

    final multipleNotifier = Provider.of<MultipleChangeNotifier>(context);

    final counter = multipleNotifier.counter;

    final message = multipleNotifier.message;

    return Scaffold(

      appBar: AppBar(

        title: Text('Multiple ChangeNotifier'),

      ),

      body: Column(

        mainAxisAlignment: MainAxisAlignment.center,

        children: [

          Text('Counter: ${counter.count}'),

          ElevatedButton(

            onPressed: () => counter.increment(),

            child: Text('Increment'),

          ),

          SizedBox(height: 16),

          Text('Message: ${message.text}'),

          TextField(

            onChanged: (value) => message.updateText(value),

            decoration: InputDecoration(

              labelText: 'Enter Message',

            ),

          ),

        ],

      ),

    );

  }

}

void main() {

  runApp(

    ChangeNotifierProvider(

      create: (_) => MultipleChangeNotifier(),

      child: MyApp(),

    ),

  );

}

```

In the example above, the `MyWidget` widget accesses the `MultipleChangeNotifier` instance using `Provider.of<MultipleChangeNotifier>(context)`. It then retrieves the `Counter` and `Message` instances from the `MultipleChangeNotifier` to use their properties and methods within the widget.

The `ChangeNotifierProvider` wraps the `MyApp` widget at the root of the widget tree, providing the `MultipleChangeNotifier` instance to all the widgets below it that need access to it.

With this setup, whenever the `increment()` method is called on the `Counter` instance or the `updateText()` method is called on the `Message` instance, the corresponding `notifyListeners()` is triggered, and the widget tree listening to the `MultipleChangeNotifier` will be updated accordingly.
