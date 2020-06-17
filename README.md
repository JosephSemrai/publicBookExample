

# DRAFT CONTENT

Copyright © Joseph Semrai 2020 All Rights Reserved



## Firebase

Throughout the course of this book, we've been creating our own backends to server our React apps. Sometimes, there is a need for bootstrapping a quick prototype without worrying about backend logic and infrastructure. Firebase is just a set of tools that Google offers to help developers build scalable applications using cloud infrastructure.

Among these tools include a scalable NoSQL database, a authentication suite, a machine learning toolkit, web app hosting, cloud storage, powerful analytics tools, messaging tools, and much more.

Firebase is increasingly becoming a substitute for an Express backend, with it being presented as an alternative in most cases. It is an extremely powerful tool for certain use cases and should be considered when creating new projects.

We'll first explore creating our notes application (complete with user authentication) using Firebase and then transition into looking at the power of Firebase, especially when using tools like React Native (to be discussed later).

### Firebase Configuration

In order to use Firebase, we'll need to sign up for Firebase at https://firebase.google.com using a Google account. Then, in the Firebase console (https://console.firebase.google.com), we can create a project by pressing "Add Project":

![Create Project Firebase](https://i.imgur.com/lm01So1.png)

We'll then be prompted to give a project name to which we can enter anything we want (Firebase will generate an ID for us depending on this name). There will also be a few steps to walk through afterwards, but none of them should affect anything that we're about to do.

Upon creating our project, we'll be presented with a dashboard that looks like this:

![Firebase Dashboard](https://i.imgur.com/wgyCo6F.png)

We'll want to press the `</>` web logo in order to add a web app to our FIrebase project:

![Web App](https://i.imgur.com/2qPDeKv.png)

Here, we'll be prompted to enter a nickname (does not matter). We'll also be prompted to enable Firebase Hosting. This is not required, so feel free to leave it unchecked. Upon the creation of our app, we'll be given the following code block containing the `firebaseConfig` variable which we'll need later:

![Firebase Config](https://i.imgur.com/cHaEyGN.png)

> The Firebase credentials will be invalidated upon this book's release. Please do not try to use these credentials in your application.

We won't need to add the SDK scripts into our HTML files since we'll be installing them through npm and importing them into our application.

### Using Firebase in React

We'll first use `create-react-app` to create an entirely new project:

```bash
npx create-react-app firebase-project
```

We'll also need to install `firebase`:

```bash
npm i firebase
```

We'll now get started with user creation and authentication. Firebase offers a drop in UI module that will handle all of the UI flows regarding authentication for us whether it be authentication with email/password, phone number, or OAuth/federated identity providers. This component will implement the best practices for authentication without us having to do much work.

We can install the React wrapper for FirebaseUI by running the following command:

```bash
npm i react-firebaseui
```

Now, let's create our `App` component:

**/src/App.js**

```jsx
import React from 'react'
import StyledFirebaseAuth from 'react-firebaseui/StyledFirebaseAuth'
import firebase from 'firebase/app' // Allows us to create app instances
import 'firebase/auth' // Gives us Firebase authentication support

// Firebase configuration provided to us by the app creation process
const firebaseConfig = {
  apiKey: "AIzaSyDJf0kn4MHA8mMgolqu1DLh3YqrjSAt57Q",
  authDomain: "notes-app-bfbc5.firebaseapp.com",
  databaseURL: "https://notes-app-bfbc5.firebaseio.com",
  projectId: "notes-app-bfbc5",
  storageBucket: "notes-app-bfbc5.appspot.com",
  messagingSenderId: "768088207404",
  appId: "1:768088207404:web:abd4b64d40dbab447e6189",
  measurementId: "G-NBKG2S4VX5"
};

// Initializes Firebase and creates an app instance
firebase.initializeApp(firebaseConfig)

// Configure FirebaseUI.
const uiConfig = {
  // Popup signin flow rather than redirect flow.
  signInFlow: 'popup',
  // We will display Google and Facebook as auth providers. Also configures the component with the provider IDs.
  signInOptions: [
    firebase.auth.GoogleAuthProvider.PROVIDER_ID,
    firebase.auth.FacebookAuthProvider.PROVIDER_ID
  ],
  callbacks: {
    // `signInSuccessWithAuthResult` is called with the user object when the user has successfully logged in
    signInSuccessWithAuthResult: () => false // Empty function as we do not want to do anything just yet
  }
}

const App = () => {
  return (
    <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()}/>
  )
}

export default App
```

Read the comments in the code block to get a detailed explanation of what's going on in the component.

Here, we simply imported the required Firebase packages (the base app package and the Firebase auth package to give us authentication support), created the configuration constants `firebaseConfig` and `uiConfig`, initialized Firebase with `initializeApp(firebaseConfig)`, and rendered `<StyledFirebaseAuth />` with our `uiConfig` and `firebase.auth()` method passed into it.

`firebaseConfig` simply comes from the code block that was provided to us from Google earlier. `uiConfig` simply contains a bunch of options (explained above in the comments) that our component will take in.

There are plenty of other features and options that come with `react-firebaseui`, but this knowledge should be enough for now.

Our application should look like this now:

![Application](https://i.imgur.com/rOyUkY1.png)

Before we actually manage our state with regard to handling the creation/retrieval of the user, we have to learn more about the `useEffect()` hook as well as the concept of subscriptions.

### Subscriptions and `useEffect()` Clean Up

Subscriptions, conceptually, are a concept of reactive programming. Subscriptions are connections between a subscriber and a publisher. A publisher will simply create a subscription for every single subscriber that is subscribed to it, to which requests and data are sent over this subscription. The subscriber that had "subscribed" to the publisher will react to changes in data sent down the subscription.

We will have to employ this concept when working with Firebase, as we will have to "subscribe" to the authentication state of our user in order to perform actions. For example, we will subscribe to the state and perform actions on a success, failure, etc.

Previously, we had talked about lifecycle methods in React class components. One lifecycle method was `componentWillUnmount` where we could clean up subscriptions

With effects, we can clean up subscriptions by returning a function from our effect function. If our effect returns a function, React will run it when its time to clean it up:

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    API.subscribeToStatus(props.user.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      API.unsubscribeToStatus(props.user.id, handleStatusChange);
    };
  });

  //...
```

React will "clean up" an effect when the component unmounts (as the name for the lifecycle method points to).

Our usage of these concepts with Firebase is as follows:

```jsx
useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange() // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe()
  }, [])
```

Here, we subscribe to the authentication state and deal with it in `onChange`. The `onAuthStateChanged()` function returns the unsubscribe function for the observer (subscriber). We then return a function that will call this unsubscribe function, to which React will call this function upon unmounting.

#### Managing User State Through Subscriptions

We'll now observe/subscribe the authentication state of our user to which we'll manage our state dependent on what our provider (Firebase) tells us:

**/src/App.js**

```jsx
import React, { useEffect } from 'react'

//... EXISTING CONFIGURATION

const onAuthStateChange = () => {
  return firebase.auth().onAuthStateChanged(user => {
    if (user) {
      console.log("The user is logged in.")
    } else {
      console.log("The user is not logged in.")
    }
  })
}  

const App = () => {
  
  
  useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange() // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe()
  }, [])

  return (
    <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()}/>
  )
}

export default App
```

Here, we define a function `onAuthStateChange()` which simply calls `firebase.auth().onAuthStateChanged()` internally. `firebase.auth().onAuthChanged()` takes in a callback function to which it'll pass in the `user` object upon a successful authentication. We take advantage of this fact to log to the console information about whether we have logged in or not.

`firebase.auth().onAuthStateChanged()` also returns the unsubscribe function that we need to call upon unmounting, so we'll return the value of this call in `onAuthStateChange()` so that the effect that we had described earlier can return it to be called upon unmounting.

#### Enabling Authentication Methods

Now, our initial setup for authentication is just about complete. If we try to log in using a provider right now, we'll receive the following error:





![Identity provider disabled](https://i.imgur.com/yeJtXOM.png)

This is simply because we had not enabled these identity providers in our Firebase dashboard just yet. We can visit the "Authentication" tab of the dashboard and toggle the Google sign-in method in just a few clicks.

![Authentication](https://i.imgur.com/EJerjV2.png)

> Make sure to select a project support email address

Facebook is almost as simple. As we did with Passport authentication, Firebase asks us to enter in a App ID and App secret as well as allow the redirect URL.

Now, we can attempt to sign into our application:

![Successful console](https://i.imgur.com/jGjgura.png)

Here, we see the console log that we are expecting upon logging in. The callback function that we had provided to `firebase.auth().onAuthStateChanged()` will be called every single time our user's authentication state changes (Firebase will notify our application).

Now, one change we might want to make is to store our user in our component's state and render a UI for the user when they're signed in. We'll eventually extend this UI to support creating/viewing notes using more of Firebase's features.

**/src/App.js**

```jsx
import React, { useEffect, useState } from 'react'
import StyledFirebaseAuth from 'react-firebaseui/StyledFirebaseAuth'
import firebase from 'firebase/app' // Allows us to create app instances
import 'firebase/auth' // Gives us Firebase authentication support
import NewNote from './components/NewNote'
import Notes from './components/Notes'

