---
layout: page
title:  "Creating AdminCP Pages"
categories: [plugins]
---

## Create a new AdminCP Page

### Why use AdminCP Pages
When you want to offer administrative options beyond [Plugin Settings](basics.md/#plugin-settings) for admins of a board, similar to plugins such as 'BAM Announcement Manager' or 'My Advertisements', it can often be a good idea to consider adding custom admincp pages.

### Step 1. Deciding on a Module
Before adding a custom admincp page, look at the type of options you want to offer and where they belong in the AdminCP. In example if your plugin offers custom announcements, there is a chance the page fits better under 'Forums & Posts' more so than 'Tools & Maintenance'.

There are 5 modules to choose from and they are:
#### Configuration
_codename=config_

The configuration module is where most if not all settings for your board and plugins are located.

Probable Usage: If your plugin has extensive settings that maintains its own CRUD operations, such as custom lists that impact the usage of the plugin, but is direclty administered individually by each board admin, then Configuration is a good place for your AdminCP page.

#### Forums & Posts
_codename=forum_

This module is where the forum nodes, posts and attachments are managed. 

Probable Usage: If your plugins is a forum extension such as a 'forum banner', where you want to let admins administer the uploaded banners more easily for each forum, this could be a good place to have your AdminCP page.

#### Users & Groups
_codename=user_

Anything with managing users, moderators, permissions and groups goes here. 

Probable Usage: In case your plugin offers some extensive individual user management or group management, you can easily place your AdminCP page in this module.

#### Templates & Style
_codename=style_

This is where admins manage the themes, templates and stylesheets of their board.

Probable Usage: If your plugin offers extensions to the template editor or similar, you can place your customization options in this module.

#### Tools & Maintenance
_codename=tools_

This module has everything to do with the health of the board, all from viewing logs and statistics to managing tasks, caching and database.

Probable Usage: Maybe your plugin offers some kind of custom logging for admins or similar maintenance functionality, then you can place your logs in this module.

### Step 2. Setting up your AdminCP page
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

You can decide on an id that fits your usecase scenario, it is also good practice to have the original module included in your link. Such as the above case where the link attributes include _module_ and the value is _forum-customadmin-page_ where forum is the original module codename. The _id_ you give your admincp page must be the same as the _key_ you provide to the _$actions_ variable.

Now we can create our custom admin page file inside the forum module folder, located at */admin/modules/forum/*. Simply create a file with the same name as supplied in the array with key 'file'. In this case the filename will be *customadmin-page.php*.

### Step 3. Custom Admin Page File
Our custom AdminCP page can now be constructed. The first thing we can do is add the default tabs to the admincp page.

The dummy page will include 2 default tabs, which are tabs always shown no matter what action is being done. The top of the php file including the 2 tabs looks like this:
```
<?php

if(!defined('IN_MYBB'))
{
	die('You are not allowed direct access to this file.');
}

/* Load Language */
$lang->load('customadmin-page');

$sub_tabs['customadmin_default'] = array(
	'title' => $lang->customadmin_default,
	'link' => 'index.php?module=forum-customadmin-page',
	'description' => $lang->customadmin_default_desc
);

$sub_tabs['customadmin_new'] = array(
	'title' => $lang->customadmin_new,
	'link' => 'index.php?module=forum-customadmin-page&action=new',
	'description' => $lang->customadmin_new_desc
);

/* Output header */
$page->output_header($lang->customadmin_title);
```

Whether you use a language file or not, is an individual decision.

Now is a good time to implement the logic that decides what content to output. For this example the _action_ parameter will be used to control the output, and a simple _if_ structure. We can use the included admin classes _form_, _page_ and _table_ to build the admin page.

The following code will output _tabs_ and _footer_ but no page content. The usage of the aforementioned admin classes are explained later.
```
if ($mybb->input['action'] == 'new')
{
	$page->output_nav_tabs($sub_tabs, 'customadmin_new');
	
	/** Page content here *//
} else 
{
	$page->output_nav_tabs($sub_tabs, 'customadmin_default');
	
	/** Page content here *//
}

$page->output_footer($lang->customadmin_title_acronym);
```

### Page Content
The following will not discuss how to output content in AdminCP other than by using classes included in **MyBB** for building AdminCP pages.

#### Outputting Tables
In the _table with data_ example any iteration of data can substitue the iteration of users.

##### Simple Table
In this example we output a table that is not built iterably.

First we create a new instance (object) of the _Table_ class and we construct the table head.
```
$table = new Table;
$table->construct_header('Statistic Name', ['width' => '70%']);
$table->construct_header('Amount', ['width' => '30%', 'class' => 'align_center']);
```

The _construct_header()_ function takes 2 parameters, the title of the head and an array. The array is optional but can be used to set _class_, _style_, _width_ and _colspan_ attributes of the resulting html.

Assuming we already have the data we need stored in an object or array, we can then proceed to build the cells of the next row. This is done using the construct_cell() function of the _Table_ class. Using our instance of the class, we can build cells.

```
$table->construct_cell('Members');
$table->construct_cell($statistic['members'], ['class' => 'align_center']);
```

The construct_cell() function takes two parameters, the first one is the data of the cell, and the second one is an optional array. The optional array can be used to set _class_, _style_, _id_, _colspan_, _rowspan_, and _width_ attributes of your cell.

After constructing the necessary cells for one row, the construct_row() function is then used to build the cells into a row and appends it to the table. This function also sets the cells of the table to an empty array once finished, such that the cells can be populated again for the next row.

```
$table->construct_row();
```

The construct_row() function takes an optional array, the array can be used to set _id_ and _class_ attributes of the row.

There is two functions that can be used to output the resulting table to the page, one is output() and the other is construct_html(). The output() function uses the construct_html() function and makes it faster and easier to output the table.

The output() function takes 4 optional parameters which are _heading_, _border_, _class_, and _return_. Using the output() function on a table without giving it any parameters will build a table without any heading, with borders set to 1, class will contain _general_ and return will be set to false. If return is false the resulting table will be directly echo'ed, if set to true it will return the html.

Basic usage
```
$table->output('Statistics');
```

If you have a need to set the id of the table, you will have to use the construct_html() function, which takes 4 optional parameters where the first three are similar to the output() function. The parameters are _heading_, _border_, _class_, and _table id_. The difference from output() is that _class_ is now set to null, and that construct_html() allows to set a custom id. The construct_html() function returns the resulting html.

Basic usage
```
echo $table->construct_html('Statistics', 1, 'general', 'statistics_table');
```

##### Table with data
Most tables are built iterably, as they often contain variable data. In this short sub-chapter there will be referred to the functions used in [Simple Table](creating-admincp-pages.md#simple-table), and it is therefore recommended to read the aforementioned sub-chapter first.

Whether one starts by constructing headers or fetching data from database, the steps to constructing a table should always be followed. Construct the header, the cells for each row, the row and then the table itself. The part of a table that is usually iterable is the table rows. We use the same example from [Simple Table](creating-admincp-pages.md#simple-table) where the table contains 2 columns.

```
$table = new Table;
$table->construct_header('Statistic Name', ['width' => '70%']);
$table->construct_header('Amount', ['width' => '30%', 'class' => 'align_center']);
```

We then fetch our data from the database.

```
$query = $db->simple_select('statistics', '*');
```

Then for each row from the database, a table row can be built simply by iterating the result from the query in a while loop.

```
while ($row = $db->fetch_array($query))
{
	$table->construct_cell($row['name']);
	$table->construct_cell($row['amount']);

	$table->construct_row();
}
```

In case there is no rows in result of the query, the function num_rows() can be used to make a check. This is done after the while loop to make sure that any rows that should have been there is not there.

```
if ($table->num_rows() == 0)
{
	$table->construct_cell('No results', ['colspan' => 2, 'class' => 'align_center']);
	$table->construct_row();
}
```

This will ensure that once the table is finally output there will at least be some visual response for the user.
##### Pagination
Pagination is often used when tables contain many rows. Using the function multipage() that exists inside _inc/functions.php_, pagination can easily be built when iterating tables.

The first step in building the pagination functionality is to find out how many rows there currently is in our data set. Assuming the data is in a database table, a query can be built to find the amount of rows.

```
$query = $db->simple_select('statistics', 'count(ID) as total');
$total = $db-fetch_field($query, 'total');
```

To decide how many pages to have, there must be a decision on have many rows per page. If this is for a plugin and to make readability easy for the board administrator, it could be an option inside your plugin settings. For most usecases a fixed number will do fine, which is what will be used here.

We assume that the _total_ variable currently holds the number 27, such  that the logic can be shown easilier throughout the example. We initialize a variable that will decide how many rows that can be per page. 

```
$perpage = 10;
```

To find out how many pages there are, divice the _total_ variable with the _perpage_ variable and round the result up. This can be done using the ceil() function in php.

```
$pages = ceil($total / $perpage);
```

In case _total_ is 27 and _perpage_ is 10, the result will be that _pages_ is 3. To decide which page is currently accessed a the get input _page_ is used. If the input is not set the page should default to 1.

```
$_page = isset($mybb->input['page']) ? $mybb->get_input('page', MyBB::INPUT_INT) : 1;
$_page = $_page < 1 ? 1 : $_page;
```

By using the ternary operator two checks are made. The last thing needed to build the pagination is the url. The url should not include the _page_ get input.

```
$url = $mybb->settings['bburl'].'/admin/index.php?module=forum-customadmin-page&action=statistics';
```

In the above example the get input _action_ is populated, this is done to simply illustrate that pagination needs to know the exact routing, as it is only able to take care of the parameters it is given.

With everything ready, the multipage() function can be used to build the html. It takes four parameters, the total amount of items, the amount per page, the current page, and the url. The result will have to be manually echo'ed, and can be echo'ed before and after the table, to suit any preference.

```
$pagination = multipage($total, $perpage, $_page, $url);
echo $pagination;
```

The last step is fetching the right entries from the database. To do this more easily a new variable is introduced, which will be the offset of the database. The offset is calculated by taking the current page and subtracting 1 from it, then multiplying it with the amount of items per page.

```
$offset = ($_page - 1) * $perpage;
```

A query can then be built using the _offset_ and _perpage_ variables to filter the entries needed. 

```
$query = $db->write_query('SELECT * FROM '.TABLE_PREFIX.'statistics ORDER BY id ASC LIMIT '.$offset.', '.$perpage);
```

The resulting entries can then be iterated using a standard while loop.

If there is only 1 page, or rather if the _total_ variable is equal to or less than the _perpage_ variable, the multipage() function will return null. This ensures pagination will only be shown when usable.
#### Outputting & Handling Forms

#### Page Functionality
##### Output Header

##### Output Footer

##### Add Item to Breadcrumb

##### Output Navigation Tabs

##### Codebuttons Editor in AdminCP


##### Popup
