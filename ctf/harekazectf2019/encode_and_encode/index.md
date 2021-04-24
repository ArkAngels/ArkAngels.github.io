---
layout: default
title: Harekaze CTF 2019 - Encode and Encode
---

<h1 align="center">[Harekaze CTF 2019 - Encode and Encode]</h1>
## Challenge Descripton:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/challdesc.png"></p><br>
We are given a web and the source code (for source code you can find it in <a href="https://github.com/ArkAngels/CTF-Source-Codes/tree/master/Harekaze%20CTF%202019%20-%20Encode%20and%20Encode">link</a>).<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/index.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/about.png"></p><br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/lorem.png"></p><br>

From here, there are 3 pages that we can access; about, lorem, and source code of the <b>query.php</b>. Let's take a look at <b>query.php</b> first:
```php
<?php
error_reporting(0);

if (isset($_GET['source'])) {
  show_source(__FILE__);
  exit();
}

function is_valid($str) {
  $banword = [
    // no path traversal
    '\.\.',
    // no stream wrapper
    '(php|file|glob|data|tp|zip|zlib|phar):',
    // no data exfiltration
    'flag'
  ];
  $regexp = '/' . implode('|', $banword) . '/i';
  if (preg_match($regexp, $str)) {
    return false;
  }
  return true;
}

$body = file_get_contents('php://input');
$json = json_decode($body, true);

if (is_valid($body) && isset($json) && isset($json['page'])) {
  $page = $json['page'];
  $content = file_get_contents($page);
  if (!$content || !is_valid($content)) {
    $content = "<p>not found</p>\n";
  }
} else {
  $content = '<p>invalid request</p>';
}

// no data exfiltration!!!
$content = preg_replace('/HarekazeCTF\{.+\}/i', 'HarekazeCTF{&lt;censored&gt;}', $content);
echo json_encode(['content' => $content]);

```
From this source code, I will try to explain the important part. Starting from function <b>is_valid</b>.
```php
function is_valid($str) {
  $banword = [
    // no path traversal
    '\.\.',
    // no stream wrapper
    '(php|file|glob|data|tp|zip|zlib|phar):',
    // no data exfiltration
    'flag'
  ];
  $regexp = '/' . implode('|', $banword) . '/i';
  if (preg_match($regexp, $str)) {
    return false;
  }
  return true;
}
```
In this function, it will check our input for word specified in the <i>$banword</i> variable. If even one of the word matched with our input, it will return false.<br>
Moving on to the next part:
```php
$body = file_get_contents('php://input');
$json = json_decode($body, true);

if (is_valid($body) && isset($json) && isset($json['page'])) {
  $page = $json['page'];
  $content = file_get_contents($page);
  if (!$content || !is_valid($content)) {
    $content = "<p>not found</p>\n";
  }
} else {
  $content = '<p>invalid request</p>';
}
```
In this section, the program will receive our input and store it in <i>$body</i> variable. After that, the <i>$body</i> variable will be turned into <i>json decoded</i> form and store it in <b>$json</b> variable. Until this part, we can conclude that our input must be in <i>json encoded</i> form.<br>

Next, there's a nested if. The first if checks whether the <b>is_valid</b> function return true or false. The second condition checks whether the <i>$json</i> variable is empty or not. And the last part checks whether the <i>$json['page']</i> is empty or not. These 3 conditions must be met in order to get into the if or else it will go to "Invalid Request".<br>

If we get into the if part, the program will take the content of <i>$json['page']</i> and store it in <i>$page</i>. And then, whatever is in the <i>$page</i> variable will be used as a file name and the program will try to get the content of it and store it in <i>$content</i> variable. After that there will be another pair of conditions where if the file that the program is trying to read cannot be found and if the <b>is_valid</b> check on <i>$content</i> returns false, the content of <i>$content</i> will be "Not Found".<br>

Now, to the last part:
```php
// no data exfiltration!!!
$content = preg_replace('/HarekazeCTF\{.+\}/i', 'HarekazeCTF{&lt;censored&gt;}', $content);
echo json_encode(['content' => $content]);
```
In this part, the program will replace string that contains <b>HarekazeCTF{blablabla}</b> to <b>HarekazeCTF{&lt;censored&gt;}<b>. And after that, the program will print out the content of <i>$content</i> in <i>json encoded</i> form.<br>

<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/check_input.png"></p><br>

Now what? Even if we can read the flag file located in <i>/flag</i>, the one we get isn't the real flag.<br>

Well, the writer also got confused on how to bypass the <i>preg_match</i> validation. But then, the writer found something interesting on stackoverflow (forgot the url) that has this statement.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/json_info.png"></p><br>
Turns out that JSON supports unicode and can translate unicode character. This way we can bypass the <i>preg_match</i> because the program doesn't detect any word specified in <i>$banword</i> variable.<br>

So now, with the help of online tools that translate string into unicode characters, the writer converted "/flag" string into unicode and we can do something like this:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/input.png"></p><br>

Yep, we can read the file now. But we're not there yet, because the output is still "HarekazeCTF{&lt;censored&gt;}". How to bypass the last <i>preg_replace</i>? Because we already bypassed the <i>preg_match</i> check, means we can use even features that are specified in <i>$banword</i>. And the writer found a way to read the file, by encode it to Base64 using one of php wrapper available which is "<a href="https://www.idontplaydarts.com/2011/02/using-php-filter-for-local-file-inclusion/">php://</a>". So by using:
  ```
  php://filter/convert.base64-encode/resource=<file>
  ```
We can encode the content of any file specified in <file>. So? we encode the wrapper with resource "/flag" into unicode and send it to the server.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/final_input.png"></p>

With that, we can read the "/flag" after we decode it.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/harekaze2019_encode/flag.png"></p><br>
Flag: HarekazeCTF{turutara_tattatta_ritta}
