# Home Assistant Apps (Add-ons)

**Document Purpose:** Documentation for Home Assistant apps installed on this system

**Location in HA:** Settings → Apps

---

## Installed Apps

This system has five apps installed:

1. **Home Assistant Google Drive Backup** - Secondary backup system
2. **Studio Code Server** - Web-based YAML editor
3. **Claude Code** - AI-assisted automation development
4. **Matter Server** - Matter protocol controller (background service)
5. **Open Thread Border Router** - Thread border router via ZBT-2 hardware (background service)

---

# Home Assistant Google Drive Backup

**Type:** Backup Solution  
**Purpose:** Secondary backup system with Google Drive cloud storage  
**Status:** Active

---

## What is Google Drive Backup?

An add-on/app that automatically manages backups between Home Assistant and Google Drive. Provides a second independent backup system beyond the built-in HA backup with Nabu Casa.

**Key Features:**
- Automatic backups to Google Drive
- Independent of Nabu Casa
- Configurable retention policies
- Web interface for backup management
- Automatic cleanup of old backups
- One-click restore capability

---

## Role in This System

**Redundant Backup System:**
This provides a **second backup system** independent of the built-in HA backup:

1. **Built-in HA Backup** → Nabu Casa cloud
2. **Google Drive Backup app** → Google Drive cloud (westcombecasslake@gmail.com)

**Why Two Systems?**
- Redundancy in case one backup system fails
- Different cloud providers (Nabu Casa vs Google)
- Additional protection for critical home automation system

---

## Configuration

**Google Account:**
- **Account:** westcombecasslake@gmail.com (house email)
- **Transfers with house** - account credentials provided to new owner

**Backup Location:**
- Google Drive folder dedicated to HA backups
- Automatic upload after each backup
- Configurable retention (keep X days/weeks)

**Backup Schedule:**
- Configured within app settings
- Typically daily or after changes
- Automatic cleanup of old backups

---

## Accessing Google Drive Backup

**Launch Icon (Primary):**
- Launch icon in left HA app nav bar
- Opens web interface within HA app

**Via Apps Page:**
1. Settings → Apps
2. Click "Home Assistant Google Drive Backup"
3. Opens web interface
4. View backup status, create backups, restore

**Configuration:**
- Settings tab in app interface
- Configure schedule, retention
- Manage Google Drive authorization
- Set backup naming

---

## Creating and Restoring Backups

**Automatic Backups:**
- Scheduled automatically
- Uploads to Google Drive
- Cleanup per retention policy

**Manual Backup:**
1. Open Google Drive Backup web interface
2. Click "Backup Now"
3. Wait for completion
4. Automatically uploaded to Google Drive

**Restore:**
1. Open Google Drive Backup web interface
2. Select backup from list
3. Click "Restore"
4. Confirm
5. System restarts with restored configuration

---

## Important Notes

- **Secondary backup system** - complements built-in HA backup
- **Google Drive storage** - separate from Nabu Casa cloud
- **Account transfers** - westcombecasslake@gmail.com provided to new owner
- **Redundancy** - provides additional protection
- **Independent** - continues working even if Nabu Casa subscription lapses

---

# Claude Code

**Type:** AI-Assisted Development
**Purpose:** AI coding assistant with direct access to Home Assistant files
**Status:** Active

---

## What is Claude Code?

