---
title:  "Leverage laravel's form request to make your code cleaner"
date:   2018-04-24 19:57:00
categories: [Laravel]
tags: [laravel, programming]
---
Laravel has a powerful feature called **Form Request** to perform form validation logic. If we look at laravel's official documentation, we can find **Form Request** under [**Validation**](https://laravel.com/docs/5.6/validation#form-request-validation) section. But it can be used for much more purposes than just a validation.

First of all, let's see how we can make a request class. Laravel has the following artisan command to create a form request class:
```
php artisan make:request CreateProductRequest
```
The above command will generate a file inside `app/Http/Requests` directory.

The file will contain two methods, `authorize` and `rules`. The `authorize` method is used to check if the user has authority to make this request. While in `rules` method, we specify different rules that needs to be passed before moving to the controller. 

To apply this validation rules, we just need to inject this request class in our controller method like below:
```php
public function store(CreateProductRequest $request)
{
	...
}
```
You can view more about it on laravel's [official documentation](https://laravel.com/docs/5.6/validation#introduction).

Besides validation, request classes can be used to optimize and make controller classes look cleaner. 

### Custom Rule class
In many cases, [built-in validation rules](https://laravel.com/docs/5.5/validation#available-validation-rules) will be fine. But there will be many cases when custom rules need to be made. 

Lets say, we have two tables, `categories` and `products`. `products` table contains a foreign key `category_id` which links to 	`categories`. A category can have multiple products and product name should be unique in each category.

| categories |
|------------|  
| id         |      
| name       |  

| products |
|------------|  
| id         |      
| name       |  
| category_id |

While adding and editing products, we need to check if the product with the given name is already used. This logic can be written in controller but it will make the method look dirty. Instead, we can create a custom **Rule** class introduced from laravel 5.5.

A custom validation rule class can be creating by implementing `
Illuminate\Contracts\Validation\Rule` interface or with the following artisan command:
```
php artisan make:rule UniqueProduct
```
It will create a file inside `app\Rules` directory. The Rule interface has two methods, `passes` and `message`. `passes` is where we put logic required for the validation. In `message`, we can simply return a string if the validation fails.
```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;
use Illuminate\Support\Facades\DB;

class UniqueProduct implements Rule
{
	private $ignore = [];
	private $categoryId;

	public function __construct($categoryId)
	{
		$this->categoryId = $categoryId;
	}

	public function ignore($column, $value)
	{
		$this->ignore = [$column, $value];
		return $this;
	}

	public function passes($attribute, $value)
	{
		$product = DB::table('products')
		->where(function ($query) use ($value) {
			return $query
			->where('name', $value)
			->where('products.category_id', $this->categoryId);
		});

		// While editing product, ignore current product
		if (2 === count($this->ignore)) {
			$product->where($this->ignore[0], '<>', $this->ignore[1]);
		}

		// if > 0, Product with the name already exists
		if ($product->count() > 0) {
			return false;
		}

		return true;
	}

	public function message()
	{
		return 'The :attribute has already been taken.';
	}
}
```
First of all, the id of the category is passed through the constructor. In the `passes` method, at first,  there is a query to check if a product with the same name exists in the table, no fancy here. After that there is a condition to ignore the current product's row. While editing, we don't have to check the current product's data, so for that, we just need to pass the identifier of the product in an array to the ignore method, eg. `['id', 1]`.  And in `message` method, we just return the error message for the validation.

To use this rule, in the form request class, we only need to instantiate the rule class.
```php
CreateProductRequest.php
...
public function rules()
{
	return [
		'category_id' => 'required',
		'name' => [
			'bail',
			'required',
			new UniqueProduct($this->only('category_id'))
		]
	];
}

UpdateProductRequest.php
...
public function rules()
{
	return [
		'category_id' => 'required',
		'name' => [
			'bail',
			'required',
			(new UniqueProduct($this->only('category_id')))
			->ignore('id', $this->route()->parameter('productId'))
		]
	];
	// productId is obtained from the route param
	// Route::put('products/{productId}', '..ProductController@update');
}
```
So now both your controller and request class will look cleaner.

### Adding/Editing request elements
There will be a time when we have to add some elements to the request object. Also, we may need to modify request elements before validating or storing it to the database.

Lets say, while editing a product, the request has manufactured year, date and day as seperate elements, and we need to merge them together to create a single date. Similarly, let's say product's attributes are in json format and we need to convert it to array.

One way is to modify them in the controller or service class after receiving the request object. Since, we are talking about making our code cleaner, we can utilize request classes to make our changes.

As of laravel 5, laravel calls `getValidatorInstance` method and it gets through several methods, to call `all()` to get the input data for validation. We can override this method in form request class to add our own, or modify the request elements.

### Adding elements to request data
`merge` method will merge the request data with our own array data, which can be useful to add our own data to the request object.
```php
public function all()
{
	$this->merge(['date_of_manufacture' => implode(
		'-',
		[
			$request->get('year'),
			$request->get('month'),
			$request->get('day')
		]
	)]);
	return parent::all();
}
```

### Editing request data
We can retrieve all the input data in an array from `parent::all()` . Then we can modify the existing elements of the array. 
```php
public function all()
{
	$input = parent::all();
	// assuming attributes will be json encoded data
	// convert to into associative array
	// can be useful for array validation
	$input['attributes'] = json_decode($this->get('attributes'), true);

	// replaces the request data with new data
	$this->replace($input);
	return parent::all();
}
```

So by these ways, we can make our code cleaner by utilizing the request classes, and use them beyond validation.