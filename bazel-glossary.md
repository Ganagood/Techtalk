# Bazel Glossary for Presenters

A complete reference with definitions, syntax, and execution examples for all Bazel terms and commands used in this demo.

---

## Core Bazel Concepts

### 1. Bazel
**Definition:** A build and test system that uses an explicit dependency graph and caching to run only necessary work.

**Simple explanation:** Bazel is a smart planner that avoids rebuilding everything when you change one file.

**Why it matters:** Saves engineering time by making builds fast, correct, and scalable.

---

### 2. Target
**Definition:** A named buildable or testable unit, such as an app, library, or test.

**Analogy:** A target is one building in a city.

**Syntax in BUILD.bazel:**
```bazel
cc_binary(
    name = "demo_app",
    srcs = ["main.cc"],
    deps = ["//lib:math_utils"],
)
```

**Execution example:**
```bash
bazel run //app:demo_app          # Runs the binary target
bazel test //tests:math_utils_test # Tests the target
```

---

### 3. Rule
**Definition:** The recipe Bazel uses to build a target (e.g., cc_binary, cc_library, py_binary).

**Analogy:** Rule is the construction method for that building.

**Common rules in C++:**
- `cc_binary`: Executable
- `cc_library`: Reusable code
- `cc_test`: Unit test

**Syntax example:**
```bazel
cc_library(
    name = "math_utils",
    srcs = ["math_utils.cc"],
    hdrs = ["math_utils.h"],
)
```

---

### 4. Label
**Definition:** The unique address of a target, like `//app:demo_app`.

**Format:** `//package:target_name`

**Analogy:** Label is the GPS address of a building.

**Examples:**
- `//app:demo_app` - Target demo_app in package app
- `//lib:math_utils` - Target math_utils in package lib
- `//tests:math_utils_test` - Target math_utils_test in package tests

---

### 5. Package
**Definition:** A directory that contains a BUILD.bazel file.

**Analogy:** A package is a neighborhood in the city.

**Example structure:**
```
demo-project/
├── app/
│   ├── BUILD.bazel      (app is a package)
│   └── main.cc
├── lib/
│   ├── BUILD.bazel      (lib is a package)
│   ├── math_utils.cc
│   └── math_utils.h
└── tests/
    ├── BUILD.bazel      (tests is a package)
    └── math_utils_test.cc
```

---

### 6. BUILD.bazel
**Definition:** File in each package where targets and their dependencies are declared.

**Simple explanation:** This is where we describe what to build and what each thing depends on.

**Syntax example:**
```bazel
cc_binary(
    name = "demo_app",
    srcs = ["main.cc"],
    deps = [
        "//lib:math_utils",
        "//lib:string_utils",
    ],
)
```

**Execution example:**
```bash
bazel query //...        # Shows all targets Bazel knows about
```

---

### 7. MODULE.bazel
**Definition:** File used by Bzlmod to declare external dependencies and module metadata.

**Analogy:** This is the import manifest for outside materials your city needs.

**Syntax example:**
```bazel
module(
    name = "bazel_techtalk_demo",
    version = "1.0",
)
```

---

## Dependency Graph and Query Terms

### 1. Dependency
**Definition:** A target required by another target to build or run correctly.

**Analogy:** One building needs roads to other buildings.

**Syntax in BUILD.bazel:**
```bazel
cc_binary(
    name = "demo_app",
    srcs = ["main.cc"],
    deps = ["//lib:math_utils"],  # This is a dependency
)
```

---

### 2. Dependency Graph
**Definition:** The full map of targets and their relationships.

**Simple explanation:** Bazel decisions come from this map, not from guesswork.

**Execution example:**
```bash
bazel query //...
# Output: Shows all targets and their structure
```

---

### 3. Transitive Dependency
**Definition:** A dependency of your dependency.

**Analogy:** Friend of a friend in the graph.

**Example:**
- App depends on math_utils
- math_utils depends on common_utils
- common_utils is a transitive dependency of app

---

### 4. deps(...) Query
**Definition:** Shows all dependencies needed by a target.

**Simple explanation:** This tells us everything this target relies on.

**Syntax:**
```bash
bazel query 'deps(//TARGET:name)'
```

**Execution example:**
```bash
bazel query 'deps(//app:demo_app)'

# Output:
# //app:demo_app
# //lib:math_utils
# //lib:string_utils
# // and all transitive deps
```

**What to say when you run it:**
"This is the exact impact surface behind the app. These are all the things that must be built before the app can run."

---

### 5. rdeps(...) Query
**Definition:** Reverse dependencies; shows what is impacted by a target.

**Simple explanation:** If I change this library, who else feels the blast radius?

