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

#### Add source code to access java native interface

- Add java class Codegen.java

        package org.android.echoprint.lib;
        
        /**
         * Created by jtentor on 22/10/2016.
         */
        
        /**
         * Codegen.java
         *
         * Created by Alex Restrepo on 1/22/12.
         * Copyright (C) 2012 Grand Valley State University (http://masl.cis.gvsu.edu/)
         *
         * Permission is hereby granted, free of charge, to any person obtaining a copy of
         * this software and associated documentation files (the "Software"), to deal in
         * the Software without restriction, including without limitation the rights to
         * use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
         * of the Software, and to permit persons to whom the Software is furnished to do
         * so, subject to the following conditions:
         *
         * The above copyright notice and this permission notice shall be included in all
         * copies or substantial portions of the Software.
         *
         * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
         * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
         * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
         * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
         * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
         * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
         * SOFTWARE.
         */
        
        /**
         * Codegen class<br>
         * This class bridges the native Codegen library with the Java side...
         *
         * @author Alex Restrepo (MASL)
         *
         */
        public class Codegen
        {
            private final float normalizingValue = Short.MAX_VALUE;
        
            native String codegen(float data[], int numSamples);
        
            static
            {
                System.loadLibrary("echoprint-android-lib");
            }
        
            /**
             * Invoke the echoprint native library and generate the fingerprint code.<br>
             * Echoprint REQUIRES PCM encoded audio with the following parameters:<br>
             * Frequency: 11025 khz<br>
             * Data: MONO - PCM enconded float array
             *
             * @param data PCM encoded data as floats [-1, 1]
             * @param numSamples number of PCM samples at 11025 KHz
             * @return The generated fingerprint as a compressed - base64 string.
             */
            public String generate(float data[], int numSamples)
            {
                return codegen(data, numSamples);
            }
        
            /**
             * Invoke the echoprint native library and generate the fingerprint code.<br>
             * Since echoprint requires the audio data to be an array of floats in the<br>
             * range [-1, 1] this method will normalize the data array transforming the<br>
             * 16 bit signed shorts into floats.
             *
             * @param data PCM encoded data as shorts
             * @param numSamples number of PCM samples at 11025 KHz
             * @return The generated fingerprint as a compressed - base64 string.
             */
            public String generate(short data[], int numSamples)
            {
                // echoprint expects data as floats, which is the native value for
                // core audio data, and I guess ffmpeg
                // Android records data as 16 bit shorts, so we need to normalize the
                // data before sending it to echoprint
                float normalizeAudioData[] = new float[numSamples];
                for (int i = 0; i < numSamples - 1; i++)
                    normalizeAudioData[i] = data[i] / normalizingValue;
        
                return this.codegen(normalizeAudioData, numSamples);
            }
        }

- Find $JDKPath in File | Other Settings | Default Project Structure...
  It could be C:\Program Files\Android\Android Studio\jre
- Open a cmd console in ./lib/src/main/java to generate header file needed
- To generate header file in ./cpp folder execute:

        "C:\Program Files\Android\Android Studio\jre\bin\javah.exe" -d ../cpp org.android.echoprint.lib.Codegen

