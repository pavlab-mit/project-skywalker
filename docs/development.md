# Development Guide

This guide covers setting up a development environment, creating custom MOOS behaviors, and contributing to the Project Skywalker platform.

## Development Environment Setup

### Local Development Machine

For developing and testing MOOS missions locally before deploying to the UAV:

#### macOS Setup

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install cmake git subversion

# Clone MOOS-IvP
cd ~/
git clone https://github.com/moos-ivp/moos-ivp.git
cd ~/moos-ivp
./build.sh

# Add to PATH (add to ~/.zshrc or ~/.bash_profile)
export PATH=$PATH:~/moos-ivp/bin
```

#### Linux Setup

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y build-essential cmake git subversion \
    libfltk1.3-dev freeglut3-dev libpng-dev libjpeg-dev \
    libxft-dev libxinerama-dev

# Clone MOOS-IvP
cd ~/
git clone https://github.com/moos-ivp/moos-ivp.git
cd ~/moos-ivp
./build.sh

# Add to PATH
echo 'export PATH=$PATH:~/moos-ivp/bin' >> ~/.bashrc
source ~/.bashrc
```

#### Windows Setup

Not officially supported. Consider:
- WSL2 (Windows Subsystem for Linux) with Ubuntu
- Docker container with MOOS-IvP
- Virtual machine with Linux

### IDE and Tools

#### Text Editors

**VS Code (Recommended)**
```bash
# Install VS Code
brew install --cask visual-studio-code

# Useful extensions:
# - C/C++
# - CMake Tools
# - Markdown All in One
```

**Emacs**
```bash
brew install emacs
```

**Vim/Neovim**
```bash
brew install neovim
```

#### Version Control

```bash
# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Recommended: use SSH keys for GitHub
ssh-keygen -t ed25519 -C "your.email@example.com"
# Add to GitHub: Settings → SSH Keys
```

## MOOS Application Development

### Application Structure

Basic MOOS application structure:

```
MyApp/
├── CMakeLists.txt
├── MyApp.h
├── MyApp.cpp
├── MyApp_Info.h
├── MyApp_Info.cpp
├── main.cpp
└── README.md
```

### Creating a Custom MOOS Application

#### 1. Generate Skeleton

MOOS-IvP provides a tool to generate application templates:

```bash
cd ~/moos-ivp-uav-base/src/
GenMOOSApp MyApp "Your Name"
```

This creates a basic application structure with:
- Header and source files
- Main entry point
- CMakeLists.txt
- Info/help functions

#### 2. Define Application Logic

Edit `MyApp.h`:

```cpp
#ifndef MY_APP_HEADER
#define MY_APP_HEADER

#include "MOOS/libMOOS/Thirdparty/AppCasting/AppCastingMOOSApp.h"

class MyApp : public AppCastingMOOSApp
{
public:
   MyApp();
   ~MyApp() {};

protected: // Standard MOOS functions
   bool OnNewMail(MOOSMSG_LIST &NewMail);
   bool Iterate();
   bool OnConnectToServer();
   bool OnStartUp();

protected: // Configuration functions
   void registerVariables();

protected: // Application specific functions
   bool handleConfigParam(std::string param, std::string value);
   void doSomething();

protected: // State variables
   double m_my_variable;
   std::string m_status;
};

#endif
```

Edit `MyApp.cpp`:

