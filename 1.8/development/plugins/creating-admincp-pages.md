---
layout: page
title:  "Creating AdminCP Pages"
categories: [plugins]
---

## Create a new AdminCP Page

### Why use AdminCP Pages
When you want to offer administrative options beyond [Plugin Settings](basics/#plugin-settings) for admins of a board, similar to plugins such as 'BAM Announcement Manager' or 'My Advertisements', it can often be a good idea to consider adding custom admincp pages.

#### Step 1. Deciding on a Module
Before adding a custom admincp page, look at the type of options you want to offer and where they belong in the AdminCP. In example if your plugin offers custom announcements, there is a chance the page fits better under 'Forums & Posts' more so than 'Tools & Maintenance'.

There are 5 modules to choose from and they are:
##### Configuration
_codename=config_
The configuration module is where most if not all settings for your board and plugins are located.

Probable Usage: If your plugin has extensive settings that maintains its own CRUD operations, such as custom lists that impact the usage of the plugin, but is direclty administered individually by each board admin, then Configuration is a good place for your AdminCP page.

##### Forums & Posts
_codename=forum_
This module is where the forum nodes, posts and attachments are managed. 

Probable Usage: If your plugins is a forum extension such as a 'forum banner', where you want to let admins administer the uploaded banners more easily for each forum, this could be a good place to have your AdminCP page.

##### Users & Groups
_codename=user_
Anything with managing users, moderators, permissions and groups goes here. 

Probable Usage: In case your plugin offers some extensive individual user management or group management, you can easily place your AdminCP page in this module.

##### Templates & Style
_codename=style_
This is where admins manage the themes, templates and stylesheets of their board.

Probable Usage: If your plugin offers extensions to the template editor or similar, you can place your customization options in this module.

##### Tools & Maintenance
_codename=tools_
This module has everything to do with the health of the board, all from viewing logs and statistics to managing tasks, caching and database.

Probable Usage: Maybe your plugin offers some kind of custom logging for admins or similar maintenance functionality, then you can place your logs in this module.

#### Step 2. Setting up your AdminCP page
