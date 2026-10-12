Wikipedia Reader
================

An app that fetches and displays article summaries from the Wikipedia API.

Implemented as part of the [Official Flutter Learning Pathway.](https://docs.flutter.dev/learn/pathway/tutorial/set-up-state-project)

The following are notes I took when covering these lessons.

Lessons Covered
---------------

The following lessons have been covered:

-   Making HTTP requests in Flutter.
-   Managing application state with ChangeNotifier.
-   Using the MVVM architecture pattern.
-   Rebuild UI when state changes

Making HTTP requests in Flutter.
--------------------------------

The overarching pattern that this tutorial implements is called Model-View-ViewModel or MVVM. MVVM is an architectural pattern used in client apps that separates your app into three layers:

-   Model: Handles data operations.
-   View: Displays the UI.
-   ViewModel: Manages state and connects the two.

Wikipedia provides a REST API that returns JSON data about articles. 

<https://en.wikipedia.org/api/rest_v1/page/random/summary>

The function getRandomArticleSummary() within the ArticleModel will make the initial request to the API, and return the response (or exception).

For HTTP requests, we use async and await keywords, and return a 'Future', which is essentially a promise that something will be returned.

The Uri.https constructor safely builds URLs by handling encoding and formatting. This approach is more reliable than string concatenation, especially when dealing with special characters or query parameters.

We then need to decode the returned JSON response from the Wikipedia API, and convert it into our own custom object, Summary.

Managing application state with ChangeNotifier.
-----------------------------------------------

When developers talk about state-management in Flutter, they're essentially referring to the pattern by which your app updates the data it needs to render correctly and then tells Flutter to re-render the UI with that new data.

In MVVM, this responsibility falls to the ViewModel layer, which sits between and connects your UI to your Model layer. In Flutter, ViewModels use Flutter's ChangeNotifier class to notify the UI when data changes.

To use ChangeNotifier, extend it in your state management class to gain access to the notifyListeners() method, which triggers UI rebuilds when called.

Therefore, we need to create an ArticleViewModel that extends ChangeNotifier.

Then we need to add the following properties to

1.  Link the Model
2.  Define the object we are handling (Summary)
3.  Handle errors

final ArticleModel model;

 Summary? summary;

 Exception? error;

We want to update the constructor, so that the API request is automatically sent when the ArticleViewModel is created:

  ArticleViewModel(this.model) {

    fetchArticle();

  }


  Future<void> fetchArticle() async {

summary = await model.getRandomArticleSummary();

notifyListeners();

}

Constructors cannot be asynchronous, hence why we need fetchArticle as a separate function. notifyListeners() will inform the UI of the update.

Rebuild UI when state changes
-----------------------------

The view layer is your UI, and in Flutter, that refers to your app's widgets. As it pertains to this tutorial, the important part is wiring up your UI to respond to data changes from the ViewModel. ListenableBuilder is a widget that can "listen" to a ChangeNotifier, and automatically rebuilds when it's provided ChangeNotifier calls notifyListeners().

So we implement a StatefulWidget, called the ArticleView. (extends StatefulWidget).

Then we instantiate it with the initState() function, where we will call viewModel.fetchArticle(), thus bringing the View, ViewModel and Model together.

In this case, we set this view up as the startup page by overriding the build function of the MainApp class.

  @override

  Widget build(BuildContext context) {

return const MaterialApp(home: ArticleView());

  }

We then wrap the UI components in a ListenableBuilder, to listen for state changes and pass them a ChangeNotifier object. In this case, the ArticleViewModel extends ChangeNotifier.

body: ListenableBuilder(

        listenable: viewModel,

        builder: (context, child) {

          return const Center(child: Text('Loading...'));

        },

      ),

In order to handle the possible view model states, we must recall the ArticleViewModel, which has three properties that the UI is interested in:

-   Summary? summary
-   bool isLoading
-   Exception? Error

Depending on the combined state of these properties, the UI can display different widgets. So we use Dart's support for switch expressions to handle all possible combinations in a clean, readable way. (see ArticleView)

This is an excellent example of how a declarative, reactive framework like Flutter and a pattern like MVVM work together: The UI is rendered based on the state and updates when a state changes demands it, but it doesn't manage any state or the process of updating itself. The business logic and rendering are completely separate from each other.