// Firebase configuration provided to us by the app creation process
const firebaseConfig = {
  apiKey: "AIzaSyDJf0kn4MHA8mMgolqu1DLh3YqrjSAt57Q",
  authDomain: "notes-app-bfbc5.firebaseapp.com",
  databaseURL: "https://notes-app-bfbc5.firebaseio.com",
  projectId: "notes-app-bfbc5",
  storageBucket: "notes-app-bfbc5.appspot.com",
  messagingSenderId: "768088207404",
  appId: "1:768088207404:web:abd4b64d40dbab447e6189",
  measurementId: "G-NBKG2S4VX5"
};

// Initializes Firebase and creates an app instance
firebase.initializeApp(firebaseConfig)

// Configure FirebaseUI.
const uiConfig = {
  // Popup signin flow rather than redirect flow.
  signInFlow: 'popup',
  // We will display Google and Facebook as auth providers. Also configures the component with the provider IDs.
  signInOptions: [
    firebase.auth.GoogleAuthProvider.PROVIDER_ID,
    firebase.auth.FacebookAuthProvider.PROVIDER_ID
  ],
  callbacks: {
    // `signInSuccessWithAuthResult` is called with the user object when the user has successfully logged in
    signInSuccessWithAuthResult: () => false // Empty function as we do not want to do anything just yet
  }
}

const onAuthStateChange = (setUser) => {
  return firebase.auth().onAuthStateChanged(user => {
    if (user) {
      console.log("The user is logged in.", user)
      setUser(user)
    } else {
      console.log("The user is not logged in.")
    }
  })
}

const App = () => {
  const [user, setUser] = useState()

  const handleSignOut = () => {
    firebase.auth().signOut()
    setUser()
  }

  useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange(setUser) // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe()
  }, [])

  return (
    <>
      <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()}/>

      {user ?
        <>
          <NewNote />
          <button onClick={handleSignOut}>Sign Out</button>
          {JSON.stringify(user)}
        </> : 
        <p>Please Sign In</p>}
    </>
  )
}

export default App
```

Here, we made a few changes. `onAuthStateChange()` now takes in a `setUser` parameter which is a function given to us by `useState()` to set the `user` piece of state.

If we have not set the `user` piece of state to a value other than `null`, we render a "Please Sign In" screen with the login screen.

If we are signed in, we render a stringified version of the `user` piece of state as well as a "Sign Out" button that calls `handleSignOut()` upon a click. This `handleSignOut()` event handler clears the `user` piece of state and calls `firebase.auth().signOut()`.

We keep `StyledFirebaseAuth` in the DOM despite being logged in as `firebaseUI` expects the firebaseUI widget element to always be in the DOM or else it will error.

With this, we have successfully created an application that supports authentication with Firebase and stores the user object in its state. With this, you could apply Redux as we did in previous applications, but, for the scope of this project, we'll keep everything in our component's state.

### Firebase Cloud Firestore

Now, let's start adding the ability to create and fetch notes. For this, we'll take advantage of another Firebase feature, *Cloud Firestore*.

Firebase offers two types of database solutions that support real-time data syncing (sends data between devices without the need for polling or continuous updates):

* "**Cloud Firestore** is Firebase's newest database for mobile app development. It builds on the successes of the Realtime Database with a new, more intuitive data model. Cloud Firestore also features richer, faster queries and scales further than the Realtime Database." (Firebase Official Docs)
* "**Realtime Database** is Firebase's original database. It's an efficient, low-latency solution for mobile apps that require synced states across clients in realtime." (Firebase Official Docs)

These databases come with extra features with their SDKs such as being able to store and serve changes on a local cache when offline, to which these updates are served to the cloud when the device comes online.

### Setting up Cloud Firestore

Provisioning a database with Firebase is also quite easy, just visit your dashboard to enable Cloud Firestore.

Upon visiting your dashboard, click the "Database" tab and then "Create Database".

![Image](https://i.imgur.com/N5yjdAR.png)

You'll then be asked to select a rule set for your database. As we do not have a rule set configured just yet and are not deploying this app, press "Start in **test mode**".

![Rules](https://i.imgur.com/Mb6Uf6X.png)

After this, press the "Next" button and then "Done" to finish the process.

Now, let's talk about using Cloud Firestore.

Cloud Firestore, like mongoDB, is a document database where our data is stored in documents and collections.

With Cloud Firestore, we do not need to "create" or "delete" collections, upon the creation of the first document in a collection, the collection simply "exists". If all of the documents in a collection disappear, the collection no longer exists.

#### Firestore "References"

For the following explanations, assume that we store a reference to `firebase.firestore()` in the `db` variable:

```js
var db = firebase.firestore()
```

Every single document in Cloud Firestore is referred to by its location in the database. This is akin to finding a document via some criteria using `mongoose` and storing the resulting instance in a variable:

```js
const exampleUserDocumentRef = db.collection('users').doc('exampleUser')
```

We can also create references that point to collections as follows:

```js
const usersCollectionRef = db.collection('users')
```

Now, to illustrate hierarchal data, we must introduce the concept of *subcollections*.

Previously, our user object contained a `notes` property containing an array of `Note` IDs. Upon populating these fields, a given user document would effectively contain all of the user's note documents as well.

We can achieve the same functionality with subcollections:

```jsx
const noteRef = db.collection('users').doc('exampleUser')
                  .collection('notes').doc('note1')
```

This is similar to how we structured our data with mongoDB in that our documents can eventually contain other documents, but with Firestore, we must follow the alternating pattern of collections and documents.

A document cannot contain a reference to a document and a collection cannot contain a reference to a collection. This concept will begin to make more sense when we create our application surrounding these principles.

#### Creating/Overwriting a Document

Now, how might we go about creating a document? With `mongoose` we were able to use the `new` keyword along with the model to create an instance of the model, to which we could save the instance to the database to create our document.

With Cloud Firestore, it's just as easy, or even easier. We can dynamically create new collections (recall that there is no need to manually create or delete collections) and documents in those collections with a ID of our choice or an auto-generated ID:

```js
db.collection("notes").doc("firstNote").set({
  content: "This note has a set ID",
  flagged: false
})

db.collection("notes").add({
  content: "This note has an auto-generated ID",
  flagged: false
})
```

Here, notice how we can call the `add()` method upon `db.collection("notes")` to add a document with the passed in object as its content. The ID will be auto-generated for us in this case.

We can manually specify an ID by calling `doc(ID)` upon `db.collection("notes")` to which we can just `set()` the document with the ID that we had specified with the content of the object passed to `set()`.

We can also just generate a reference to a document and set the data of the document later:

```jsx
// Creates the new document with an auto-generated ID
const noteRef = db.collection("notes").doc()

// Set the data of the document later
noteRef.set(data)
```

This illustrates the concept of `doc()` creating a new document (with an auto-generated ID or a specified ID if passed in) and `set()` setting the content of the reference. `add()` appears to just do this in one step.

#### Updating a document

To update certain fields of a document without updating the entire document, just call the `update()` method on a document or a reference:

```js
const noteRef = db.collection("notes").doc("firstNote")

noteRef.update({
  content: "Updated content!"
})
.then(function() {
  console.log("Document successfully updated.");
})
.catch(function(error) {
  // The document may not exist
  console.error("Error updating document: ", error);
});
```

The fields that we specify in the object passed to `update()` will replace the fields in the document.

We can also access nested fields through "dot notation" within quotes:

```js
noteRef.update({
    "content.type": "text"
})
```

This would update the `type` property in the `content` object:

```js
{
  firstNote: {
    content: {
      type: "text",
      textContent: "This text is in a nested object!"
    }
  }
}
```

If we do not use this "dot notation" to update a nested field and instead just try using an object within an object to target a field, we'll end up overwriting the entire nested property:

```js
db.collection("notes").doc("firstNote").update({
  content: {
    type: "image"
  }
}).then(function() {
  console.log("Note updated")
})
```

This would result in the entire `content` property being overwritten with the object we had specified (instead of just the `type` property being replaced).

```js
{
  firstNote: {
    content: {
      type: "image"
    }
  }
}
```

#### Deleting a Document

We can easily delete a document by calling the `delete()` method upon it:

```js
db.collection("notes").doc("firstNote").delete()
  .then(() => console.log("Document deleted successfully"))