- Add C++ code in ./lib/src/main/cpp 

        //
        // Created by jtentor on 22/10/2016.
        //
        
        /**
         * edu_gvsu_masl_echoprint_Codegen.cpp
         * jni
         *
         * Created by Alex Restrepo on 1/22/12.
         * Copyright (C) 2012 Grand Valley State University (http://masl.cis.gvsu.edu/)
         *
         * Permission is hereby granted, free of charge, to any person obtaining a copy of
         * this software and associated documentation files (the "Software"), to deal in
         * the Software without restriction, including without limitation the rights to
         * use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
         * of the Software, and to permit persons to whom the Software is furnished to do
         * so, subject to the following conditions:
         *
         * The above copyright notice and this permission notice shall be included in all
         * copies or substantial portions of the Software.
         *
         * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
         * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
         * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
         * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
         * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
         * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
         * SOFTWARE.
         */
        
        //#include <android/log.h>
        #include <string.h>
        #include <jni.h>
        #include "org_android_echoprint_lib_Codegen.h"
        #include "codegen/Codegen.h"
        
        JNIEXPORT jstring JNICALL Java_org_android_echoprint_lib_Codegen_codegen
                (JNIEnv *env, jobject thiz, jfloatArray pcmData, jint numSamples)
        {
            // get the contents of the java array as native floats
            float *data = (float *)env->GetFloatArrayElements(pcmData, 0);
        
            // invoke the codegen
            Codegen c = Codegen(data, (unsigned int)numSamples, 0);
            const char *code = c.getCodeString().c_str();
        
            // release the native array as we're done with them
            env->ReleaseFloatArrayElements(pcmData, data, 0);
        
            // return the fingerprint string
            return env->NewStringUTF(code);
        }


#### Add CMakeList.txt to Android Library Module and link with gradle

. Edit CMakeList.txt with

        # Sets the minimum version of CMake required to build
        
        cmake_minimum_required(VERSION 3.4.1)
        
        # Creates and names a library, sets it as STATIC or SHARED
        # Provides the relative paths to its source code.
        
        add_library( # Sets the name of the library.
                     echoprint-android-lib
        
                     # Sets the library as a shared library.
                     SHARED
        
                     # Provides a relative path to your source file(s).
                     # Associated headers in the same location as their source
                     # file are automatically included.
                     src/main/cpp/AndroidCodegen.cpp
                     src/main/cpp/codegen/Codegen.cpp
                     src/main/cpp/codegen/Whitening.cpp
                     src/main/cpp/codegen/SubbandAnalysis.cpp
                     src/main/cpp/codegen/MatrixUtility.cpp
                     src/main/cpp/codegen/Fingerprint.cpp
                     src/main/cpp/codegen/Base64.cpp
                     src/main/cpp/codegen/AudioStreamInput.cpp
                     src/main/cpp/codegen/AudioBufferInput.cpp
                   )
        
        # Provides a relative path to header file(s)
        
        include_directories( # Specifies a path to header file(s).
                             src/main/cpp/codegen/
                             src/main/cpp/boost_1_46_1/
                           )
        
        # Searches for a specified prebuilt library, stores the path as a variable
        
        find_library( # Sets the name of the path variable.
                      z-lib
        
                      # Specifies the name of the NDK library that
                      # you want CMake to locate.
                      z
                    )
        
        # Specifies libraries CMake should link to your target library. You
        # can link multiple libraries, such as libraries you define in the
        # build script, prebuilt third-party libraries, or system libraries.
        
        target_link_libraries( # Specifies the target library.
                               echoprint-android-lib
        
                               # Links the target library to the log library
                               # included in the NDK.
                               ${z-lib}
                             )

 
. Edit build.gradle in ./lib/ to Link c++ project with gradle 
   add entries **externalNativeBuild { cmake { ...** for cppFlags "-fexceptions" and path "CMakeLists.txt" values

        apply plugin: 'com.android.library'
        
        android {
            compileSdkVersion 24
            buildToolsVersion "24.0.3"
        
            defaultConfig {
                minSdkVersion 21
                targetSdkVersion 24
                versionCode 1
                versionName "1.0"
        
                testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
                externalNativeBuild {
                    cmake {
                        cppFlags "-fexceptions"
                    }
                }
        
            }
            buildTypes {
                release {
                    minifyEnabled false
                    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
                }
            }
            externalNativeBuild {
                cmake {
                    path "CMakeLists.txt"
                }
            }
        }
        
        dependencies {
            compile fileTree(dir: 'libs', include: ['*.jar'])
            androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
                exclude group: 'com.android.support', module: 'support-annotations'
            })
            compile 'com.android.support:appcompat-v7:24.2.1'
            testCompile 'junit:junit:4.12'
        }

. Sync, do not warry it will take a lot of time to index all files and build symbols 
. Build the project

