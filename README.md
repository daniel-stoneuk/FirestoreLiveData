# FirestoreLiveData

Transforms Firebase Cloud Firestore queries, collections, and documents into `LiveData`. **Forked from [ptrbrynt/FirestoreLiveData](https://github.com/ptrbrynt/FirestoreLiveData) with support for unregistering listeners.**

Inspired by blog posts by [The Firebase Blog](https://firebase.googleblog.com/2017/12/using-android-architecture-components.html)

## TODO

Classes that need to be updated

- [ ] QueryLiveData
- [ ] DocumentLiveData
- [ ] CollectionLiveData
- [ ] FirebaseAuthLiveData

## Getting Started

First, add the JitPack repository to your **project-level** `build.gradle` file:
```gradle
allprojects {
  repositories {
    ...
    maven { url 'https://jitpack.io' }
  }
}
```

To add FirestoreLiveData to your app, add the following dependency to your module's `build.gradle` file:
```gradle
implementation 'com.github.daniel-stoneuk:FirestoreLiveData:Tag'
```

## Creating Models

The first step in transforming your Firestore queries is to create a model class. All model classes must extend `FirestoreModel` in order to work with this library:
```kotlin
data class Book(
  var title: String? = null,
  var author: String? = null,
  var timestamp: Timestamp? = null
): FirestoreModel()
```

It's also important **not** to include an 'id' property in your models, as this is included in `FirestoreModel`.

## Transforming queries, collections, and document references

You can transform any instance of `Query`, `CollectionReference`, or `DocumentReference` into a `LiveData` like so:

```kotlin
val db = FirebaseFirestore.getInstance()

val myQuery = db.collection("books").orderBy("title").asLiveData<Book>()

val myCollection = db.collection("books").asLiveData<Book>()

val myDocument = db.collection("books").document(documentId).asLiveData<Book>()
```

## Resources

The `asLiveData<T>()` methods return instances of `LiveData<FirestoreResource<T>>` for documents, and `LiveData<FirestoreResource<List<T>>>` for collections and queries.

The `FirestoreResource` class wraps the result of your query or reference and includes some additional information:

```kotlin
val myQuery = db.collection("books").orderBy("title").asLiveData<Book>()

// In your fragment or activity
myQuery.observe(this, Observer { resource: FirestoreResource<List<Book>> ->
  val status: Status = resource.status // Either SUCCESS, ERROR, or LOADING
  val data: List<Book>? = resource.data // The result of your query or reference, when status is SUCCESS
  val throwable: Throwable? = resource.throwable // If an error occurred, the details are here, when status is ERROR
  val errorMessage: String? = resource.errorMessage // A shortcut to resource.throwable?.localizedMessage
}
```

## Resource Observer

As a helpful utility, the library also includes a `ResourceObserver` class, which is an extension of `Observer` and provides easy callbacks for the three states.

You can implement a `ResourceObserver` like this:

```kotlin
myQuery.observe(this, object: ResourceObserver<List<Book>> {
  override fun onSuccess(data: List<Book>?) {
    // Handle successful result here
  }
  
  override fun onError(throwable: Throwable?, errorMessage: String?) {
    // Handle errors here
  }
  
  override fun onLoading() {
    // Handle loading state e.g. display a loading animation
  }
}
```

## Data Operations

If you have an instance of `CollectionLiveData`, you can add a new object to the collection like this:
```kotlin
val myCollection = db.collection("books").asLiveData<Book>()

myCollection.add(Book("A Book", "An Author", Timestamp.now()))

```

Similarly, you can perform the following operations on `DocumentLiveData`:

```kotlin
val myDocument = db.collection("books").document(documentId).asLiveData<Book>()

// Replaces the current data
myDocument.set(Book("Another Book", "A Superior Author", Timestamp.now())

// Update the value of a field
myDocument.update("title", "Updated Title")

// Update the values of multiple fields
myDocument.update(mapOf(Pair("title", "Updated Title"), Pair("author", "New Author")))

```

## Bonus

### `TaskResult` and `TaskLiveData`

All the operations described above return `TaskLiveData` instances. These are `LiveData`s containing `TaskResult`s, which are a bit like `FirestoreResource`s, but for `Task`s!
```kotlin
val task = myDocument.update("title", "Updated Title")

task.observe(lifecycleOwner, Observer { taskResult ->
  val status: TaskStatus = taskResult.status // Either SUCCESS, CANCELLED, FAILED, or RUNNING
  val data = taskResult.data // The data attached to the result of the task. Only present for add operations on collections, and will be the reference to the newly added item
  val exception: Exception? = taskResult.exception // If an error occurred, this exception will provide information
})
```

### Auth State LiveData

Also included is a `LiveData` wrapper for the current `FirebaseAuth` state:
```kotlin
FirebaseAuth.getInstance().currentUserLiveData.observe(lifecycleOwner, Observer {
  // Handle auth state changes here
})
```

### Observe Queries, DocumentReference, and CollectionReferences directly

Also included is a handy shortcut to translate `Query`s, `DocumentReference`s, and `CollectionReference`s into `LiveData` and observe them, as one function call!

```kotlin
val myQuery = db.collection("books").orderBy("published")
myQuery.observe<Book>(lifecycleOwner, Observer { books: FirestoreResource<List<Book>> ->
    // Here are the books!
})
```