```

It is important to note that deleting a document does not remove the documents in its sub collections.

This means that if our note document had a `users` subcollection to which it had the documents of the users that have access to the notes. If we delete the note document, that subcollection still exists.

We can still access `/notes/firstNote/users/firstUser` even though `/notes/firstNote` no longer exists.

It is not recommended to delete collections from a web client. Upon hearing web client, you might wonder, how will we protect our data if we're making these requests on the client side? Since the user can manipulate anything on the browser side, we'll take advantage of Cloud Firestore security rules in order to restrict access to certain documents and operations.

#### Getting Data

There are two ways to retrieve documents/collections/the result of queries in Cloud Firestore.

We can receive this data through:

* Calling a method to retrieve data that matches our criteria
* Setting a listener to retrieve update/data-change events

##### Calling Methods to Retrieve Data

We can retrieve documents by calling `get()` upon their reference:

```js
const userRef = db.collection("users").doc("firstUser")

userRef.get().then(doc => {
  if (doc.exists) {
    console.log("Document data:", doc.data()) 
  } else {
    console.log("Document does not exist!")
  }
}).catch(error => {
  console.log("Error getting document:", error);
})
```

Get multiple documents from a collection based on a condition:

```js
db.collection("notes").where("flagged", "==", true)
  .get()
  .then(querySnapshot => 
    querySnapshot.forEach(
      doc => console.log(doc.data())
    )
  )
  .catch(error => {
    console.log("Error getting documents: ", error);
  })
```

Here, we fetch all the documents that match the condition specified to `where()`. We can get all documents from a collection by just using `get()` on a collection without calling `where()`. For now, just think about `querySnapshot` as a "snapshot" of our collection, to which we can iterate over it to access our documents.

#### Setting up Listeners

We can listen for changes/updates to a document with the `onSnapshot()` method.

An initial call to `onSnapshot()` with a valid callback creates a document "snapshot" immediately with the current contents of the document.

A `DocumentSnapshot` simply contains the data read from a document in our database. The data can be extracted through calling `data()` upon it, to which `undefined` will be returned if the document does not exist.

Upon any update to the document, our document snapshot will be updated and our callback function will be called again with the new document snapshot:

```js
db.collection("notes").doc("firstNote")
  .onSnapshot(doc => console.log("Data: ", doc.data()))
```

An interesting behavior to note is that any updates that we make locally will instantly invoke these snapshot listeners. These listeners do not wait for the server to write the changes in an effort to reduce latency. With this, we're given access to `doc.metadata.hasPendingWrites` which is true when our local changes have not been written to the backend/server just yet (as a result, the source of changes is our local app when the value of `hasPendingWrites` is `true`):

```js
db.collection("notes").doc("firstNote")
  .onSnapshot(doc => {
    const source = doc.metadata.hasPendingWrites ? "Local" : "Server"
    console.log(source, " data: ", doc.data())
  })
```

We can also listen to entire collections or multiple documents in a collection:

```js
db.collection("notes")
  .onSnapshot(querySnapshot => {
    var notes = []
    querySnapshot.forEach(doc => {
        notes.push(doc.data())
    })
  })
```

Here, we take in a `QuerySnapshot` which contains zero or more `DocumentSnapshot` objects. We can access these documents as an array through the `docs` property or by iterating through it using `forEach()`.

Using the `docs` property to achieve the above would be as follows:

```js
db.collection("notes")
  .onSnapshot(querySnapshot => {
    const notes = querySnapshot.docs
  })
```

It is important to note that our array will contain `QueryDocumentSnapshot`s, so we will later have to iterate over the array and call `data()` upon each element in order to get the same result as the above `notes` var in the first example.

As with the `onAuthStateChange()` listener, these listeners return the appropriate `unsubscribe()` function that we can use to detach our listeners:

```js
const unsubscribe = db.collection("notes")
					  .onSnapshot(() => { /* */ })


// Stop listening for changes
unsubscribe()
```

#### Firestore Security Rules

Cloud Firestore security rules allow us to control access to our documents and collections. It is entirely possible to implement a role-based system with Firestore so that we can restrict access to certain roles or users.

These security rules follow the following syntax:

```js
service cloud.firestore { // Describes the service
  match /databases/{database}/documents { // Declares the path that the rule should match to, to which we use {database} to describe the documents of every single database in the project
    // ...
  }
}
```

Now, we can nest `match` statements inside this `match` statement that just encompasses all of the databases in our project:

```js
service cloud.firestore {
  match /databases/{database}/documents {

    // Match any document in the 'notes' collection
    match /notes/{note} {
      allow read: if <condition>;
      allow write: if <condition>;
    }
  }
}
```

The items wrapped in braces "{}" are wildcards and will make the rule apply to any document in the collection. For example, `/notes/{note}` will match every single note document in the `notes` collection. When using wildcards, we're also given access to the document name/ID through the wildcard, so we can check for matches involving a given document's ID in the `<condition>`.

There are plenty of conditions and structures that we can use for our rule set. If you are considering on using Firestore for a production app in the future, it may help to read the official docs to completion and refer to sources such as Fireship for examples on implementing full-scale role-based authentication systems.

For now, to handle a basic rule set, we're going to allow reading and writing to our notes collection if the user is signed in, we can handle this with the following statements:

```js
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow the user to access documents in the 'notes' collection
    // only if they are authenticated.
    match /notes/{note} {
      allow read, write: if request.auth.uid != null; // Checks if the request is authenticated
    }
  }
}
```

As an example for the use of a wildcard in a condition, if we wanted to restrict updating a user object to only the user, we would write a rule similar to the following:

```js
service cloud.firestore {
  match /databases/{database}/documents {
    // Checks if the uid of the requesting user matches the ID of the user document being updated (they should match if the user is the same)
    match /users/{userId} {
      allow read, update, delete: if request.auth.uid == userId;
      allow create: if request.auth.uid != null; // Doesn't allow the creation of another user object if the user is signed in
    }
  }
}
```

With this, we're ready to start adding some functionality to our application. There are plenty of other features to Cloud Firestore, but those features outscope the introduction that we're providing here. If you choose to use Firebase in a future, larger project, it may be beneficial to read the official Firebase docs.

### Interacting with Cloud Firestore in React

Now, equipped with Cloud Firestore, let's begin adding the note taking functionality to our Firebase application.

Let's first add the ability to create notes, we'll handle this in a new component. Let's create the **components** directory and create a new module in it:

**/src/components/NewNote.js**

```jsx
import React, { useState } from 'react'
import firebase from 'firebase/app'
import 'firebase/firestore'

const NewNote = () => {
  const [formValue, setFormValue] = useState("")

  const handleSubmit = async () => {
    firebase.firestore().collection('notes').add({
      content: formValue,
      flagged: false
    })
    .then(() => console.log("Document successfully written!"))
    .catch(error => console.error("Error writing document: ", error))

    setFormValue("")
  }

  return (
    <>
      <input type="text" value={formValue} onChange={event => setFormValue(event.target.value)} />
      <button onClick={handleSubmit}>Create</button>
    </>
  )
}

export default NewNote
```

Here, in our `NewNote` component, much of the logic and syntax surrounding this component should be very familiar. This component simply renders a controlled text input element to which we send to Firestore to create a new document with the value of our `formValue` piece of state in our `handleSubmit()` handler.

Now, we need a way to view our notes. We can do this by simply requesting the entire collection and displaying it:

**/src/components/Notes.js**

```js
import React, { useState, useEffect } from 'react'
import firebase from 'firebase/app'
import 'firebase/firestore'

const Notes = () => {
  const [notes, setNotes] = useState([])

  useEffect(() => {
    const unsubscribe = firebase.firestore().collection("notes")
      .onSnapshot(querySnapshot => {
        let notes = []
        querySnapshot.forEach(d => {
          let noteData = d.data()
          noteData.id = d.id
          notes.push(noteData)
        })
        setNotes(notes)
      })
    return unsubscribe
  }, [])

  return (
    <>
      {notes.map(note => (
        <li key={note.id}>{note.content}</li> 
      ))}
    </>
  )
}