**Syntax:**
```bash
bazel query 'rdeps(//SOURCE:..., //TARGET:name)'
```

**Execution example:**
```bash
bazel query 'rdeps(//..., //lib:math_utils)'

# Output:
# //lib:math_utils
# //app:demo_app
# //tests:math_utils_test
```

**What to say when you run it:**
"Now we see everything that depends on math_utils. If we change this library, these targets need rebuilt."

---

### 6. kind(...) Filter
**Definition:** Filters graph results to only targets matching a rule type.

**Simple explanation:** From all impacted things, show me just tests.

**Syntax:**
```bash
bazel query 'kind(".*test", deps(//TARGET:name))'
```

**Execution example:**
```bash
bazel query 'kind(".*test", rdeps(//..., //lib:math_utils))'

# Output:
# //tests:math_utils_test
```

**What to say when you run it:**
"This shows all tests affected by a change to math_utils."

---

## Execution and Build Behavior

### 1. Action
**Definition:** One concrete execution step Bazel performs (compile, link, run test).

**Analogy:** A single construction task on a building.

**Examples of actions:**
- Compile math_utils.cc → math_utils.o
- Link objects → demo_app binary
- Run tests

---

### 2. aquery
**Definition:** Action query; shows low-level actions Bazel will execute.

**Simple explanation:** query shows structure; aquery shows actual work.

**Syntax:**
```bash
bazel aquery 'mnemonic("ACTION_TYPE", deps(//TARGET:name))'
```

**Execution example:**
```bash
bazel aquery 'mnemonic("CppCompile", deps(//app:demo_app))'

# Output shows compile commands Bazel will run
```

**What to say when you run it:**
"Now we see the exact commands Bazel will execute. This is the action plan."

---

### 3. Mnemonic
**Definition:** The action type name used by Bazel (CppCompile, CppLink, TestRunner, etc.).

**Analogy:** Job category label for each action.

**Common mnemonics in C++:**
- `CppCompile` - Compile C++ source
- `CppLink` - Link object files
- `TestRunner` - Run test binary

**Syntax:**
```bash
bazel aquery 'mnemonic("CppCompile", deps(//app:demo_app))'
```

---

### 4. Incremental Build
**Definition:** Rebuild only what changed and what depends on it.

**Simple explanation:** We changed one street, so we reroute only nearby traffic.

**Execution example:**
```bash
# First run - full build
bazel run //app:demo_app

# Edit lib/math_utils.cc

# Second run - only recompile math_utils and link app
bazel run //app:demo_app
```

**What to observe:** Second run is much faster.

---

### 5. Up-to-date Result
**Definition:** Bazel found valid previous outputs and reused them instead of redoing work.

**Simple explanation:** Same inputs, same outputs, no wasted rebuild.

**Execution example:**
```bash
bazel run //app:demo_app
# First time output:
# //app:demo_app up-to-date (real work)

bazel run //app:demo_app
# Second time output:
# //app:demo_app up-to-date (reused)
```

---

### 6. Hermetic Build
**Definition:** Build action depends only on declared inputs, not hidden machine state.

**Analogy:** Cooking from a fixed recipe with only listed ingredients.

**Why it matters:** Prevents "works on my machine" failures.

**Example:**
```bazel
cc_library(
    name = "math_utils",
    srcs = ["math_utils.cc"],  # All inputs declared
    hdrs = ["math_utils.h"],   # No hidden system headers as inputs
    deps = ["//lib:string_utils"],  # Explicit dependencies
)
```

---

### 7. Deterministic Result
**Definition:** Same inputs produce same outputs every time.

**Simple explanation:** Reliable and predictable, on laptop and CI.

**Benefit:** Fewer random failures in CI pipelines.

---

### 8. Reproducible Build
**Definition:** Different environments can get the same outcome from the same inputs.

**Simple explanation:** Works consistently across machines, not just mine.

**Example:** Your laptop and CI produce identical binaries.

---

### 9. Caching (Local and Remote)
**Definition:** Reuse previously computed outputs instead of recomputing.

**Analogy:** Reusing a prepared dish instead of cooking from scratch again.

**Local cache:**
```bash
# First run - builds and caches
bazel run //app:demo_app

# Second run - reuses from local cache
bazel run //app:demo_app
```

**Remote cache:** Multiple machines share one cache server for even faster builds.

---

## Commands in Your Demo

### 1. bazel query //...
**What it does:** Lists all targets in the workspace graph.

**What to say:** "This is the map of what Bazel knows exists."

**Syntax:**
```bash
bazel query //...
```

**Execution example:**
```bash
$ bazel query //...

# Output:
# //app:demo_app
# //lib:math_utils
# //lib:string_utils
# //tests:math_utils_test
```

