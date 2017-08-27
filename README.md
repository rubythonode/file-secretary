# Laravel File Secretary
Get rid of anything related to files in Laravel, This package handles all for you. Anything we mean.

## What does this package do?
 1. **Handles your public assets** (.css, .js, .etc) to be served through your CDN provider.
 Unlike other solutions 
 there is no runtime i/o needed for retrieving the unique id needed for cache purging on deploys.
 2. **Handles all the image resizing needs** with simple configuration, Images are generated on the fly
 for once, and are stored in your CDN provider, They could be served without the participation of PHP
 all handled with a simple *nginx snippet*. 
 3. **Detects redundant files**, File names are generated based on the filesize + a hash function.
 so redundant files could not exist technically, You can implement your own file name generator, too.
 4. **Handles basic files** with a simple method call. They can be served without the participation of PHP.
 5. **Allows Database Tracking** (Optional), you can use the eloquent model to relate files to your other models, easily.
 You can also implement your own eloquent model for more flexibility.
 6. **Simple functions** for dealing with resizable image urls, file urls, asset urls etc.
 7. **A Simple controller for serving private/public files** can be used to serve both resizable images, and basic files.
 You can implement your own access control for serving based on config.
 
## Installation
```
composer require reshadman/file-secretary 1.*
```

## Configuration
To publish configuration:
```
php artisqan vendor:publish --provider=Reshadman\FileSecretary\Infrastructure\FileSecretaryServiceProvider
```

> By default the migration for database tracking is also published, you delete it if you don't want the functionality.

### Configuration data

#### File name generator function:
```php
<?php 
return [
    // Other config elements...
    'file_name_generator' => function (\Reshadman\FileSecretary\Application\PresentedFile $presentedFile) {
        // This prevents multiple files with the same contents.
        // And it is too rare, to have two different files with the same hash and size.
        // You could also add an additional hash, but it will increase the filename size
        // Which may lead to some problems in Windows systems.
        $size = $presentedFile->getFileInstance()->getSize();
        $hash = sha1_file($presentedFile->getFileInstance()->getPath());
        return  $size . '-' . $hash;
    },
];
``` 

#### Database tracking:
Database tracking is only possible with eloquent models, In the next releases
they will be eloquent independent.

You can use the default model, with your custom table name, or you can create your own model
and address it in the config file.

Your model should implement the following interface:
```php
<?php

\Reshadman\FileSecretary\Application\PersistableFile::class;

```

or you can simply extend the package's default eloquent model.

```php
<?php 
return [
    // Other config elements...
    'eloquent' => [
        'model' => \Reshadman\FileSecretary\Application\EloquentPersistedFile::class,

        'table' => 'system__files'
    ],
];
``` 
### Image templates:
The package offers a very simple way for manipulating images, we call them **Templates**. It is based on [Intervention Package](http://image.intervention.io/). 
With a COC wrapper. You name your templates in the config file, map them to a template class, and they will be manipulated, 
stored and served on the cloud as simple as that, you just need to follow the configuration spec.
By default a default template is included in the package which allows:
 - Resizing: (defined width, defined height), (defined width, auto height and vice versa you can also define the fit strategy)
 - Encodings: Images can be encoded with the exact encoding of their parent file, or you can specify your needed encodings.
 - Quality: You can specify the quality, too
 - Stripping: Sometimes the ICC profiles of the images are embedded in the file. Removing
 them can reduce the file size dramatically. You can also use this option.
```php
<?php 
return [
    // Other config elements...
    'available_image_templates' => [
        'companies_logo_200x200' => [
            'class' => \Reshadman\FileSecretary\Infrastructure\Images\Templates\DynamicResizableTemplate::class,
            'args' => [
                'width' => 200,
                'height' => 200,
                'encodings' => null, // When null only parent file encoding is allowed.
                'strip' =>  false, // removes the ICC profile when imagick is used.
            ],
        ],
        'companies_logo_201xauto' => [
            'class' => \Reshadman\FileSecretary\Infrastructure\Images\Templates\DynamicResizableTemplate::class,
            'args' => [
                'width' => 201,
                'height' => null, // Height will be calculated automatically
                'mode' => \Reshadman\FileSecretary\Infrastructure\Images\TemplateManager::MODE_FIT, // The image will fit
                'encodings' => [
                    'png' // Ony png extension is served otherwise it throws 404 exception
                ]
            ],
        ],
    ],
];
```  


## Running the Integration Tests
 There are integration tests written for this package. To run integration
tests do as the following:

 1. Create your `phpunit.xml` file based on the packages's `phpunit.dist.xml`: `cp phpunit.dist.xml phpunit.xml`
 
 2. Fill the phpunit config with your environment variables.
 The package has been tested with **Rackspace** Object storage, to prove the 
 functionality in cloud. You can change the `phpunit.xml` file and the configs in `fixtures/config/`
 to integrate them with your testing environment.
 3. Run the tests with `vendor/bin/phpunit --debug`
 
> Currently there is no isolated object unit testing for this package. 
> They will be added in next releases.

## Package Roadmap
 1. Writing more integration tests + isolated object unit tests.
 2. Use more semantic names for features, class names and methods names.
 3. Make the tracking, eloquent independent.
 4. Refactor the code both for design and performance.

## About the package
This package has been extracted from [*jobinja.ir - The leading job board and career platform in Iran*](https://jobinja.ir),
This is part of the work for making [jobinja.ir](https://jobinja.ir), [12factor.net](http://12factor.net) compatible.

## License

The MIT License (MIT). Please see License File for more information.