```cpp
#include "MyApp.h"
#include "MBUtils.h"

MyApp::MyApp()
{
   m_my_variable = 0.0;
   m_status = "idle";
}

bool MyApp::OnStartUp()
{
   AppCastingMOOSApp::OnStartUp();

   STRING_LIST sParams;
   m_MissionReader.EnableVerbatimQuoting(false);
   if(!m_MissionReader.GetConfiguration(GetAppName(), sParams))
      reportConfigWarning("No config block found for " + GetAppName());

   STRING_LIST::iterator p;
   for(p=sParams.begin(); p!=sParams.end(); p++) {
      string orig  = *p;
      string line  = *p;
      string param = tolower(biteStringX(line, '='));
      string value = line;

      bool handled = false;
      if(param == "my_param")
         handled = handleConfigParam(param, value);

      if(!handled)
         reportUnhandledConfigWarning(orig);
   }

   registerVariables();
   return true;
}

bool MyApp::OnConnectToServer()
{
   registerVariables();
   return true;
}

bool MyApp::Iterate()
{
   AppCastingMOOSApp::Iterate();

   // Main application logic here
   doSomething();

   // Publish results
   Notify("MY_OUTPUT", m_my_variable);

   AppCastingMOOSApp::PostReport();
   return true;
}

bool MyApp::OnNewMail(MOOSMSG_LIST &NewMail)
{
   AppCastingMOOSApp::OnNewMail(NewMail);

   MOOSMSG_LIST::iterator p;
   for(p=NewMail.begin(); p!=NewMail.end(); p++) {
      CMOOSMsg &msg = *p;
      string key    = msg.GetKey();
      double dval   = msg.GetDouble();
      string sval   = msg.GetString();

      if(key == "MY_INPUT") {
         m_my_variable = dval;
      }
   }

   return true;
}

void MyApp::registerVariables()
{
   AppCastingMOOSApp::RegisterVariables();
   Register("MY_INPUT", 0);
}

bool MyApp::handleConfigParam(std::string param, std::string value)
{
   if(param == "my_param") {
      m_my_variable = atof(value.c_str());
      return true;
   }
   return false;
}

void MyApp::doSomething()
{
   // Your application logic
   m_my_variable += 1.0;
}
```

#### 3. Build Application

```bash
cd ~/moos-ivp-uav-base
./build.sh
```

#### 4. Test Application

Create test mission `test_myapp.moos`:

```moos
ServerHost = localhost
ServerPort = 9000
Community  = test

ProcessConfig = ANTLER
{
  MSBetweenLaunches = 200
  
  Run = MOOSDB        @ NewConsole = false
  Run = MyApp         @ NewConsole = true
}

ProcessConfig = MyApp
{
  AppTick   = 4
  CommsTick = 4
  
  my_param = 100
}
```

Run:
```bash
pAntler test_myapp.moos
```

### Creating Custom Behaviors

MOOS-IvP uses behavior-based autonomy. Create custom behaviors for specific tasks.

#### Behavior Structure

```cpp
#include "IvPBehavior.h"

class BHV_MyBehavior : public IvPBehavior {
public:
   BHV_MyBehavior(IvPDomain);
   ~BHV_MyBehavior() {};

   bool setParam(string, string);
   IvPFunction* onRunState();
   void onSetParamComplete();
   void onRunToIdleState();
   void onIdleToRunState();

protected:
   double m_desired_heading;
   double m_desired_speed;
};
```

See MOOS-IvP documentation for detailed behavior development.

## Testing

### Unit Testing

For MOOS applications, test individual functions:

```cpp
// test_myapp.cpp
#include "MyApp.h"
#include <assert.h>

void testMyFunction() {
   MyApp app;
   // Test logic
   assert(app.someFunction() == expected_value);
}

int main() {
   testMyFunction();
   return 0;
}
```

### Integration Testing

Test with MOOS in the loop:

1. Create minimal mission file
2. Launch with pAntler
3. Send test inputs with uPokeDB
4. Monitor outputs with uXMS

```bash
# Terminal 1: Launch mission
pAntler test.moos

# Terminal 2: Send test input
uPokeDB test.moos MY_INPUT=123

# Terminal 3: Monitor
uXMS test.moos
```

### Simulation Testing

Before flying, test in simulation:

1. Use shore-side MOOS applications
2. Simulate vehicle behavior
3. Test coordination logic
4. Validate against expected behavior

## Debugging

### MOOS Debugging Tools

**uXMS** - Variable viewer:
```bash
uXMS mission.moos
```

**uMS** - Process monitor:
```bash
uMS mission.moos
```

**alogview** - Log viewer:
```bash
alogview logs/mylog.alog
```

**uPokeDB** - Variable setter/getter:
```bash
uPokeDB mission.moos MY_VAR=value
uPokeDB mission.moos MY_VAR?
```

### GDB Debugging

For C++ debugging:

```bash
# Compile with debug symbols
./build.sh -d

# Run with GDB
gdb --args MyApp mission.moos

# Common GDB commands:
# break MyApp.cpp:42   - Set breakpoint
# run                  - Start program
# continue             - Continue execution
# print variable       - Print variable value
# backtrace            - Show call stack
```

### Log Files

MOOS automatically logs to `.alog` files:

```bash
# View log
alogview logs/mylog.alog

# Extract specific variables
aloggrep logs/mylog.alog NAV_X NAV_Y

# Convert to CSV
alog2csv logs/mylog.alog
```

## Code Style and Standards

### C++ Style

