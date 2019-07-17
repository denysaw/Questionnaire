# PHP Developer Coding Exemplar
My answers to [Rebilly](https://rebilly.com/) questionnaire.

## Instructions
This coding exemplar is designed to highlight your skills and areas of expertise with the PHP language in general. We rely mainly on PHP as our web technology of choice. Because of this, it's important that developers have a well-rounded understanding of the PHP language.

Please complete the following questions to the best of your ability. If you are unable to solve a question, please indicate as such. You should feel free to use the PHP manual and other resources to solve these questions; after all, we expect that our developers will use their problem-solving skills at work! Some questions are intended to be difficult, while others are meant to be easy or obvious. Please post your answers in a Gist, using Markdown format, and send the link for review.

This exercise should take approximately one hour to complete.

Good luck!

## Question 1
You've been tasked with identifying a string that contains the word "superman" (case insensitive). You've written the following code:
```php
<?php

if(!strpos(strtolower($str), 'superman')) {
	throw new Exception;
}
```

QA has come to you and said that this works great for strings like "I love superman", but an exception is generated for strings like "Superman is awesome!", which should not happen. Explain why this occurs, and show how you would solve this issue (you must use strpos() in your answer).


## Answer 1
Hehe)) Because it's a typical type coercion ) 0, which is returned as result of `strpos()` (in a case when string starts with a `superman`) interpreted by PHP as a boolean `false`, that's why exception is throwing in that case. So all we need is just use tripple-equal (`===`) comparison operator, which compares not just values, but also types:
```php
if (strpos(strtolower($str), 'superman') === false) {
    throw new Exception;
}
```

## Question 2
A client has called and said that they're noticing performance problems on their database when searching for a user by email address. You've checked, and the following query is running:

SELECT * FROM users WHERE email = 'user@test.com';

You run the EXPLAIN command and get the following results:
```
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | users | ALL  | NULL          | NULL | NULL    | NULL | 10320 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
```
Offer a theory as to why the performance is slow.

## Answer 2
Performance is slow because there're already a ton of users with those test emails ))  
And I bet they don't have even simple index there, so it's even a usual text search.  
So these are the steps they need to achieve (from high to low priority): 
- Add an unique index on the `email` column (to avoid users with same emails):
```sql
CREATE UNIQUE INDEX unique_email_index ON users(email);
```
- Add ReCaptcha to their SignUp form to prevent bots from signing up with test emails.
- Use email verification to free DB of users, which didn't confirm their accounts during a month (e.g.).

## Question 3
Starting with PHP 5.5, what is the recommended way to hash user passwords? Show some code that illustrates how you would hash a user's password, and also how you would check that a password supplied by a user matches the password on file.

If your client cannot upgrade to PHP 5.5, and you have to hash passwords using existing PHP libraries, what is one library/package that makes this easy?

## Answer 3
For PHP5.5+ recommended way of user passwords' hashing is a standard method `password_hash()`, which has few algorithm options (`bcrypt` (default one), `blowfish` and `argon2*`). It's usage very simple:
```php
$hash = password_hash('supersecretone', PASSWORD_DEFAULT);
 
$correct = password_verify($_POST['password'], $hash);
 
echo 'Password is '. ($correct ? '' : 'in'). 'correct';
```

If you still have PHP<5.5 you shouldn't use vulnerable `md5`/`sha1` algs, simply composer require (or clone from github) famous `password_compat` library and use same methods as you would work under PHP5.5+

## Question 4
You're given a sorted index array that contians no keys. The array contains only integers, and your task is to identify whether or not the integer you're looking for is in the array. Write a function that searches for the integer and returns true or false based on whether the integer is present. Describe how you arrived at your solution.

## Answer 4
Hehe ) There're already even few built-in function for such needs: `in_array()`, which returns bool, which equals whether array contains a needle-var or nope :) And `array_search()`, which does pretty same things, just returns key of a found value instead of just `true`. For finding only integers I would just pass `true` as 3rd method's parameter (`strict`) to any of those methods:
```php
public function intInArray(int $needle, array $haystack): bool
{
    return in_array($needle, $haystack, true);
    // or return array_search($needle, $haystack, true) !== false;
}
```

But if you meant to code it myself without using standard one, I would do it like that:
```php
public function intInArray(int $needle, array $haystack): bool
{
    foreach ($haystack as $el) {
        if ($needle === $el) return true;
    }
     
    return false;
}
```

