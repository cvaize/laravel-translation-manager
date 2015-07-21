# Laravel Translation Manager

Note that this package is originally based on Barry vd. Heuvel's <barryvdh@gmail.com> excellent **barryvdh/laravel-translation-manager** package but heavily reworked to add [New Features](#NewFeatures).

This is a package to manage Laravel translation files. It does not replace the Translation system but augments it with:

- import/export the php files to a database
- make translations editable through a web interface.
- allow in-database translations to override the ones in the language files. Used to update translations on server clusters where updating translation files is not possible (like AWS EC2) or would cause server code to be out of sync.

The workflow would be:

- Import translations: Read all translation files and save them in the database
- Find all translations in php/twig sources
- Optionally: Listen to missing translation with the custom Translator
- Translate all keys through the webinterface
- Export: Write all translations back to the translation files or cache if in production environment where writing of files is not an option.

This way, translations can be saved in git history and no overhead is introduced in production.

![Translator Page ](https://raw.githubusercontent.com/vsch/laravel-translation-manager/master/images/ScreenShot_main.png)

## Current Limitations

This package was developed as part of a rush project on which translation management was desperately needed but not budgeted into the schedule. Having started with the `barryvdh/laravel-translation-manager` and needing extra features to make my life easier I decided to add a few of the features and hacked them into place to get a working version fast but have not yet refactored these for human consumption or ease of maintenance. I am in the process of doing this as time permits. I will be addressing these limitations in the near future or sooner if someone contacts me with a request to prioritize the clean-up of a specific area. See [To Do List](#ToDo)

## Installation

1. Require this package in your composer.json and run composer update (or run `composer require vsch/laravel-translation-manager:*` directly):

    "vsch/laravel-translation-manager": "0.1.x"

2. After updating composer, add the ServiceProviders to the providers array in app/config/app.php

```php
    //'Illuminate\Translation\TranslationServiceProvider',
    'Vsch\TranslationManager\TranslationServiceProvider',
    'Vsch\TranslationManager\ManagerServiceProvider',
```

The TranslationServiceProvider is an extension to the standard functionality and is required in order for the web interface to work properly. It is backward compatible with the existing Translator since it is a subclass of it and only overrides implementation for new features.

3. You need to run the migrations for this package

    $ php artisan migrate --package="vsch/laravel-translation-manager"

4. You need to publish the config file for this package. This will add the files `app/config/packages/vsch/laravel-translation-manager/config.php` and `app/config/packages/vsch/laravel-translation-manager/local/config.php`, where you can configure this package.

    $ php artisan config:publish vsch/laravel-translation-manager

5. You need to publish the web assets used by the translation manager web interface. This will add the assets to `public/packages/vsch/laravel-translation-manager`

    $ php artisan asset:publish vsch/laravel-translation-manager

6. You have to add the Controller to your routes.php, so you can set your own url/filters.

```php
    Route::group(array('before' => 'auth'), function ()
    {
        Route::controller('translations', 'Vsch\TranslationManager\Controller');
    });
```

This example will make the translation manager available at `http://yourdomain.com/translations`

## Web interface

When you have imported your translation (via buttons or command), you can view them in the web interface (on the url you defined the with the controller).
You can click on a translation and an edit field will popup. All translations are saved when the edit dialog is closed unless it is closed with the cancel button. Clicking anywhere on the page outside the edit dialog will save the current changes.
When a translation is not yet created in a different locale, you can also just edit it to create it.

Using the buttons on the web interface, you can import/export the translations. For publishing translations, make sure your application can write to the language directory or optionally configure it to do in-database publishing using the cache by adding:

## Artisan Commands

You can also use the commands below.

### Import command

The import command will search through app/lang and load all strings in the database, so you can easily manage them.

    $ php artisan translations:import
    
Note: By default, only new strings are added. Translations already in the DB are kept the same. If you want to replace all values with the ones from the files, 
add the `--replace` (or `-R`) option: `php artisan translations:import --replace`

### Find translations in source

The Find command/button will look search for all php/twig files in the app directory, to see if they contain translation functions, and will try to extract the group/item names.
The found keys will be added to the database, so they can be easily translated.
This can be done through the web interface, or via an Artisan command.

    $ php artisan translations:find

### Export command

The export command will write the contents of the database back to app/lang php files.
This will overwrite existing translations and remove all comments, so make sure to backup your data before using.
Supply the group name to define which groups you want to publish.

    $ php artisan translations:export <group>

For example, `php artisan translations:export reminders` when you have 2 locales (en/nl), will write to `app/lang/en/reminders.php` and `app/lang/nl/reminders.php`

### Clean command

The clean command will search for all translation that are NULL and delete them, so your interface is a bit cleaner. Note: empty translations are never exported.

    $ php artisan translations:clean

### Reset command

The reset command simply clears all translation in the database, so you can start fresh (by a new import). Make sure to export your work if needed before doing this.

    $ php artisan translations:reset

<a id="NewFeatures"></a>
## New Features

These features were added to the original barryvdh/laravel-translation-manager package.

- translation manager web-interface is localized. Current version has English and Russian. Others can be easily added by adding package translation overrides in `app/lang/packages/{locale}/laravel-translation-manager/messages.php` files. If message.php files are added for en and ru locales then their contents will override the language files included in the package.
- allows in-database translations to override the translations in files.
- publishing translations on production systems where updating translation files would only update a single server, can be configured to use the cache for serving up the modified translations. This allows translations to be updated live without having to redeploy a new version of the language files.
- Translation service can be put into 'in place edit' mode that enables editing of translations where they appear on the page.
  - this eliminates the need to peruse code to find the translation group/key combination that is used for the resulting string and then looking for it in the translation files or in the web interface. Simply enable in-place editign mode and click on the string you wish to edit.
  - This may require some editing of view files to handle: string values that should not be links because they are not shown or are used for HTML attribute values, `<button>` contents that don't display links and other edge cases.
- changes to database translations that have not been published show a difference between previously published/imported translations and current unpublished changes.
- soft delete of translations that are not physically deleted from the database until translations are published.
- translation page has a dash board view showing all unpublished changes, missing translations and deleted translations.
- handling of translation files in nested directories under `lang/` and package translations under `lang/packages` to allow managing package translation overrides.
- missing translation key logging that can be used in a production environment by setting 1 of N sessions to actually log missing translations. Since checking for missing translation requires hitting the database for every translation, it can be a heavy load on the DB server. This configuration setting allows randomly selecting 1 of N user sessions to be marked as checking for missing translations allowing the benefit of finding missing translations while reducing the load burden on the server.
- in place edit mode inside the search dialog to allow editing of translation in the search result.
- extra buttons added to the bootstrap x-edit component for frequent operations:
  - change case of translation or selection within translation: lowercase, first cap
  - create plural forms for use in `choice()`, currently English is automatically created and Russian will do its best by using Yandex translator to derive the plural forms.
  - recall translation text from last saved or last published.
  - simulated copy/paste buttons. Work only if the page is not reloaded and only within the translation edit dialog.
- exported language files are formatted to align `=>` for a given level, making it easier to deal with these files manually if needed.

## TODO

This package is still very much in development although it is successfully being used to manage translations. Here is a list of to do's and limitations:

- MySQL DB is assumed for queries in those places where Eloquent was too cumbersome or too inefficient. I will be refactoring all DB access that bypasses Eloquent into a TranslationRepository interface class so that new DB access will only need to create a new repository implementation class for a specific DB interface.
- Yandex assisted translations are only implemented for Russian, although Yandex does support other languages. This will be addressed to use all other available options.
- Google translate was not used since it has no free option. However it is a simple change in the translator.js file to handle alternate translation engines.
- Mismatched translations dashboard view assumes that English version of the translations is always correct and it is other languages that should have the same translations for different group/key combinations whose English text matches. For example if English messages.test1 = 'Test' and messages.test2 = 'Test' then for other languages the translations for these two keys will be flagged as a mismatch if they are not the same. This was a idiosyncrasy of the project for which this module was developed and if you find it useful then edit the index.blade.php file and change `@if(false && !empty($mismatches))` to `@if(true && !empty($mismatches))` on the line that precedes the mismatched dashboard `<div>`.
- key operations that allow creating new keys and also keys permuted by suffixes, moving, copying, deleting keys are a bit of a kludge. I am planning to rework the web interface to make these cleaner. However, if you desperately need these to save a lot of typing and editing, the current version will do the trick.
- Create a Laravel 5 compatible branch.
- ability to download a zip file that contains all translation files that would be generated by publish operation. This would be useful to get an updated copy of translations from a production server if an SQL connection from local dev environment is not available.

- Suggestions and priority requests are welcome. :)
