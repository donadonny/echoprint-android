# Echoprint Codegen Android creating and compilation instructions

Using Android Studio 2.2.2 on Windows

## Project Setup

### Echoprint Android Project

- Start a new Android Studio project
- Check C++ support
- Select platforms minimum SDK
- Select Empty Activity
- Change activity values as you need
- Select Toolchain Default for default CMake setting
- Check Exceptions Support to enable C++ exception handling
- Not check Runtime Type Information Support

  You have a first app with native C++ support
  Test build and run Android Studio demo app 

### Setup Android Library Module

#### Create Android Library Module

- Add a new Module
- Select Android Library
- Set Library name, Folder name and Package name

#### Add Echopring codegen sources

- Add cpp folder to ./lib/src/main
- Add codegen folder to ./lib/src/main/cpp
- Clone or download https://github.com/jtentor/echoprint-codegen, it is a fork from https://github.com/spotify/echoprint-codegen
- Copy all files from ./echoprint-codegen/src/ folder to ./lib/src/main/cpp/codegen/ in the Android Library project
- In ./lib/src/main/cpp/codegen folder rename all .cxx files to .cpp

#### Add Boost C++ Libraries

- Download from http://www.boost.org/ boost_1_46_1 libraries release or the last one 
- Decompress all into boost_x_xx_x folder
- You need boost headers files only
- Remove all folders except boost
- Move the folder boost_x_xx_x to cpp in the Android Library Project
  Do not be anxious they are a lot of files to index, boost_1_62_0 have 12278 files so you can use boost_1_46_1 with 7715 files
  The path ./boost_x_xx_x sometimes is referred to as $BOOST_ROOT, to compile anything in Boost you need to add this path in #include path
