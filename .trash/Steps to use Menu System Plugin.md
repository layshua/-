# Steps to Use the Menu System Plugin 

## 1. Create or open an Unreal Engine 5 project 

(For new projects, it is recommended that you use the Third Person project template, as it already has a working character which is replicated). 

## 2. Configure the project to use the Steam Online Subsystem 

- Go to Edit -> Plugins -> search for Online Subsystem Steam 

- Check Enabled 

- Click Restart Now 

- In the project folder, go to Config 

---

- Open DefaultEngine.ini 

Add the following to the bottom of DefaultEngine.ini (copy them directly from Unreal Engine’s Documentation page) 

https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/Online/Steam/ 

- Save and close. The full file should appear as follows (with your project name where it says     “MenuSystem”) 

- Open DefaultGame.ini and add: 

---

- Compile the Visual Studio Project (CTRL + SHIFT + B) 

## 3. Add the Plugin to the Project Folder 

- Copy the Plugins folder into your project: 

- Inside Plugins, you should see MultiplayerSessions 

- Close Unreal Engine and Visual Studio. Delete your Binaries, Intermediate, and Saved folders.     Right-click your .uproject and select Generate Visual Studio project files. 

- Open your Visual Studio project solution (.sln). You should now see a Plugins folder in the     Solution Explorer 

---

- Open your project in Unreal Engine. It will ask if you would like to rebuild modules. Select **Yes**. 

- In the Content Browser, click Settings. Check **Show Plugin Content**. 

- You will now have two extra folders in the Content Browser: Multiplayer Sessions Content and     Multiplayer Sessions C++ Classes. 

---

- Go to the Multiplayer Sessions Content folder. There is a Widget Blueprint called WBP_Menu. 

## 4. Create a Lobby Level 

- Create a new level, and save it in your project (it doesn’t matter where you save it or what you     name it) 

- Open your starting level. 

- Open the Level Blueprint. Use Create Widget to create a new widget, selecting **WBP_Menu** as     the widget class. Connect this to **Begin Play**. 

---

- Drag off of the Return Value output and search for Menu Setup. Select it. 

- Change the Number of Public Connections to the number of players you want in your game 

- Change the lobby path to the path to your lobby level (use /Game/ instead of /Content/) and     leave out the .umap extension. 

---

## 5. Play Test the Game 

- Go to Platforms -> <platform> -> Package Project. 

- Create a new folder or select a folder destination 

---

- Click Select Folder 

- Upload to a Google drive (or anywhere else) and download it onto a second machine 

- Run the Steam client in the background. Make sure both computers are set to the same region     (Steam -> Settings -> Downloads -> Download Region). 

---

- On one machine, launch the game and click Host 

- On the other machine, launch the game and click Join 

- Test Play! 

