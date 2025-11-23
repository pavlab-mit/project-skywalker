# Project Skywalker Documentation

Welcome to the Project Skywalker documentation! This directory contains comprehensive guides for building and operating autonomous UAV systems using MOOS-IvP on the Skywalker X8 platform.

## 📖 Documentation Index

### Getting Started
- **[Documentation Home](index.md)** - Overview and navigation
- **[Installation Guide](installation.md)** - Set up Raspberry Pi and MOOS-IvP
- **[Quick Start](quickstart.md)** - Get running quickly

### Core Guides
- **[Usage Guide](usage.md)** - Operating the system and running missions
- **[Architecture](architecture.md)** - System design and components

### Hardware & Configuration
- **[Hardware Assembly](hardware.md)** - Physical integration
- **[Radio Setup](radio-setup.md)** - RFD900x and mesh networking
- **[Firmware Guide](firmware.md)** - ArduPilot configuration

### Development & Contributing
- **[Development Guide](development.md)** - Creating custom applications
- **[Contributing](contributing.md)** - How to contribute

### Reference
- **[Troubleshooting](troubleshooting.md)** - Common problems and solutions
- **[Authors](authors.md)** - Contributors and acknowledgements
- **[Changelog](changelog.md)** - Version history

## 🌐 Viewing the Documentation

### GitHub Pages
The documentation is available as a website at:
https://pavlab-mit.github.io/project-skywalker/

### Locally
To view locally, you can:

1. **Browse on GitHub** - Click any `.md` file above
2. **Clone the repository**
   ```bash
   git clone https://github.com/pavlab-mit/project-skywalker.git
   cd project-skywalker/docs
   ```
3. **Use a Markdown viewer** - Any editor that supports Markdown

### Build with Jekyll (Optional)
To build the site locally with Jekyll:

```bash
# Install Jekyll and dependencies
gem install bundler jekyll

# Create Gemfile
cat > Gemfile << EOF
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
gem "jekyll-feed"
gem "jekyll-sitemap"
gem "jekyll-seo-tag"
EOF

# Install dependencies
bundle install

# Serve locally
bundle exec jekyll serve

# Visit http://localhost:4000
```

## 📚 Additional Resources

### In This Repository
- `../Info/` - Detailed technical guides and design decisions
- `../README.md` - Main project README

### External Links
- [MOOS-IvP Documentation](https://oceanai.mit.edu/moos-ivp/)
- [ArduPilot Documentation](https://ardupilot.org/)
- [RFD900x User Manual](https://rfdesign.com.au/)

## 📧 Contact

- **General**: team@pavlab-mit.org
- **Technical**: chbenj36@gmail.com
- **Issues**: [GitHub Issues](https://github.com/pavlab-mit/project-skywalker/issues)

## 🤝 Contributing

We welcome contributions to improve the documentation! See the [Contributing Guide](contributing.md) for details.

---

**Start here**: [Documentation Home](index.md)
