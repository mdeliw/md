# PHP, MySQL

[toc]

## Basics

```php
<?php
    // comment
    /* comment */
    echo "Hello";
	$var = 0;
	$var = 1.2;
	$var = "string";
	
	// array
	$team = array('1', '2');
	echo $team[1];

	// multi-dimensional array
	$oxo = array(
        array('x', ' ', 'o'),
        array('o', 'o', 'x'),
        array('x', 'o', ' ')
    );
	echo $oxo[1][2];

	//equal and identical
	// ==, ===, !=, !==

	// string concatenation
	echo "This " . $msg . " is concatenated";
	$newMsg .= $oldMsg;

	// literel string without replacement
	$msg = 'This message will not replace $variable';

	// literel string with replacement
	$msg = "This message will replace the $count with the value of count";

	// constant
	define("ROOT_LOCATION", "/usr/local/www/");

	// function
	function longdate($timestamp) {
	    return date("l F jS Y", $timestamp);
  	}
	echo longdate(time());

	// global var
	global $is_logged_in;

	// local variable inside a function retaining a state when the function 
	// is called multiple times.
	static $count = 0;
	
?>
```

## Expressions and Control flow

```php
$y = 3 * (abs(2 * $x) + 4);

echo "a: [" . TRUE  . "]<br>"; // a:[1]
echo "a: [" . (20 > 9) . "]<br>"; // a[1]

if ($days_to_new_year < 30) {
    echo "Not long now till new year";  // Statement
}

// else, elseif, switch, ?, while, do..while, for, 

// casting
```

## Functions and Objects

```php
require_once "library.php";

if (function_exists("array_combine")) {
    echo "Function exists";
}

class User {
    const ENGLISH = 0;
    public $name, $password;
    static $static_property = "I'm static";
    function save_user() {
        echo "Save User code goes here";
    }
    
    static function pwd_string() {
      echo "Please enter your password";
    }
}
$object = new User;
$temp   = new User('name', 'password');
print_r($object);
$object->name = "Joe";

// with constructor
class User {
    function __construct($param1, $param2) {
      public $username = "Guest";
    }
    
    function __destruct() {
      // Destructor code goes here
    }
    
    function get_password() {
      return $this->password;
    }
}
```

## Arrays

```php
$paper[] = "Copier";
$paper[] = "Inkjet";
$paper[] = "Laser";
$paper[] = "Photo";

for ($j = 0 ; $j < 4 ; ++$j)
    echo "$j: $paper[$j]<br>";

$paper['copier'] = "Copier & Multipurpose";
$paper['inkjet'] = "Inkjet Printer";
$paper['laser']  = "Laser Printer";
$paper['photo']  = "Photographic Paper";

// associative
$p2 = array('copier' => "Copier & Multipurpose",
            'inkjet' => "Inkjet Printer",
            'laser'  => "Laser Printer",
            'photo'  => "Photographic Paper");

// foreach
$paper = array("Copier", "Inkjet", "Laser", "Photo");
$j = 0;

foreach($paper as $item) {
    echo "$j: $item<br>";
    ++$j;
}


$paper = array('copier' => "Copier & Multipurpose",
               'inkjet' => "Inkjet Printer",
               'laser'  => "Laser Printer",
               'photo'  => "Photographic Paper");

foreach($paper as $item => $description)
    echo "$item: $description<br>";

// list
list($a, $b) = array('Alice', 'Bob');
echo "a=$a b=$b";

// multidimensional
$products = array(
    
    'paper' => array(

        'copier' => "Copier & Multipurpose",
        'inkjet' => "Inkjet Printer",
        'laser'  => "Laser Printer",
        'photo'  => "Photographic Paper"),

    'pens' => array(

        'ball'   => "Ball Point",
        'hilite' => "Highlighters",
        'marker' => "Markers"),

    'misc' => array(

        'tape'   => "Sticky Tape",
        'glue'   => "Adhesives",
        'clips'  => "Paperclips"
    )
);

foreach($products as $section => $items)
    foreach($items as $key => $value)
        echo "$section:\t$key\t($value)<br>";

// is_array, count, sort, shuffle, explode, extract, compact, 
```

## Practical Use

```php
printf();
time();
date();

// files handling
    
```

## MySQL

```php
$hn = 'localhost';
$db = 'publications';
$un = 'username';
$pw = 'password';

// connect to db
require_once 'login.php';
$conn = new mysqli($hn, $un, $pw, $db);
if ($conn->connect_error) die("Fatal Error");

// fetching a result
require_once 'login.php';
$connection = new mysqli($hn, $un, $pw, $db);

if ($connection->connect_error) die("Fatal Error");

$query  = "SELECT * FROM classics";
$result = $connection->query($query);

if (!$result) die("Fatal Error");

$rows = $result->num_rows;

for ($j = 0 ; $j < $rows ; ++$j) {
    $result->data_seek($j);
    echo 'Author: '  .htmlspecialchars($result->fetch_assoc()['author'])  .'<br>';
    $result->data_seek($j);
    echo 'Title: '   .htmlspecialchars($result->fetch_assoc()['title'])   .'<br>';
    $result->data_seek($j);
    echo 'Category: '.htmlspecialchars($result->fetch_assoc()['category']).'<br>';
    $result->data_seek($j);
    echo 'Year: '    .htmlspecialchars($result->fetch_assoc()['year'])    .'<br>';
    $result->data_seek($j);
    echo 'ISBN: '    .htmlspecialchars($result->fetch_assoc()['isbn'])    .'<br><br>';
}

$result->close();
$connection->close();

// fetching a row
require_once 'login.php';
$conn = new mysqli($hn, $un, $pw, $db);
if ($conn->connect_error) die("Fatal Error");

$query  = "SELECT * FROM classics";
$result = $conn->query($query);
if (!$result) die("Fatal Error");

$rows = $result->num_rows;

for ($j = 0 ; $j < $rows ; ++$j)  {
    $row = $result->fetch_array(MYSQLI_ASSOC);

    echo 'Author: '   . htmlspecialchars($row['author'])   . '<br>';
    echo 'Title: '    . htmlspecialchars($row['title'])    . '<br>';
    echo 'Category: ' . htmlspecialchars($row['category']) . '<br>';
    echo 'Year: '     . htmlspecialchars($row['year'])     . '<br>';
    echo 'ISBN: '     . htmlspecialchars($row['isbn'])     . '<br><br>';
}

$result->close();
$conn->close();
```

