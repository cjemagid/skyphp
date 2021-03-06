# SkyPHP 3.0

A lightweight PHP5 framework for building scalable HTML5 websites quickly.

PHP 5.4 is required

## Features

- PHP-FIG Standards Compliant [<http://www.php-fig.org/>]
- `Sky\Page` organizes your webpages and URLs
    - Database folders - define your urls with database rows
    - Queryfolders - make your querystrings pretty
    - Templates
        - Auto-minify and bundle CSS/JS (SASS coming soon)
        - Ajax navigation
    - Inherit folder (virtual symlink)
- `Sky\Model` is an ORM that uses AQL
- `Sky\Api` is a framework for creating REST APIs
- `Sky\Db` supports Master/Slave DB environments (tested with PostgreSQL & MySQL)
- `Sky\Memcache` supports redundant Memcached servers
- Cascading codebases and hooks
- CMS add-on codebase available [<https://github.com/SkyPHP/cms>]

## Documentation

### `Sky\Page`

#### Sample page
`/pages/test/test.php`
```php
<?php

$this->title = 'Test Page';
//$this->js[] = '/pages/test/test.js'; // this is done automatically
//$this->css[] = '/pages/test/test.css'; // this is done automatically
$this->js[] = '/lib/js/some-other-library.js';
$this->head[] = '<meta property="example" content="you can add html to your head">';
$this->template('html5', 'top');
// this stuff appears in the body tag
?>
    <h1>This is a Test Page</h1>
<?
    // The Page object ($this) contains useful methods and properties.
    d($this); // dump the Page object

$this->template('html5', 'bottom');
```

### `Sky\Model`

#### Saving data objects to the database
```php
<?php

use \My\Models\artist;

$artist = new artist([                  // create the object
    'name' => 'Anthrax'
]);                                     // but don't save to the database yet

if ($artist->_errors) {                 // validation errors are generated in real-time
    d($artist->_errors);                // dump _errors array in a nice html format
}

$artist->name = 'Slayer';               // set the artist's name
$artist->save();                        // save the object to the database

$artist->set([                          // change multiple fields
    'name' => 'Slayer',                 // *name is not saved in this case because only
    'state' => 'CA',                    // modified fields are saved
    'albums' => [                       // you can easily save nested objects
        [
            'name' => 'Diabolus in Musica',
            'year' => 1998
        ]
    ]
])->save();

$artist->update([                       // shorthand for updating fields in the database
    'name' => 'Slanthrax'
]);

$artist = artist::insert([              // shorthand for inserting new objects to the db
    'name' => 'Anthrax',
    'state' => 'NY'
]);
echo $artist->id; // 5                  // get the newly created artist_id
```

#### Getting data objects from the database
```php
<?php

$aritst_id = 5;
$artist = new artist($artist_id);       // get artist from database by primary key
$artist = artist::get($artist_id);      // this is the same as above
echo $artist->name; // Anthrax

$artist = artist::getOne([              // get one artist using some criteria
    'where' => "name = 'Slayer'"        // ('where' can be a string or array)
])->update([                            // and change their city
    'city' => 'Huntington Park'
]);

$artists = artist::getMany([            // get 100 "the" bands from Brooklyn
    'where' => [
        "name ilike 'The %'",
        "city = 'Brooklyn'",
        "state = 'NY'"
    ],
    'limit' => 100
]);
foreach ($artists as $artist) {         // display each artist and number of albums
    $qty = count($artist->albums);
    echo "{$artist->name} have {$qty} albums.<br />";
}

$number_of_bands = artist::getCount([   // get a count of artists in Brooklyn
    'where' => "city = 'Brooklyn' and state = 'NY'"
]);

$artist_ids = artist::getList([         // get an array of every artist.id in NY
    'where' => "state = 'NY'"
]);
```

#### Sample Model
`/lib/My/Models/artist.php`
```php
<?php

namespace Crave\Models;

class artist extends \Sky\Model
{

    const AQL = "
        artist {
            name,
            bio,
            [artist_type],
            [artist_genre]s as genres
        }
        ct_promoter_artist {
            ct_promoter_id
        }
    ";

    /**
     *
     */
    public static $_meta = [

        'possibleErrors' => [
            'invalid_state' => [
                'message' => 'Please enter a valid two character state abbreviation.',
                'type' => 'invalid',
                'fields' => ['state']
            ]
        ],

        'requiredFields' => [
            'name' => 'Artist Name'
        ],

        'readOnlyProperties' => [
            'artist_type'
        ],

        'cachedLists' => [
            'artist_type_id'
        ]
    ];

    /**
     * Validates 'state' field only if $this->state is not null
     */
    public function validate_state()
    {
        if (strlen($this->state) != 2) {
            $this->addError('invalid_state');
        }
    }

    /**
     *
     */
    public function validate()
    {

    }

}
```