export default Notes
```

Here, we register an effect that gets the `notes` collection from Firestore upon the component being mounted. We then iterate over the snapshot to create an array filled with all of the document's data. We then create a list of the notes with their `key`s set to their `id` values which have been autogenerated.

Notice how we have to create a new object `noteData` set to the document's data retrieved by calling `data()`. Since we need the document's ID for a key value and for further operations, we'll manually add it to the `noteData` object before pushing it onto the array.

We are listening for updates from the database, so upon creating a note, we instantly notify all listeners locally (as mentioned above) as well as any listeners connected to the database (when we actually save our changes to the database). As a result of this, our state is automatically updated and we do not have to do any manual state management to see the results of our note creation.

Requesting entire collections is not possible with larger datasets. Firestore offers an excellent feature set, to which *pagination* is a feature. Pagination allows us to split our massive collections into "pages" of documents, to which we can serve the pages one at a time.

We'll now update our `App` component to include these two new components and remove the rendering of the `user` object to which we receive the following result:

![Result](https://i.imgur.com/Bn7g853.png)

Now, let's add the ability to toggle the flagged property of our notes:

**/src/components/Notes.js**

```jsx
import React, { useState, useEffect } from 'react'
import firebase from 'firebase/app'
import 'firebase/firestore'

const Notes = () => {
  const [notes, setNotes] = useState([])

  useEffect(() => {
    const unsubscribe = firebase.firestore().collection("notes")
      .onSnapshot(querySnapshot => {
        let notes = []
        querySnapshot.forEach(d => {
          let noteData = d.data()
          noteData.id = d.id
          notes.push(noteData)
        })
        setNotes(notes)
      })
    return unsubscribe
  }, [])

  const onToggle = note => {
    const noteRef = firebase.firestore().collection("notes").doc(note.id)

    noteRef.update({
      flagged: !note.flagged
    })
    .then(() => {
      console.log("Document successfully updated!")
    })
    .catch(error => {
        // The document probably doesn't exist.
        console.error("Error updating document: ", error)
    })
  }

  return (
    <>
      {notes.map(note => (
        <div key={note.id}>
          <input type="checkbox" onChange={() => onToggle(note)} checked={note.flagged} /> {note.content}
        </div>
      ))}
    </>
  )
}

export default Notes
```

Here, we simply render a checkbox for every note in our function provided to `notes.map()`. This checkbox is set to the `flagged` property of the given note in our iteration, to which this is a controlled checkbox. We also register an `onChange` handler that calls `onToggle(note)`. In `onToggle()`, we take in the note and find the specified document referring to the note and store it in `noteRef`. Then, we simply make use of the `update()` method to flip the `flagged` property.

Our application now looks like this with the ability to toggle the `flagged` property of a note:

![Test](https://i.imgur.com/szF6Cre.png)

With this, we have created a note-taking application using Firebase, to which we had not needed to create our own backend server. Firebase is a powerful tool and should be considered for projects where the scale or timeframe may not allow for a full, custom backend solution.

## React Native

Now, for the duration of this book, we've been working with React, a framework created by Facebook for "data-driven interfaces". React is an excellent choice for most web applications due to its declarative and extensible nature.

Now, React is mainly meant for web apps and web interfaces, but there is a whole other world of applications that run "natively" on platforms such as Windows, iOS, macOS, Android, etc. rather than a web browser. 

React Native is almost a logical extension to React, with it enabling us to write these native applications using the same virtual DOM, stateful components, layouts, etc. that we are used to with React.

React Native also comes with a large benefit behind the mantra "Write Once, Run Anywhere" where we can reuse most of our code for deployment on multiple platforms. With native applications, you would previously have to write a separate application for each target that you wished to support. With React Native, it is entirely possible to write "once" and deploy to iOS, Android, Windows, etc.

These native applications, when run on their respective platforms, often come with much better experiences than a web app would provide in a browser. There are other technologies such as Progressive Web Applications that are attempting to bridge this gap, but native applications still provide the most seamless experience.

React Native primitives render actual native platform UI elements leading to an almost-native experience.

> It is important to note that you'll only be able to target and build for iOS from a macOS supported device.

### Expo

Making our native development experience even easier, Expo is a toolset that handles a lot of the headache that comes with setting up React Native applications (analogous to how `create-react-app` eased our introduction to React). It provides a set of APIs that work extremely well without any need for us to dive into native code, allowing us to stay entirely within the realm of JavaScript.

This comes with the downside of not being able to write any native code/native "modules" for anything that we can't achieve with the set of APIs given to us, limiting us to a certain extent. We can always "eject" our application (just like we could with `create-react-app`), but, after ejecting, we are left to our devices on managing everything without Expo's help.

Explained further: Expo apps are written in pure JS and will not “drop down” to the native Android or iOS layer. This brings us to the main caveat with Expo which is that you will not be able to use any native modules (any native to JS bridges). In order to use native modules, Expo allows you to eject your pure JavaScript project from Expo to ExpoKit. With this, everything that you already have built will keep working as it did before, but your project will no longer live in the Expo client, so you will have to configure and build your projects yourself without Expo features.

#### Getting Started with Expo

Let's explore the suite of tools that Expo provides us with:

* Snack - Allows for a playground environment where you can run React Native in the browser
* CLI - A command-line interface where developers can share, publish, and serve their projects
* Client - Runs Expo projects on actual phone hardware while developing with features such as hot reloading, etc. It is also possible to build Expo applications for web clients contributing to a true "Write Once, Run Anywhere".
* SDK - Bundles and exposes APIs and cross-platform libraries to be used by the developer
  * Includes maps, icons, images, animations, constants (height of statusbar, etc.), and more.

With this, let's install Expo by running the following command:

```bash
npm i -g expo-cli
```

If you're using macOS, it's also recommended to install Watchman (tool used for monitoring changes):

```bash
brew install watchman
```

We can now initialize a project using the `expo init` command. Throughout this process, you may be prompted to create an Expo account and sign into it from the CLI and any client apps that you may use:

```bash
expo init notes-app
```

We'll then be prompted to choose a workflow, to which we'll choose the "blank" managed workflow:

![image-20200227223901655](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200227223901655.png)

Managed workflows give us the above advantages of Expo to which Expo manages lots of the complexity of native applications for us.

This gives us an application structure similar to `create-react-app`:

![image-20200227234039946](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200227234039946.png)

Upon running `expo start` to run our application, the following window will appear in our browser:

![image-20200227234002892](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200227234002892.png)

Here, we're given information related to our app as well as options to start our application on a client/simulator. Our terminal will also give us similar information:

![image-20200227234122759](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200227234122759.png)

For our purposes, we'll be using an iOS simulator to view our application. If you're using a Windows computer, you may use an Android emulator. We'll press `i` to proceed with launching an iOS simulator. You must have xCode installed already and a simulator initialized (Android simulators require Android Studio + an AVD). We are now given access to our app:

![image-20200227234726600](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200227234726600.png)

Changes to our application code will automatically "refresh" our app.

Before we actually start with our application code, let's explore some of the main differences between React and React Native.

#### Differences between React and React Native

Although it might look like it sometimes, React Native does not use HTML at all in order to render the app. Recall that our app is rendered down to native primitives for performance reasons.

#### Primitives

Now, there are plenty of similarities, however. For example, `View` behaves similarly to `div` and `Text` to `p`. This difference in primitive elements, however, means that we won't be able to use any React libraries that render HTML, SVG, or canvas elements.

#### Styling

The difference in primitive elements also means that we cannot approach styling in the exact same way. Some CSS-in-JS libraries, such as `styled-components` allow us to similarly style our elements as we did with React. Traditionally, however, instead of CSS, we use a `StyleSheet`

```jsx
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

