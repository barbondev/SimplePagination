Simple Pagination
=================

[![Build Status](https://travis-ci.org/AshleyDawson/SimplePagination.svg?branch=develop)](https://travis-ci.org/AshleyDawson/SimplePagination)

Simple pagination library implements a paging interface on collections of things.

Installation
------------

You can install Simple Pagination via [Composer](https://getcomposer.org/). To do that, simply `require` the 
package in your `composer.json` file like so:

```json
{
    "require": {
        "ashleydawson/simplepagination": "1.0.*"
    }
}
```

Then run `composer update` to install the package.

How Simple Pagination Works
---------------------------

I've tried to make Simple Pagination as easy to use and as flexible as possible. There are four main elements that
describe the operation of Simple Pagination. These are:

1. Paginator service
2. Item total callback
3. Slice callback
4. Pagination model

The **Paginator** service performs the pagination algorithm, generating the page range and item collection slices.
When it's done it will return a **Pagination** model filled with the item collection slice and meta information.

The two main operations the **Paginator** service will perform on your collection (or data set) are denoted by two
callback methods passed to the **Paginator** service. The first one is the **Item total callback**. This callback is
used to determine the total number of items in your collection (returned as an integer). The second one is the 
**Slice callback**. This callback actually slices your collection given an **offset** and **length** argument.

The idea behind using these callbacks is so that Simple Pagination is kept, well, simple! The real power comes with
the flexibility. You can use Simple Pagination with just about any collection you want. From simple arrays to database
lists to [Doctrine](http://www.doctrine-project.org/) collections to [Solr](http://lucene.apache.org/solr/) result 
sets - we're got you covered! It really doesn't matter what we slice - as long as it's a collection of things and you 
can count and slice it.

Basic Usage
-----------

Ok, lets go with the most basic example - paginating over an array.

```php
use AshleyDawson\SimplePagination;

// Build a mock list of items we want to paginate through
$items = array(
    'Banana',
    'Apple',
    'Cherry',
    'Lemon',
    'Pear',
    'Watermelon',
    'Orange',
    'Grapefruit',
    'Blackcurrant',
    'Dingleberry',
    'Snosberry',
    'Tomato',
);

// Instantiate a new paginator service
$paginator = new Paginator();

// Set some parameters
$paginator
    ->setItemsPerPage(10) // Give us a maximum of 10 items per page
    ->setPagesInRange(5) // How many pages to display in navigation (e.g. if we have a lot of pages to get through)
;

// Pass our item total callback
$paginator->setItemTotalCallback(function () use ($items) {
    return count($items);
});

// Pass our slice callback
$paginator->setSliceCallback(function ($offset, $length) use ($items) {
    return array_slice($items, $offset, $length);
});

// Paginate the item collection, passing the current page number (e.g. from the current request)
$pagination = $paginator->paginate((int)$_GET['page']);

// Ok, from here on is where we'd be inside a template of view (e.g. pass $pagination to your view)

// Iterate over the items on this page
foreach ($pagination->getItems() as $item) {
    echo $item . '<br />';
}

// Let's build a basic page navigation structure
foreach ($pagination->getPages() as $page) {
    echo '<a href="?page=' . $page . '">' . $page . '</a> ';
}
```

There are lots of other pieces of meta data held within the `$pagination` instance. These can be used for building
first, last previous and next buttons.