So I've just walk thru the array elements and on first match I'm totally exiting the method with a positive result to avoid waisting a time on a walking thru the rest elements. And I didn't use $match var + break from loop to economy few bytes of RAM :)

## Question 5
During a large data migration, you get the following error: Fatal error: Allowed memory size of 134217728 bytes exhausted (tried to allocate 54 bytes). You've traced the problem to the following snippet of code:

```php
$stmt = $pdo->prepare('SELECT * FROM largeTable');
$stmt->execute();
$results = $stmt->fetchAll(PDO::FETCH_ASSOC);
foreach ($results as $result) {
	// manipulate the data here
}
```

Refactor this code so that it stops triggering the memory error.

## Answer 5
Hehe ) You just shouldn't fetch the rows all together. Fetch data with chunks of 100-1000 records using OFFSET/LIMIT:
```php
$stmt = $pdo->prepare('SELECT * FROM largeTable LIMIT ?, ?');
$limit = 100;
$offset = 0;

while (true) {
    $stmt->execute([$offset, $limit]);
    
    $results = $stmt->fetchAll(PDO::FETCH_ASSOC);
    if (!$results) break;
    
    foreach ($results as $result) {
        // manipulate the data here
    }
    
    $offset += $limit;
}
```

## Question 6
Write a function that takes a phone number in any form and formats it using a delimiter supplied by the developer. The delimiter is optional; if one is not supplied, use a dash (-). Your function should accept a phone number in any format (e.g. 123-456-7890, (123) 456-7890, 1234567890, etc) and format it according to the 3-3-4 US block standard, using the delimiter specified. Assume foreign phone numbers and country codes are out of scope.

Note: This question CAN be solved using a regular expression, but one is not REQUIRED as a solution. Focus instead on cleanliness and effectiveness of the code, and take into account phone numbers that may not pass a sanity check.

## Answer 6
Without optional country code prefix regular exrpession appeared to be not very huge/hard.  
So basically cutting away non-digits and then splitting digits into 3 number groups:
```php
public function formatIt(string $phone, string $delimiter = '-'): string
{
    $phone = preg_replace('/\D/', '', $phone);
    $pattern = '/^(\d{3})(\d{3})(\d{4})$/';
    $replacement = join($delimiter, ['$1', '$2', '$3']);
     
    return preg_replace($pattern, $replacement, $phone);
}
```

## Question 7
In production, we'll be caching to memcache. On staging, we'll be caching to APC. In development, we won't be caching at all. Design a library that allows you to store and retrieve data from the cache (only two methods required) and fits the requirements of all three environments. Consider making use of anything appropriate (e.g. traits, classes, interfaces, abstract classes, closures, etc) to solve this problem.

Note: This is an architecture question. Please focus on the design of your library, rather than implementation or the specific caches I've described.