An add-on that runs Claude (Anthropic's AI) directly inside the Home Assistant container, with full read/write access to all configuration files. Replaces the Samba + external editor workflow — files are read and edited in place, no file transfers needed.

**Key Features:**
- Direct read/write access to `/homeassistant/` files
- AI-assisted YAML writing, review, and debugging
- Bash command execution within the container
- Code search and analysis across all packages
- Git integration for version control
- Persistent memory across sessions

---

## Role in This System

**Primary automation development tool:**
- Write and review YAML automation packages
- Conduct subsystem reviews (cross-check YAML against docs)
- Maintain documentation alongside code
- Run git commits after each change

**Replaces Samba for file work:**
- Files are edited directly in the container — no download/upload cycle
- Changes take effect immediately without file transfers

---

## Accessing Claude Code

**Via Apps Page:**
1. Settings → Apps
2. Click "Claude Code"
3. Opens terminal session with Claude

---

## Important Notes

- **Requires Anthropic API key** - configured in add-on settings
- **Direct file access** - operates on live configuration files; changes apply immediately
- **Git recommended** - commit changes after each session for rollback capability
- **Session memory** - maintains context across conversations via memory files in `/root/.claude/`

---

# Studio Code Server

**Type:** Web-Based Code Editor  
**Purpose:** Full Visual Studio Code experience integrated in Home Assistant  
**Status:** Active

---

## What is Studio Code Server?

An add-on/app that provides a full Visual Studio Code (VSCode) editor accessible through your web browser, integrated directly into Home Assistant.

**Key Features:**
- Full VSCode editor in browser
- Syntax highlighting for YAML
- IntelliSense and autocomplete
- Integrated file browser
- Multi-file editing
- Extensions support
- Terminal access
- Git integration

---

## Role in This System

**Manual Code Editor:**
- Direct YAML editing with syntax highlighting and entity validation
- Useful for quick edits, browsing the file tree, or when Claude Code is not needed
- Edit YAML configuration files
- Create and modify automations
- Edit scripts and packages
- Syntax validation
- File management
- Terminal access to HA

**Advantages Over Simple Editor:**
- Advanced YAML syntax checking
- Multiple files open simultaneously
- Better error detection
- More powerful search/replace
- Extensions for HA development
- Entity name validation and dropdown

---

## Accessing Studio Code Server

**Launch Icon (Primary):**
- Launch icon in left HA app nav bar
- Click icon to open Studio Code Server
- Window opens within HA app (integrated experience)

**Via Apps Page:**
1. Settings → Apps
2. Click "Studio Code Server"
3. Opens within HA interface

**Best Practice:**
After completing work, **stop the app** to free up system resources. While it doesn't noticeably affect system performance, Studio Code Server uses significant resources when running.

**To Stop:**
1. Settings → Apps → Studio Code Server
2. Click "Stop"
3. Restart when needed for editing

---

## Common Tasks

**Edit YAML Files:**
1. Open Studio Code Server
2. Navigate to `/config/packages/` (or other folder)
3. Open file
4. Edit with full syntax highlighting
5. Save (changes apply immediately)

**Home Assistant Validation Features:**
- **HA-specific YAML validation** - catches Home Assistant configuration errors
- **Entity name dropdown** - IntelliSense shows available entities as you type
- **Entity validation** - verifies entity IDs exist in your system
- Syntax highlighting shows errors in real-time
- Autocomplete for HA services and attributes

**Check Configuration:**
- Use built-in YAML validation
- Syntax highlighting shows errors
- IntelliSense helps with entity IDs
- Red underlines indicate problems

**Terminal Access:**
- Built-in terminal for command-line tasks
- Can restart HA services
- View logs
- Run commands

---

## Configuration

**Editor Settings:**
- Theme preferences
- Font size
- Keybindings
- Extensions

**Access:**
- Protected by HA authentication
- Additional password optional
- Local network access
- **Can be used remotely** via Nabu Casa remote access
- **Has been used remotely** to make system changes

**Note on Remote Use:**
Studio Code Server can be accessed remotely through Nabu Casa's remote UI. However, be cautious with remote editing and ensure changes are validated before saving.

---

## Extensions

**HA Extensions Available:**
- Home Assistant Config Helper
- YAML Language Support
- YAML Validator
- File Icons
- Git integration tools

**Installation:**
- Extensions panel in VSCode
- Search and install
- Enhances editing experience

---

## Important Notes

- **Full VSCode** - complete editor, not simplified
- **Browser-based** - no desktop software needed
- **YAML editing** - best tool for editing HA configuration
- **Syntax validation** - prevents config errors
- **HA-specific features** - entity validation, dropdown, HA YAML checking
- **Terminal access** - advanced users can run commands
- **Protected** - requires HA authentication
- **Remote access capable** - can be used via Nabu Casa remote UI
- **Resource usage** - Uses significant resources when running. Doesn't seem to affect system performance, but **shut down after completing work** to free resources
- **Recommended tool** - this was used to build the entire HA system

---

# Matter Server

**Type:** Protocol Infrastructure
**Purpose:** Matter controller required for all Matter devices
**Status:** Active (must remain running)

---

## What is Matter Server?

A background add-on that runs the Python Matter Server, implementing the Matter smart home protocol controller. All Matter devices in this system require this server to communicate with Home Assistant.

**Key Features:**
- Implements Matter controller for Home Assistant
- Manages Matter device commissioning and communication
- Required for all Matter/Thread device operation
- Runs as a background service with no user interface

---

## Role in This System

**Required infrastructure for Matter devices:**
- IKEA air quality sensors (CO2, PM2.5, TVOC) in airquality.yaml
- Bathroom fan switches (middle bath, east bath)

**This app must remain running at all times.** Stopping it disconnects all Matter devices from Home Assistant.

---

## Important Notes

- **Background service** - no user interface, just needs to be running
- **Do not stop** - disconnects all Matter devices
- **Auto-starts** - configured to start automatically with HA
- See: matter-thread-integration.md

---

# Open Thread Border Router

**Type:** Protocol Infrastructure
**Purpose:** Thread border router using Silicon Labs ZBT-2 hardware
**Status:** Active (must remain running)

---

## What is Open Thread Border Router?

A background add-on that provides Thread border router functionality using the Silicon Labs ZBT-2 chip built into the Home Assistant Green. Thread devices communicate via this border router to reach the IP network and Home Assistant.

**Key Features:**
- Thread border router using ZBT-2 hardware built into HA Green
- Bridges Thread mesh network to IP network
- Required for all Thread device communication
- Runs as a background service with no user interface

---

## Role in This System

**Required infrastructure for Thread devices:**
- IKEA air quality sensors communicate via Thread
- Bathroom fan switches (middle bath, east bath) use Thread/Matter

**This app must remain running at all times.** Stopping it disconnects all Thread devices.

---

## Important Notes

- **Background service** - no user interface, just needs to be running
- **Do not stop** - disconnects all Thread devices
- **Hardware dependent** - requires Silicon Labs ZBT-2 installed in HA Green
- **Auto-starts** - configured to start automatically with HA
- See: matter-thread-integration.md

---

# App Management

## Accessing Apps

**Location:** Settings → Apps

**Interface:**
- Shows all installed apps
- Install new apps from app store
- Start/stop/restart apps
- View logs
- Configure settings
- Launch icons for apps with interfaces appear in left navigation bar

---

## App Updates

**Automatic Updates:**
- Can be enabled per app
- Apps update automatically
- Or manual updates via interface

**Before Updating:**
- Create backup
- Review release notes
- Check for breaking changes

---

## Migration Notes for New Owner

**All Apps Transfer:**
- Google Drive Backup → Account: westcombecasslake@gmail.com
- Studio Code Server → Protected by HA authentication
- Claude Code → Requires new owner's Anthropic API key
- Matter Server → No account needed, transfers automatically
- Open Thread Border Router → No account needed, transfers automatically

**After Ownership Transfer:**
1. Update Google Drive Backup with new Google account (or keep house account)
2. Studio Code Server automatically works with new HA credentials
3. Claude Code: replace Anthropic API key in add-on settings with your own
4. Matter Server and Open Thread Border Router require no changes
5. Verify all Matter/Thread devices are online after transfer

---

## Official Resources

**Home Assistant App Documentation:**
- https://www.home-assistant.io/addons/

**Specific Apps:**
- Google Drive Backup: https://github.com/sabeechen/hassio-google-drive-backup
- Studio Code Server: https://github.com/hassio-addons/addon-vscode
- Matter Server: https://github.com/home-assistant-libs/python-matter-server
- Open Thread Border Router: https://github.com/home-assistant/addons/tree/master/openthread_border_router

---

## Important Notes

- **Five apps installed** - Google Drive Backup, Studio Code Server, Claude Code, Matter Server, Open Thread Border Router
- **Google Drive account** - westcombecasslake@gmail.com (transfers with house)
- **Backup redundancy** - Two independent backup systems for protection
- **Infrastructure apps** - Matter Server and Open Thread Border Router must always be running
- **Code editing** - Studio Code Server for manual YAML editing, Claude Code for AI-assisted development
- **All protected** - Apps require authentication for security
- **Location** - Settings → Apps
- **Launch icons** - Apps with interfaces show in left navigation bar

---

**Last Updated:** February 2026  
**Document Maintainer:** Homeowner
