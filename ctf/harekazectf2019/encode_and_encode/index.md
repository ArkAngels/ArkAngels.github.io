---
layout: default
title: [Harekaze CTF 2019 - Encode and Encode]
---

<h1 align="center">[Harekaze CTF 2019 - Encode and Encode]</h1>
## Challenge Descripton:
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/harekaze2019_encode/challdesc.png"></p><br>
We are given a web and the source code (for source code you can find it in <a href="https://github.com/ArkAngels/CTF-Source-Codes/tree/master/Harekaze%20CTF%202019%20-%20Encode%20and%20Encode">link</a>).<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/harekaze2019_encode/index.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/harekaze2019_encode/about.png"></p><br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/harekaze2019_encode/lorem.png"></p><br>

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
