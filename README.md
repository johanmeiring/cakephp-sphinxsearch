# Sphinx plugin for CakePHP

## Installation

You can install this plugin into your CakePHP application using [composer](http://getcomposer.org).

The recommended way to install composer packages is:

```
composer require voycey/sphinxsearch-cakephp3
```

##Basic Documentation

I designed this as a replacement for the binary API access for Sphinxsearch that I was using on 2.x (https://github.com/voycey/sphinxsearch-cakephp2)

It currently has one function and that is to query the provided index and return the matching records in a CakePHP friendly format (In this case as Query objects and Entities).

##How to use

1 Install the package with composer as above
2 Add ````Plugin::load('Sphinx');```` to your bootstrap.php
3 Attach the behaviour to a table you wish to search on (that has an index for it!)

```php
<?php 
class PostsTable extends Table
{

    /**
     * Initialize method
     *
     * @param  array $config The configuration for the Table.
     * @return void
     */
    public function initialize(array $config)
    {
        parent::initialize($config);

        $this->table('posts');
        $this->displayField('title');
        $this->primaryKey('id');

        $this->addBehavior('Timestamp');
        $this->addBehavior('Tools.Slugged', [
                                    'label' => 'title', 
                                    'unique' => true, 
                                    'mode' => 'ascii', 
                                    'case' => 'low']);
        $this->addBehavior('Sphinx.Sphinx');
    }
}
?>
```

4 Perform a search through the behaviour directly (This will return a query object), it takes an array of the following parameters:

  1 ````index```` - this is the index you want to search against
  
  2 ````term````` - this is the term you want to search for
  
  3 ````match_fields```` - these are the fields you want to search against (default is search whole index)
  
  4 ````pagination```` - this is a standard Cake 3 pagination array - allows you to define how your data comes back, what fields it contains and what Models are contained.
  

Here is an example unit test that works for me.
```php

public function testBehaviour()
{
    $paginate = [
        'order' => [
            'Posts.id asc'
        ],
        'fields' => [
            'id', 'title', 'user_id'
        ],
        'contain' => [
            'Comments' => [
                'fields' => ['id', 'post_id']
            ],
            'Categories' => [
                'fields' => ['id', 'CategoriesPosts.post_id']
            ],
            'Types' => [
                'fields' => ['id', 'name']
            ]
        ]
    ];

    $query = $this->Posts->search([
                                'index' => 'idx_toolkit', 
                                'term' => 'Ten', 
                                'match_fields' => 'title', 
                                'paginate' => $paginate]);
    
    $row = $query->first();

    $this->assertInstanceOf('Cake\ORM\Query', $query);
    $this->assertInstanceOf('Cake\ORM\Entity', $row);

}
```
###TODO
* Allow for custom configuration to be passed in
* Hook into afterSave and have the Sphinx index updated (this isn't a priority for me as my indexes don't need to be live but please submit a pull request if you want to add this)
* Work out how to test this easily on Travis (again - help appreciated)
