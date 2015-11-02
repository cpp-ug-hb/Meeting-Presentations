class: center, middle

# CMake: Von Hello World zum <br> modularen Softwaresystem

Finn Haedicke<br>
C++ User Group Meeting Bremen - 2015-03-26

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
  <img alt="Creative Commons Lizenzvertrag" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
</a>


.small[
  Hilfe: `?`, Notizen: `p`
]

???

# Korrekturen nach dem Talk

* `install( ... INCLUDES DESTINATION )` ist einfacher als target_include_directories
* Portable installation der CMake files
* Fehlende Boost_INCLUDE_DIRS hinzugefügt


---

# Agenda

* CMake

* Hello World

* Eigene Pakete

???

* Nicht genug Zeit um CMake komplett abzudecken
* Am Beispiel von einem simplen projekt zum Aufteilen in mehrere Projekte.


---

# Was ist CMake?

* Cross Plattform Build System
* Generiert Native Projekte (Make, Ninja, Xcode, Visual Studio, ...)
* Abhängigkeiten und Modularisierung
* Cross-Compilation
* Erweiterbar
* Gut für Qt 5


* Bauen: cmake, ccmake, cmake-gui
* Testen: ctest
* Packen: cpack
* http://www.cmake.org/
* http://www.cmake.org/documentation/

???

* Wenn etwas aus dem Talk mitnehmen, dann: `ninja` ausprobieren
* Qt5 Integration der Libraries, MOC, Resources, ...
* Gutes Dependency tracking cpp -> header, target -> target, ...

---

# CMake Alternativen?

.pull-left[
## Generatoren
* Autotools
* qmake
* gyp
]
.pull-right[
## All-in-one
* scons
* jam
* waf
]

.pull-left[
## Builder
* make
* ninja
* nmake
* jom
]

.pull-right[
## IDEs
* QtCreator
* Visual Studio
* Eclipse
* CLion
]


???

Generatoren
* Autotools kann nur Unix und Make
* QMake kann nur Make
* gyp ähnliche features wie CMake, verwendet für Google Chrome, V8 (noch)

All-in-one
* Habe ich wenig mit gearbeitet
* Jam wird von Boost verwendet.

Builder
* Wer möchte große Makefiles von Hand schreiben, daher Generatoren
* Ninja hat das als Philosophie


IDEs
* Portierbare Konfigurationen?
* Funktioniert nur, wenn alle die gleiche nehmen.
* Continuous Integration?

---

# CMake mit einer Datei

* `CMakeLists.txt`
* `main.cpp`

```cpp
int main() {
  std::cout << "Hello World" << std::endl;
  return 0;
}
```

```cmake
cmake_minimum_required( VERSION 3.1 )
project( main )

add_executable( hello main.cpp )
install( TARGETS hello DESTINATION bin )
```

```sh
mkdir build  && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make install
```

???

* Bauen außerhalb des Source Ordner.
  * Mehrere Builds mit einem Source Mögliche (Debug/Release oder Compiler)
* CMake baut nicht selber sondern generiert Makefiles
* Auf Linux/Unix ist wird per default mit Make gebaut und einem Compiler aus dem PATH (cc/gcc)
* Default install prefix: /usr/local/

---

# CMake Parameter

* `-DCMAKE_BUILD_TYPE=...`
  * Release: `-O3 -DNDEBUG`
  * Debug: `-g`
  * MinSizeRel: `-Os -DNDEBUG`
  * RelWithDebInfo: `-O2 -g -DNDEBUG`
* `-DCMAKE_INSTALL_PREFIX=...`
* `-DCMAKE_CXX_COMPILER=...`, `-DCMAKE_C_COMPILER`
  * oder `export CC=...`, `export CXX=...`


Alles über cmake-gui oder ccmake änderbar (außer Compiler)

???

* Auf anderen Compilern die entsprechenden flags


---

# CMake Generatoren

`-G "<Generator>"`
.compactlist[
* Borland Makefiles
* MSYS Makefiles
* MinGW Makefiles
* NMake Makefiles
* NMake Makefiles JOM
* **Ninja**
* **Unix Makefiles**
* Watcom WMake
* Visual Studio 6
* Visual Studio 7
* Visual Studio 7 .NET 2003
* Visual Studio 8 2005
* Visual Studio 9 2008
* Visual Studio 10 2010
* Visual Studio 11 2012
* Visual Studio 12 2013
* **Visual Studio 14 2015**
* **Xcode**
* CodeBlocks
* CodeLite
* Eclipse CDT4
* KDevelop3
* Kate
* Sublime Text 2
]

