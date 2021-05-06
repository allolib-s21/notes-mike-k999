# Notes from Michael K.


## Switching the default audio devices

### Switching audio output devices within the current version of allolib (as of 4/20/2021)
The current version of [allolib](https://github.com/AlloSphere-Research-Group/allolib) does not have a way of detecting changes in the number of available audio devices, or detecting a change of default device.  The only way to update the device is to either manually check for a default device change at regular intervals (which could seriously hurt performance), or have the user change the default device when desired.  I provide a method of doing the latter.

1. **Method 1 - the "hack":**
This method is deemed as a "hack" because it is a very quick solution that could cause issues.  It is however better than nothing, as an unprompted change of audio device will cause [allolib_playground](https://github.com/allolib-s21/allolib_playground) to crash anyway.  At least this method will give you a way of manually switching between devices to avoid these crashes, however it will probably be seen as a pain in any user's eyes.  Here is how to make it work:
   1. First we will need to override the `bool onKeyDown(Keyboard const& k)` function of of the [App](https://allosphere-research-group.github.io/allolib-doc/classal_1_1_app.html) class.  This code is used for handling key presses on the keyboard.  We use this function so the user is able to switch the default audio ouptut device by using keyboard shortcuts.  In order to do this, inside your class extending `al::App` we need to add the following code, if it does not already exist:
      ```cpp
      bool onKeyDown(al::Keyboard const& k) override {
        if (al::ParameterGUI::usingKeyboard()) {  // Ignore keys if GUI is using them
          return true;
        }
        // we will be adding more code here as well soon...
        // there may be existing code here
      }
      ```
   2. If the code above is already in your project, you can skip that step.  We now need to decide how the user will go about changing the default output.  I have chosen <kbd>Ctrl</kbd>+<kbd>d</kbd> for deactivate and <kbd>Ctrl</kbd>+<kbd>a</kbd> for activate.  When the user uses the shortcut to deactivate, they will disconnect their app from sound output, and when the user uses the shortcut to activate, we will find the default audio device and connect to it for output.  We will now add the next couple lines to do this, after the first if statement shown above, and within the `onKeyDown` function:
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
   3. *Caveats*: This code assumes that the new default output device you connect to has a sample rate of 48000 and has 2 audio channels.  If this is not the case the according code can be modified.  You can also compute the ascii code outside of the if statements if desired in order to use the value in all if statements.
   4. *Instructions for user use*:  When the user desires to change the ouput device, they must first deactivate their current output device (with <kbd>Ctrl</kbd>+<kbd>d</kbd>) *while it is still connected*, ensure the new device they want to connect to is the default device, then re-activate the audio output on the new device (with <kbd>Ctrl</kbd>+<kbd>a</kbd>).  If the user removes an audio device before it is deactivated the application will crash (this happens even without this code as I mentioned above, so it is better to at least have the option to deactivate it).  If the user tries to activate any device while a stream to some device is already open, allolib/RtAudio will prevent this from happening and so no issues will occur.

2. **Method 2 - the new and improved "hack":**
Rather than including the second block of code from the original "hack", you can include this code:
   ```cpp
      if(k.ctrl()) {
         int asciiCode = k.key();
         if(asciiCode == 100) {
            audioIO().close();
         }
         if(asciiCode == 97) {
            al::AudioDevice dev_out = al::AudioDevice::defaultOutput();
            configureAudio(dev_out, dev_out.defaultSampleRate(), 512, dev_out.channelsOutMax(), 0);
            audioIO().start()
         }
      }
   ```
The benefit of this code over the previous method is that it will start the current audio device while making sure we configure it with the proper parameters.  There is an example in my [alloplayground demos repo](https://github.com/allolib-s21/alloplayground-mike-k999).
You can also use a similar method for switching input devices.  (Coming soon!)

