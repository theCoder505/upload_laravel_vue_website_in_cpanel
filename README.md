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



That's all.