???

* Probiert mal Windows-Executables mit Ninja zu bauen.
* Muss nicht angegeben werden, Nur wenn default nicht genutzt werden soll.
* Wie Compiler, muss beim ersten cmake lauf angegeben werden. Zum Testen: neuen build Ordner nehmen.

---

# CMake mit mehreren Dateien

* `CMakeLists.txt`
* `main.cpp`
* `print.cpp`
* `print.hpp`

```cpp
*#include "print.hpp"

int main() {
  print();
  return 0;
}
```

```cmake
...
add_executable( hello
  main.cpp
  print.cpp
  print.hpp    ## optional, included for IDE
)
```


???

* header optional
* Endung `.cpp` optional

---

# CMake mit eigenen Libraries

* `CMakeLists.txt`
* `main.cpp`
* `libprint/`
  * `CMakeLists.txt`
  * `print.hpp`
  * `print.cpp`


```cpp
*#include <print.hpp>

int main() {
  print();
  return 0;
}
```

???

* Eine CMakeLists.txt pro Ordner
* Verweis auf Dateien in Unterordner auch möglich. Aber nicht im sinne des Beispiels.

---

# CMake mit eigenen Libraries (2)

```cmake
cmake_minimum_required(VERSION 3.1)
project( main )

add_subdirectory( libprint )

add_executable( hello main.cpp )
*target_link_libraries( hello print )
install( TARGETS hello DESTINATION bin )
```

```cmake
add_library( print STATIC
  print.cpp
  print.hpp
)
*target_include_directories( print PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} )
```

???

* add_subdirectory um Weitere Ordner mit CMakeLists.txt zu bauen.
* add_library kann auch SHARED, INTERFACE, OBJECT oder MODULE sein.
* (oder IMPORTED um aus bestehenden Libraries ein CMake Target zu machen.)
* für target `print` setzen wir das include_dir für alle die das Target verwenden.

---

# Target flags


* target_link_libraries - Welche Libraries dieses Target braucht
* target_include_directories - Welch Includes dieses Target braucht
* target_compile_definitions - Welch Defines dieses Target braucht


* PRIVATE - nur für dieses Target
* INTERFACE - nur für andere Targets, die das Target verwenden.
* PUBLIC - für beide

_seit CMake 2.8.12_

???

* Besser als include_directories und link_libraries weil die Abhängigkeiten
  lokal definiert werden.
* Vor 2.8.12 waren PRIVATE, PUBLIC und INTERFACE komplizierter zu definieren.
* Kann für Windows DLL import/export genutzt werden;
  PRIVATE define für dllexport setzen, INTERFACE (oder unset) für dllimport.


---

# Mehrere Projekte - find_package

Aus Ordner `libprint/` wird Projekt `Print`
* `CMakeLists.txt`
* `PrintConfig.cmake.in`
* `print.cpp`
* `print.hpp`


```cmake
cmake_minimum_required(VERSION 3.1)
project( main )

*find_package( Print 1.0 REQUIRED )

add_executable( hello main.cpp )
target_link_libraries( hello print )
install( TARGETS hello DESTINATION bin )
```


???

* Projekt ist schon recht groß geworden. Lieber aufteilen...
* Aus add_subdirectory() wird find_package(...)
* Rest bleibt
* Wie geht das?


---

# Mehrere Projekte - install

* Include für Export setzen
* Library Exportieren

``` cmake
cmake_minimum_required(VERSION 3.0)
project(Print)


add_library( print STATIC print.cpp print.hpp)

install( TARGETS print DESTINATION lib
* EXPORT Print-export
* INCLUDES DESTINATION include
)
install( FILES print.hpp DESTINATION include)

```

???

* INSTALL_INTERFACE hinzugefügt, nur aktiv wenn Target installiert und exportiert wurde.
* install() mit EXPORT, damit target exportiert werden kann.

---

# Mehrere Projekte - Config und Export

