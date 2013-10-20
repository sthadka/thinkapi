Title: Making Codeigniter clean URLs cleaner
Slug: making-codeigniter-clean-urls-cleaner
Date: 2008-10-07 19:19
Category: Codeigniter
Tags: codeigniter
Author: Sukumar Yethadka
Summary: A simple guide to customize your codeignter URLs.

Usually when working with codeigniter you come across the use case where you
need to remove the extra "index" appearing in the URL for purposes
ranging from shorter URLs to SEO.

For example,

    :::text
    Current URL style:
    http://example.com/controller/method/category1/category2/page-name
    Target URL style:
    http://example.com/category1/category2/page-name.html


As you can see, the difference in the target URL is the absence of the
controller name and the method name and the presence of the extension ".html".
Now, lets see how to achieve this and why the extra extension is required.

Below is my version of .htaccess.

    :::text
    RewriteEngine on

    RewriteRule ^([a-z0-9_-]+)\.html$ index.php/page/$1 [L]

    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond $1 !^(index\.php|asset|robots\.txt)
    RewriteRule ^(.*)$ index.php/$1 [L]

The line that is extra is 3 which redirects all the requests for ".html"; files to the controller named "page".

Now, lets look at the page controller which will handle these requests.

    :::php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Page extends Controller
    {

        function __construct()
        {
            parent::Controller();
        }

        function _remap()
        {
            $segment_count = $this->uri->total_segments();
            $segments = $this->uri->segment_array();

            if($segment_count == 0)
            {
                $this->data['title'] = 'Page Title';
                $this->load->view('home', $this->data);
            }
            elseif($segment_count == 2) // Single page
            {
                $page_url = $segments[2];
                // Handle the page request
            }
            elseif($segment_count == 3) // Page with category
            {


                $category = $segments[2];
                $page_url = $segments[3];
                // Handle the category + page request
            }
        }
    }

In this way you can have pages with smaller and cleaner URLs.

Tips:

1. You can use any extension you like. e.g; in case you have static HTML files to server, you can use the extension ".html" for static and ".htm" for non-static files.
2. You can have any number of URL stubs before the ".html" extension and you can access all these stubs in the controller.

Cons:

1. The URL needs an extra extension.
2. Only one controller per extension.

Do let me know what you think about this or if there is any way to improve on this.

## Old comments

**Ruben Müller**

Why don't you use the routes.php?

<a href="http://codeigniter.com/user_guide/general/routing.html" rel="nofollow">http://codeigniter.com/user_guide/general/routing.html</a>

---

**Bob**

I'm wondering why you would want the extension (.html, .htm, .php, whatever). If you're looking for shorter, cleaner URLs, why not:

Current: <a href="http://example.com/controller/method/category1/category2/page-name" rel="nofollow">http://example.com/controller/method/category1/category2/page-name</a>

Target : <a href="http://example.com/controller/method/category1/category2" rel="nofollow">http://example.com/controller/method/category1/category2</a>

or to really chop it up:

Target : <a href="http://example.com/category1/category2" rel="nofollow">http://example.com/category1/category2</a>

I suppose it makes a difference if your controller/method names are married to your site structure (i.e., <a href="http://example.com/products/view/cat1/cat2" rel="nofollow">http://example.com/products/view/cat1/cat2</a>) or not (<a href="http://example.com/whichpage/displayproducts/cat1/cat2" rel="nofollow">http://example.com/whichpage/displayproducts/cat1/cat2</a>) but at least to the way I code, I'm not seeing the benefits of this method.

Cheers,
Bob

---

**Sukumar Yethadka**

@Ruben
Using routes is an option. But in order to get URLs like

<a href="http://example.com/category1/category2/page-name" rel="nofollow">http://example.com/category1/category2/page-name</a>

without the controller name and the method name ("index" for example); the route would be (Please note that I haven't tested this below route)

    :::php
    $route['(:any)'] = 'page/$1';

Then you would be able to use any other controller since all the requests are routed to "page" controller.
Do let me know if there is a better routing method I missed.

@Bob
The main reasons I didn't want

Target : <a href="http://example.com/controller/method/category1/category2" rel="nofollow">http://example.com/controller/method/category1/category2</a>

is because my URLs would have looked like

<a href="http://example.com/page/index/category1/category2" rel="nofollow">http://example.com/page/index/category1/category2</a>

AFAIK, google considers the URL segments to be more important the closer they are to the domain name.

Yes,
Target : <a href="http://example.com/category1/category2" rel="nofollow">http://example.com/category1/category2</a>

is quite possible and using routes at that. However, I think we wouldn' t be able to use other controllers because of this (Unless we have some filter that checks for the existence of the controller and then the category). Using extensions helps to map a particular extension to a specific controller whithout affecting any other controllers.

One of the main use cases I can think of is for a small blog similar to wordpress.

Hope that makes sense.

---

**Henry Weismann**

How about:

Current Url: <a href="http://example.com/controller-method/category1/category2" rel="nofollow">http://example.com/controller-method/category1/category2</a>

OR

Current Url: <a href="http://example.com/subfolder-controller-method/category1/category2" rel="nofollow">http://example.com/subfolder-controller-method/category1/category2</a>

is because my URLs would have looked like

Target Url: <a href="http://example.com/index.php/controller/method/category1/category2" rel="nofollow">http://example.com/index.php/controller/method/category1/category2</a>

OR

Target Url: <a href="http://example.com/index.php/controller/method/index/category1/category2" rel="nofollow">http://example.com/index.php/controller/method/index/category1/category2</a>

    :::text
    RewriteEngine on
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond $1 !^(index\.php|asset|robots\.txt)
    RewriteRule ^([^/.]+)-([^/.]+)-([^/.]+)/(.*)$ index.php/$1/$2/$3/$4 [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond $1 !^(index\.php|asset|robots\.txt)
    RewriteRule ^([^/.]+)-([^/.]+)/(.*)$ index.php/$1/$2/$3 [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond $1 !^(index\.php|asset|robots\.txt)
    RewriteRule ^(.*)$ index.php/$1 [L]

Using Routes:

    :::php
    $route['^([^/.]+)-([^/.]+)/(.*)'] = "$1/$2/$3";
    $route['^([^/.]+)-([^/.]+)-([^/.]+)/(.*)'] = "$1/$2/$3/$4";

I have no idea if that will work as I just wrote it now.  Feel free to test it.

---

**Henry Weismann**

Should read:
Target Url: <a href="http://example.com/index.php/controller/method/category1/category2" rel="nofollow">http://example.com/index.php/controller/method/category1/category2</a>

OR

Target Url: <a href="http://example.com/index.php/subfolder/controller/method/index/category1/category2" rel="nofollow">http://example.com/index.php/subfolder/controller/method/index/category1/category2</a>

---

**Sukumar Yethadka**

@Henry,
Your code looks good, but my aim is to remove the controller and the method name from the URL.
Target URL style: <a href="http://example.com/category1/category2/page-name" rel="nofollow">http://example.com/category1/category2/page-name</a>
Your method is useful to merge the forward slashes in the URL.

Useful code nevertheless, thanks!

---

**Sean**

One problem I notice about CI Urls is how they look inside of google on your website snippet. They all include a ../ inside the URL have you noticed this and do you have any idea how to remove that ../ it wastes space and is distracting.
Thanks,
Sean

---

**Sukumar Yethadka**

@Sean,
I didn't really get what you are referring to. You can check the usage of this code snippet on <a href="http://www.fourseasonsvineyards.com/" rel="nofollow">http://www.fourseasonsvineyards.com/</a>
As you can see there are no "../" in the URLs anywhere. Do let me know if I am missing something.