Follow MOOS-IvP conventions:
- Class names: `UpperCamelCase`
- Functions: `lowerCamelCase`
- Variables: `m_lower_with_underscore` (member vars)
- Constants: `UPPER_CASE`
- Indentation: 2 spaces

### Documentation

Document your code:

```cpp
//---------------------------------------------------------
// Procedure: doSomething
//   Purpose: Brief description of what this function does
//      Note: Any additional notes

void MyApp::doSomething()
{
   // Implementation
}
```

### Git Commit Messages

Good commit message format:

```
Short summary (50 chars or less)

More detailed explanation if needed. Wrap at 72 characters.
Explain what and why, not how.

- Bullet points are okay
- Use present tense ("Add feature" not "Added feature")
```

## Building and Deployment

### Building for Raspberry Pi

**Option 1: Build on Raspberry Pi** (Recommended)
```bash
# SSH to Pi
ssh skywalker1

# Navigate to source
cd ~/moos-ivp-uav-base

# Build (use -j2 for parallel build)
./build.sh -j2
```

**Option 2: Cross-compile** (Advanced)
- Requires cross-compilation toolchain
- More complex setup
- Faster builds on powerful machine

### Deployment

```bash
# Method 1: Git pull on Pi
ssh skywalker1
cd ~/moos-ivp-uav-base
git pull
./build.sh -j2

# Method 2: SCP files
scp my_new_mission.moos skywalker1:~/missions/

# Method 3: Rsync
rsync -avz ~/missions/ skywalker1:~/missions/
```

## Continuous Integration

### GitHub Actions (Optional)

Add `.github/workflows/build.yml`:

```yaml
name: Build MOOS-IvP

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Install dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y build-essential cmake git subversion
    
    - name: Build MOOS-IvP
      run: |
        ./build.sh
```

## Contributing Guidelines

See [Contributing Guide](contributing.md) for:
- How to submit changes
- Pull request process
- Code review expectations
- Branch naming conventions

## Useful Resources

### MOOS-IvP Documentation
- [Official Documentation](https://oceanai.mit.edu/moos-ivp/)
- [Quick Start](https://oceanai.mit.edu/moos-ivp/pmwiki/pmwiki.php?n=Main.QuickStart)
- [Application Development](https://oceanai.mit.edu/moos-ivp/pmwiki/pmwiki.php?n=Main.ApplicationDevelopment)

### ArduPilot Development
- [Dev Guide](https://ardupilot.org/dev/)
- [MAVLink Protocol](https://mavlink.io/)

### Example Code
- MOOS-IvP tree: `~/moos-ivp/ivp/src/`
- UAV applications: `~/moos-ivp-uav-base/src/` (private repo)
- Example missions: `~/moos-ivp/ivp/missions/`

### Community
- [MOOS-IvP Forum](https://groups.google.com/g/moos-ivp)
- Project team: team@pavlab-mit.org

## Development Workflow

Typical development cycle:

```
1. Create feature branch
   git checkout -b feature/my-feature

2. Write code
   - Implement feature
   - Add tests
   - Document

3. Build and test locally
   ./build.sh
   pAntler test.moos

4. Commit changes
   git add .
   git commit -m "Add my feature"

5. Push to GitHub
   git push origin feature/my-feature

6. Create pull request
   - Describe changes
   - Link to issues
   - Request review

7. Address feedback
   - Make changes
   - Push updates

8. Merge when approved
```

## Best Practices

### For MOOS Applications
- ✅ Keep Iterate() fast (runs at AppTick rate)
- ✅ Register only needed variables
- ✅ Use AppCasting for status reporting
- ✅ Handle all configuration errors gracefully
- ✅ Log important events
- ✅ Test with various AppTick rates

### For Behaviors
- ✅ Check preconditions in onRunState()
- ✅ Return NULL if inactive
- ✅ Use InfoBuffer for debug output
- ✅ Handle edge cases
- ✅ Test priority interactions

### For Safety
- ✅ Always test in simulation first
- ✅ Implement proper error handling
- ✅ Validate all inputs
- ✅ Test failsafe scenarios
- ✅ Log critical events

## Next Steps

- **[Contributing Guide](contributing.md)** - How to contribute
- **[Usage Guide](usage.md)** - Running the system
- **[Troubleshooting](troubleshooting.md)** - Common issues

---

**Previous**: [Firmware Guide](firmware.md) | **Next**: [Contributing Guide](contributing.md)