```

In industry, people typically bundle the StyleSheet with the component with the logic being that each component should be independent. There are also compelling reasons to move styles outside of a component, such as being able to change one color to change the primary color of your app, etc. Generally, though, it is just easier to keep the StyleSheet within the component for organization purposes.

You may also notice that `StyleSheet.create()` is quite similar to `makeStyles()` from MaterialUI.

App layouts in React Native are almost entirely built using flexbox (although it behaves a bit differently than in CSS). The learning curve is quite low, however, when it comes to styling React Native apps, especially if you use a component library.

Additionally, we no longer have access to CSS animations, but rather are given access to the `Animated` API which allows us to create animated elements tied to gestures, etc.

#### Interactions

Interactions with our application are also done through the JavaScript touch events web API, `PanResponder`. This gives us access to the gesture state and other information regarding interactions.

#### Navigation

Navigation is quite a large difference in that we can't user `react-router` to handle our App's navigation. With this, React Navigation gives us an excellent set of components and a platform to handle our application routing and navigation. We'll explore its use in a moment.

#### Developer Tools

React Native provides us with some developer tools out of the box including an inspector (not as good as the inspector for the web with Chrome Developer Tools), hot reloading features, etc. We are also given access to Chrome Developer Tools and Redux DevTools, but there are a few things that we have to consider and configure differently than with React.

## Firebase + Navigation in React Native

Expo does not support the `react-native-firebase` module, so we'll have to use the same web module (full JS) that we used with our previous web app that used Firebase. This means that our Firebase related tasks will be performed in the JavaScript thread only, and we will be able to use the same syntax that we used with our web client. We'll also be making use of `react-navigation` for our application routing and navigation.

We'll now get started with creating the same Firebase application that we did earlier with Firebase in React, but in React Native (with additional styling to show off how styling is approached in React Native).

Here, we're going to be introducing a new component organization scheme that is very common in the React Native world (and increasingly so in the React world). We will assemble components into displayable "screens" found in the **/screens** directory. Our file structure will resemble the following:

![image-20200229115155300](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200229115155300.png)

This may change with larger projects, but, generally, reusable components are found in the **/components** folder, any static assets in the **/assets** folder, our services (similar to React) being found in the **/services** folder, etc.

One interesting thing to note is that if we build for the web, our "compiled" files will be found in the **/web-build** folder. We'll also add an ESLint plugin for React Native by running the following command:

```bash
npm i --save-dev eslint eslint-plugin-react eslint-plugin-react-native eslint-plugin-jest eslint-plugin-react-hooks
```

We'll then add the plugin to our **.eslintrc.js** file:

**/.eslintrc.js**

```js
module.exports = {
    //...
    "plugins": [
        "react",
        "jest",
        "react-hooks",
        "react-native"
    ],
    //...
```

### Creating Screens in React Native

With React Navigation, we're given many ways to navigate between screens, modals, and more. This includes: tab navigation, drawer navigation (sliding up to access a screen), stack navigation (stacked screens on top of each other upon an event), and deep linking (accessing a link that brings you to a screen on the app, similar to routing in React). 

You can also embed navigators inside other navigators. For example, we can put a stack navigator inside a tab navigator, so that we can view a tab (for example a "Your Profile" tab like on Instagram) and push other screens on top of the view in the stack navigator (such as displaying another profile after tapping on their like on top of your profile).

This all culminates from a "landing navigator", usually a tab navigator that handles switching to various stack navigators that can manage pushing other screens on top of the current view.

Let's start by creating the boilerplate for the various screens of our application (we'll then implement some initial navigation). This code is identical across all screens beside the names used:

**/screens/HomeScreen.js**

```js
import React from 'react'
import { StyleSheet, Text, View } from 'react-native'

const HomeScreen = () => {
  return (
    <View style={styles.container}>
      <Text>Home Screen</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})

export default HomeScreen
```

**/screens/RegisterScreen.js**

```jsx
import React from 'react'
import { StyleSheet, Text, View } from 'react-native'

const RegisterScreen = () => {
  return (
    <View style={styles.container}>
      <Text>Register Screen</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})

export default RegisterScreen
```

**/screens/LoginScreen.js**

```js
import React from 'react'
import { StyleSheet, Text, View } from 'react-native'

const LoginScreen = () => {
  return (
    <View style={styles.container}>
      <Text>Login Screen</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})

export default LoginScreen
```

Now, we'll start implementing our navigation between these screens so that we can start to create our authentication flow. We'll need to install `react-navigation` first:

```bash
npm i @react-navigation/native @react-navigation/stack @react-navigation/bottom-tabs
```

We'll also install its dependencies (needed by most configurations):

```bash
expo install react-native-gesture-handler react-native-reanimated react-native-screens react-native-safe-area-context @react-native-community/masked-view
```

Here, we used `expo install` in order to install these dependencies into our Expo managed project.

**/App.js**

```jsx
import { NavigationContainer } from '@react-navigation/native'
import { createStackNavigator } from '@react-navigation/stack'
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs'
import React, { useState } from 'react'
import { Ionicons } from '@expo/vector-icons'
import HomeScreen from './screens/HomeScreen'
import LoginScreen from './screens/LoginScreen'
import RegisterScreen from './screens/RegisterScreen'

// Handle Firebase here

const Stack = createStackNavigator()
const Tab = createBottomTabNavigator()

// Stack navigator for our authentication flow
const AuthNavigator = () => (
  <Stack.Navigator>
    <Stack.Screen name="Register" component={RegisterScreen}/>
    <Stack.Screen name="Login" component={LoginScreen}/>
  </Stack.Navigator>
)

// Tab navigator for our "app" when we're signed in containing screens related to the functionality of the app
const NotesNavigator = () => (
  <Tab.Navigator
    screenOptions={({ route }) => ({
      // eslint-disable-next-line react/display-name
      tabBarIcon: ({ focused, color, size }) => { // `tabBarIcon` set to a function that returns the icon that we want to render with a given tab
        let iconName

        if (route.name === 'Home') {
          iconName = focused
            ? 'ios-information-circle'
            : 'ios-information-circle-outline'
        }
        return <Ionicons name={iconName} size={size} color={color} />
      },
    })}
    tabBarOptions={{
      activeTintColor: 'tomato',
      inactiveTintColor: 'gray',
    }}
  >
    <Tab.Screen name="Home" component={HomeScreen}/>
  </Tab.Navigator>
)

const App = () => {
  const [user, setUser] = useState({}) // Set to a blank object to simulate a user being signed in

  return (
    <NavigationContainer>
      <Stack.Navigator headerMode='none'>
        {!user ? ( // If we're not signed in, display the auth navigator
          <>
            <Stack.Screen name="Auth" component={AuthNavigator} />
          </>
        ) : (
          <>
            {/* User is signed in */}
            <Stack.Screen name="Notes" component={NotesNavigator} />
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  )
}

export default App
```

Here, quite a bit of interesting stuff is going on. We create navigators by importing the respective navigator creator, for example `createStackNavigator()` and use that to create the its respective `Navigator` and `Screen` components. All screens are wrapped by `Screen` components, to which these `Screen` components are children of a `Navigator` component.

We can then use these `Navigator` components inside other `Navigators`. In the code block above, we also make use of the `NavigationContainer` provider which provides all of our screens with navigation context (through a `navigation` prop).

Read the comments in the code block above to get an objective summary of each function in the `App` component. In summary, we had created a `AuthNavigator` (stack navigator containing screens related to registering/logging in) and a `NotesNavigator` (tab navigator containing screens related to creating and viewing notes). The `NotesNavigator` has some additional configuration related to setting color configurations and supplying the `tabBarIcon` property which takes in a function that returns the respective icon that we want to render dependent on the information passed into it.

`AuthNavigator` and `NotesNavigator` are then wrapped in a stack navigator with `headerMode` set to `'none'` (so we won't see an extra heading in our application). We then choose which navigator to render dependent on the value of `user`. Finally, everything gets wrapped in a `NavigationContainer` provider which will provide navigation state information to our components. This information includes  a`navigation` prop that allows us to navigate using the helper methods `navigate`, `goBack`, etc. and a `route` prop that contains the screen's state/data (for example, we can pass parameters to screens). We can also access the `navigation` prop using the `useNavigation()` hook.

Let's now create the UI elements for the various screens before implementing our functionality through Firebase:

**/screens/LoginScreen.js**

```jsx
import React from 'react'
import { StyleSheet, Text, View, TextInput, TouchableOpacity } from 'react-native'

const LoginScreen = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        Welcome back!
      </Text>

      <View style={styles.error}>
        <Text style={styles.errorText}>Error</Text>
      </View>

      <View style={styles.form}>
        <Text style={styles.inputLabel}>Email Address</Text>
        <TextInput style={styles.input} autoCapitalize="none" />

        <Text style={styles.inputLabel}>Password</Text>
        <TextInput style={styles.input} secureTextEntry autoCapitalize="none" />
      </View>

      <TouchableOpacity style={styles.button}>
        <Text style={styles.buttonText}>
          Sign In
        </Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.registerView}>
        <Text>Don&apos;t have an account? <Text style={{ fontWeight: 'bold' }}>Sign Up</Text> </Text>
      </TouchableOpacity>

    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'flex-start',
    backgroundColor: 'white'
  },
  title: {
    marginVertical: 60,
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center'
  },
  error: {
    marginBottom: 20,
    alignItems: 'center',
    justifyContent: 'center',
    marginHorizontal: 40,
    backgroundColor: '#e74c3c',
    borderRadius: 8,
    padding: 10
  },
  errorText: {
    color: 'white',
    fontWeight: 'bold'
  },
  form: {
    marginHorizontal: 40
  },
  inputLabel: {
    fontSize: 13,
    fontWeight: 'bold',
    color: '#2c3e50'
  },
  input: {
    borderBottomWidth: StyleSheet.hairlineWidth,
    height: 40,
    fontSize: 15,
    borderBottomColor: '#34495e',
    marginBottom: 30
  },
  button: {
    marginHorizontal: 40,
    marginVertical: 15,
    backgroundColor: '#2ecc71',
    borderRadius: 8,
    height: 60,
    alignItems: 'center',
    justifyContent: 'center'
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
    fontSize: 15
  },
  registerView: {
    alignItems: 'center',
    justifyContent: 'center',
    height: 60
  }
})

