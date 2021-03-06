Simple Pagination
=================

[![Build Status](https://travis-ci.org/AshleyDawson/SimplePagination.svg?branch=develop)](https://travis-ci.org/AshleyDawson/SimplePagination)

Simple pagination library implements a paging interface on collections of things. If you'd like to 
use the Simple Pagination within a Symfony 2 project then why not try my [Simple Pagination Bundle](https://github.com/AshleyDawson/SimplePaginationBundle).

Installation
------------

You can install Simple Pagination via [Composer](https://getcomposer.org/). To do that, simply `require` the 
package in your `composer.json` file like so:

```json
{
    "require": {
        "ashleydawson/simple-pagination": "1.0.*"
    }
}
```

Then run `composer update` to install the package.

How Simple Pagination Works
---------------------------

I've tried to make Simple Pagination as easy to use and as flexible as possible. There are four main elements that
describe the operation of Simple Pagination. These are:

* Paginator service
* Item total callback
* Slice callback
* Pagination model

The **Paginator** service performs the pagination algorithm, generating the page range and item collection slices.
When it's done it will return a **Pagination** model filled with the item collection slice and meta information.

The two main operations the **Paginator** service will perform on your collection (or data set) are denoted by two
callback methods passed to the **Paginator** service. The first one is the **Item total callback**. This callback is
used to determine the total number of items in your collection (returned as an integer). The second one is the 
**Slice callback**. This callback actually slices your collection given an **offset** and **length** argument.

The idea behind using these callbacks is so that Simple Pagination is kept, well, simple! The real power comes with
the flexibility. You can use Simple Pagination with just about any collection you want. From simple arrays to database
lists to [Doctrine](http://www.doctrine-project.org/) collections to [Solr](http://lucene.apache.org/solr/) result 
sets - we're got you covered! It really doesn't matter what we paginate - as long as it's a collection of things and you 
can count and slice it.

Basic Usage
-----------

Ok, lets go with the most basic example - paginating over an array.

```php
use AshleyDawson\SimplePagination\Paginator;

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

// Set some parameters (optional)
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

There are lots of other pieces of meta data held within the [pagination object](#pagination-object). These can be used for building
first, last previous and next buttons.

MySQL Example
-------------

Let's take the example above and use a MySQL result set instead of an array.

```php
use AshleyDawson\SimplePagination\Paginator;

// Instantiate a new paginator service
$paginator = new Paginator();

// Set some parameters (optional)
$paginator
    ->setItemsPerPage(10) // Give us a maximum of 10 items per page
    ->setPagesInRange(5) // How many pages to display in navigation (e.g. if we have a lot of pages to get through)
;

// Pass our item total callback
$paginator->setItemTotalCallback(function () {

    // Run count query
    $result = mysql_query("SELECT COUNT(*) FROM `TestData`");
    
    // Return the count (the value of the first result column), cast as an integer
    return (int)mysql_result($result, 0);
});

// Pass our slice callback
$paginator->setSliceCallback(function ($offset, $length) {
    
    // Run slice query
    $result = mysql_query("SELECT `Name` FROM `TestData` LIMIT {$offset}, {$length}");
    
    // Build a collection of items
    $collection = array();
    while ($row = mysql_fetch_assoc($result)) {
        $collection[] = $row;
    }
    
    // Return the collection
    return $collection;
});

// Paginate the item collection, passing the current page number (e.g. from the current request)
$pagination = $paginator->paginate((int)$_GET['page']);

// Ok, from here on is where we'd be inside a template of view (e.g. pass $pagination to your view)

// Iterate over the items on this page
foreach ($pagination->getItems() as $item) {
    echo $item['Name'] . '<br />';
}

// Let's build a basic page navigation structure
foreach ($pagination->getPages() as $page) {
    echo '<a href="?page=' . $page . '">' . $page . '</a> ';
}
```

It really doesn't matter what sort of collection you return from the Paginator::setSliceCallback() callback. It will
always end up in Pagination::getItems().

<a name="pagination-object"></a>Pagination Object
-------------------------------------------------

The result of the Paginator::paginate() operation is to produce a Pagination model object, which carries the item collection for 
the current page plus the meta information for the collection, e.g. pages array, next page number, previous page number, etc.
Please see below for a list of properties that the Pagination object has.

* **items** : mixed (Collection of items for the current page)
* **pages** : array (Array of page numbers in the current range)
* **totalNumberOfPages** : int (Total number of pages)
* **currentPageNumber** : int (Current page number)
* **firstPageNumber** : int (First page number)
* **lastPageNumber** : int (Last page number)
* **previousPageNumber** : int | null (Previous page number)
* **nextPageNumber** : int | null (Next page number)
* **itemsPerPage** : int (Number of items per page)
* **totalNumberOfItems** : int (Total number of items)
* **firstPageNumberInRange** : int (First page number in current range)
* **lastPageNumberInRange** : int (Last page number in current range)

A good example of using the Pagination object is to build a simple pagination navigation structure:

```php
// Render the first page link
echo '<a href="?page=' . $pagination->getFirstPageNumber() . '">First Page</a> ';

// Render the previous page link (note: the previous page number could be null)
echo '<a href="?page=' . $pagination->getPreviousPageNumber() . '">Previous Page</a> ';

// Render page range links
foreach ($pagination->getPages() as $page) {
    echo '<a href="?page=' . $page . '">' . $page . '</a> ';
}

// Render the next page link (note: the next page number could be null)
echo '<a href="?page=' . $pagination->getNextPageNumber() . '">Next Page</a> ';

// Render the last page link
echo '<a href="?page=' . $pagination->getLastPageNumber() . '">Last Page</a>';
```

