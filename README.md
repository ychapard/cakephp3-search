# CakePHP Search

Search provides a search module for CakePHP applications.
Added :
* A field ( search_data default) is automatically filled with the values ​​of the fields you want to index
* Search word is separated into words (space char) so search is base on multiples words

## Requirements

The master branch has the following requirements:

* CakePHP 3.0.0 or greater.
* PHP 5.4.16 or greater.
* SQLite or another database driver that CakePHP can talk to. By default
  DebugKit will use SQLite, if you need to use a different database see the
  Database Configuration section below.

## Installation

* Install the plugin with composer from your CakePHP Project's ROOT directory
(where composer.json file is located)
```sh
php composer.phar require ychapard/cakephp3-search "dev-master"
```

* Load the plugin by adding following to your `config/bootstrap.php`
```php
Plugin::load('Search');
```

or running command
```sh
./bin/cake plugin load Search
```

## Usage

* Add a search config method in your table
```php
use Search\Manager;

...

public function searchConfiguration()
{
    $search = new Manager($this);
    $search
    ->value('author_id', [
        'field' => $this->aliasField('author_id')
    ])
    ->like('q', [
        'before' => true,
        'after' => true,
        'field' => [$this->aliasField('title'), $this->aliasField('title')]
    ]);
    return $search;
}
```

* Add The Behavior in your initialize method
```php
public function initialize(array $config)
{
    ...
    $this->addBehavior('Search.Search',[
        'field' => ['name','title','short_text'],
        'dest_field' => 'search_data',
        'replacement' => ' '
    ]);
    ...
}
```

* Example of index controller for a model `Article`
```php
public function index()
{
    $query = $this->Countries
        ->find('search', $this->Articles->filterParams($this->request->query))
        ->where(['title !=' => null])
        ->order(['Article.id' => 'asc'])
        ->contain([
            'Comments'
        ]);
    $this->set('articles', $this->paginate($query));
}
```

The `search` finder and the `filterParams()` method are dynamically provided by the
`Search` behavior.

* Then add the component search in the necessary methods (for our example index)
```php
public function index()
{
    $this->loadComponent('Search.Prg');
}
```

* Instead, you can add this in your AppController to enable component in all index methods
```php
public function initialize()
{
    parent::initialize();
    if ($this->request->action === 'index') {
        $this->loadComponent('Search.Prg');
    }
}
```

## Filtering your data
Once you have completed all the setup you can now filter your data by passing
query params in your index method. Using the Article example given above, you
could filter your articles using the following.

`example.com/articles?q=cakephp`

Would filter your list of articles to any article with "cakephp" in the `title`
or `content` field. You might choose to make a `get` form which posts the filter
directly to the url, or create links manually.