export default LoginScreen
```

This currently renders the following result:

![image-20200229175849936](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200229175849936.png)

We can now start implementing these features with Firebase. First, we'll install `firebase` by running the following command:

```bash
npm i firebase
```

Now, let's configure Firebase. We can use the same web credentials that we used earlier for our application:

**/services/firebaseService.js**

```jsx
import firebase from 'firebase/app'

// Firebase configuration provided to us by the app creation process
const firebaseConfig = {
  apiKey: 'AIzaSyDJf0kn4MHA8mMgolqu1DLh3YqrjSAt57Q',
  authDomain: 'notes-app-bfbc5.firebaseapp.com',
  databaseURL: 'https://notes-app-bfbc5.firebaseio.com',
  projectId: 'notes-app-bfbc5',
  storageBucket: 'notes-app-bfbc5.appspot.com',
  messagingSenderId: '768088207404',
  appId: '1:768088207404:web:abd4b64d40dbab447e6189',
  measurementId: 'G-NBKG2S4VX5'
}

// Initializes Firebase and creates an app instance
try {
  !firebase.apps.length ? firebase.initializeApp(firebaseConfig) : firebase.app() // Grabs the Firebase instance if it already exists
} catch (err) {
  // Catches 'already initialized' errors and logs it to enable hot reloading to continue to work
  if (!/already exists/.test(err.message)) {
    console.error('Firebase initialization error raised', err.stack)
  }
}

export default firebase
```

Here, we simply used the configuration from the previous web application that used Firebase and initialized our app with those credentials using `initializeApp()`. We wrapped the initialization call in a `try`/`catch` block as, when using the hot reloading features to rapidly develop with React Native, initialization will be called multiple times. We also moved our Firebase configuration and initialization to its own service module. This will help reduce the amount of re-initializations that we receive when hot reloading as well as help organize our code. These are all fallbacks in the case that something goes wrong when trying to either initialize our app or retrieve our app (in the case that there is more than 0 Firebase apps initialized).

Now, we can make a few changes to our `LoginScreen` component in order to actually implement logging in. We will be making use of email and password authentication which we have not used in the past with Firebase:

**/screens/LoginScreen.js**

```jsx
import React, { useState } from 'react'
import { StyleSheet, Text, View, TextInput, TouchableOpacity } from 'react-native'
import firebase from '../services/firebaseService'

const LoginScreen = () => {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState()

  const handleLogin = () => {
    firebase
      .auth()
      .signInWithEmailAndPassword(email, password)
      .catch(err => setError(err.message))
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        Welcome back!
      </Text>

      {error ?
        <View style={styles.error}>
          <Text style={styles.errorText}>{error}</Text>
        </View>
        : null}

      <View style={styles.form}>
        <Text style={styles.inputLabel}>Email Address</Text>
        <TextInput
          style={styles.input}
          autoCapitalize="none"
          onChangeText={email => setEmail(email)}
        />

        <Text style={styles.inputLabel}>Password</Text>
        <TextInput
          style={styles.input}
          secureTextEntry
          autoCapitalize="none"
          onChangeText={password => setPassword(password)}
        />
      </View>

      <TouchableOpacity style={styles.button} onPress={handleLogin}>
        <Text style={styles.buttonText}>
          Sign In
        </Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.registerView}>
        <Text>Don&apos;t have an account? <Text style={{ fontWeight: 'bold', color: '#2ecc71' }}>Sign Up</Text> </Text>
      </TouchableOpacity>

    </View>
  )
}

//...

export default LoginScreen
```

Here, we created three pieces of state `email`, `password`, and `error` using `useState()`. We control these pieces of state similarly to how we would in a React application with regard to providing functions that update our state.

You'll notice that with our `TextInput` elements, we made use of `onChangeText` rather than `onChange` which allows us to not deal with the `event` object, but rather just provide a function that takes in the actual text content of the input element. Instead of `onClick`, in React Native, we have `onPress` (since you won't be clicking with a mouse in a mobile application).

The `onPress` handler of our main button calls the `handleLogin()` function which takes in our state and uses `firebase.auth()` to try to log in with an email and password passed in to it. We catch any errors and set the appropriate `error` piece of state to the value of its message. With this, we changed the error view to only render when there is an error with the text being the error. 

Additionally, we just used `<TouchableOpacity>` to create our buttons as it is a wrapper component for making views respond to touches. The "opacity" part of the component comes from the fact that the opacity of the wrapped view is decreased upon being touched, dimming it. There is also a `<TouchableHighlight>` component.

Now, if we try to sign in using a random email and password, we'll receive the following error:

![image-20200229191818559](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200229191818559.png)

This is because we still need to enable email and password authentication in our Firebase dashboard:

![image-20200229191923131](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200229191923131.png)

After this is done, we can proceed to implement user registration. We'll first need a way to navigate to the registration screen. This can be accomplished through taking in the `navigation` prop (or by using the `useNavigation()`) hook. We'll then call `navigate('Register')` upon it as that will navigate to the screen with the name `"Register"` in the same navigator:

**/screens/LoginScreen.js**

```jsx
//...

	  <TouchableOpacity style={styles.registerView} onPress={() => navigation.navigate('Register')}>
      <Text>Don&apos;t have an account? <Text style={{ fontWeight: 'bold', color: '#2ecc71' }}>Sign Up</Text> </Text>
    </TouchableOpacity>

//...
```

Pressing "Don't have an account? **Sign Up**" will now take us to our register screen. We'll update our register screen as follows:

**/screens/RegisterScreen.js**

```jsx
import React, { useState } from 'react'
import { StyleSheet, Text, View, TextInput, TouchableOpacity } from 'react-native'
import firebase from '../services/firebaseService'

const RegisterScreen = ({ navigation }) => {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [name, setName] = useState('')
  const [error, setError] = useState()

  const handleRegister = () => {
    firebase
      .auth()
      .createUserWithEmailAndPassword(
        email,
        password
      )
      .then(userCredentials => { // Takes the user credentials that are returned from the create action, accesses the user property and calls the updateProfile method to set the person's name
        return userCredentials.user.updateProfile({
          displayName: name
        })
      })
      .catch(err => setError(err.message))
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        {'Hey!\nSign up to get started.'}
      </Text>

      {error ?
        <View style={styles.error}>
          <Text style={styles.errorText}>{error}</Text>
        </View>
        : null}

      <View style={styles.form}>
        <Text style={styles.inputLabel}>Full Name</Text>
        <TextInput
          style={styles.input}
          autoCapitalize="none"
          onChangeText={name => setName(name)}
        />

        <Text style={styles.inputLabel}>Email Address</Text>
        <TextInput
          style={styles.input}
          autoCapitalize="none"
          onChangeText={email => setEmail(email)}
        />

        <Text style={styles.inputLabel}>Password</Text>
        <TextInput
          style={styles.input}
          secureTextEntry
          autoCapitalize="none"
          onChangeText={password => setPassword(password)}
        />
      </View>

      <TouchableOpacity style={styles.button} onPress={handleRegister}>
        <Text style={styles.buttonText}>
          Sign Up
        </Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.registerView} onPress={() => navigation.navigate('Login')}>
        <Text>Already have an account? <Text style={{ fontWeight: 'bold', color: '#2ecc71' }}>Log In</Text> </Text>
      </TouchableOpacity>

    </View>
  )
}

