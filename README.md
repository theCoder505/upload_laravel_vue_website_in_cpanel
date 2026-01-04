See this video:
https://youtu.be/NeQYfp7h_pQ

Then in copy the shared app.blade.php file.


Then in routes/web.php file do this:
At starting add this:
use Illuminate\Support\Facades\Artisan;



then also add this:
// Clearing & Optimizing Routes, Views & Website
Route::get('/clear', function () {
    Artisan::call('cache:clear');
    Artisan::call('config:clear');
    Artisan::call('config:cache');
    Artisan::call('view:clear');
    Artisan::call('storage:link');
    return "Cleared!";
});





It might sshow this:
symlink(): No such file or directory

Then add public folder in the root directory and run again.
then hit /clear url again.






Now update the .htaccess file:
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews -Indexes
    </IfModule>
    RewriteEngine On
    
    # Redirect /storage/ to /public/storage/ for static files
    RewriteRule ^storage/(.*)$ public/storage/$1 [L]
    
    # Handle Authorization Header
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
    
    # Handle X-XSRF-Token Header
    RewriteCond %{HTTP:x-xsrf-token} .
    RewriteRule .* - [E=HTTP_X_XSRF_TOKEN:%{HTTP:X-XSRF-Token}]
    
    # Redirect Trailing Slashes If Not A Folder...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} (.+)/$
    RewriteRule ^ %1 [L,R=301]
    
    # Send Requests To Front Controller...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>


That's all.
