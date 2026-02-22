# Backup Integration

**Integration Type:** System Backup Service  
**Devices in System:** 1 service  
**Primary Use:** Home Assistant configuration backups  
**Status:** Active

---

## What is Backup Integration?

The Backup integration is a built-in Home Assistant service that creates full backups of the entire HA configuration, including settings, automations, integrations, add-ons, and user data.

**Key Features:**
- Full system backups
- Includes all configuration files
- Add-on data backup
- Automated backup scheduling
- Cloud storage via Nabu Casa
- One-click restore capability
- Backup before updates

---

## Role in This System

**Primary Function:**
- **System backup and recovery** - protects Home Assistant configuration
- **Cloud backup via Nabu Casa** - automatic off-site storage

**How It's Used:**
- Automatic backups on schedule
- Cloud backup through Nabu Casa subscription
- Manual backups before major changes
- Backup before HA updates
- Disaster recovery capability

**Additional Backup System:**
This is the built-in Home Assistant backup. A **second backup system** using the Google Drive Backup add-on/app is also configured. See `/docs/apps.md` for details on the Google Drive backup system.

---

## Home Assistant Configuration

### Integration Setup

**Location:** Settings â†’ System â†’ Backups

**Built-in Service:**
- No configuration required
- Automatically available in all HA installations
- Cloud backup via Nabu Casa subscription
- Local storage + cloud storage

**Nabu Casa Cloud Backup:**
- Automatic upload to Nabu Casa cloud
- Included with Home Assistant Cloud subscription
- Secure off-site storage
- Easy restore from cloud

---

## Backup Contents

**What's Included:**
- All YAML configuration files
- Automations and scripts
- Integration settings
- User accounts and preferences
- Add-on configurations
- Custom components
- Dashboards and views
- Secrets file

**What's NOT Included:**
- Historical sensor data (database)
- Large media files (optional)
- Temporary files

---

## Creating Backups

**Manual Backup:**
1. Settings â†’ System â†’ Backups
2. Click "Create Backup"
3. Choose full or partial backup
4. Wait for completion
5. Automatically uploads to Nabu Casa cloud

**Automatic Backups:**
- Configured in backup settings
- Scheduled daily/weekly
- Automatic cloud upload
- Automatic cleanup of old backups

---

## Restoring from Backup

**Restore Process:**
1. Settings â†’ System â†’ Backups
2. Select backup to restore (local or cloud)
3. Choose full or partial restore
4. Confirm restoration
5. System restarts with restored configuration

**Restore from Cloud:**
- Access cloud backups from any HA instance
- Download from Nabu Casa cloud
- Restore on new hardware
- No local backup needed

**When to Restore:**
- After failed update
- Hardware replacement
- Configuration errors
- Disaster recovery

---

## Best Practices

**Backup Frequency:**
- Before major changes
- Before Home Assistant updates
- Regular schedule (daily recommended)
- After significant automation changes

**Storage:**
- Nabu Casa cloud (automatic)
- Google Drive backup (secondary system - see apps.md)
- Maintain multiple backup versions
- Test restore process periodically

**Redundancy:**
This system has **two backup systems**:
1. Built-in HA backup with Nabu Casa cloud
2. Google Drive Backup add-on/app (separate redundancy)

---

## Nabu Casa Subscription

**Home Assistant Cloud:**
- Monthly subscription service ($6.50/month or $65/year)
- Includes cloud backups
- Remote access to HA
- Voice assistant integration
- **Account:** westcombecasslake@gmail.com (house email)

**Ownership Transfer:**
Nabu Casa account will be transferred to new owner. See `transferring-ownership.md` for complete account transfer procedures including how to change account email and update payment method while maintaining active subscription.

**Backup Benefits:**
- Automatic cloud upload
- Secure off-site storage
- Access from anywhere
- Hardware failure protection
- Easy restore to new hardware

---

## Migration Notes

**For New Owner:**
- Backups contain entire HA configuration
- Nabu Casa subscription transfers with house
- Cloud backups accessible with account
- All automations and integrations preserved
- Accounts and integrations need re-authentication

**If Replacing Home Assistant:**
1. Access Nabu Casa cloud backups
2. Install HA on new system
3. Restore from cloud backup
4. Re-authenticate integrations (Pushover, Ring, YoLink, etc.)
5. Verify all systems operational

**Account Information:**
- Nabu Casa account on house email: westcombecasslake@gmail.com
- Account will be transferred to new owner (see `transferring-ownership.md` for process)
- New owner can change account email and update payment method
- Subscription remains active during transition
- Google Drive backup also available (see apps.md)

---

## Official Resources

**Home Assistant Documentation:**
- Backup documentation: https://www.home-assistant.io/common-tasks/os/#backups
- Restore guide: https://www.home-assistant.io/common-tasks/os/#restoring-a-backup
- Nabu Casa: https://www.nabucasa.com

---

## Important Notes

- **Built-in service** - no setup required
- **Nabu Casa cloud backup** - included with subscription
- **Essential for disaster recovery** - regular backups critical
- **Includes all configuration** - complete system restore possible
- **Cloud storage** - protected against local hardware failure
- **Two backup systems** - Built-in + Google Drive for redundancy
- **Integrations need re-auth** - some accounts require login after restore
- **Test restores periodically** - verify backups work
- **Before updates** - always backup before major HA updates
- **Account transfers** - Nabu Casa subscription transfers with house

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
