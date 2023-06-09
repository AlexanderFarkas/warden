# Getting Started
1. Define Store
    ```dart
   // import warden 
   import "package:warden/warden.dart"; 
   
    class CounterStore extends Store {
      // define observable using `observable` member function.
      late final counter = observable(0);
      increment() => counter.value++;
    }
    ```
2. Watch value with `Warden`:
    ```dart
   final vm = CounterStore();
   
   // ...
   
   Widget build(BuildContext context) {
      return Warden(
        builder: (context, watch) =>  OutlinedButton(
          onPressed: vm.increment,
          child: Text("${watch(vm.counter)}")
        ),
      );
   }
    ```

# Computed
Use `computed` to derive from `observable`s:

```dart
class CounterStore extends Store {
  late final counter = observable(0);
  late final counterSquared = computed((watch) => watch(counter) * watch(counter));
}
```

# Dispose
Every class extending `Store` has `dispose` method. Call it once you don't need the store anymore:
```dart
myCounterStore.dispose();
```

# Stream
Every `observable`/`computed` could be converted to a stream.

Consider a complex Github Api example using `rxdart`:
```dart
class RepositoryListStore extends Store {
   RepositoryListStore(this.githubApi) {
    final subscription = search
        .asStream()
        .doOnData((value) {
          items.value = value.isEmpty ? const Success(SearchResult(items: [])) : const Loading();
        })
        .debounceTime(const Duration(milliseconds: 500))
        .where((value) => value.isNotEmpty)
        .switchMap<AsyncValue<SearchResult>>((value) async* {
          if (value != search.value) return;
          yield await Result.guard(() => githubApi.searchRepositories(value));
        })
        .listen((value) => items.value = value);

    onDispose.add(subscription.cancel);
  }

  final GithubApi githubApi;

  late final items = observable<AsyncValue<SearchResult>>(const Success(SearchResult(items: [])));
  late final search = observable('');
}

```