``` cmake
include(CMakePackageConfigHelpers)

if(UNIX)
  set(CONFIG_DIR lib/Print/cmake)
else()
  set(CONFIG_DIR CMake)
endif()

configure_package_config_file( PrintConfig.cmake.in
  ${CMAKE_BINARY_DIR}/PrintConfig.cmake
  INSTALL_DESTINATION lib/Print/cmake)

write_basic_package_version_file(
  "${CMAKE_BINARY_DIR}/PrintConfigVersion.cmake"
*  VERSION 1.0 COMPATIBILITY SameMajorVersion)

install( FILES
    ${CMAKE_BINARY_DIR}/PrintConfig.cmake
    ${CMAKE_BINARY_DIR}/PrintConfigVersion.cmake
  DESTINATION lib/Print/cmake
)

*install( EXPORT Print-export DESTINATION lib/Print/cmake)
```

???

* unter Linux/Unix und Windows sucht CMake die Paket-Config in
  unterschiedlichen Verzeichnissen.
* PrintConfig.cmake generieren, lädt die exportierten targets
* PrintConfigVersion.cmake generieren, prüft ob die Version kompatibel ist
* beide und die exports nach `lib/Print/cmake` installieren


---

# Worüber wir nicht gesprochen haben

.pull-left[
* Testen mit CTest
* Installer mit CPack
* CMake die Scriptsprache
* Shared Libraries
* Functionen und Module
* ExternalProject
* Portable Commands mit `cmake -E`
* Object Libraries
* Generator Expressions
]

.pull-right[
* Properties
* Toolchains / Cross-Compilation
* Platform Checks
* find_file, find_library and find_program
* add_custom_target und add_custom_command
* CCacheFile.txt
* CDash
]


---

class: center, middle

# Ende

---

class: center, middle

# Bonus Round