## Answer 7
If show the things simple and compact I would code CacheDriverInterface:
```php
<?php
 
interface CacheDriverInterface
{
 
    // Here usual cache methods go (phpDoc only for the first method for the sake of the page length :)
 
    /**
     * Get an item by key.
     *
     * @param string $key
     * @return mixed
     */
    public function get(string $key);
 
    public function many(array $keys);
 
    public function put(string $key, $value, int $seconds);
 
    public function putMany(array $values, int $seconds);
 
    public function increment(string $key, $value = 1);
 
    public function decrement(string $key, $value = 1);
 
    public function forever(string $key, $value);
 
    public function forget(string $key);
 
    public function flush();
     
    ...
     
}
```
Write few CacheDrivers, which implement our interface:
```php
<?php
 
class MemcacheDriver implements CacheDriverInterface
{
 
    private $memcache;
 
    public function __construct()
    {
        $this->memcache = new Memcached();
        $this->memcache->addServer(env('CACHE_SERVER'), env('CACHE_PORT'));
    }
 
    /**
     * {@inheritDoc}
     */
    public function get(string $key)
    {
        $this->memcache->get($key);
    }
 
    /**
     * {@inheritDoc}
     */
    public function many(array $keys)
    {
        $this->memcache->getMulti($keys);
    }
 
    /**
     * {@inheritDoc}
     */
    public function put(string $key, $value, int $seconds)
    {
        $this->memcache->set($key, $value, $seconds);
    }
 
    /**
     * {@inheritDoc}
     */
    public function putMany(array $values, int $seconds)
    {
        $this->memcache->setMulti($values, $seconds);
    }
 
    ...
 
}
```
```php
<?php
 
class ApcDriver implements CacheDriverInterface
{
 
    /**
     * {@inheritDoc}
     */
    public function get(string $key)
    {
        apcu_fetch($key);
    }
 
    /**
     * {@inheritDoc}
     */
    public function many(array $keys)
    {
        return array_map(function($key) {
            return $this->get($key);
        }, $keys);
    }
 
    /**
     * {@inheritDoc}
     */
    public function put(string $key, $value, int $seconds)
    {
        apcu_store($key, $value, $seconds);
    }
 
    /**
     * {@inheritDoc}
     */
    public function putMany(array $values, int $seconds)
    {
        foreach ($values as $key => $value) {
            $this->put($key, $value, $seconds);
        }
    }
 
    ...
 
}
```
```php
<?php
 
class NocacheDriver implements CacheDriverInterface
{
 
    /**
     * {@inheritDoc}
     */
    public function get(string $key)
    {
        return null;
    }
 
    /**
     * {@inheritDoc}
     */
    public function many(array $keys)
    {
        return null;
    }
 
    /**
     * {@inheritDoc}
     */
    public function put(string $key, $value, int $seconds) {}
 
    /**
     * {@inheritDoc}
     */
    public function putMany(array $values, int $seconds) {}
 
    ...
 
}
```
Then I would create CacheService which is CacheManager at once in the single file. During own instantiation it creates proper CacheDriver according to our local/stage/prod `.env` file for the further usage. Surely we could use cache Repository/Factory here, but I wanted to keep things smallest as possible to show only the idea:
```php
<?php
 
declare(strict_types=1);
// here go namespaces and uses
 
final class Cache
{
 
    const SUPPORTED_DRIVERS = [
        'memcache' => MemcacheDriver::class,
        'apc'      => ApcDriver::class,
        'nocache'  => NoCacheDriver::class,
    ];
 
    /** @var CacheDriverInterface */
    private $driver;
 
 
    public function __construct()
    {
        $driver = env('CACHE_DRIVER'); // require symfony/dotenv
 
        if (!in_array($driver, array_keys(self::SUPPORTED_DRIVERS))) {
            $driver = 'nocache';
        }
 
        $driverClass  = self::SUPPORTED_DRIVERS[$driver];
        $this->driver = new $driverClass();
    }
 
    /**
     * @param string $key
     * @param $default
     * @return mixed
     */
    public function get(string $key, $default = '')
    {
        $value = $this->driver->get($key);
        
        if (!$value && $default) {
            $value = is_callable($default) ? $default() : $default;
            $this->driver->put($key, $value);
        }
        
        return $value;
    }
 
    // Here you could implement all needed interface's methods or just pass calls thru:
 
    public function __call($method, $args)
    {
        return $this->driver->$method(...$args);
    }
}
```
Now if our project uses some DI (Dependency Injection), we could simply autowire it into any class constructor and use it. Also example shows how not to be left dataless when no cache used (`development` case):
```php
<?php
 
class SomeClass
{
 
    /** @var UserRepository */
    protected $users;
  
    /** @var Cache */
    protected $cache;
     
    ...
     
    public function __construct(UserRepository $users, Cache $cache)
    {
        $this->users = $users;
        $this->cache = $cache;
    }
    
    /**
     * @return int
     */
    public function getUserCount(): int
    {
        return $this->cache->get('key', function() {
            return count($this->UsersRepository->findAll());
        });
    }
}
```
Or if we like Laravel style, we could code Cache as a static facade:
```php
<?php
 
declare(strict_types=1);
// here go namespaces, uses, etc.
 
final class Cache
{
 
    const SUPPORTED_DRIVERS = [
        'memcache' => MemcacheDriver::class,
        'apc'      => ApcDriver::class,
        'nocache'  => NoCacheDriver::class,
    ];
 
    /** @var CacheDriverInterface */
    private static $driver;
 
    private static function getDriver()
    {
        if (!self::$driver) {
            $driver = env('CACHE_DRIVER'); // require symfony/dotenv
 
            if (!in_array($driver, array_keys(self::SUPPORTED_DRIVERS))) {
                $driver = 'nocache';
            }
 
            $driverClass  = self::SUPPORTED_DRIVERS[$driver];
            self::$driver = new $driverClass();
        }
 
        return self::$driver;
    }
 
    /**
     * @param string $key
     * @param $default
     * @return mixed
     */
    public static function get(string $key, $default = '')
    {
        $value = self::getDriver()->get($key);
         
        ...
}
```
And call it from anywhere like:
```php
public function getUserCount(): array
{
    return Cache::get('users', function() {
        return User::all()->count;
    });
}
```

