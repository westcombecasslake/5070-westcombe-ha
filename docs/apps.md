# Home Assistant Apps (Add-ons)

**Document Purpose:** Documentation for Home Assistant apps installed on this system

**Location in HA:** Settings → Apps

---

## Installed Apps

This system has three apps installed:

1. **Home Assistant Google Drive Backup** - Secondary backup system
2. **Samba Share** - Network file access  
3. **Studio Code Server** - Web-based YAML editor

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

# Samba Share

**Type:** Network File Access  
**Purpose:** Expose Home Assistant folders via SMB/CIFS for network access  
**Status:** Active

---

## What is Samba Share?

An add-on/app that exposes Home Assistant configuration folders to your local network using the SMB/CIFS protocol. Allows access to HA files from other computers on the network.

**Key Features:**
- Network access to HA config files
- Windows/Mac/Linux compatible
- Edit files remotely
- Browse directories
- Copy files to/from HA

---

## Role in This System

**Network File Access:**
- Move files from Claude AI web client downloads to HA directories
- Primary use: Transfer files to `/config/packages/` and `/config/docs/`
- Copy files to/from Home Assistant
- Browse HA directory structure
- Access configuration files from network

**Common Uses:**
- Move documentation files created in Claude to `/config/docs/`
- Transfer YAML packages from downloads to `/config/packages/`
- Copy backup files to local computer
- Upload files to HA (scripts, images, etc.)
- Review logs and configuration
- Edit files with desktop editor if needed

---

## Accessing Samba Share

**From Windows:**
1. Open File Explorer
2. Navigate to: `\\homeassistant` or `\\192.168.50.54`
3. Enter credentials if prompted
4. Browse available folders

**From Mac:**
1. Finder → Go → Connect to Server
2. Enter: `smb://homeassistant` or `smb://192.168.50.54`
3. Enter credentials if prompted
4. Browse available folders

**From Linux:**
1. File manager → Connect to Server
2. Enter: `smb://homeassistant` or `smb://192.168.50.54`
3. Enter credentials
4. Browse available folders

---

## Available Folders

**Key directories for file transfers:**
- `/config/packages/` - **Primary target for YAML package files**
- `/config/docs/` - **Primary target for documentation files**
- `/config` - Main configuration folder
- `/addons` - Add-on data
- `/backup` - Local backups
- `/share` - Shared data folder
- `/ssl` - SSL certificates

**Typical Workflow:**
1. Create files in Claude AI web client
2. Download files to local computer
3. Access HA via Samba Share on network
4. Copy files to `/config/packages/` or `/config/docs/`
5. Files immediately available to Home Assistant

---

## Configuration

**Access Credentials:**
- Username/password configured in Samba Share settings
- Different from HA login credentials
- Can be unique credentials for network access

**Network Settings:**
- Accessible on local network only
- Not exposed to internet
- Uses standard SMB/CIFS ports

---

## Important Notes

- **Local network only** - not accessible from internet
- **File editing** - use with caution, validate YAML syntax
- **Configuration access** - provides direct access to all config files
- **Credentials needed** - separate from HA login
- **Windows/Mac/Linux** - compatible with all platforms
- **Security** - ensure strong password for network access

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

**Primary Code Editor:**
- **Recommended way to add and check code** in Home Assistant
- **This was used to build the entire system** - all YAML packages created here
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
- Samba Share → Set new credentials if desired
- Studio Code Server → Protected by HA authentication

**After Ownership Transfer:**
1. Update Google Drive Backup with new Google account (or keep house account)
2. Change Samba Share credentials if desired
3. Studio Code Server automatically works with new HA credentials
4. Test all apps functionality

---

## Official Resources

**Home Assistant App Documentation:**
- https://www.home-assistant.io/addons/

**Specific Apps:**
- Google Drive Backup: https://github.com/sabeechen/hassio-google-drive-backup
- Samba Share: https://github.com/home-assistant/addons/tree/master/samba
- Studio Code Server: https://github.com/hassio-addons/addon-vscode

---

## Important Notes

- **Three apps installed** - Google Drive Backup, Samba, Studio Code Server
- **Google Drive account** - westcombecasslake@gmail.com (transfers with house)
- **Backup redundancy** - Two independent backup systems for protection
- **Network access** - Samba provides file access from other computers
- **Code editing** - Studio Code Server for advanced YAML editing
- **All protected** - Apps require authentication for security
- **Local network** - Apps accessible on local network, not internet (by default)
- **Location** - Settings → Apps
- **Launch icons** - Apps with interfaces show in left navigation bar

---

**Last Updated:** February 2026  
**Document Maintainer:** Homeowner
