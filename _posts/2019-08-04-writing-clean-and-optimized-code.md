---
title:  "Writing clean and optimized code"
date:   2019-08-04 21:02:00
categories: [Tutorial]
tags: [programming]
---
A code, whether it is a good or a bad one, depends ultimately on a developer. A bad code may work functionally, but the software will bring uncertain bugs and will be difficult to scale in the future. Writing a clean and optimized code takes time, and practice. But once written, it will not just reduce bugs and uncertainty, but also help other developers to understand the code properly.

There are many ways one can write clean and optimized code. Few of them are as follows:

### Write intuitive class, method and variable names
Naming things are hard. A developer must take out his/her time to choose proper names instead of writing unrelated ones. Improper names will not just waste the time of other developers, but also the one who writes, in the future.

Consider the following simple example:

```js
const fn = 'Super';
const ln = 'Man';
  
function getData(fn, ln) {
	return `${fn} ${ln}`;
}

getData(fn, ln);
```
Here, a developer can understand the code only after reading the whole logic. Now, lets consider the following code: 
```js
const firstName = 'Super';
const lastName = 'Man';
  
function getFullName(firstName, lastName) {
	return `${firstName} ${lastName}`;
}

getFullName(firstName, lastName);
```
Anyone can understand the code just by reading the variable and method names.

### Follow coding standards
Coding standards are the set of guidelines and best practices for writing a software. In every programming languages, there are certain rules or conventions that the developers follow to make the code consistent and clean. These standards help the developers in any part of the world write the same code and work together without any problems.

### YAGNI Principle
**YAGNI** which stands for **You Aren't Gonna Need It**,  states that a developer should not add the functionality unless it is truely necessary.  Assuming the future requirements unnecessary and adding unrequired functionality with add code complexity, and can make the code unoptimized.

Lets say there is a requirement to activate a client. One might think of future and write a function to handle the activation feature of multiple clients.

```php
function changeStatus(array ids)
{
	foreach($ids as $id) {
		$client = $this->clientService->find($id);
		$client->activate();
	}
}
```

It is ok if there will definately be the requirement in the future. But for now, it only adds complexity in the code. Instead of sending an id of the client, the id needs to be wrapped around an array to use this function. So, do not over engineer the solution of the problem that might occur in the future.

### Review your own code
To ensure that the code you wrote is clean and optimized enough, review your own code regularly, or atleast before merging the code. Reading code allow` developers to find any possible fixes and changes that may not have found while writing. While reviewing the code, a developer can also find a possible way that he/she can simplify the particular code.

In conclusion, writing a clean code is all about simplifying the code, and making it consistent through out the program. Coding standards help to make the code consistent, and a developer should always take his/her time out to read about the best practices and how others are solving the particular problem.