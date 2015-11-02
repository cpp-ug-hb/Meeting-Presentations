class: center, middle

# Qualitätssicherung in C++:  Ein Toolflow Überblick

Stefan Frehse <br/>
C++ User-Group Meeting - 2015-03-26


<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
  <img alt="Creative Commons Lizenzvertrag" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
</a>

---

# Agenda 
- Was ist Qualität
- Toolunterstützung
- Beispiel: `cppcheck`, `clang`-sanitizer
- Diskussion über den Einsatz solcher Tools

---

class: center, middle

# Qualität?

---

# Qualität

- Quellcode (wie z.B. Dead code, uninitialized variables, Coding style)
- Laufzeit (wie z.B. Memory-Leaks, Race conditions, Memory consumption)

---

# Tools - nicht kommerziell (unvoll. Liste)
## Statische Analyse
 - `cppcheck`: statische Quellcodeanalye [[1]](http://bit.ly/1kyZB3f)
 - `vera++`: Style checker und Refactoring Tool [[2]](https://bitbucket.org/verateam/vera/wiki/Home)
 - `flint`: linter [[3]](http://github.com/facebook/flint) 

## Dynamische Analyze
 - Clang sanitizer (e.g., Memory, Thread, Static) [[4]](http://llvm.org)
 - valgrind (callbgrind, cachegrind, helgrind) [[5]](http://valgrind.org)
 - klee: symbolic virtual machine on top of LLVM [[6]](http://klee.github.io/)
 - cbmc: formale Beweisführung [[7]](http://www.cprover.org/cbmc/)

---
# Tools - kommerziell
 - cppcat (_statisch_) ein visual studio plugin [[8]](http://cppchat.com)
 - Coverity Code Advisor von Coverity (kostenlos für öffentliche Github Projekte, ansonsten kostenpflichtig) [[9]](http://coverity.com)
 - CppDepend von CoderGears [[10]](http://cppdepend.com)
 - QA-C und QA-C++ von QA-Systems [[11]](http://www.qa-systems.de/)

---
class: center, middle

# Beispiel: `cppcheck` auf Lingeling Quellcode [[12]](http://fmv.jku.at/lingeling/)

---

# [cppcheck](http://cppcheck.sourceforge.net/)
## statischer Checker für C/C++

- Out of bounds checking
- Memory leaks checking
- Detect possible null pointer dereferences
- Check for uninitialized variables
- Check for invalid usage of STL
- Checking exception safety
- Warn if obsolete or unsafe functions are used
- Warn about unused or redundant code
- Detect various suspicious code indicating bugs

---
# `cppcheck`: Grad der Analyse
- `error`, `warning`, `style`, `performance`, `portability`, `information`
- `all` umfasst alle vorhandenen Checks
- Beispiel: `performance` empfiehlt Quellcode stellen zur Laufzeitverbesserung

---

## Beispiel: examples.cpp

	int example1()
	{
	  char a[10];
	  a[10] = 0;
	  return 0;
	}

	void example2()
	{
	  int *p;
	  *p = 10;
	}
---

## Überprüfung von `examples.cpp` mit `cppcheck`
### `$: cppcheck --enable=all examples.cpp`

	Checking examples.cpp...
	examples.cpp:4: style: Variable 'a' is assigned a value that is never used.
	examples.cpp:10: style: Variable 'p' is not assigned a value.
	examples.cpp:4: error: Array 'a[10]' accessed at index 10, which is out of bounds.
	examples.cpp:11: error: Uninitialized variable: p
	Checking usage of global functions..
	examples.cpp:1: style: The function 'example1' is never used.
	examples.cpp:8: style: The function 'example2' is never used.

---
## Komplexeres Beispiel: SAT-solver Lingeling
- geschrieben in C
- ~26 kLOC (`sloccount`)

### `cppcheck`
- `cppcheck -j4 --enable=performance $PATHTOLINGELING`
- `performance` Empfehlung für schnelleren Code 
- Laufzeit: 160 Sekunden

---

## Ausgabe: `cppcheck` für `Lingeling`
	[lglmain.c:316]: (error) Common realloc mistake: 'targets'
		nulled but not freed upon failure

## Quellcode: `lglmain.c:316`
	static void lglpushtarget (int target) {
	  if (ntargets == sztargets)
	      targets = realloc (targets, sizeof *targets *
	                   (sztargets = sztargets ? 2*sztargets : 1));
			       targets[ntargets++] = target;
        }


---

## Ausgabe: `cppcheck` für `Lingeling`
	[lglib.c:25795] -> [lglib.c:25800]: (performance) Variable 'sum' is reassigned 
		a value before the old one has been used.

## Quellcode: '[lglib.c:25795] -> [lglib.c:25800]'
	sum = s->moved.bin + s->moved.trn + s->moved.lrg;
	lglprs (lgl,
	      "mins: %lld learned lits, %.0f%% minimized",
	      (LGLL) s->lits.learned, lglpcnt (min, s->lits.nonmin));

	sum = s->mincls.min + s->mincls.bin + s->mincls.size + s->mincls.deco;


---
## Ausgabe: `cppcheck` für `Lingeling`
	[lglib.c:8653]: (error) Invalid abs() argument nr 1.
	A non-boolean value is required.

## Quellcode: `lglib.c:8655`, `lgl->bias` is an `int`
	assert (abs( lgl->bias <= 1));

---

# `clang`-sanitizer:
- `address`-sanitizer:
	- Out-of-bounds accesses to heap, stack and globals
	- Use-after-free
	- Use-after-return (to some extent)
	- Double-free, invalid free
	- Memory leaks (experimental)
- `memory`-sanitizer:
	- detection of uninitialized reads
- `thread`-sanitizer:
	- detects data races

- Buildvorgang mit den *flags*: z.B. `-fsanitize=memory`
---

# Beispiel: `cmake` Buildvorgang mit clang und `sanitizer`
## Buildvorgang:
	$ export CC=clang && export CXX=clang++
	$ export CFLAGS=-fsanitize=memory && export CXXFLAGS=-fsanitize=memory
	$ ./configure && make
## Ausgabe: Nach Ausführung der Tests
	==8192== WARNING: MemorySanitizer: use-of-uninitialized-value
		...
	SUMMARY: MemorySanitizer: use-of-uninitialized-value ??:0
	cmsys::SystemTools::SplitPath(std::string const&
	, std::vector<std::string, std::allocator<std::string> >&, bool)

---

# Nutzung verschiedener Tools 
- *use-of-uninitialized-value* nicht durch `cppcheck` gefunden
- Nutzung verschiedener Tools um eine breite Abdeckung zu ermöglichen

---

class: center, middle
# Integration in einen Toolflow?

---

# Workflow / Toolflow 
- Ausführung der Checks vor den Commit?
- Durchführung innerhalb eines Continuous Integration
- automatisches Code review mit `gerrit`?
- Pull-request erst gültig wenn Checks erfolgreich sein 
- Orchestrierung verschiedener Tools, wie z.B. `cppcheck`, `clang`, `valgrind`

---
class: center, middle
# Diskussion

---
# Warum nicht gleich morgen einsetzen?
- Kompliziert einzusetzen?
- Kein Nutzen?
- Laufzeit zu hoch?
---

# Ende
.smaller[
Dieses Werk von <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">Stefan Frehse</span>
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