.compactlist[
* [PrintConfig.cmake.in](#PrintConfig)
* [C++11/14](#cpp11)
* [ExternalProject](#externalproject)
* [find_package(...)](#find-modules)
* [find_package(Boost)](#find-boost)
* [cmake -E](#cmake-E)
]

---
name: PrintConfig
# PrintConfig.cmake.in

```cmake
@PACKAGE_INIT@

include(${CMAKE_CURRENT_LIST_DIR}/Print-export.cmake)
```

---
name: cpp11

# C++ 11/14

C++ Standart setzen:
* Explicit
  ```cmake
  set_property(TARGET main PROPERTY CXX_STANDARD 11)
  ```
* oder Global
  ```cmake
  set(CMAKE_CXX_STANDARD 11)
  ```

* oder compile features benutzen
  ```cmake
  target_compile_features(main PUBLIC cxx_auto_type cxx_lambdas)
  ```

???

http://www.cmake.org/cmake/help/v3.2/manual/cmake-compile-features.7.html


---
name: externalproject
# Mehrere Projekte Bauen - ExternalProject

```cmake
include(ExternalProject)

ExternalProject_Add(
  Print
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/Print
  INSTALL_DIR ${CMAKE_BINARY_DIR}/install/Print
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install/main
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
)

ExternalProject_Add(
  main
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/main
  INSTALL_DIR ${CMAKE_BINARY_DIR}/install/main
  CMAKE_ARGS
    -DCMAKE_PREFIX_PATH=${CMAKE_BINARY_DIR}/install/Print
    -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install/main
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
  DEPENDS Print
)
```

???
http://www.cmake.org/cmake/help/v3.2/module/ExternalProject.html


---
name: find-modules
# find_package für externe libraries

Viele Find-Module mit CMake ausgeliefert:

.compactlist.f55[
* FindALSA
* FindArmadillo
* FindASPELL
* FindAVIFile
* FindBISON
* FindBLAS
* FindBacktrace
* **FindBoost**
* FindBullet
* FindBZip2
* FindCABLE
* FindCoin3D
* FindCUDA
* FindCups
* FindCURL
* FindCurses
* FindCVS
* FindCxxTest
* FindCygwin
* FindDart
* FindDCMTK
* FindDevIL
* FindDoxygen
* FindEXPAT
* FindFLEX
* FindFLTK2
* FindFLTK
* FindFreetype
* FindGCCXML
* FindGDAL
* FindGettext
* FindGIF
* FindGit
* FindGLEW
* FindGLUT
* FindGnuplot
* FindGnuTLS
* FindGSL
* FindGTest
* FindGTK2
* FindGTK
* FindHDF5
* FindHg
* FindHSPELL
* FindHTMLHelp
* FindIce
* FindIcotool
* FindImageMagick
* FindIntl
* FindITK
* FindJasper
* FindJava
* FindJNI
* FindJPEG
* FindKDE3
* FindKDE4
* FindLAPACK
* FindLATEX
* FindLibArchive
* FindLibLZMA
* FindLibXml2
* FindLibXslt
* FindLua50
* FindLua51
* FindLua
* FindMatlab
* FindMFC
* FindMotif
* FindMPEG2
* FindMPEG
* FindMPI
* FindOpenAL
* FindOpenCL
* FindOpenGL
* FindOpenMP
* FindOpenSceneGraph
* FindOpenSSL
* FindOpenThreads
* FindosgAnimation
* FindosgDB
* Findosg_functions
* FindosgFX
* FindosgGA
* FindosgIntrospection
* FindosgManipulator
* FindosgParticle
* FindosgPresentation
* FindosgProducer
* FindosgQt
* Findosg
* FindosgShadow
* FindosgSim
* FindosgTerrain
* FindosgText
* FindosgUtil
* FindosgViewer
* FindosgVolume
* FindosgWidget
* FindPackageHandleStandardArgs
* FindPackageMessage
* FindPerlLibs
* FindPerl
* FindPHP4
* FindPhysFS
* FindPike
* FindPkgConfig
* FindPNG
* FindPostgreSQL
* FindProducer
* FindProtobuf
* FindPythonInterp
* FindPythonLibs
* FindQt3
* FindQt4
* FindQt
* FindQuickTime
* FindRTI
* FindRuby
* FindSDL_image
* FindSDL_mixer
* FindSDL_net
* FindSDL
* FindSDL_sound
* FindSDL_ttf
* FindSelfPackers
* FindSquish
* FindSubversion
* FindSWIG
* FindTCL
* FindTclsh
* FindTclStub
* **FindThreads**
* FindTIFF
* FindUnixCommands
* FindVTK
* FindWget
* FindWish
* FindwxWidgets
* FindwxWindows
* FindXercesC
* FindX11
* FindXMLRPC
* FindZLIB
]

---
name: find-boost
# find_package für Boost

* [find_package](http://www.cmake.org/cmake/help/v3.2/command/find_package.html)
* [cmake-modules.7](http://www.cmake.org/cmake/help/v3.2/manual/cmake-modules.7.html),
  Beispiel: [Boost](http://www.cmake.org/cmake/help/v3.2/module/FindBoost.html)

```cmake
find_package( Boost
  1.57.0 EXACT REQUIRED
  COMPONENTS thread chrono system
)
...
add_executable(main main.cpp)
target_include_directories(main PRIVATE ${Boost_INCLUDE_DIRS})
target_link_libraries(main PRIVATE
  ${Boost_THREAD_LIBRARY} ${Boost_CHRONO_LIBRARY} ${Boost_SYSTEM_LIBRARY})
```

```sh
cmake -DBOOST_ROOT=$boost $src
   oder
export CMAKE_PREFIX_PATH=$boost
cmake $src
   oder
cmake $src   # auto-detect
```
???

Alternativ auch `${Boost_LIBRARIES}`


---
name: cmake-E

# cmake -E

```
Usage: cmake -E [command] [arguments ...]
Available commands:
  chdir dir cmd [args]...
  compare_files file1 file2
  copy file destination
  copy_directory source destination
  copy_if_different in-file out-file
  echo [string]...
  echo_append [string]...
  env [--unset=NAME]... [NAME=VALUE]... COMMAND [ARG]...
  environment
  make_directory dir
  md5sum file1 [...]
  remove [-f] file1 file2 ...
  remove_directory dir
  rename oldname newname
  tar [cxt][vf][zjJ] file.tar [file/dir1 file/dir2 ...]
  sleep <number>...
  time command [args] ...
  touch file
  touch_nocreate file
```

---

class: center

# Ende
.smaller[
Dieses Werk von <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">Finn Haedicke</span>
ist lizenziert unter einer <a rel="license"
href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons
Namensnennung - Weitergabe unter gleichen Bedingungen 4.0 International
Lizenz</a>.
]
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
  <img alt="Creative Commons Lizenzvertrag" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
</a>

.left[
* [remark.js](http://remarkjs.com/) ([GitHub](https://github.com/gnab/remark))
* [highlight.js](https://highlightjs.org/)
* Fonts
  * [PT Sans](http://www.google.com/fonts/specimen/PT+Sans)
  * [Yanone Kaffeesatz](http://www.google.com/fonts/specimen/Yanone+Kaffeesatz)
]
