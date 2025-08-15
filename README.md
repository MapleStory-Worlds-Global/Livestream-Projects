# Livestream-Projects

## Table of Contents
- [Overview](#overview)
- [Mod File Import Instructions](#mod-file-import-instructions)
- [ModPackage file Import Instructions](#modpackage-file-import-instructions)
- [Downloads](#download)
---

## Overview
This repository provides various feature packages for use in MapleStory Worlds (MSW).

Each project folder includes a .mod file along with the mlua source files used in the package.
You can download any package you need and import it into your MSW project.

See the README file in each package for detailed usage information.

## Mod File Import Instructions

### What is a Mod File?
A mod file contains a copy of a full project that can be imported into a MapleStory Worlds project. When importing a .mod file, it will import the entire project at the state that it was when it was created, but will overwrite all the files of your current project. 

### Create a new Project
Create a new project in MapleStory Worlds through "Create" > "Create New" using any template  (The project will be overwritten by the project files.)
![Import Mod](https://github.com/MapleStory-Worlds-Global/GithubAssets/blob/2def2cfbc9b9f5465d55c411be8516db4f1fbac9/HowToImportModProject/HowTo_1.png "Create a new Project")
![Import Mod](https://github.com/MapleStory-Worlds-Global/GithubAssets/blob/2def2cfbc9b9f5465d55c411be8516db4f1fbac9/HowToImportModProject/HowTo_2.png "Choose any template")

### Download the .mod File
Click download on your desired project in the download section to download the mod file associated with the project. 

### Import the .mod File to Maker Project
Import the .mod file by navigating to File > Import from File to import the project file and overwrite the current project with the project downloaded in the previous step. 
![Import Mod](https://github.com/MapleStory-Worlds-Global/GithubAssets/blob/2def2cfbc9b9f5465d55c411be8516db4f1fbac9/HowToImportModProject/HowTo_4.png "Import the mod into your new project")

### Follow the readme instructions 
Reference the project page found in the associated github project folder for project specific instructions or additional info (Link to project page also found in the download section).
 
## ModPackage File Import Instructions

### What is a ModPackage File?
A mod package file, as the name suggests, is a package of certain scripts, assets, models, etc from a certain MapleStory Worlds Project. A mod package does not share the entire project, but rather allows the sharing of parts of your project with other creators, or with your other projects. 

### Download the .modpackage file 
Download the modpackage file from the download section of the github repository. 

### Import the .modpackage file 
Import the .modpackage file into the project you wish to import. 
![Import Mod](https://github.com/MapleStory-Worlds-Global/GithubAssets/blob/7fe64477e9e49f86623a5bba007bc62bf78e0d31/HowToImportModPackage/HowTo_2.png "Import the package")

### Select Assets for import 
Select which assets from the package that you wish to import. 
As a rule of thumb, if you're importing a package from MapleStory-Worlds-Global repositories, the assets in the "Core" folder will be the critical main logic behind the project, and the "Sample" is an optional implementation of the assets in the "Core" folder. 
![Import Mod](https://github.com/MapleStory-Worlds-Global/GithubAssets/blob/7fe64477e9e49f86623a5bba007bc62bf78e0d31/HowToImportModPackage/HowTo_3.png "Select import assets")

### Follow the readme instructions 
Reference the project page found in the associated github project folder for project specific instructions or additional info (Link to project page also found in the download section).

## Download

| Projects | Description | Documentation | Project Files Download |
|---------|-------------|---------|--------------------|
| Combat Demo | Learn the basics of combat with MapleStory Worlds! | [View](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/tree/main/CombatDemo) | [Download Mod](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CombatDemo/CombatDemo_V1.modpackage) | 
| Cutscene Demo | Learn to implement cutscene sequences of your very own! | [View](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/tree/main/CutsceneDemo) | [Download Mod](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CutsceneDemo/CutsceneDemo_V1.modpackage) |
| Character Animations Demo | Learn to work with animations using MapleStory Worlds! | [View](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/tree/main/CharacterAnimationsDemo) | [Download Mod](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CharacterAnimationsDemo/StateAnimationDemo_V1.modpackage) |

## License

This project is licensed under the MIT License.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the LICENSE.(https://opensource.org/licenses/MIT)
