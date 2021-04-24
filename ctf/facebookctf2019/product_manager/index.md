---
layout: default
title: Facebook CTF 2019 - Product Manager
---

<h1 align="center">[Facebook CTF 2019 - Product Manager]</h1>

## Challenge Description:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/challdesc.png"></p><br>
We are given a web with a source code. Let's take a look at the website first.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/home.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/add.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/view.png"></p><br>
So in this website, we can add a product but in order to add something, we need to provide a <i>secret</i> which has to be 10 characters or more and alphanumeric. After we add a product, we can go to view and then we can view our product by submitting the product name and the secret. Let's start by adding a product.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/add_test.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/added.png"></p><br>
And then let's try to view.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/view_test.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/viewed_test.png"></p><br>
So, when we added a product, by submitting name and secret, we can get the product name and the description. Hmm... interesting. Since we are provided with the source code, let's take a look at it.<br>
```php
// index.php
<?php

require_once("db.php");

$products = get_top_products();

require_once("header.php");
?>

<p>
  <ul>
<?php
foreach ($products as $product) {
  echo "<li>" . htmlentities($product['name']) . "</li>";
}
?>
  </ul>
</p>

<?php require_once("footer.php");
```
```php
// db.php
<?php
/*
CREATE TABLE products (
  name char(64),
  secret char(64),
  description varchar(250)
);

INSERT INTO products VALUES('facebook', sha256(....), 'FLAG_HERE');
INSERT INTO products VALUES('messenger', sha256(....), ....);
INSERT INTO products VALUES('instagram', sha256(....), ....);
INSERT INTO products VALUES('whatsapp', sha256(....), ....);
INSERT INTO products VALUES('oculus-rift', sha256(....), ....);
*/
error_reporting(0);
require_once("config.php"); // DB config

$db = new mysqli($MYSQL_HOST, $MYSQL_USERNAME, $MYSQL_PASSWORD, $MYSQL_DBNAME);

if ($db->connect_error) {
  die("Connection failed: " . $db->connect_error);
}

function check_errors($var) {
  if ($var === false) {
    die("Error. Please contact administrator.");
  }
}

function get_top_products() {
  global $db;
  $statement = $db->prepare(
    "SELECT name FROM products LIMIT 5"
  );
  check_errors($statement);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  $products = [];
  while ( ($product = $res->fetch_assoc()) !== null) {
    array_push($products, $product);
  }
  $statement->close();
  return $products;
}

function get_product($name) {
  global $db;
  $statement = $db->prepare(
    "SELECT name, description FROM products WHERE name = ?"
  );
  check_errors($statement);
  $statement->bind_param("s", $name);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  $product = $res->fetch_assoc();
  $statement->close();
  return $product;
}

function insert_product($name, $secret, $description) {
  global $db;
  $statement = $db->prepare(
    "INSERT INTO products (name, secret, description) VALUES
      (?, ?, ?)"
  );
  check_errors($statement);
  $statement->bind_param("sss", $name, $secret, $description);
  check_errors($statement->execute());
  $statement->close();
}

function check_name_secret($name, $secret) {
  global $db;
  $valid = false;
  $statement = $db->prepare(
    "SELECT name FROM products WHERE name = ? AND secret = ?"
  );
  check_errors($statement);
  $statement->bind_param("ss", $name, $secret);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  if ($res->fetch_assoc() !== null) {
    $valid = true;
  }
  $statement->close();
  return $valid;
}
```
```php
// view.php
<?php

require_once("db.php");
require_once("header.php");

function handle_post() {
  global $_POST;

  $name = $_POST["name"];
  $secret = $_POST["secret"];

  if (isset($name) && $name !== ""
        && isset($secret) && $secret !== "") {
    if (check_name_secret($name, hash('sha256', $secret)) === false) {
      return "Incorrect name or secret, please try again";
    }

    $product = get_product($name);

    echo "<p>Product details:";
    echo "<ul><li>" . htmlentities($product['name']) . "</li>";
    echo "<li>" . htmlentities($product['description']) . "</li></ul></p>";
  }

  return null;
}

$error = handle_post();
if ($error !== null) {
  echo "<p>Error: " . $error . "</p>";
}
?>
<form action="/view.php" method="POST">
  Name: <input type="text" name="name" /><br />
  Secret: <input type="password" name="secret" /><br />
  <input type="submit" value="View" />
</form>

<?php require_once("footer.php");
```
```php
// add.php
<?php

require_once("db.php");
require_once("header.php");

function validate_secret($secret) {
  if (strlen($secret) < 10) {
    return false;
  }
  $has_lowercase = false;
  $has_uppercase = false;
  $has_number = false;
  foreach (str_split($secret) as $ch) {
    if (ctype_lower($ch)) {
      $has_lowercase = true;
    } else if (ctype_upper($ch)) {
      $has_uppercase = true;
    } else if (is_numeric($ch)) {
      $has_number = true;
    }
  }
  return $has_lowercase && $has_uppercase && $has_number;
}

function handle_post() {
  global $_POST;

  $name = $_POST["name"];
  $secret = $_POST["secret"];
  $description = $_POST["description"];

  if (isset($name) && $name !== ""
        && isset($secret) && $secret !== ""
        && isset($description) && $description !== "") {
    if (validate_secret($secret) === false) {
      return "Invalid secret, please check requirements";
    }

    $product = get_product($name);
    if ($product !== null) {
      return "Product name already exists, please enter again";
    }

    insert_product($name, hash('sha256', $secret), $description);

    echo "<p>Product has been added</p>";
  }

  return null;
}

$error = handle_post();
if ($error !== null) {
  echo "<p>Error: " . $error . "</p>";
}
?>
<form action="/add.php" method="POST">
  Name of your product: <input type="text" name="name" /><br />
  Secret (10+ characters, smallcase, uppercase, number) : <input type="password" name="secret" /><br />
  Description: <input type="text" name="description" /><br />
  <input type="submit" value="Add" />
</form>

<?php require_once("footer.php");
```
From this source code, we have an information for the MySQL table for <i>products</i>, and the initial insert. According to the source code, the flag is in the description of <i>facebook</i> product. But there is the problem, the secret that we input for <i>view</i>, will be hashed using SHA256 and compared to the one in the database. And we cannot perform any SQL Injection because of the prepared statement. XSS also not possible because of the <i>htmlentities</i> (Yeah right the flag is in the database so XSS probably off-topic). So how can we get the flag when we don't have the plaintext of the secret for <i>facebook</i> product? Brute-force? Perhaps? Let's take a closer look at the source code.<br>
```php
// add.php
function handle_post() {
  global $_POST;

  $name = $_POST["name"];
  $secret = $_POST["secret"];
  $description = $_POST["description"];

  if (isset($name) && $name !== ""
        && isset($secret) && $secret !== ""
        && isset($description) && $description !== "") {
    if (validate_secret($secret) === false) {
      return "Invalid secret, please check requirements";
    }

    $product = get_product($name);
    if ($product !== null) {
      return "Product name already exists, please enter again";
    }

    insert_product($name, hash('sha256', $secret), $description);

    echo "<p>Product has been added</p>";
  }

  return null;
}
```
In this part of source code, we can conclude that we cannot input the same product name since it will be a duplicate.<br>
```php
function check_name_secret($name, $secret) {
  global $db;
  $valid = false;
  $statement = $db->prepare(
    "SELECT name FROM products WHERE name = ? AND secret = ?"
  );
  check_errors($statement);
  $statement->bind_param("ss", $name, $secret);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  if ($res->fetch_assoc() !== null) {
    $valid = true;
  }
  $statement->close();
  return $valid;
}
```
And for this function, it will check when we are about to view a product to check whether the <i>name</i> AND the <i>secret</i> that we input matches with the one in the database.<br>
```php
function get_product($name) {
  global $db;
  $statement = $db->prepare(
    "SELECT name, description FROM products WHERE name = ?"
  );
  check_errors($statement);
  $statement->bind_param("s", $name);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  $product = $res->fetch_assoc();
  $statement->close();
  return $product;
}
```
This function will try to get the product that we input and return it. Hang on, why when we about to get the product it only checks for the name? Doesn't the program should check for the secret as well? Hmm... suspicious. Seems like this is our entry point. But how? We cannot input <i>facebook</i> again since it is already in the database. Nope, actually there is this vulnerability that the writer found on the internet last year called <b href="https://blog.lucideus.com/2018/03/sql-truncation-attack-2018-lucideus.html">SQL Truncation Attack</b>. The writer cannot explain it clearly about this SQL Truncation Attack sorry but you guys can read about it in the link.<br>

The point of this attack this time is we can input something <b>different</b> but not quite different(?). From the PHP part, if we input "facebook\<space\>" it will be recognized as it is. But when it is being inserted to the database, MySQL somehow <i>trimmed</i> the space. So, from here, if we input "facebook\<space\>" and our own secret, in MySQL it will be "facebook" and \<my secret\>. So there will be 2 products named "facebook" but with 2 different secrets.<br>

So from there, we can input something like this:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/add_product.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/added.png"></p><br>
From here, we can simply input "facebook" with our own secret to get the flag.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/get_flag.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/facebookctf2019_product/flag.png"></p><br>
Flag: fb{4774ck1n9_5q1_w17h0u7_1nj3c710n_15_4m421n9_:)}
