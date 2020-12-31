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
To bind your menu item to one of the admincp modules, you will need to create two functions that are run in admincp. These functions are then run when needed by utilizing 2 [Hooks](hooks/).

The first hook you will need is: 
```
  $plugins->add_hook('admin_{module_codename}_menu', 'callback_function');
```
This hook will call your callback function when navigating to the module corresponding to the codename. Simply replace {module_codename} with any of the codenames listed above.

The second hook you will need is the action handler for your page. You will need to create a separate file that handles the actual content of the page, we will create this file after setting up the plugin file properly. The hook looks similar to the first hook, and again you will have to replace the module codename with the corresponding codename listed above. (Must be the same for both hooks for this to work).
```
  $plugins->add_hook('admin_{module_codename}_action_handler', 'callback_function_action_handler');	
```

It is also good to wrap these plugin hooks in a check to see if the plugin is run whilst accessing AdminCP (Since this is essentially the only time when these functions needs to be called).

An example of the two hooks wrapped in a check where the forum module is used:
```
if(defined('IN_ADMINCP')) {
	$plugins->add_hook('admin_forum_menu', 'callback_function');
	$plugins->add_hook('admin_forum_action_handler', 'callback_function_action_handler');	
}
```

Next up we need to create the callback functions that need to run. The two hooks also sends a pointer with the function call, and we will need to populate our information by editing the variable.

The process is pretty straightforward, here are two example codes of the functions called for the forum module:
```
function callback_function(&$sub_menu) {
    $sub_menu[] = ['id' => 'customadmin-page', 'title' => 'Custom Admin Page', 'link' => 'index.php?module=forum-customadmin-page'];
}

function callback_function_action_handler(&$actions) {
    $actions['customadmin-page'] = ['active' => 'customadmin-page', 'file' => 'customadmin-page.php'];
}
```

Now we can create our custom admin page file inside the forum module folder, located at */admin/modules/forum/*. Simply create a file with the same name as supplied in the array with key 'file'. In this case the filename will be *customadmin-page.php*.

#### Step 3. Custom Admin Page File