## Question 8
Write a complete set of unit tests for the following code:
```php
function fizzBuzz($start = 1, $stop = 100)
{
	$string = '';
	
	if($stop < $start || $start < 0 || $stop < 0) {
		throw new InvalidArgumentException;
	}
	
	for($i = $start; $i <= $stop; $i++) {
		if($i % 3 == 0 && $i % 5 == 0) {
			$string .= 'FizzBuzz';
			continue;
		}
		
		if($i % 3 == 0) {
			$string .= 'Fizz';
			continue;
		}
		
		if ($i % 5 == 0) {
			$string .= 'Buzz';
			continue;
		}
		
		$string .= $i;
	}
	
	return $string;
}
```

## Answer 8
```php
<?php
 
declare(strict_types=1);
 
require_once 'fizzBuzz.php';
 
use PHPUnit\Framework\TestCase;
 
final class FizzBuzzTest extends TestCase
{
    public function testStartGreaterThanStop()
    {
        $this->expectException(InvalidArgumentException::class);
        fizzBuzz(10,1);
    }
 
    public function testNegativeStart()
    {
        $this->expectException(InvalidArgumentException::class);
        fizzBuzz(-1, 2);
    }
 
    public function testNegativeStop()
    {
        $this->expectException(InvalidArgumentException::class);
        fizzBuzz(-3, -1);
    }
 
    public function testFizz()
    {
        $this->assertEquals('Fizz', fizzBuzz(3, 3));
    }
 
    public function testBuzz()
    {
        $this->assertEquals('Buzz', fizzBuzz(5, 5));
    }
 
    public function testFizzBuzz()
    {
        $this->assertEquals('FizzBuzz', fizzBuzz(15, 15));
    }
 
    public function testOneToFizz()
    {
        $this->assertEquals('12Fizz', fizzBuzz(1, 3));
    }
 
    public function testOneToBuzz()
    {
        $this->assertEquals('12Fizz4Buzz', fizzBuzz(1, 5));
    }
 
    public function testOneToFizzBuzz()
    {
        $this->assertEquals('12Fizz4BuzzFizz78FizzBuzz11Fizz1314FizzBuzz', fizzBuzz(1, 15));
    }
}
```

## Question 9
I've developed a class called Select to represent the SELECT statements I'd normally write for a database. I want to be able to use the Select objects as queries and automatically cast them to strings, but when I use them in PDOStatement::execute() I get the following error: Catchable fatal error: Object of class Select could not be converted to string. What should I change in my Select class so that this error goes away?

## Answer 9
Hehe) You simply need to add `__toString()` magic method and inside of it iterate all current elements of your QueryLanguage concatenating their contents to a string. So something like that:
```php
public function __toString()
{
    $query = '';
 
    foreach($this->statements as $stmt) {
        $query .= $stmt->getSQL();
    }
    
    return $query;
}
```

## Question 10
I have an array of file names:
```php
$files = [
	'/usr/share/nginx/wordpress/wp-content/themes/index.php',
	'/usr/share/nginx/wordpress/wp-content/themes/mytheme.php',
	'/usr/share/nginx/wordpress/wp-content/plugins/myplugin.php',
	'/usr/share/nginx/wordpress/wp-content/plugins/akismet.php',
	'/usr/share/nginx/wordpress/wp-content/uploads/november.jpg',
];
```

Below is a mixed list of specific file names, as well as file paths, that should be excluded. For example, ALL files under a file path should be excluded, but if the value is an actual file name, only that specific file should be excluded.

```php
$exclude = [
	'/usr/share/nginx/wordpress/wp-content/uploads',
	'/usr/share/nginx/wordpress/wp-content/plugins/myplugin.php',
];
```

For example, you'll want to exclude the uploads directory (and all files in it), but ONLY the myplugin.php file.

Devise a method for applying the exclusion list against the file list WITHOUT nested foreach() loops.

## Answer 10
Nice one ;)
```php
public function excludeFromList($list, $exclude)
{
    return array_filter($list, function($path) use ($exclude) {
        $filter = false;

        foreach($exclude as $exPath) {
            $filter = $path == $exPath || !pathinfo($exPath, PATHINFO_EXTENSION) && strpos($path, $exPath) === 0;
            if ($filter) break;
        }

        return !$filter;
    });
}
```
