### 1. Project Setup

1. **Create a New Android Project**

   - In Android Studio, create a new project using the `Empty Views Activity` template, ensuring `Kotlin` is selected as the language.

2. **Enable View Binding**

   - In your `build.gradle.kts (Module: app)` file, enable View Binding using the recommended syntax:

     ```kotlin
     android {
         ...
         buildFeatures {
             viewBinding = true
         }
     }
     ```

3. **Add Dependencies**

   - Ensure the dependencies for the Navigation Component and ViewModel are correctly set up.
   - Update `libs.versions.toml` with:

     ```toml
     [versions]
     androidx-navigation = "2.9.4"
     androidx-lifecycle = "2.9.3"

     [libraries]
     androidx-navigation-fragment-ktx = { module = "androidx.navigation:navigation-fragment-ktx", version.ref = "androidx-navigation" }
     androidx-navigation-ui-ktx = { module = "androidx.navigation:navigation-ui-ktx", version.ref = "androidx-navigation" }
     androidx-lifecycle-viewmodel-ktx = { module = "androidx.lifecycle:lifecycle-viewmodel-ktx", version.ref = "androidx-lifecycle" }
     ```

   - In `build.gradle.kts (Module: app)`, add:

     ```kotlin
     dependencies {
         // ...
         // Navigation Component dependencies
         implementation(libs.androidx.navigation.fragment.ktx)
         implementation(libs.androidx.navigation.ui.ktx)

         // ViewModel dependencies
         implementation(libs.androidx.lifecycle.viewmodel.ktx)
         // ...
     }
     ```

### 2. Define the App's Navigation

1. **Add a Navigation Graph**

   - Create a navigation resource file `nav_graph.xml` under `res/navigation`.
   - Add two fragments (`LandingFragment` and `GameFragment`) to the navigation graph.

   Here’s a basic layout for the navigation graph:

   ```xml
   <navigation xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       app:startDestination="@id/landingFragment">

       <!-- TODO: Update the package paths accordingly -->
       <fragment
           android:id="@+id/landingFragment"
           android:name="pt.ipleiria.worksheet_spike_views.LandingFragment"
           android:label="LandingFragment"
           tools:layout="@layout/fragment_landing">
           <action
               android:id="@+id/action_landingFragment_to_gameFragment"
               app:destination="@id/gameFragment" />
       </fragment>

       <fragment
           android:id="@+id/gameFragment"
           android:name="pt.ipleiria.worksheet_spike_views.GameFragment"
           android:label="GameFragment"
           tools:layout="@layout/fragment_game" />
   </navigation>
   ```

2. **Add a Navigation Host to MainActivity**

   - In `activity_main.xml`, add a `NavHostFragment` to host your navigation.

   ```xml
   <androidx.fragment.app.FragmentContainerView
       android:id="@+id/nav_host_fragment"
       android:name="androidx.navigation.fragment.NavHostFragment"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       app:navGraph="@navigation/nav_graph"
       app:defaultNavHost="true" />
   ```

### 3. Implement the ViewModel

1. **Create a ViewModel to Store the Game State**

   - Create a `GameViewModel` class to track the number of games played and won.

   ```kotlin
   import androidx.lifecycle.LiveData
   import androidx.lifecycle.MutableLiveData
   import androidx.lifecycle.ViewModel
   
   class GameViewModel : ViewModel() {
       private var _playedGames = MutableLiveData(0)
       val playedGames: LiveData<Int> = _playedGames

       private var _wonGames = MutableLiveData(0)
       val wonGames: LiveData<Int> = _wonGames

       fun incrementPlayedGames() {
           _playedGames.value = (_playedGames.value ?: 0) + 1
       }

       fun incrementWonGames() {
           _wonGames.value = (_wonGames.value ?: 0) + 1
       }
   }
   ```

### 4. Create the Landing Fragment

1. **Create the UI for `LandingFragment`**

   - Create a layout file `fragment_landing.xml` with a `Button` to start the game and a `TextView` to display game statistics.

   ```xml
   <LinearLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:orientation="vertical"
       android:gravity="center">

       <TextView
           android:id="@+id/game_stats"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="Played games: 0 | Won games: 0" />

       <Button
           android:id="@+id/play_button"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="Play" />
   </LinearLayout>
   ```

2. **Implement Logic in `LandingFragment`**

   - In `LandingFragment.kt`, observe the `GameViewModel` to update the stats.

   ```kotlin
   import android.os.Bundle
   import androidx.fragment.app.Fragment
   import android.view.LayoutInflater
   import android.view.View
   import android.view.ViewGroup
   import androidx.fragment.app.activityViewModels
   import androidx.navigation.fragment.findNavController
   
   class LandingFragment : Fragment() {

       private val viewModel: GameViewModel by activityViewModels()

       override fun onCreateView(
           inflater: LayoutInflater, container: ViewGroup?,
           savedInstanceState: Bundle?
       ): View? {
           val binding = FragmentLandingBinding.inflate(inflater, container, false)

           viewModel.playedGames.observe(viewLifecycleOwner) { played ->
               updateGameStats(binding, played, viewModel.wonGames.value ?: 0)
           }

           viewModel.wonGames.observe(viewLifecycleOwner) { won ->
               updateGameStats(binding, viewModel.playedGames.value ?: 0, won)
           }

           binding.playButton.setOnClickListener {
               viewModel.incrementPlayedGames()
               // Navigate to the game fragment
               findNavController().navigate(R.id.action_landingFragment_to_gameFragment)
           }

           return binding.root
       }

       private fun updateGameStats(binding: FragmentLandingBinding, played: Int, won: Int) {
           binding.gameStats.text = "Played games: $played | Won games: $won"
       }
   }
   ```

### 5. Create the Game Fragment

1. **Create the UI for `GameFragment`**

   - Create a layout file `fragment_game.xml` with two `ToggleButtons` to reveal images.

   ```xml
   <LinearLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:orientation="horizontal"
       android:gravity="center">

       <ToggleButton
           android:id="@+id/toggleButton1"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content" />

       <ToggleButton
           android:id="@+id/toggleButton2"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content" />
   </LinearLayout>
   ```

2. **Implement Logic in `GameFragment`**

   - In `GameFragment.kt`, check if both buttons are toggled to increment the number of games won.

   ```kotlin
   import android.os.Bundle
   import androidx.fragment.app.Fragment
   import android.view.LayoutInflater
   import android.view.View
   import android.view.ViewGroup
   import androidx.fragment.app.activityViewModels
   import androidx.navigation.fragment.findNavController
   
   class GameFragment : Fragment() {

       private val viewModel: GameViewModel by activityViewModels()

       override fun onCreateView(
           inflater: LayoutInflater, container: ViewGroup?,
           savedInstanceState: Bundle?
       ): View? {
           val binding = FragmentGameBinding.inflate(inflater, container, false)

           binding.toggleButton1.setOnCheckedChangeListener { _, isChecked1 ->
               val isChecked2 = binding.toggleButton2.isChecked
               if (isChecked1 && isChecked2) {
                   viewModel.incrementWonGames()
                   findNavController().popBackStack()
               }
           }

           binding.toggleButton2.setOnCheckedChangeListener { _, isChecked2 ->
               val isChecked1 = binding.toggleButton1.isChecked
               if (isChecked1 && isChecked2) {
                   viewModel.incrementWonGames()
                   findNavController().popBackStack()
               }
           }

           return binding.root
       }
   }
   ```

### 6. To Learn More

- Developing Android Apps with Kotlin
    - https://www.udacity.com/enrollment/ud9012
    - Udacity course on developing Android apps in the Kotlin programming language.

