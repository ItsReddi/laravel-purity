<p align="center">
  <img src="/art/purity-logo.png" alt="Social Card of Laravel Purity">
  <h2 align="center">Elegant way to filter and sort</h2>
</p>

Laravel Purity is an elegant and efficient filtering and sorting package for Laravel, designed to simplify complex data filtering and sorting logic. By simply adding `filter()` to your Eloquent query, you can add the ability for frontend users to apply filters.

Features :
 - verios filter methods
 - filter by relation coulmns
 - custom filters
 - multi-column sort

Laravel Purity is not only developer-friendly but also front-end developer-friendly. Frontend developers can effortlessly use filtering and sorting of the APIs by using the popular [JavaScript qs](https://www.npmjs.com/package/qs) package.

> **Note**
> if you are front-end developer and what to make queries in an API that uses this package head to [queries](#queries-and-javascript-examples) section

the way that this package handles filters is inspired by strapi [filter](https://docs.strapi.io/dev-docs/api/rest/filters-locale-publication#filtering) and [sort](https://docs.strapi.io/dev-docs/api/rest/sort-pagination#sorting) functionality.

## Installation
Install the package via composer by this command:
   ```sh
   composer require abbasudo/laravel-purity 
   ```
Get configs (`configs/purity.php`) file to customize package's behavior by this command:
   ```sh
   php artisan vendor:publish --tag=purity 
   ```
## Basic Usage
### Filters
Add `Filterable` trait to your model to get filters functionalities.
```php
use Abbasudo\Purity\Traits\Filterable;

class Post extends Model
{
    use Filterable;
    
    //
}
```
now add `filter()` to your model query in the controller.
```php
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        return Post::filter()->get();
    }
}
```
By default it gives access to all filters available. here is the list of [avalable filters](#avalable-filters). if you want to explicitly specify which filters to use in this call head to [allowed filters](#allowed-filters) section.

### Sort
Add `Sortable` trait to your model to get sorts functionalities.
```php
use Abbasudo\Purity\Traits\Sortable;

class Post extends Model
{
    use Sortable;
    
    //
}
```
now add `filter()` to your model query in the controller.
```php
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        return Post::sort()->get();
    }
}
```
Now sort can be applied as instructed in [sort usage](#usage-examples).
## Advanced Usage
### Allowed Filters
The system validates allowed filters in the following order of priority:
- Filters passed as an array to the `filter()` function.
- Filters declared in the `$filters` variable in the model.
- Filters specified in the `filters` configuration in the `configs/purity.php` file.



## Queries and javascript examples
This section is a guide for front-end developers who want to use an API that uses this package. 
### Avalable Filters
Queries can accept a filters parameter with the following syntax:

`GET /api/posts?filters[field][operator]=value`

**By Default** the following operators are available:
| Operator       | Description                                      |
| -------------- | ------------------------------------------------ |
| `$eq`          | Equal                                            |
| `$eqi`         | Equal (case-insensitive)                         |
| `$ne`          | Not equal                                        |
| `$lt`          | Less than                                        |
| `$lte`         | Less than or equal to                            |
| `$gt`          | Greater than                                     |
| `$gte`         | Greater than or equal to                         |
| `$in`          | Included in an array                             |
| `$notIn`       | Not included in an array                         |
| `$contains`    | Contains                                         |
| `$notContains` | Does not contain                                 |
| `$containsi`   | Contains (case-insensitive)                      |
| `$notContainsi`| Does not contain (case-insensitive)              |
| `$null`        | Is null                                          |
| `$notNull`     | Is not null                                      |
| `$between`     | Is between                                       |
| `$startsWith`  | Starts with                                      |
| `$startsWithi` | Starts with (case-insensitive)                   |
| `$endsWith`    | Ends with                                        |
| `$endsWithi`   | Ends with (case-insensitive)                     |
| `$or`          | Joins the filters in an "or" expression          |
| `$and`         | Joins the filters in an "and" expression         |

#### Simple Filtering

> **Tip**
>   in javascript use [qs](https://www.npmjs.com/package/qs) directly to generate complex queries instead of creating them manually. Examples in this documentation showcase how you can use `qs`.

Find users having 'John' as first name

`GET /api/users?filters[name][$eq]=John`
  ```js
  const qs = require('qs');
const query = qs.stringify({
  filters: {
    username: {
      $eq: 'John',
    },
  },
}, {
  encodeValuesOnly: true, // prettify URL
});

await request(`/api/users?${query}`);
  ```
Find multiple restaurants with ids 3, 6, 8

`GET /api/restaurants?filters[id][$in][0]=3&filters[id][$in][1]=6&filters[id][$in][2]=8`
  ```js
  const qs = require('qs');
const query = qs.stringify({
  filters: {
    id: {
      $in: [3, 6, 8],
    },
  },
}, {
  encodeValuesOnly: true, // prettify URL
});

await request(`/api/restaurants?${query}`);
  ```
#### Complex Filtering
Complex filtering is combining multiple filters using advanced methods such as combining $and & $or. This allows for more flexibility to request exactly the data needed.

Find books with 2 possible dates and a specific author.

`GET /api/books?filters[$or][0][date][$eq]=2020-01-01&filters[$or][1][date][$eq]=2020-01-02&filters[author][name][$eq]=Kai%20doe`
```js
const qs = require('qs');
const query = qs.stringify({
  filters: {
    $or: [
      {
        date: {
          $eq: '2020-01-01',
        },
      },
      {
        date: {
          $eq: '2020-01-02',
        },
      },
    ],
    author: {
      name: {
        $eq: 'Kai doe',
      },
    },
  },
}, {
  encodeValuesOnly: true, // prettify URL
});

await request(`/api/books?${query}`);
```
#### Deep Filtering
Deep filtering is filtering on a relation's fields.

Find restaurants owned by a chef who belongs to a 5-star restaurant

`GET /api/restaurants?filters[chef][restaurants][stars][$eq]=5`
```js
const qs = require('qs');
const query = qs.stringify({
  filters: {
    chef: {
      restaurants: {
        stars: {
          $eq: 5,
        },
      },
    },
  },
}, {
  encodeValuesOnly: true, // prettify URL
});

await request(`/api/restaurants?${query}`);
```
### Apply Sort
Queries can accept a sort parameter that allows sorting on one or multiple fields with the following syntaxes:

`GET /api/:pluralApiId?sort=value` to sort on 1 field

`GET /api/:pluralApiId?sort[0]=value1&sort[1]=value2` to sort on multiple fields (e.g. on 2 fields)

The sorting order can be defined with:
- `:asc` for ascending order (default order, can be omitted)
- `:desc` for descending order.

#### Usage Examples

Sort using 2 fields

`GET /api/articles?sort[0]=title&sort[1]=slug`

```js
const qs = require('qs');
const query = qs.stringify({
  sort: ['title', 'slug'],
}, {
  encodeValuesOnly: true, // prettify URL
});

await request(`/api/articles?${query}`);
```



Sort using 2 fields and set the order

`GET /api/articles?sort[0]=title%3Aasc&sort[1]=slug%3Adesc`

```js
const qs = require('qs');
const query = qs.stringify({
  sort: ['title:asc', 'slug:desc'],
}, {
  encodeValuesOnly: true, // prettify URL
});

await request(`/api/articles?${query}`);
```

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
