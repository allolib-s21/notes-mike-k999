# Notes from Michael K.


## Switching the default audio devices

### Switching audio output devices within the current version of allolib (as of 4/20/2021)
The current version of [allolib](https://github.com/AlloSphere-Research-Group/allolib) does not have a way of detecting changes in the number of available audio devices, or detecting a change of default device.  The only way to update the device is to either manually check for a default device change at regular intervals (which could seriously hurt performance), or have the user change the default device when desired.  I provide a method of doing the latter.

1. **Method 1: the "hack":**
This method is deemed as a "hack" because it is a very quick solution that could cause issues.  It is however better than nothing, as an unprompted change of audio device will cause [allolib_playground](https://github.com/allolib-s21/allolib_playground) to crash anyway.  At least this method will give you a way of manually switching between devices to avoid these crashes, however it will probably be seen as a pain in any user's eyes.  First we go over the code to be added.
   1. First we will need to override the `bool onKeyDown(Keyboard const& k)` function of of the [App](https://allosphere-research-group.github.io/allolib-doc/classal_1_1_app.html) class.  In order to do this, inside your class extending `al::App` we need to add the following code, if it does not already exist:
      ```cpp
      bool onKeyDown(Keyboard const& k) override {
        if (ParameterGUI::usingKeyboard()) {  // Ignore keys if GUI is using them
          return true;
        }
        // there may be existing code here
        // we will be adding more code here as well soon...
      }
      ```
This code is used for handling key presses on the keyboard.  In this method the user is able to switch the default audio ouptut device by using keyboard shortcuts.
   
   2. 

   
   2. If the code above is already in your project, you can skip that step.  We will now add these next couple lines after the first if statement shown above, and within the `onKeyDown` function:
    ```cpp
      if(k.ctrl()) {
        int asciiCode = k.key();
        if(asciiCode == 100) { // ctrl-d pressed => deactivate current output device
          audioIO().close();
        }
        if(asciiCode == 97) { //ctrl-a pressed => activate the current default output device
          configureAudio(48000., 512, 2, 0);
          audioIO().start();
        }
      }
    ```
    
