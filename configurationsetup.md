### Instructions for Configuring Firebase WebGL Authentication

**Goal:** Ensure the Firebase JavaScript SDK is correctly initialized with the Web App configuration when the Unity WebGL build runs in a browser, enabling Firebase Authentication (e.g., Anonymous Sign-In) to function correctly.

**Context:** The project uses the Firebase C# SDK, which for WebGL builds, relies on the underlying Firebase JavaScript SDK. The JS SDK must be initialized using the configuration from a registered Firebase Web App.

**Firebase Web App Configuration (Provided by User):**

```js
const firebaseConfig = {
  apiKey: "AIzaSyBUcnIgG9n-NfAmJeKpmpKY68uUQ_qYUOA",
  authDomain: "cryptonclicker.firebaseapp.com",
  databaseURL: "https://cryptonclicker-default-rtdb.firebaseio.com",
  projectId: "cryptonclicker",
  storageBucket: "cryptonclicker.firebasestorage.app", // Verify this matches the exact console output if issues arise
  messagingSenderId: "583959164948",
  appId: "1:583959164948:web:8284bc1473c710efbacf01",
  measurementId: "G-LEGVWJ64T1" // Optional
};
```

**Steps:**

1.  **Create Custom WebGL Template Directory:**
    *   Verify if the directory `Assets/WebGLTemplates` exists in the Unity project. If not, create it.
    *   Inside `Assets/WebGLTemplates`, create a new subdirectory named `FirebaseTemplate`.

2.  **Copy Base Template Files:**
    *   Locate the default Unity WebGL templates. The path is typically `[UnityEditorInstallationPath]/Editor/Data/PlaybackEngines/WebGLSupport/BuildTools/WebGLTemplates/`.
    *   Copy the `index.html` file (and any other files like `css` or `images` if using the `Default` template) from either the `Minimal` or `Default` template folder into the newly created `Assets/WebGLTemplates/FirebaseTemplate` directory.

3.  **Edit `index.html`:**
    *   Open the file `Assets/WebGLTemplates/FirebaseTemplate/index.html` for editing.
    *   Locate the `<script>` tag responsible for loading the Unity game build. It will likely look similar to `<script src="{{{ LOADER_FILENAME }}}"></script>`.
    *   **Immediately before** this Unity loader script tag, insert the following HTML block:

        ```html
        <!-- Firebase JS SDK Includes (Compat versions for Unity Wrapper) -->
        <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-app-compat.js"></script>
        <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-auth-compat.js"></script>
        <!-- Optional: Add firebase-database-compat.js if direct JS DB access is needed -->
        <!-- <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-database-compat.js"></script> -->

        <!-- Firebase Initialization Script -->
        <script>
          // --- Firebase Configuration ---
          const firebaseConfig = {
            apiKey: "AIzaSyBUcnIgG9n-NfAmJeKpmpKY68uUQ_qYUOA", // Use the actual key provided
            authDomain: "cryptonclicker.firebaseapp.com",
            databaseURL: "https://cryptonclicker-default-rtdb.firebaseio.com",
            projectId: "cryptonclicker",
            storageBucket: "cryptonclicker.firebasestorage.app", // Use the actual bucket provided
            messagingSenderId: "583959164948",
            appId: "1:583959164948:web:8284bc1473c710efbacf01",
            measurementId: "G-LEGVWJ64T1" // Optional
          };

          // --- Initialize Firebase ---
          try {
            firebase.initializeApp(firebaseConfig);
            console.log("Firebase JS SDK Initialized from Template");
            // Optional: Initialize Analytics if needed and included
            // if (typeof firebase.analytics === 'function') {
            //   firebase.analytics();
            // }
          } catch (e) {
            console.error("Error initializing Firebase from template:", e);
          }
        </script>
        <!-- End Firebase Initialization -->
        ```

    *   Ensure the `firebaseConfig` object within the script matches the user-provided configuration exactly.

4.  **Save `index.html`**.

5.  **Select Template in Unity Build Settings:**
    *   Open the Unity Editor.
    *   Navigate to `Edit -> Project Settings`.
    *   Select the `Player` category from the left-hand list.
    *   Switch to the `WebGL` platform tab (icon that looks like a globe).
    *   Expand the `Resolution and Presentation` section.
    *   From the `WebGL Template` dropdown menu, select `FirebaseTemplate`.

6.  **Build and Test:**
    *   Save the Unity Project Settings.
    *   Perform a new WebGL build (`File -> Build Settings... -> Select WebGL -> Build`).
    *   Deploy the build output to the hosting environment (e.g., GitHub Pages).
    *   Test the application in a browser. Open the developer console (F12) to check for the "Firebase JS SDK Initialized from Template" message and monitor for any Firebase-related errors during the game's startup and authentication sequence. Ensure anonymous sign-in works as expected.