//... SAME STYLING AS LOGINSCREEN

export default RegisterScreen
```

Here, our `RegisterScreen` component is almost exactly the same as `LoginScreen`. The changes that we made include changing the header text, calling `firebase.auth().createUserWithEmailAndPassword(email, password)` in order to create our user, adding another input element for the user's name, and navigating to the login screen with the text at the bottom.

We take advantage of another Firebase feature where we access and modify the user right after its creation to add more fields to the user object.

Our register screen now looks like this:

![image-20200301000039154](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200301000039154.png)

Finally, we'll add the ability for our application to detect when we have signed in to which we'll store the user in our state:

**/App.js**

```jsx
//...

import firebase from './services/firebaseService'

//...

const App () => {
  const [user, setUser] = useState()
  
  useEffect(() => {
    const unsubscribe = firebase
      .auth()
      .onAuthStateChanged(
        user => setUser(user)
      )

    return () => unsubscribe()
  }
  , [])
  
  //...
}
```

Now, upon signing in, we'll instantly be taken to the home screen of our application since our `user` piece of state will be populated, to which we render our `NotesNavigator` stack screen only when `user` "exists".

There is no easy way to pass our `user` piece of state through our components as a prop down through our components (prop drilling), so this would require the use of Redux or other methods given to us by React Navigation.

This presents an excellent opportunity to discuss the use of another React feature that attempts to provide some of the functionality of Redux bundled in React.

### React Context (Web + React Native)

Now, let's explore the problem of needing the `user` piece of state in other areas of our application without passing it as a prop down the tree (prop drilling).

To get started, we need to first create the "context". This will be used to later create "providers" and "consumers" (explained later).

Let's first create our "context" file:

**/context.js**

```jsx
import React from 'react'

export const UserContext = React.createContext([null, () => {}])
```

Here, we create a context. This is like creating a store with Redux (except here, instead of creating a global store object, we're just creating a global user object).

We export this context object for use in our components.

Now, in our `App` component, we can wrap our entire app in the provider component given to us by the context object in order to provide the child components with the context:

**/App.js**

```jsx
//...

import { UserContext } from './context'

//...

  return (
    <UserContext.Provider value={user}>
      <NavigationContainer>
        <Stack.Navigator screenProps headerMode='none'>
          {!user ? (
            <>
              <Stack.Screen name="Auth" component={AuthNavigator} />
            </>
          ) : (
            <>
              {/* User is signed in */}
              <Stack.Screen name="Notes" component={NotesNavigator} />
            </>
          )}
        </Stack.Navigator>
      </NavigationContainer>
    </UserContext.Provider>
  )
}
```

This provider component will allow consuming components to subscribe to context changes. We can consume this value from the provider by using the consumer component given to us by the context object. Let's consume the user context object in our `HomeScreen` component:

**/screens/HomeScreen.js**

```jsx
import React from 'react'
import { StyleSheet, Text, View } from 'react-native'
import { UserContext } from '../context'

const HomeScreen = () => {
  return (
    <UserContext.Consumer>
      {
        user =>
          <View style={styles.container}>
            <Text>Welcome {user.displayName}</Text>
          </View>
      }
    </UserContext.Consumer>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})

export default HomeScreen
```

The `<Context.Consumer>` component subscribes to context changes and takes a function in as a child. This function will receive the current context value and should return a React node. The function that we pass as a child above takes in the user object and makes it available to us to do whatever we please with. In this case, we simply use our access to the `user` global state (context) and access its display name.

We can update the context by adding a function to our context which allows us to simply call this function with any value passed into it in order to replace the context value:

**/context.js**

```jsx
import React from 'react'

export const UserContext = React.createContext({
  user: null,
  setUser: () => {}
})
```

We'll also need to provide a value for this blank function that actually implements how it'll update our state/the context value:

**/App.js**

```jsx
//...

import { UserContext } from './context'

//...

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <NavigationContainer>
        <Stack.Navigator screenProps headerMode='none'>
          {!user ? (
            <>
              <Stack.Screen name="Auth" component={AuthNavigator} />
            </>
          ) : (
            <>
              {/* User is signed in */}
              <Stack.Screen name="Notes" component={NotesNavigator} />
            </>
          )}
        </Stack.Navigator>
      </NavigationContainer>
    </UserContext.Provider>
  )
}
```

Now, we can get this blank function and call it from within our component to update our context value:

**/screens/HomeScreen.js**

```jsx
import React from 'react'
import { StyleSheet, Text, View } from 'react-native'
import { UserContext } from '../context'
import { TouchableOpacity } from 'react-native-gesture-handler'

const HomeScreen = () => {
  return (
    <UserContext.Consumer>
      {
        ({ user, setUser }) =>
          <View style={styles.container}>
            <Text>Welcome {user.displayName}</Text>
            <TouchableOpacity onPress={() => setUser('test')}>
              <Text>
                Reset User
              </Text>
            </TouchableOpacity>
          </View>
      }
    </UserContext.Consumer>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})

export default HomeScreen
```

Here, we just destructure the `user` and `setUser` values from the object that the consumer will provide its child function with. Then, we can use the `setUser()` function to update the value of our context.

You can think of creating the context as providing default values for all the pieces of our store and the provider as actually taking in the values/providing the values.

Now, let's start implementing our note taking features. In order to interact with Firestore, we'll need to require its SDK in our `firebaseService` module:

```jsx
import firebase from 'firebase/app'
require('firebase/auth')
require('firebase/firestore')

//...
```

At the time of writing, Firebase currently has a bug where they depend on certain `window` elements only present in the browser environment (as a result, it will break in React Native). We can fix this by adding a "polyfill" of sorts to the `App` module:

```bash
npm i base-64
```

Now, import it and substitute the values in the `App` module:

**/App.js**

```jsx
//...

import {decode, encode} from 'base-64'

if (!global.btoa) {  global.btoa = encode }

if (!global.atob) { global.atob = decode }

//...
```

Now, we'll create a new page where we'll create our notes:

**/screens/CreateNoteScreen.js**

```jsx
import React, { useState } from 'react'
import { StyleSheet, Text, View, TextInput, TouchableOpacity } from 'react-native'
import firebase from '../services/firebaseService'

const NewNoteScreen = () => {
  const [formValue, setFormValue] = useState('')
  const [error, setError] = useState()
  const [message, setMessage] = useState()

  const handleSubmit = async () => {
    firebase
      .firestore()
      .collection('notes')
      .add({
        content: formValue,
        flagged: false
      })
      .then(() => setMessage('Successfully created note!'))
      .catch(err => setError(err.message))
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        Create a Note
      </Text>

      {error ?
        <View style={styles.error}>
          <Text style={styles.errorText}>{error}</Text>
        </View>
        : null}

      {message ?
        <View style={styles.message}>
          <Text style={styles.messageText}>{message}</Text>
        </View>
        : null}

      <View style={styles.card}>
        <TextInput
          style={styles.input}
          onChangeText={text => setFormValue(text)}
          value={formValue}
        />

        <TouchableOpacity onPress={handleSubmit} style={styles.button}>
          <Text style={styles.buttonText}>Submit</Text>
        </TouchableOpacity>

      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'flex-start',
    alignItems: 'center'
  },
  title: {
    marginVertical: 80,
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center'
  },
  input: {
    borderBottomWidth: StyleSheet.hairlineWidth,
    height: 40,
    fontSize: 15,
    borderBottomColor: '#34495e',
    marginBottom: 15
  },
  button: {
    marginHorizontal: 40,
    marginVertical: 10,
    backgroundColor: '#2ecc71',
    borderRadius: 8,
    height: 60,
    alignItems: 'center',
    justifyContent: 'center'
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
    fontSize: 15
  },
  error: {
    marginBottom: 20,
    alignItems: 'center',
    justifyContent: 'center',
    marginHorizontal: 40,
    backgroundColor: '#e74c3c',
    borderRadius: 8,
    padding: 10
  },
  errorText: {
    color: 'white',
    fontWeight: 'bold'
  },
  message: {
    marginBottom: 20,
    alignItems: 'center',
    justifyContent: 'center',
    marginHorizontal: 40,
    backgroundColor: '#2ecc71',
    borderRadius: 8,
    padding: 10
  },
  messageText: {
    color: 'white',
    fontWeight: 'bold'
  },
  card:{
    padding: 15,
    width: '90%',
    shadowColor: '#000000',
    shadowOffset: {
      width: 0,
      height: 3
    },
    shadowRadius: 5,
    shadowOpacity: 0.3,
    backgroundColor: 'white',
    borderRadius: 6
  }
})

