# Contributing Guide

Thank you for your interest in contributing to Project Skywalker! This guide will help you get started.

## Ways to Contribute

There are many ways to contribute to this project:

- 🐛 **Report bugs** - Submit detailed bug reports
- 📝 **Improve documentation** - Fix typos, clarify instructions, add examples
- 💡 **Suggest features** - Propose new capabilities or improvements
- 🔧 **Submit code** - Fix bugs or implement new features
- 🧪 **Test** - Verify functionality and report results
- 📚 **Share knowledge** - Write tutorials, blog posts, or guides
- 💬 **Help others** - Answer questions in issues and discussions

## Getting Started

### Prerequisites

Before contributing, ensure you have:

1. **GitHub Account** - [Create one](https://github.com/signup) if needed
2. **Development Environment** - See [Development Guide](development.md)
3. **Understanding of MOOS-IvP** - Review [documentation](https://oceanai.mit.edu/moos-ivp/)
4. **Familiarity with Git** - Basic git knowledge required

### Fork and Clone

1. **Fork the repository**
   - Click "Fork" button on GitHub
   - Creates your personal copy

2. **Clone your fork**
   ```bash
   git clone https://github.com/YOUR-USERNAME/project-skywalker.git
   cd project-skywalker
   ```

3. **Add upstream remote**
   ```bash
   git remote add upstream https://github.com/pavlab-mit/project-skywalker.git
   ```

4. **Keep your fork synced**
   ```bash
   git fetch upstream
   git checkout main
   git merge upstream/main
   ```

## Contribution Workflow

### 1. Create an Issue

Before starting work, create or comment on an issue:

- Describe what you plan to do
- Get feedback from maintainers
- Avoid duplicate work

**For bugs:**
- Describe what happened
- Provide steps to reproduce
- Include system information
- Attach logs if applicable

**For features:**
- Explain the use case
- Describe proposed solution
- Discuss alternatives considered

### 2. Create a Branch

Create a descriptive branch name:

```bash
git checkout -b feature/add-new-behavior
git checkout -b fix/radio-connection-bug
git checkout -b docs/improve-installation-guide
```

**Branch naming conventions:**
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation updates
- `refactor/` - Code refactoring
- `test/` - Test additions or fixes

### 3. Make Changes

#### Code Changes

Follow these guidelines:

**Style**
- Follow existing code style
- Use meaningful variable names
- Comment complex logic
- Keep functions focused and small

**Testing**
- Test your changes thoroughly
- Add unit tests if applicable
- Test on actual hardware when possible
- Document test procedures

**Documentation**
- Update relevant docs
- Add docstrings to functions
- Update README if needed
- Include usage examples

#### Documentation Changes

For documentation contributions:

- Use clear, concise language
- Include examples and code snippets
- Add links to related sections
- Check spelling and grammar
- Verify all links work

### 4. Commit Changes

Write clear commit messages:

```bash
git add .
git commit -m "Add support for dual GPS configuration

- Implement GPS blending logic
- Update parameter documentation
- Add configuration examples
- Test with dual GPS setup"
```

**Commit message format:**
- First line: Brief summary (50 chars or less)
- Blank line
- Detailed description if needed
- Use present tense ("Add feature" not "Added feature")
- Reference issues: "Fixes #123" or "Relates to #456"

### 5. Push Changes

```bash
git push origin feature/add-new-behavior
```

### 6. Create Pull Request

1. Go to your fork on GitHub
2. Click "Pull Request" button
3. Fill out the PR template:

**Title:** Clear, concise description

**Description:** Include:
- What changes were made
- Why the changes are needed
- How to test the changes
- Related issues (Fixes #123)
- Screenshots (if UI changes)
- Breaking changes (if any)

**Example PR description:**

```markdown
## Description
Adds support for dual GPS configuration with automatic blending.

## Motivation
Users with dual GPS modules need automatic blending for improved 
accuracy and redundancy.

## Changes
- Added GPS blending logic to sensor processing
- Updated configuration documentation
- Added example mission files
- Tested with Here3 GPS modules

## Testing
- [ ] Tested on bench with dual GPS
- [ ] Verified GPS switching works
- [ ] Tested in flight (50km mission)
- [ ] Documentation updated

## Related Issues
Fixes #123
Relates to #456

## Screenshots
(If applicable)

## Breaking Changes
None
```

### 7. Code Review

**Expect feedback:**
- Maintainers will review your PR
- May request changes
- Be responsive to comments
- Don't take criticism personally

**Address feedback:**
```bash
# Make requested changes
git add .
git commit -m "Address review feedback"
git push origin feature/add-new-behavior
```

**Updates appear automatically** in your PR.

### 8. Merge

Once approved:
- Maintainer will merge your PR
- Your changes are now in main branch
- Celebrate! 🎉

## Code Standards

### C++ Code

```cpp
// Class names: UpperCamelCase
class MyNewBehavior : public IvPBehavior {
public:
   MyNewBehavior();
   ~MyNewBehavior() {};

   // Function names: lowerCamelCase
   bool setParam(string, string);
   IvPFunction* onRunState();

protected:
   // Member variables: m_lower_with_underscore
   double m_desired_heading;
   string m_status_message;
   
   // Constants: UPPER_CASE
   static const int MAX_ITERATIONS = 100;
};
```

### Mission Files

```moos
// Clear structure
ServerHost = localhost
ServerPort = 9000
Community  = skywalker1

// Comments explain purpose
ProcessConfig = ANTLER
{
  MSBetweenLaunches = 200
  
  // One line per process
  Run = MOOSDB          @ NewConsole = false
  Run = pHelmIvP        @ NewConsole = false
}

// Organized parameters
ProcessConfig = pHelmIvP
{
  AppTick   = 4
  CommsTick = 4
  
  // Group related settings
  behaviors  = behaviors.bhv
  domain     = course:0:359:360
  domain     = speed:0:25:26
}
```

### Documentation

Use Markdown for documentation:

```markdown
# Main Title

Brief introduction paragraph.

## Section Title

Content with **bold** and *italic* where appropriate.

### Subsection

- Use bulleted lists
- For series of items
- Keep items parallel

1. Numbered lists
2. For sequential steps
3. Or ranked items

Code blocks with syntax highlighting:
```bash
command --with-options
```

[Links](https://example.com) to related resources.
```

## Testing Requirements

### Before Submitting PR

- [ ] Code compiles without warnings
- [ ] All tests pass
- [ ] New features have tests
- [ ] Documentation updated
- [ ] Commit messages clear
- [ ] Branch up to date with main

### Testing Checklist

**For Code Changes:**
- [ ] Tested locally
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] No regression in existing functionality

**For Hardware Changes:**
- [ ] Tested on bench
- [ ] Tested in flight (if applicable)
- [ ] Multiple configurations tested
- [ ] Failure modes tested

**For Documentation:**
- [ ] All links work
- [ ] Commands tested
- [ ] Code examples run
- [ ] Screenshots current

## Documentation Guidelines

### Writing Style

- Use clear, simple language
- Write in second person ("You can...")
- Use active voice
- Be concise but thorough
- Include examples

### Structure

```markdown
# Title

Brief overview (1-2 paragraphs).

## Prerequisites

List requirements.

## Step 1: First Action

Instructions with code examples.

## Step 2: Next Action

More instructions.

## Troubleshooting

Common issues and solutions.

## Next Steps

Links to related guides.
```

### Code Examples

Always test code examples:

```bash
# This should actually work
cd ~/project-skywalker
./build.sh
```

Include expected output when helpful:

```bash
$ ./build.sh
Building MOOS-IvP...
[100%] Built target MyApp
Build complete!
```

## Reporting Bugs

### Before Reporting

1. Search existing issues
2. Update to latest version
3. Verify it's reproducible
4. Collect information

### Bug Report Template

```markdown
## Description
Clear description of the bug.

## To Reproduce
Steps to reproduce:
1. Go to '...'
2. Run command '...'
3. Observe error

## Expected Behavior
What should happen.

## Actual Behavior
What actually happens.

## Environment
- Hardware: Skywalker X8, CubeOrange+, RPi 4B 8GB
- Software: ArduPilot 4.5.7, MOOS-IvP [version]
- OS: Raspberry Pi OS 64-bit
- Branch/Commit: main @ abc1234

## Logs
```
Paste relevant logs here
```

## Additional Context
Screenshots, error messages, etc.
```

## Suggesting Features

### Feature Request Template

```markdown
## Problem Statement
Describe the problem this feature would solve.

## Proposed Solution
How should it work?

## Alternatives Considered
What other solutions did you consider?

## Use Cases
Who would benefit and how?

## Additional Context
Mockups, examples, references.
```

## Community Guidelines

### Code of Conduct

We are committed to providing a welcoming and inclusive environment:

- **Be respectful** - Value diverse opinions and experiences
- **Be collaborative** - Work together constructively
- **Be patient** - Remember everyone was a beginner once
- **Be professional** - Keep discussions focused and productive
- **Be considerate** - Your work affects others
- **Be open** - Accept constructive criticism gracefully

### Unacceptable Behavior

- Harassment or discrimination
- Personal attacks or insults
- Trolling or inflammatory comments
- Publishing private information
- Unprofessional conduct

### Reporting Issues

If you experience or witness unacceptable behavior:
- Contact: team@pavlab-mit.org
- All complaints reviewed and investigated
- Confidentiality maintained

## Getting Help

### Resources

- **Documentation**: [docs/](index.md)
- **MOOS-IvP Docs**: [moos-ivp.org](https://oceanai.mit.edu/moos-ivp/)
- **Issues**: [GitHub Issues](https://github.com/pavlab-mit/project-skywalker/issues)

### Contact

- **General**: team@pavlab-mit.org
- **Repository Access**: chbenj36@gmail.com
- **GitHub Issues**: For bug reports and features

## Recognition

Contributors will be:
- Listed in [Authors](authors.md)
- Credited in release notes
- Acknowledged in relevant documentation

Thank you for contributing to Project Skywalker!

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.

---

**Previous**: [Development Guide](development.md) | **Next**: [Authors](authors.md)