---

### 2. bazel run //app:demo_app
**What it does:** Builds what is needed, then executes the app target.

**What to say:** "First run does real work; second run usually demonstrates reuse."

**Syntax:**
```bash
bazel run //TARGET:name
```

**Execution example:**
```bash
# First run (cold)
$ bazel run //app:demo_app
# Compiling...
# Output: Bazel Demo
#         Subtotal: 134.5
#         Total with tax: 143.715

# Second run (cached)
$ bazel run //app:demo_app
# up-to-date
# Output: Bazel Demo
#         Subtotal: 134.5
#         Total with tax: 143.715
```

---

### 3. bazel test //tests:math_utils_test
**What it does:** Builds and runs the test target.

**What to say:** "Tests are first-class citizens in the same graph."

**Syntax:**
```bash
bazel test //PACKAGE:test_name
```

**Execution example:**
```bash
$ bazel test //tests:math_utils_test

# Output:
# //tests:math_utils_test PASSED
```

---

### 4. bazel query 'deps(//app:demo_app)'
**What it does:** Shows all dependencies of the app.

**What to say:** "This is the exact impact surface behind the app."

**Syntax:**
```bash
bazel query 'deps(//TARGET:name)'
```

**Execution example:**
```bash
$ bazel query 'deps(//app:demo_app)'

# Output:
# //app:demo_app
# //lib:math_utils
# //lib:string_utils
```

---

### 5. bazel query 'rdeps(//..., //lib:math_utils)'
**What it does:** Shows what depends on math_utils.

**What to say:** "This gives change impact before we edit anything risky."

**Syntax:**
```bash
bazel query 'rdeps(//SOURCE:..., //TARGET:name)'
```

**Execution example:**
```bash
$ bazel query 'rdeps(//..., //lib:math_utils)'

# Output:
# //lib:math_utils
# //app:demo_app
# //tests:math_utils_test
```

---

### 6. bazel aquery 'mnemonic("CppCompile", deps(//app:demo_app))'
**What it does:** Shows compile actions for app dependencies.

**What to say:** "Now we can inspect the execution plan, not only the graph."

**Syntax:**
```bash
bazel aquery 'mnemonic("ACTION_TYPE", deps(//TARGET:name))'
```

**Execution example:**
```bash
$ bazel aquery 'mnemonic("CppCompile", deps(//app:demo_app))'

# Output:
# action 'CppCompile'
#  Inputs: //lib:math_utils [math_utils.cc]
#  Command: c++ -c math_utils.cc -o math_utils.o
#
# action 'CppCompile'
#  Inputs: //app [main.cc]
#  Command: c++ -c main.cc -o main.o
```

---

## Team-Level Terms

### 1. CI Reliability
**Definition:** Stable and consistent build/test behavior in automation pipelines.

**Simple explanation:** Less random red builds, fewer surprises.

**Bazel benefit:** Hermetic builds reduce CI noise.

---

### 2. Flaky Test or Flaky Build
**Definition:** Intermittent failure not caused by real code changes.

**Simple explanation:** Hermetic and explicit inputs help reduce this noise.

**Bazel improvement:** Explicit dependencies and hermetic builds reduce flakiness.

---

### 3. Monorepo Scale
**Definition:** Many projects in one repo sharing tooling and dependencies.

**Simple explanation:** Bigger repos benefit more from selective rebuild and cache reuse.

**Bazel advantage:** Graph-based approach scales linearly, not quadratically.

---

## Quick Reference: Common Phrases to Use

| Phrase | Use When |
|--------|----------|
| "Same inputs, same outputs, so reuse is expected." | Explaining second run speed |
| "The graph explains the result we just saw." | After showing query output |
| "Let us make a prediction before we run this command." | Before each demo command |
| "This is where Bazel starts compounding value across teams." | Transitioning to adoption talk |
| "If inputs are unchanged, results are reused." | Explaining core concept |
| "Incremental rebuild only what changed." | After code edit demo |
| "The dependency graph is inspectable, not magical." | Explaining query power |

---

## Pre-Demo Checklist

- [ ] Understand target vs rule vs label
- [ ] Practice bazel query //... in demo-project/
- [ ] Practice bazel run //app:demo_app (twice, to show cache)
- [ ] Practice bazel test //tests:math_utils_test
- [ ] Practice the math_utils.cc edit workflow
- [ ] Practice bazel query 'deps(//app:demo_app)'
- [ ] Practice bazel query 'rdeps(//..., //lib:math_utils)'
- [ ] Test ./scripts/start_practice_container.sh
- [ ] Verify Docker is running
- [ ] Prepare audience README link

---

End of Glossary
