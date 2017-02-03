# GAMMA

## What is this project?

GAMMA is a very simple and easy to understand lightweight MVC *(Model-View-Controller)* framework written in PHP. GAMMA is NOT a professional framework and it does not come with all the heavy stuff real frameworks have. That said, if you just need to show some pages, with a few database calls and some basic AJAX calls, without spending a lot of time learning a robust and complex PHP Framework, or reading massive documentations, then GAMMA might be very useful for you.

### Features

* Easy to install
* Simple and clean structure
* Native and easy to understand PHP code.
* Commented code
* Support for ajax calls
* Demo Application (Netflix Movies)


### Prerequisites

* Web Server (Apache, NGINX)
* Mysql Server (5.0+)
* PHP 5.0+
* mod_rewrite activated (see guidelines below)


### Installing

1. Create a MySQL Database, Username, Password and grant privileges.

2. Checkout a copy of the source files and head over to your app's config in /config/config.php and update the MySQL database credentials (created in previous step). 

```php
define('DB_HOST', 'localhost');
define('DB_NAME', 'db_name');
define('DB_USER', 'db_user');
define('DB_PASSWORD', 'db_password');
```

3. Head to http://localhost/netflix-movies *(assuming the source files are served on this URL)*.

## Quick Start

### General Structure

The application's URL-path translates requests directly to the controllers and their methods inside app/controllers.

`example.com/movies/exampleOne` will do what the *exampleOne()* method in app/controllers/moviescontroller.php says.

`example.com/movies/exampleOne/param1/param2` will do what the *exampleOne(param1, param2)* method in app/controllers/moviescontroller.php says, and will pass param1, param2 as parameters to it.

`example.com/movies` will do what the *index()* method in app/controllers/home.php says.

`example.com` will do what the *index()* method in app/controllers/movies.php says (default fallback).

`example.com/movies/view/17` will do what the *view(id)* method in app/controllers/movies.php says and
will pass `17` as a parameter to it. 

*Note: If there is no view() method in the Movies Controller, it will fallback to the file in app/core/controller.class.php view() method, also other methods like delete(), add(), viewall().*

### Views and Templates

Header and Footer are global to all the views (`app/views/header.php`,`app/views/footer.php`). All views of a model must be located inside a folder with the model name, and the filename must be same as method name (in our Netflix Movies demo, views are inside: `app/views/movies/` and the view for *methodexample()* is called methodexample.php). 

Let's look at the methodexample()-method in the Movies Controller (`app/controllers/moviescontroller.php`): This simply get today days name, set it (to use it into the view) and render the corresponding view (`app/views/movies/methodexample.php`):

`app/controllers/moviescontroller.php (methodexample())`
```php
function methodexample()
{
    $day = date("l");
    $this->set('day', $day);
}
```
`app/views/movies/methodexample.php`
```html
<h2>This is an simple View example</h2>
<b>Today is:</b> <?=$day?>
```

As you can see, passing variables to the view is also very simple, using the method *set(variable_name, value)* you can set as many variables as you need, and access them from the view as normal variables.


### Example with Data

Let's look into the list()-method in the Movies Controller (`app/controllers/moviescontroller.php`): Everything is extremely reduced and simple: $this->Movie->search($search) simply calls the search()-method in `app/model/movies.php` with searched term as a parameter, the returned result will be stored using *set()* to be used in the view:

```php
function list()
{
    $search = isset($_POST["searchterm"]) ? $_POST["searchterm"] : '';
    $this->set('active','list');
    $this->set('search',$search);
    if($search)
    {
        $this->set('title','Results for '.$search . ' - Netflix Movies');
        $this->set('data',$this->Movie->search($search));   
    }
    else
    {
        $this->set('title','All Results - Netflix Movies');
        $this->set('data',$this->Movie->selectAll());
    }
}
```

For extreme simplicity, all data-handling methods are in the Model (in this case `app/model/movies.php`). This is for sure not really professional, but the most simple implementation. Have a look how search()-method in movies.php looks like:

```php
function search($search)
{
    return $this->query('SELECT * FROM '.$this->_table.' WHERE  
        (show_title LIKE \'%'.mysqli_real_escape_string($this->_dbHandle,$search).'%\') OR
        (show_cast LIKE \'%'.mysqli_real_escape_string($this->_dbHandle,$search).'%\') OR
        (director LIKE \'%'.mysqli_real_escape_string($this->_dbHandle,$search).'%\') OR
        (summary LIKE \'%'.mysqli_real_escape_string($this->_dbHandle,$search).'%\')
        ');   
}
```

The result, here $data, can then easily be used directly inside the view files (in this case `app/views/movies/list.php`, in a simplified version of our Netflix Movies example):

```php
<table class="table table-striped">
  <thead>
    <tr>
      <th>Title</th>
      <th>Year</th>
      <th>Director</th>
    </tr>
  </thead>
  <tbody>
  <?php foreach ($data as $dataitem):?>    	
      <tr>
        <td>
            <?php echo $dataitem['Movie']['show_title']?>
        </td>
      <td><?php echo $dataitem['Movie']['release_year']?></td>
      <td><?php echo $dataitem['Movie']['director']?></td>
      <td>
      </tr>
    <?php endforeach?>
  </tbody>
</table>
```



## AJAX Calls

When a controller needs to be requested using Ajax, simply call `$this->ajax()` inside the controller, and it will return the view without global Header and Footer, for example, top5()-method in `app/controllers/moviescontroller.php`:

```php
function top5($order="rating", $search = "")
{
  $this->ajax();
  $this->set('data',$this->Movie->top5($order, $search)); 
}
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Others

### Server config for nginx

```nginx
server {
    server_name default_server _;   # Listen to any servername
    listen      [::]:80;
    listen      80;

    root /var/www/html/gamma-folder/;

    location / {
        index index.php;
        try_files /$uri /$uri/ /index.php?url=$uri;
    }

    location ~ \.(php)$ {
        fastcgi_pass   unix:/var/run/php5-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
*Note: gamma-folder is an example, and 
be changed according your installation folder*

### How to activate mod_rewrite?
* [Ubuntu 14.04 LTS](http://www.dev-metal.com/enable-mod_rewrite-ubuntu-14-04-lts/)
* [Ubuntu 12.04 LTS](http://www.dev-metal.com/enable-mod_rewrite-ubuntu-12-04-lts/)
* [EasyPHP on Windows](http://stackoverflow.com/questions/8158770/easyphp-and-htaccess)
* [AMPPS on Windows/Mac OS](http://www.softaculous.com/board/index.php?tid=3634&title=AMPPS_rewrite_enable/disable_option%3F_please%3F)
* [XAMPP for Windows](http://www.leonardaustin.com/blog/technical/enable-mod_rewrite-in-xampp/)
* [MAMP on Mac OS](http://stackoverflow.com/questions/7670561/how-to-get-htaccess-to-work-on-mamp)


## Authors

* [**Walter Gammarota**](https://github.com/WalterGammarota)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
