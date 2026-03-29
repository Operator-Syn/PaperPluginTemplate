# MinecraftPlugin — Developer Onboarding Guide

A Paper plugin project built with Gradle and Java. This guide walks you through setting up your environment from scratch, even if you've never worked with Java or Minecraft plugins before.

---

## Table of Contents

- [What You'll Need](#what-youll-need)
- [Step 1 — Install SDKMAN](#step-1--install-sdkman)
- [Step 2 — Install Java and Gradle](#step-2--install-java-and-gradle)
- [Step 3 — Clone the Repository](#step-3--clone-the-repository)
- [Step 4 — Activate Project SDK Versions](#step-4--activate-project-sdk-versions)
- [Step 5 — Open in VSCode](#step-5--open-in-vscode)
- [Step 6 — Build the Plugin](#step-6--build-the-plugin)
- [Step 7 — Run Tests](#step-7--run-tests)
- [Step 8 — Run the Development Server](#step-8--run-the-development-server)
- [Project Structure](#project-structure)
- [Common Commands](#common-commands)
- [Troubleshooting](#troubleshooting)

---

## What You'll Need

- A Linux or macOS machine (Windows users: use WSL2)
- [VSCode](https://code.visualstudio.com/) installed
- A terminal
- Git installed (`git --version` to check)
- A Minecraft Java Edition account (to join the dev server)

---

## Step 1 — Install SDKMAN

SDKMAN is a version manager for Java and Gradle — similar to `nvm` for Node.js. It lets you install and switch between SDK versions easily.

```bash
curl -s "https://get.sdkman.io" | bash
```

After installation, restart your terminal or run:

```bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Verify it works:

```bash
sdk version
```

You should see a SDKMAN version number printed.

---

## Step 2 — Install Java and Gradle

This project requires **Java 21** and **Gradle 8.13**. Install them via SDKMAN:

```bash
sdk install java 21.0.2-open
sdk install gradle 8.13
```

(Optionally) Set them as your defaults: (Although you shouldn't do this if your switching between java versions)

```bash
sdk default java 21.0.2-open
sdk default gradle 8.13
```

Verify both are installed correctly:

```bash
java -version   # should say OpenJDK 21
gradle -version # should say Gradle 8.13
```

---

## Step 3 — Clone the Repository

```bash
git clone https://github.com/yashindo/minecraft-ni-brad.git
cd minecraft-ni-brad
```

---

## Step 4 — Activate Project SDK Versions

This project includes a `.sdkmanrc` file that pins the exact Java and Gradle versions used. Run this once inside the project directory:

```bash
sdk env install
```

This installs the versions specified in `.sdkmanrc` if you don't have them yet. Then activate them for your current shell:

```bash
sdk env
```

---

## Step 5 — Open in VSCode

```bash
code .
```

When VSCode opens, install the following extensions if prompted:

- **Extension Pack for Java** — Java language support
- **Gradle for Java** — Gradle build integration

VSCode will automatically detect the Gradle project and index the source files. Wait for it to finish (watch the status bar at the bottom).

---

## Step 6 — Build the Plugin

```bash
./gradlew :lib:build
```

The compiled plugin jar will be output to:

```
lib/build/libs/lib-1.0.0.jar
```

> **Note:** The version number in the filename comes from `gradle.properties`. Change `version=1.0.0` there to bump the plugin version.

---

## Step 7 — Run Tests

This project uses [MockBukkit](https://github.com/MockBukkit/MockBukkit) to unit test the plugin without needing a real Minecraft server.

Run all tests:

```bash
./gradlew :lib:test --rerun
```

View the HTML test report in your browser:

```
lib/build/reports/tests/test/index.html
```

---

## Step 8 — Run the Development Server

This project uses the [run-paper](https://github.com/jpenilla/run-paper) Gradle plugin to automatically download and run a Paper server with your plugin loaded.

**First time only — accept the EULA:**

```bash
echo "eula=true" > lib/run/eula.txt
```

Then start the server:

```bash
./gradlew :lib:runServer
```

The server will:
1. Download Paper automatically (first run only)
2. Build your plugin
3. Load it into the server

**To join the server:** Open Minecraft Java Edition 1.21, go to Multiplayer → Add Server and enter:
```
localhost
```

**To run server commands:** Type directly in the terminal where `runServer` is running:
```
op yourMinecraftUsername
gamemode creative yourMinecraftUsername
```

**To stop the server:**
```
stop
```

---

## Project Structure

```
PaperPluginTemplate/
├── gradle/
│   ├── libs.versions.toml          # Dependency version catalog
│   └── wrapper/
│       ├── gradle-wrapper.jar      # Gradle bootstrapper (committed intentionally)
│       └── gradle-wrapper.properties
├── lib/
│   ├── build.gradle.kts            # Build config — dependencies, plugins, tasks
│   └── src/
│       ├── main/
│       │   ├── java/org/application/
│       │   │   └── Library.java    # Main plugin class (extends JavaPlugin)
│       │   └── resources/
│       │       └── plugin.yml      # Plugin metadata — name, version, main class
│       └── test/
│           ├── java/org/application/
│           │   └── LibraryTest.java # Unit tests using MockBukkit + TestNG
│           └── resources/
│               └── plugin.yml
├── .sdkmanrc                       # Pins Java and Gradle versions for this project
├── gradle.properties               # Project version and Gradle settings
├── gradlew                         # Unix Gradle wrapper script
├── gradlew.bat                     # Windows Gradle wrapper script
└── settings.gradle.kts             # Root project name
```

### Key Files Explained

**`gradle.properties`** — Change the plugin version here. It gets injected into `plugin.yml` automatically at build time:
```properties
version=1.0.0
```

**`plugin.yml`** — The server reads this to identify your plugin. The `${version}` placeholder is replaced by Gradle:
```yaml
name: MinecraftPlugin
version: "${version}"
main: org.application.Library
api-version: 1.21
```

**`build.gradle.kts`** — Defines dependencies and build behavior. Paper API is `compileOnly` meaning it's available when writing code but not bundled in the jar — the server provides it at runtime.

---

## Common Commands

| Command | Description |
|---|---|
| `./gradlew :lib:build` | Compile and package the plugin jar |
| `./gradlew :lib:test --rerun` | Run all unit tests |
| `./gradlew :lib:runServer` | Start the dev Paper server |
| `./gradlew :lib:dependencies` | Show resolved dependency tree |
| `./gradlew :lib:clean` | Delete build outputs |
| `./gradlew --stop` | Stop all Gradle daemons |

---

## Troubleshooting

**`sdk: command not found`**
Run `source "$HOME/.sdkman/bin/sdkman-init.sh"` or restart your terminal.

**`./gradlew: Permission denied`**
```bash
chmod +x gradlew
```

**VSCode shows red errors on import statements**
Open the command palette (`Ctrl+Shift+P`) and run:
```
Java: Clean Language Server Workspace
```
Then reload the window.

**Gradle keeps creating new daemons**
Make sure your `gradle.properties` has:
```properties
org.gradle.jvmargs=-Duser.country=PH -Duser.language=en
```
Replace `PH` with your country code.

**Server won't start — EULA error**
```bash
echo "eula=true" > lib/run/eula.txt
```

**`configuration-cache` error when running server**
Make sure `org.gradle.configuration-cache=false` is set in `gradle.properties`.