export default NewNoteScreen
```

Here, our logic pertaining to the creation of a note is very similar to how it was with our React application.

We'll also have to update our `NotesNavigator` to include our new screen:

**/App.js**

```jsx
//...

import NewNoteScreen from './screens/NewNoteScreen'

//...

const NotesNavigator = () => (
  <Tab.Navigator
    screenOptions={({ route }) => ({
      // eslint-disable-next-line react/display-name
      tabBarIcon: ({ focused, color, size }) => {
        let iconName

        if (route.name === 'Home') {
          iconName = focused
            ? 'ios-information-circle'
            : 'ios-information-circle-outline'
        } else if (route.name === 'New Note') {
          iconName = focused
            ? 'ios-add-circle'
            : 'ios-add'
        }
        return <Ionicons name={iconName} size={size} color={color} />
      },
    })}
    tabBarOptions={{
      activeTintColor: 'tomato',
      inactiveTintColor: 'gray',
    }}
  >
    <Tab.Screen name="Home" component={HomeScreen}/>
    <Tab.Screen name="New Note" component={NewNoteScreen}/>
  </Tab.Navigator>
)

//...
```

We also created a new condition related to `route.name === 'New Note'` to that we can set the `iconName` to an appropriate icon for the tab.

Our screen now looks like this:

![image-20200301155117620](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200301155117620.png)

Now, we'll just listen to the notes collection of our application and display the notes in our home screen:

**/screens/HomeScreen.js**

```jsx
import React, { useEffect, useState } from 'react'
import { StyleSheet, Text, View, Switch } from 'react-native'
import { UserContext } from '../context'
import { TouchableOpacity } from 'react-native-gesture-handler'
import firebase from '../services/firebaseService'

const HomeScreen = () => {

  const [notes, setNotes] = useState([])


  useEffect(() => {
    const unsubscribe = firebase.firestore().collection('notes')
      .onSnapshot(querySnapshot => {
        let notes = []
        querySnapshot.forEach(d => {
          let noteData = d.data()
          noteData.id = d.id
          notes.push(noteData)
        })
        setNotes(notes)
      })
    return unsubscribe
  }, [])

  const onToggle = note => {
    const noteRef = firebase.firestore().collection('notes').doc(note.id)

    noteRef.update({
      flagged: !note.flagged
    })
      .then(() => {
        console.log('Document successfully updated!')
      })
      .catch(error => {
        // The document probably doesn't exist.
        console.error('Error updating document: ', error)
      })
  }


  return (
    <UserContext.Consumer>
      {
        ({ user, setUser }) =>
          <View style={styles.container}>
            <Text style={styles.title}>{`Welcome ${user.displayName},\nHere are your notes`}</Text>

            {notes.map(note => (
              <View style={styles.card} key={note.id}>
                <Switch onChange={() => onToggle(note)} value={note.flagged} />
                <Text style={styles.noteText}>
                  {note.content}
                </Text>
              </View>
            ))}

            <TouchableOpacity style={styles.reset} onPress={() => setUser('test')}>
              <Text>
                Reset User
              </Text>
            </TouchableOpacity>
          </View>
      }
    </UserContext.Consumer>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'flex-start',
    alignItems: 'flex-start',
    marginHorizontal: 20
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginTop: '40%',
    marginBottom: '10%'
  },
  card:{
    flexDirection: 'row',
    marginTop: 10,
    padding: 15,
    width: '100%',
    shadowColor: '#000000',
    shadowOffset: {
      width: 0,
      height: 3
    },
    shadowRadius: 5,
    shadowOpacity: 0.3,
    backgroundColor: 'white',
    borderRadius: 6,
    alignItems: 'center'

  },
  noteText: {
    marginHorizontal: 10,
    width: '80%'
  },
  reset: {
    marginTop: 15
  }
})

export default HomeScreen
```

Here, again, our logic is very similar to our React application (see the pattern?). The main difference here is that we use a `Switch` component instead of an `input` element of a checkbox type. The switch behaves similarly, but takes in a `value` property rather than a `checked` property to determine whether its toggled or not.

Our application looks like this currently:

![image-20200301165407161](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200301165407161.png)

Now, let's explore one difference between these "native" applications and web applications made with React that run in our web browser. If we add too many notes to the point where our notes can no longer fit on one screen, we'll run into the following issue:

![image-20200301165703029](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200301165703029.png)

Here, our final few notes get cut off, but we have no way of scrolling to see the other notes. Unlike web applications with React, upon an overflow, a scrollbar will not appear that allows us to scroll.

We can add this scrolling behavior by introducing the components `<ScrollView>` or `<FlatList>`. `<FlatList>` takes in a list of data and lazily renders a component provided to it. This comes with drastic performance benefits with larger sets of data. `<ScrollView>` just renders all of its child components at once which comes with a performance downside. It is easier to implement, so it may work better for some smaller lists.

Let's work on implementing `<FlatList>` here. `<FlatList>` takes in a plain array through its `data` prop. It also takes in the component that we want to render in the form of a function provided to `renderItem`. The function provided to `renderItem` will take in the `item` to render, the `index` of the item if we need it, and a `separators` object (contains more metadata and allows us to change the separators of the list).

With this, we'll first refactor our note "component" into its own module as follows:

**/components/Note.js**

```jsx
import React from 'react'
import { StyleSheet, View, Switch, Text } from 'react-native'
import firebase from '../services/firebaseService'

const onToggle = note => {
  const noteRef = firebase.firestore().collection('notes').doc(note.id)

  noteRef.update({
    flagged: !note.flagged
  })
    .then(() => {
      console.log('Document successfully updated!')
    })
    .catch(error => {
      // The document probably doesn't exist.
      console.error('Error updating document: ', error)
    })
}

const Note = ({ note }) => {

  return (
    <View style={styles.card} key={note.id}>
      <Switch onChange={() => onToggle(note)} value={note.flagged} />
      <Text style={styles.noteText}>
        {note.content}
      </Text>
    </View>
  )
}

const styles = StyleSheet.create({
  card:{
    flexDirection: 'row',
    marginTop: 10,
    padding: 15,
    width: '100%',
    shadowColor: '#000000',
    shadowOffset: {
      width: 0,
      height: 3
    },
    shadowRadius: 5,
    shadowOpacity: 0.3,
    backgroundColor: 'white',
    borderRadius: 6,
    alignItems: 'center'

  },
  noteText: {
    marginHorizontal: 10,
    width: '80%'
  }
})

export default Note
```

Here, we interact with Firestore directly to update the notes. This component takes the note to render in and renders the appropriate component, complete with the ability to toggle its flagged property. We can then update our home screen to render a `<FlatList>` that renders these `Note` components:

**/screens/HomeScreen.js**

```jsx
import React, { useEffect, useState } from 'react'
import { StyleSheet, Text, View, FlatList } from 'react-native'
import { UserContext } from '../context'
import { TouchableOpacity } from 'react-native-gesture-handler'
import firebase from '../services/firebaseService'
import Note from '../components/Note'

const HomeScreen = () => {

  const [notes, setNotes] = useState([])

  useEffect(() => {
    const unsubscribe = firebase.firestore().collection('notes')
      .onSnapshot(querySnapshot => {
        let notes = []
        querySnapshot.forEach(d => {
          let noteData = d.data()
          noteData.id = d.id
          notes.push(noteData)
        })
        setNotes(notes)
      })
    return unsubscribe
  }, [])

  return (
    <UserContext.Consumer>
      {
        ({ user, setUser }) =>
          <View style={styles.container}>
            <Text style={styles.title}>{`Welcome ${user.displayName},\nHere are your notes`}</Text>

            <FlatList
              data={notes}
              style={styles.noteList}
              renderItem={({ item }) => (
                <Note
                  id={item.id}
                  note={item}
                />
              )}
            />

            <TouchableOpacity style={styles.reset} onPress={() => setUser('test')}>
              <Text>
                Reset User
              </Text>
            </TouchableOpacity>
          </View>
      }
    </UserContext.Consumer>
  )
}

//... STYLES

export default HomeScreen
```

This yields the final result of our application where we can now scroll through our notes:

![image-20200301184324117](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20200301184324117.png)

This was a survey of the power and usage of React Native. Should you continue with learning to create these native applications, there are plenty of tools and additional features out there worth exploring. You may even choose to not use Expo and just use React Native itself to get the latest React Native features.
