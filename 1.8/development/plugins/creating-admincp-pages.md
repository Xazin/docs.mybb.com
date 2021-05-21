---
layout: page
title:  "Creating AdminCP Pages"
categories: [plugins]
---

## Create a new AdminCP Page

### Why use AdminCP Pages
When you want to offer administrative options beyond [Plugin Settings](basics.md/#plugin-settings) for admins of a board, similar to plugins such as 'BAM Announcement Manager' or 'My Advertisements', it can often be a good idea to consider adding custom AdminCP pages.

### Step 1. Deciding on a Module
Before adding a custom AdminCP page, look at the type of options you want to offer and where they belong in the AdminCP. In example if your plugin offers custom announcements, there is a chance the page fits better under 'Forums & Posts' more so than 'Tools & Maintenance'.

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
    $sub_menu[] = array('id' => 'customadmin-page', 'title' => 'Custom Admin Page', 'link' => 'index.php?module=forum-customadmin-page');
}

function callback_function_action_handler(&$actions) {
    $actions['customadmin-page'] = array('active' => 'customadmin-page', 'file' => 'customadmin-page.php');
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

## Page Content
The following will not discuss how to output content in AdminCP other than using classes included in **MyBB** to build AdminCP pages.

### Outputting Tables

#### Simple Table
First, we create a new instance (object) of the _Table_ class, and we construct the table head.

```
$table = new Table;
$table->construct_header('Statistic Name', array('width' => '70%'));
$table->construct_header('Amount', array('width' => '30%', 'class' => 'align_center'));
```

The _construct_header()_ function takes two parameters, the title of the head and an array. The array is optional but can be used to set _class_, _style_, _width_ and _colspan_ attributes of the resulting HTML.

Assuming we already have the data we need in an object or array, we can build the following row's cells using the construct_cell() function of the _Table_ class.

```
$table->construct_cell('Members');
$table->construct_cell($statistic['members'], array('class' => 'align_center'));
```

The construct_cell() function takes two parameters, the first one is the cell's data, and the second one is an optional array. The optional array can set _class_, _style_, _id_, _colspan_, _rowspan_, and _width_ attributes of the table cell.

After constructing the necessary cells for one row, the construct_row() function is then used to build the cells into a row and appends it to the table. This function also sets the table cells to an empty array once finished, such that the cells can be populated again for the next row. The function construct_row() takes an optional array as a parameter that can set _id_ and _class_ attributes of the row.

```
$table->construct_row();
```

Outputting the resulting table can be done in two different ways. The standard method uses the output() function, and the other more irregular method uses the output_html() function.

The output() function takes 4 optional parameters which are _heading_, _border_, _class_, and _return_. The parameters all contain default values, where _heading_ = '', _border_ = 1, _class_ = 'general', and _return_ = false.

Basic usage
```
$table->output('Statistics');
```

If there is a need to set the table's ID attribute, the construct_html() function takes a parameter different from the output() function. The parameters of construct_html() are _heading_, _border_, _class_, and _table id_. The difference between the two functions, output() and construct_html(), is that for construct_html() the _class_ defaults to NULL, and the last parameter is now _id_. A major difference as well is that construct_html() only returns the resulting HTML and does not echo/print it.

Basic usage
```
echo $table->construct_html('Statistics', 1, 'general', 'statistics_table');
```

#### Table with data
There will be references to the functions used in this short sub-chapter [Simple Table](creating-admincp-pages#simple-table). 

We use the same example from [Simple Table](creating-admincp-pages#simple-table), where the table contains two columns.

```
$table = new Table;
$table->construct_header('Statistic Name', array('width' => '70%'));
$table->construct_header('Amount', array('width' => '30%', 'class' => 'align_center'));
```

We then fetch our data from the database.

```
$query = $db->simple_select('statistics', '*');
```

Then for each row from the database, a table row can be built simply by iterating the query result in a while loop.

```
while ($row = $db->fetch_array($query))
{
	$table->construct_cell($row['name']);
	$table->construct_cell($row['amount']);

	$table->construct_row();
}
```

In case there are no rows in the result of the query, the function num_rows() can be used to make a check. After the while loop, the check ensures that any rows that should have been there are not there.

```
if ($table->num_rows() == 0)
{
	$table->construct_cell('No results', array('colspan' => 2, 'class' => 'align_center'));
	$table->construct_row();
}
```

Doing the check from the above example will ensure that the user always has some form of visual response.

#### Pagination
When tables contain many rows, pagination helps with readability. Using the function multipage() that exists inside _inc/functions.php_, pagination can easily be built when iterating tables.

The first step in building the pagination functionality is to find out how many rows there currently are in the relevant dataset. Assuming the data is in a database table, the number of rows is retrieved using a query.

```
$query = $db->simple_select('statistics', 'count(ID) as total');
$total = $db-fetch_field($query, 'total');
```

There must be a decision on how many rows per page. If this is for a plugin and to make readability easy for the board administrator, it could be an option inside the plugin's settings. A fixed number will do fine for most use cases, which is what the example will use.

We assume that the _total_ variable currently holds the number 27, such that the logic can be shown easily throughout the example. We initialize a variable that will decide how many rows that can be per page. 

```
$perpage = 10;
```

To find out how many pages there are, divide the _total_ variable with the _perpage_ variable and round the result. Using the ceil() function in PHP will round the number up.

```
$pages = ceil($total / $perpage);
```

In case _total_ is 27, and _perpage_ is 10, the result will be that _pages_ is 3. The GET input _page_ decides which page is currently accessed. If the input is empty or not set, the page should default to 1.

```
$_page = isset($mybb->input['page']) ? $mybb->get_input('page', MyBB::INPUT_INT) : 1;
$_page = $_page < 1 ? 1 : $_page;
```

In the above example, the ternary operator's usage is to make sure that the _\_page_ variable is not less than 0.

```
$url = $mybb->settings['bburl'].'/admin/index.php?module=forum-customadmin-page&action=statistics';
```

In the above example, the GET input _action_ is part of the _url_, which illustrates that the multipage() function needs to know the exact routing.

With everything ready, the multipage() function can build the HTML. It takes four parameters, the total amount of items, the amount per page, the current page, and the URL. The resulting HTML from the multipage() function echoes before or after the table.

```
$pagination = multipage($total, $perpage, $_page, $url);
echo $pagination;
```

The last step is fetching the correct entries from the database. A new variable is introduced, which will be the offset of the rows in the database. The offset is calculated by taking the current page and subtracting one from it, then multiplying it with the number of items per page.

```
$offset = ($_page - 1) * $perpage;
```

You can use a query with the LIMIT clause to specify the number of rows and an offset.

```
$query = $db->write_query('SELECT * FROM '.TABLE_PREFIX.'statistics ORDER BY id ASC LIMIT '.$offset.', '.$perpage); // Short syntax
$query = $db->write_query('SELECT * FROM '.TABLE_PREFIX.'statistics ORDER BY id ASC LIMIT '.$offset.' OFFSET '.$perpage); // Full syntax
```

You can iterate the resulting entries using a standard while loop.

Suppose there is only 1 page, or rather if the _total_ variable is equal to or less than the _perpage_ variable, the multipage() function will return null, which ensures pagination is in use only when relevant.

### Outputting & Handling Forms

### Page Functionality
#### Output Header

#### Output Footer

#### Add Item to Breadcrumb

#### Output Navigation Tabs

#### Codebuttons Editor in AdminCP

#### Popup
