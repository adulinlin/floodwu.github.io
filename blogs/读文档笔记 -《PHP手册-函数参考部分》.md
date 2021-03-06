# PHP手册-函数参考部分阅读摘要

## 查找字符串首次出现的位置 `strpos`
mixed strpos( string $haystack , mixed $needle [, int $offset = 0 ] )
if(strpos($_POST[‘email’],’@’) === false){
...
}
为了区别返回值0和false，必须使用===或!==来判断。

## 返回字符串的子串 `substr`
string substr( string $string , int $start [, int $length ] )
如果start是负数，返回的字符串将从 string 结尾处向前数第 start 个字符开始：
$rest = substr("abcdef", -3, 1); 	// 返回 "d"

## 替换字符串的子串 `substr_replace`
mixed substr_replace( mixed $string , mixed $replacement , mixed $start [, mixed $length ] )
如果没有指定$length参数，将替换从$start到字符串末尾的所有字符。
如果设定了$length参数并且为正数，表示string中被替换的子字符串的长度。如果设定为负数，它表示待替换的子字符串结尾处距离string末端的字符个数。如果没有提供此参数，那么它默认为strlen( string ) （字符串的长度）。当然，如果length为0，那么这个函数的功能为将replacement插入到string的start位置处。
print substr_replace('abcdefghijklmn','XXXX',-5,3);
abcdefghijklmn  ->  abcdefghiXXXXmn

print substr_replace('abcdefghijklmn','...',10);
abcdefghijklmn  ->  abcdefghij...

## 获取字符串长度 `strlen`
int strlen( string $string )

## 查找字符串的首次出现-区分大小写 `strstr`
string strstr( string $haystack , mixed $needle [, bool $before_needle = false ] )
返回haystack字符串从needle第一次出现的位置开始到haystack结尾的字符串。
若$before_needle为TRUE，strstr()将返回needle在haystack中的位置之前的部分。
若未发现字符串，返回FALSE。
$email  = 'name@example.com';
$domain = strstr($email, '@');
echo $domain; 	// 打印 @example.com

$user = strstr($email, '@', true); 
echo $user; 		// 打印 name

## 查找字符串的首次出现-不区分大小写 `stristr`
string stristr( string $haystack , mixed $needle [, bool $before_needle = false ] )

## 按字节反转字符串 `strrev`
string strrev( string $string )
echo strrev("Hello world!"); // 输出 "!dlrow olleH"

## 使用一个字符串分割另一个字符串 `explode`
array explode( string $delimiter , string $string [, int $limit ] )
如果设置了limit参数并且是正数，则返回的数组包含最多limit个元素，而最后那个元素将包含string的剩余部分。
如果limit参数是负数，则返回除了最后的-limit个元素外的所有元素。
如果limit是0，则会被当做1。

## 将一个一维数组的值转化为字符串 `implode`
string implode( string $glue , array $pieces )

## 结合explode函数实现按单词反转字符串
$s = 'one two three four five';
$revs = implode(' ',array_reverse(explode(' ', $s)));
print( $revs);  # five four three two one

## 生成随机数 `mt_rand`
int mt_rand ( int $min , int $max )

## 执行一个正则表达式的搜索和替换 `preg_replace`
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
$string = 'April 15, 2003';
$pattern = '/[ri]/';
$replacement = 'X';
echo preg_replace($pattern, $replacement, $string);	# ApXXl 15, 2003

## 执行一个正则表达式搜索并且使用一个回调进行替换 `preg_replace_callback`
mixed preg_replace_callback ( mixed $pattern , callable $callback , mixed $subject [, int $limit = -1 [, int &$count ]] )
这个函数的行为除了可以指定一个callback替代replacement进行替换字符串的计算，其他方面等同于preg_replace()。

## 将字符串转换为数组 `str_split`
array str_split ( string $string [, int $split_length = 1 ] )
$str = "Hello Friend";
$arr = str_split($str,2);
print_r($arr);
输出：Array ( [0] => He [1] => ll [2] => o [3] => Fr [4] => ie [5] => nd )

## implode()的别名 `join`
string join ( string $glue , array $pieces )

## 子字符串替换-区分大小写 `str_replace`
mixed str_replace ( mixed $search , mixed $replace , mixed $subject [, int &$count ] )
该函数返回一个字符串或者数组。该字符串或数组是将subject中全部的search都被replace替换之后的结果。
// <body text='black'>
echo str_replace("%body%", "black", "<body text='%body%'>");

## 子字符串替换-不区分大小写 `str_ireplace`
mixed str_ireplace ( mixed $search , mixed $replace , mixed $subject [, int &$count ] )

## 重复一个字符串 `str_repeat`
string str_repeat ( string $input , int $multiplier )
echo str_repeat("-=", 10);  // -=-=-=-=-=-=-=-=-=-=

## 将字符串的首字母转换为大写 `ucfirst`
string ucfirst ( string $str )
echo ucfirst('hello world!');	// Hello world!

## 将字符串中每个单词的首字母转换为大写
string ucwords ( string $str ) ucwords
echo ucwords('hello world!');		// Hello World!

## 将字符串转化为小写 `strtolower`
string strtolower ( string $string )
echo strtolower('Hello World!');	// hello world!

## 将字符串转化为大写 `strtoupper`
string strtoupper ( string $string )
echo strtoupper('Hello World!');	// HELLO WORLD!

## 删除字符串开头的空白字符（或其他字符） `ltrim`
string ltrim ( string $str [, string $character_mask ] )

## 删除字符串末端的空白字符（或者其他字符） `rtrim`
string rtrim ( string $str [, string $character_mask ] )

## 去除字符串首尾处的空白字符（或者其他字符） `trim`
string trim ( string $str [, string $charlist = " \t\n\r\0\x0B" ] )

## rtrim()的别名 `chop`
string chop ( string $str [, string $character_mask ] )

## 打开文件或者URL `fopen`
resource fopen ( string $filename , string $mode [, bool $use_include_path = false [, resource $context ]] )
$handle = fopen("/home/rasmus/file.txt", "r");
$handle = fopen("/home/rasmus/file.gif", "wb");
$handle = fopen("http://www.example.com/", "r");
$handle = fopen("ftp://user:password@example.com/somefile.txt", "w");

## 关闭一个已打开的文件指针 `fclose`
bool fclose ( resource $handle )

## 输出一个消息并且退出当前脚本 `exit`
void exit( [string $status] )
void exit( int $status )
尽管调用了exit()，Shutdown函数以及object destructors总是会被执行。

## 输出一个消息并且退出当前脚本，等同于exit `die`
void die([ string $status ] )

## 将行格式化为CSV并写入文件指针 `fputcsv`
int fputcsv ( resource $handle , array $fields [, string $delimiter = ',' [, string $enclosure = '"' ]] )
fputcsv()将一行（用fields数组传递）格式化为CSV格式并写入由handle指定的文件。
文件指针必须是有效的，必须指向由 fopen() 或 fsockopen() 成功打开的文件(并还未由 fclose() 关闭)。
$list = array (
    array('aaa', 'bbb', 'ccc', 'dddd'),
    array('123', '456', '789'),
    array('"aaa"', '"bbb"')
);

$fp = fopen('file.csv', 'w');
foreach ($list as $fields) {
    fputcsv($fp, $fields);
}
fclose($fp);
将生成一个如下内容的文件：
aaa,bbb,ccc,dddd
123,456,789
"""aaa""","""bbb"""

## 从文件指针中读入一行并解析 CSV 字段 `fgetcsv`
array fgetcsv( resource $handle [, int $length = 0 [, string $delimiter = ',' [, string $enclosure = '"' [, string $escape = '\\' ]]]] )
$row = 1;
if (($handle = fopen("test.csv", "r")) !== FALSE) {
    while (($data = fgetcsv($handle, 1000, ",")) !== FALSE) {
        $num = count($data);
        echo "<p> $num fields in line $row: <br /></p>\n";
        $row++;
        for ($c=0; $c < $num; $c++) {
            echo $data[$c] . "<br />\n";
        }
    }
    fclose($handle);
}

## 把字符转换为 HTML 实体 `htmlentities`
string htmlentities ( string $string [, int $flags = ENT_COMPAT | ENT_HTML401 [, string $encoding = ini_get("default_charset") [, bool $double_encode = true ]]] )
$str = "A 'quote' is <b>bold</b>";

// Outputs: A 'quote' is &lt;b&gt;bold&lt;/b&gt;
echo htmlentities($str);

// Outputs: A &#039;quote&#039; is &lt;b&gt;bold&lt;/b&gt;
echo htmlentities($str, ENT_QUOTES);	# 既转换双引号，也转换单引号

#3 把数据装入一个二进制字符串 `pack` 
string pack ( string $format [, mixed $args [, mixed $... ]] )
echo pack("C3",80,72,80);	// PHP
echo pack("A3C1A3",80,72,80);	// 80 H80

## 从二进制字符串中解析出数据 `unpack` 
array unpack ( string $format , string $data )
$binarydata = "\x04\x00\xa0\x00";
$array = unpack("cchars/nint", $binarydata);
print_r($array);  // Array ( [chars] => 4 [int] => 160 )

## 使用另一个字符串填充字符串为指定长度 `str_pad`
string str_pad ( string $input , int $pad_length [, string $pad_string = " " [, int $pad_type = STR_PAD_RIGHT ]] )
该函数返回input被从左端、右端或者同时两端被填充到指定长度后的结果。如果可选的pad_string参数没有被指定，input将被空格字符填充，否则它将被pad_string填充到指定长度。
$input = "Alien";
echo str_pad($input, 10);                      // 输出 "Alien     "
echo str_pad($input, 10, "-=", STR_PAD_LEFT);  // 输出 "-=-=-Alien"
echo str_pad($input, 10, "_", STR_PAD_BOTH);   // 输出 "__Alien___"
echo str_pad($input, 6 , "___");               // 输出 "Alien_"

## 通过一个正则表达式分隔字符串 `preg_split`
array preg_split ( string $pattern , string $subject [, int $limit = -1 [, int $flags = 0 ]] )
//使用逗号或空格(包含" ", \r, \t, \n, \f)分隔短语
$keywords = preg_split("/[\s,]+/", "hypertext language, programming");
print_r($keywords);
输出：
	Array
(
    [0] => hypertext
    [1] => language
    [2] => programming
)

## 打断字符串为指定数量的字串 `wordwrap`
string wordwrap( string $str [, int $width = 75 [, string $break = "\n" [, bool $cut = false ]]] )
$text = "The quick brown fox jumped over the lazy dog.";
$newtext = wordwrap($text, 20, "<br />\n");
echo $newtext;
输出：
	The quick brown fox<br />
jumped over the lazy<br />
dog.

## 发送原生 HTTP 头 `header` 
void header ( string $string [, bool $replace = true [, int $http_response_code ]] )
 	header()必须在任何实际输出之前调用，不管是普通的HTML标签，还是文件或PHP输出的空行，空格。这是个常见的错误，在通过include，require，或者其访问其他文件里面的函数的时候，如果在header()被调用之前，其中有空格或者空行。
// 重定向
header('Location: http://www.example.com/');	
exit;

## 读入一个文件并写入到输出缓冲 `readfile`
int readfile ( string $filename [, bool $use_include_path = false [, resource $context ]] )
// 下载pdf
header('Content-type: application/pdf');
header('Content-Disposition: attachment; filename="downloaded.pdf"');
readfile('original.pdf');

## 检测变量是否为数字或数字字符串 `is_numeric`
bool is_numeric ( mixed $var )

## 检测变量是否是布尔型  `is_bool` 
bool is_bool ( mixed $var )

## 检测变量是否是浮点型  `is_float`
bool is_float ( mixed $var )

## 检测变量是否是整数 `is_int`
bool is_int ( mixed $var )

## 检测变量是否是字符串  `is_string`
bool is_string ( mixed $var )

## 检测变量是否是一个对象 `is_object`
bool is_object ( mixed $var )

## 检测变量是否是数组 `is_array`
bool is_array ( mixed $var )

## is_int() 的别名 `is_integer`
bool is_integer ( mixed $var )

## 绝对值 `abs`
number abs ( mixed $number )
// 比较浮点数是否相等（不应该直接用==比较）
$delta = 0.00001;
$a = 1.00000001;
$b = 1.00000000;

if( abs($a - $b) < $delta ){
	print 'equal enough !';
}

## 对浮点数进行四舍五入 `round` 
float round ( float $val [, int $precision = 0 [, int $mode = PHP_ROUND_HALF_UP ]] )
PHP 默认不能正确处理类似 "12,300.2" 的字符串。

## 向上取整 `ceil` 
float ceil ( float $value )

## 向下取整 `floor` 
float floor ( float $value )

## 建立一个包含指定范围单元的数组 `range` 
array range ( mixed $start , mixed $limit [, number $step = 1 ] )
返回的数组中从 start 到 limit 的单元，包括它们本身。

## 带种子生成随机数 `mt_srand`
void mt_srand ([ int $seed ] )

## 取对数 `log`
float log ( float $arg [, float $base = M_E ] )

## 计算 e 的指数 `exp` 
float exp ( float $arg )

## 取指数 `pow` 
number pow ( number $base , number $exp )

## 以千位分隔符方式格式化一个数字 `number_format`
string number_format ( float $number [, int $decimals = 0 ] )
string number_format ( float $number , int $decimals = 0 , string $dec_point = "." , string $thousands_sep = "," )
$number = 1234.56;
$english_format_number = number_format($number);			// 1,235
$nombre_format_francais = number_format($number, 2, ',', ' ');	// 1 234,56

$number = 1234.5678;
$english_format_number = number_format($number, 2, '.', '');	// 1234.57

## 2个任意精度数字的加法计算 `bcadd` 
string bcadd ( string $left_operand , string $right_operand [, int $scale ] )
$a = '1.234';
$b = '5';
echo bcadd($a, $b);     // 6
echo bcadd($a, $b, 4);  // 6.2340
非常大或者非常小的数的运算需要使用BCMath或者GMP库。

## 2个任意精度数字的减法 `bcsub`
string bcsub ( string $left_operand , string $right_operand [, int $scale = int ] )

# 检查某个名称的常量是否存在 `defined`
bool defined( string $name )

## 检测变量是否设置 `isset` 
bool isset ( mixed $var [, mixed $... ] )

## 检查函数是否已定义 `function_exists`
bool function_exists ( string $function_name )

## 在任意进制之间转换数字 `base_convert`
string base_convert ( string $number , int $frombase , int $tobase )
$hexadecimal = 'A37334';
echo base_convert($hexadecimal, 16, 2);
输出：
	101000110111001100110100

## 格式化一个本地时间／日期 `date` 
string date ( string $format [, int $timestamp ] )
echo date("Y-m-d H:i:s");	// 2016-01-13 10:05:39

## 取得日期／时间信息 `getdate` 
array getdate ([ int $timestamp = time() ] )
print_r(getdate());
输出：
	Array ( [seconds] => 30 [minutes] => 7 [hours] => 10 [mday] => 13 [wday] => 3 [mon] => 1 [year] => 2016 [yday] => 12 [weekday] => Wednesday [month] => January [0] => 1452676050 )

## 返回当前的 Unix 时间戳 `time` 
int time ( void )

## 取得本地时间
array localtime ([ int $timestamp = time() [, bool $is_associative = false ]] ) localtime 
print_r(localtime(time(),true));
输出：
Array ( [tm_sec] => 52 [tm_min] => 10 [tm_hour] => 10 [tm_mday] => 13 [tm_mon] => 0 [tm_year] => 116 [tm_wday] => 3 [tm_yday] => 12 [tm_isdst] => 0 )

## 取得一个日期的 Unix 时间戳 `mktime` 
int mktime ([ int $hour = date("H") [, int $minute = date("i") [, int $second = date("s") [, int $month = date("n") [, int $day = date("j") [, int $year = date("Y") [, int $is_dst = -1 ]]]]]]] )
echo date("M-d-Y", mktime(0, 0, 0, 12, 32, 1997));	// Jan-01-1998

## 取得 GMT 日期的 UNIX 时间戳 `gmmktime` 
int gmmktime ([ int $hour [, int $minute [, int $second [, int $month [, int $day [, int $year [, int $is_dst ]]]]]]] )

## 设定用于一个脚本中所有日期时间函数的默认时区 `date_default_timezone_set`
bool date_default_timezone_set ( string $timezone_identifier )

## 检验日期的合法性 `checkdate` 
bool checkdate ( int $month , int $day , int $year )
var_dump(checkdate(12, 31, 2000));	// bool(true)
var_dump(checkdate(2, 29, 2001));	// bool(false)

## 将任何英文文本的日期时间描述解析为 Unix 时间戳 `strtotime` 
int strtotime ( string $time [, int $now = time() ] )
echo strtotime("now"), "\n";
echo strtotime("10 September 2000"), "\n";
echo strtotime("+1 day"), "\n";
echo strtotime("+1 week"), "\n";
echo strtotime("+1 week 2 days 4 hours 2 seconds"), "\n";
echo strtotime("next Thursday"), "\n";
echo strtotime("last Monday"), "\n";
输出：1452677160 968536800 1452763560 1453281960 1453469162 1452726000 1452466800

## 返回当前 Unix 时间戳和微秒数 `microtime` 
mixed microtime ([ bool $get_as_float ] )
echo microtime();	// 0.89424900 1452677381

## 设置变量的类型 `settype`
bool settype ( mixed &$var , string $type ) settype 
$foo = "5bar"; 			// string
$bar = true;   			// boolean
settype($foo, "integer"); 		// $foo 现在是 5   (integer)
settype($bar, "string");  	// $bar 现在是 "1" (string)

## 将回调函数作用到给定数组的单元上 `array_map`
array array_map ( callable $callback , array $arr1 [, array $... ] )
function cube($n)
{
    return($n * $n * $n);
}

$a = array(1, 2, 3, 4, 5);
$b = array_map("cube", $a);
print_r($b);
$b将变成：
Array
(
    [0] => 1
    [1] => 8
    [2] => 27
    [3] => 64
    [4] => 125
)

## 释放给定的变量 `unset`
void unset ( mixed $var [, mixed $... ] )
$a = array('1'=>'a','2'=>'b','3'=>'c','4'=>'d','5'=>'e');
unset($a['3']);
print_r($a);	// Array ( [1] => a [2] => b [4] => d [5] => e )

$a = array('a','b','c','d','e');
unset($a[2]);
print_r($a);	// Array ( [0] => a [1] => b [3] => d [4] => e )

如果在函数中 unset() 一个全局变量，则只是局部变量被销毁，而在调用环境中的变量将保持调用 unset() 之前一样的值：
function destroy_foo() {
    global $foo;
    unset($foo);
}

$foo = 'bar';
destroy_foo();
echo $foo;	// bar
正确的做法：unset($GLOBALS['bar']);

## 将数组中的内部指针向前移动一位 `next`
mixed next ( array &$array )

## 返回数组中的当前单元 `current`
mixed current ( array &$array )

## 将数组的内部指针指向第一个单元 `reset` 
mixed reset ( array &$array )
$array = array('step one', 'step two', 'step three', 'step four');
echo current($array); 		// "step one"
next($array);
next($array);
echo current($array) ; 		// "step three"
reset($array);
echo current($array) ; 		// "step one"
 
## 返回数组中当前的键／值对并将数组指针向前移动一步 `each`
array each ( array &$array )

## 把数组中的值赋给一些变量 `list` 
array list ( mixed $varname [, mixed $... ] )
$fruit = array('a' => 'apple', 'b' => 'banana', 'c' => 'cranberry');
reset($fruit);
while (list($key, $val) = each($fruit)) {
    echo "$key => $val\n";
}
输出：
a => apple
b => banana
c => cranberry

## 计算数组中的单元数目或对象中的属性个数 `count`
int count ( mixed $var [, int $mode = COUNT_NORMAL ] )

## 把数组中的一部分去掉并用其它值取代 `array_splice`
array array_splice ( array &$input , int $offset [, int $length = 0 [, mixed $replacement ]] )
把input数组中由offset和length指定的单元去掉，如果提供了replacement参数，则用其中的单元取代。input中的数字键名不被保留。
如果指定了length并且为负值，则移除从offset到数组末尾倒数length为止中间的所有的单元。
$input = array("red", "green", "blue", "yellow");
array_splice($input, 2);							// array("red", "green")

$input = array("red", "green", "blue", "yellow");
array_splice($input, 1, -1);						// array("red", "yellow")

$input = array("red", "green", "blue", "yellow");
array_splice($input, 1, count($input), "orange");		// array("red", "orange")

$input = array("red", "green", "blue", "yellow");
array_splice($input, -1, 1, array("black", "maroon"));	// array("red", "green","blue", "black", "maroon")

$input = array("red", "green", "blue", "yellow");
array_splice($input, 3, 0, "purple");				// array("red", "green","blue", "purple", "yellow");

## 返回数组中所有的值 `array_values`
array array_values ( array $input )
$array = array("size" => "XL", "color" => "gold");
print_r(array_values($array));
输出：
Array
(
    [0] => XL
    [1] => gold
)

## 将数组开头的单元移出数组 `array_shift`
mixed array_shift ( array &$array )
将 array 的第一个单元移出并作为结果返回，将 array 的长度减一并将所有其它单元向前移动一位。所有的数字键名将改为从零开始计数，文字键名将不变。使用此函数后会重置（reset()）array 指针。
$stack = array("orange", "banana", "apple", "raspberry");
$fruit = array_shift($stack);
print_r($stack);
输出：
Array
(
    [0] => banana
    [1] => apple
    [2] => raspberry
)

## 将数组最后一个单元弹出 `array_pop`
mixed array_pop ( array &$array )
弹出并返回 array 数组的最后一个单元，并将数组 array 的长度减一。如果 array 为空（或者不是数组）将返回 NULL。使用此函数后会重置（reset()）array 指针。
$stack = array("orange", "banana", "apple", "raspberry");
$fruit = array_pop($stack);
print_r($stack);
输出：
	Array
(
    [0] => orange
    [1] => banana
    [2] => apple
)

## 用值将数组填补到指定长度 `array_pad`
array array_pad ( array $input , int $pad_size , mixed $pad_value )

## 合并一个或多个数组 `array_merge`
array array_merge ( array $array1 [, array $... ] )

## 将一个或多个单元压入数组的末尾 `array_push`
int array_push ( array &$array , mixed $var [, mixed $... ] )
如果用 array_push() 来给数组增加一个单元，还不如用 $array[] = ，因为这样没有调用函数的额外负担。

## 检查给定的键名或索引是否存在于数组中 `array_key_exists`
bool array_key_exists ( mixed $key , array $search )

## 检查数组中是否存在某个值 `in_array`
bool in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] )
如果第三个参数 strict 的值为 TRUE 则 in_array() 函数还会检查 needle 的类型是否和 haystack 中的相同。

## 交换数组中的键和值 `array_flip`
array array_flip ( array $trans )
如果值的类型不对将发出一个警告，并且有问题的键／值对将不会反转。
如果同一个值出现了多次，则最后一个键名将作为它的值，所有其它的都丢失了。
$trans = array("a" => 1, "b" => 1, "c" => 2);
$trans = array_flip($trans);
print_r($trans);
输出：
Array
(
    [1] => b
    [2] => c
)

## 在数组中搜索给定的值，如果成功则返回相应的键名 `array_search`
mixed array_search ( mixed $needle , array $haystack [, bool $strict = false ] )
$array = array(0 => 'blue', 1 => 'red', 2 => 'green', 3 => 'red');
$key = array_search('green', $array); // $key = 2;
$key = array_search('red', $array);   // $key = 1;

## 用回调函数过滤数组中的单元 `array_filter`
array array_filter ( array $input [, callable $callback = "" ] )
function odd($var){	// 奇数
    return($var & 1);
}

function even($var){	// 偶数
    return(!($var & 1));
}

$array1 = array("a"=>1, "b"=>2, "c"=>3, "d"=>4, "e"=>5);
$array2 = array(6, 7, 8, 9, 10, 11, 12);

echo "Odd :\n";
print_r(array_filter($array1, "odd"));
echo "Even:\n";
print_r(array_filter($array2, "even"));
输出：
Odd :
Array
(
    [a] => 1
    [c] => 3
    [e] => 5
)
Even:
Array
(
    [0] => 6
    [2] => 8
    [4] => 10
    [6] => 12
)

## 找出最大值 `max`
mixed max ( array $values )
mixed max ( mixed $value1 , mixed $value2 [, mixed $... ] )

## 找出最小值 `min`
mixed min ( array $values )
mixed min ( mixed $value1 , mixed $value2 [, mixed $... ] )
max、min都无法得出最大或最小元素的索引。

## 对数组排序 `sort`
bool sort ( array &$array [, int $sort_flags = SORT_REGULAR ] )
不能保留元素之间的键值关系，元素会重新从0开始向上索引。

## 对数组进行排序并保持索引关系 asort
bool asort ( array &$array [, int $sort_flags = SORT_REGULAR ] )
$fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");
sort($fruits);

foreach ($fruits as $key => $val) {
    echo "$key = $val\n";
}
输出：0 = apple 1 = banana 2 = lemon 3 = orange
如果使用asort，则输出：
c = apple b = banana d = lemon a = orange

## 返回一个单元顺序相反的数组 `array_reverse`
array array_reverse ( array $array [, bool $preserve_keys = false ] )
preserve_keys如果设置为 TRUE 会保留数字的键。 非数字的键则不受这个设置的影响，总是会被保留。

## 用“自然排序”算法对数组排序 `natsort`
bool natsort ( array &$array )
$array1 = $array2 = array("img12.png", "img10.png", "img2.png", "img1.png");
asort($array1);
echo "Standard sorting\n";
print_r($array1);

natsort($array2);
echo "\nNatural order sorting\n";
print_r($array2);

输出：
Standard sorting
Array
(
    [3] => img1.png
    [1] => img10.png
    [0] => img12.png
    [2] => img2.png
)

Natural order sorting
Array
(
    [3] => img1.png
    [2] => img2.png
    [1] => img10.png
    [0] => img12.png
)

## 用“自然排序”算法对数组进行不区分大小写字母的排序 `natcasesort`
bool natcasesort ( array &$array )

## 对数组逆向排序 `rsort`
bool rsort ( array &$array [, int $sort_flags = SORT_REGULAR ] )

## 对数组进行逆向排序并保持索引关系 `arsort`
bool arsort ( array &$array [, int $sort_flags = SORT_REGULAR ] )

## 使用用户自定义的比较函数对数组中的值进行排序 `usort`
bool usort ( array &$array , callable $cmp_function )
function cmp($a, $b){
    if ($a == $b) {
        return 0;
    }
    return ($b < $a) ? 1 : -1;
}

$a = array(3, 2, 5, 6, 1);

usort($a, "cmp");

print_r($a);	// Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 5 [4] => 6 )

## 使用自然排序算法比较字符串 `strnatcmp`
int strnatcmp ( string $str1 , string $str2 )

## 对多个数组或多维数组进行排序 `array_multisort`
bool array_multisort ( array &$arr [, mixed $arg = SORT_ASC [, mixed $arg = SORT_REGULAR [, mixed $... ]]] )
$ar1 = array(10, 100, 100, 0);
$ar2 = array(1, 3, 2, 4);
array_multisort($ar1, $ar2);

var_dump($ar1);
var_dump($ar2);
输出：
array(4) {
  [0]=> int(0)
  [1]=> int(10)
  [2]=> int(100)
  [3]=> int(100)
}
array(4) {
  [0]=> int(4)
  [1]=> int(1)
  [2]=> int(2)
  [3]=> int(3)
}

## 将数组打乱 `shuffle`
bool shuffle ( array &$array )

## 移除数组中重复的值 `array_unique`
array array_unique ( array $array [, int $sort_flags = SORT_STRING ] )
$input = array("a" => "green", "red", "b" => "green", "blue", "red");
$result = array_unique($input);
print_r($result);
输出：
Array
(
    [a] => green
    [0] => red
    [1] => blue
)

## 返回数组中部分的或所有的键名 `array_keys`
array array_keys ( array $array [, mixed $search_value [, bool $strict = false ]] )
如果指定了可选参数 search_value，则只返回该值的键名。否则 input 数组中的所有键名都会被返回。

## 使用用户自定义函数对数组中的每个元素做回调处理 `array_walk`
bool array_walk ( array &$array , callable $funcname [, mixed $userdata = NULL ] )
array_walk() 不会受到 array 内部数组指针的影响。array_walk() 会遍历整个数组而不管指针的位置。
$fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");

function test_alter(&$item1, $key, $prefix){
    $item1 = "$prefix: $item1";
}

function test_print($item2, $key){
    echo "$key. $item2<br />\n";
}

echo "Before ...:\n";
array_walk($fruits, 'test_print');

array_walk($fruits, 'test_alter', 'fruit');
echo "... and after:\n";

array_walk($fruits, 'test_print');
输出：
Before ...:
d. lemon
a. orange
b. banana
c. apple
... and after:
d. fruit: lemon
a. fruit: orange
b. fruit: banana
c. fruit: apple

## 对数组中的每个成员递归地应用用户函数 `array_walk_recursive`
bool array_walk_recursive ( array &$input , callable $funcname [, mixed $userdata = NULL ] )

## 计算数组的交集 `array_intersect`
array array_intersect ( array $array1 , array $array2 [, array $ ... ] )
$array1 = array("a" => "green", "red", "blue");
$array2 = array("b" => "green", "yellow", "red");
$result = array_intersect($array1, $array2);
print_r($result);
输出：
Array
(
    [a] => green
    [0] => red
)

## 计算数组的差集 `array_diff`
array array_diff ( array $array1 , array $array2 [, array $... ] )

## 检查一个变量是否为空 `empty`
bool empty ( mixed $var )
当一个变量并不存在，或者它的值等同于FALSE，那么它会被认为不存在。如果变量不存在的话，empty()并不会产生警告。

## 产生一个可存储的值的表示 `serialize`
string serialize ( mixed $value )
返回字符串，此字符串包含了表示 value 的字节流，可以存储于任何地方

## 从已存储的表示中创建 PHP 的值 `unserialize`
mixed unserialize ( string $str )
对单一的已序列化的变量进行操作，将其转换回 PHP 的值。

## 打印变量的相关信息 `var_dump`
void var_dump ( mixed $expression [, mixed $... ] )

## 打印关于变量的易于理解的信息 `print_r`
bool print_r ( mixed $expression [, bool $return ] )

## 输出或返回一个变量的字符串表示 `var_export` 
mixed var_export ( mixed $expression [, bool $return ] )

## 打开输出控制缓冲 `ob_start`
bool ob_start ([ callback $output_callback [, int $chunk_size [, bool $erase ]]] )
当输出缓冲激活后，脚本将不会输出内容（除http标头外），相反需要输出的内容被存储在内部缓冲区中。

## 返回输出缓冲区的内容 `ob_get_contents`
string ob_get_contents ( void )
只是得到输出缓冲区的内容，但不清除它。
ob_start();
echo "Hello ";
$out1 = ob_get_contents();
echo "World";
$out2 = ob_get_contents();
ob_end_clean();
var_dump($out1, $out2);
输出：
	string(6) "Hello "
string(11) "Hello World"

## 冲刷出（送出）输出缓冲区内容并关闭缓冲 `ob_end_flush`
bool ob_end_flush ( void )

## 清空（擦除）缓冲区并关闭输出缓冲 `ob_end_clean`
bool ob_end_clean ( void )

## 获取传递给函数的参数的个数 `func_num_args`
int func_num_args ( void )

## 返回函数参数列表的某一项 `func_get_arg`
mixed func_get_arg ( int $arg_num )

## 返回一个包含函数参数列表的数组 `func_get_args`
array func_get_args ( void )

## 把第一个参数作为回调函数调用 `call_user_func`
mixed call_user_func ( callable $callback [, mixed $parameter [, mixed $... ]] )
注意，传入call_user_func()的参数不能为引用传递。可以使用call_user_func_array来实现传引用。

## 调用回调函数，并把一个数组参数作为回调函数的参数 `call_user_func_array`
mixed call_user_func_array ( callable $callback , array $param_arr )
error_reporting(E_ALL);
function increment(&$var){
    $var++;
}

$a = 0;
call_user_func('increment', $a);
echo $a."\n";		//	Warning: Parameter 1 to increment() expected to be a reference, value given in ...

call_user_func_array('increment', array(&$a));
echo $a."\n";		// 	1

## 创建一个lambda-style的匿名内部函数 `create_function`
string create_function ( string $args , string $code )
$newfunc = create_function('$a,$b', 'return "ln($a) + ln($b) = " . log($a * $b);');
echo "New anonymous function: $newfunc\n";
echo $newfunc(2, M_E) . "\n";
输出：
New anonymous function: lambda_1 
ln(2) + ln(2.718281828459) = 1.6931471805599

## 返回类或接口所实现的接口列表 `class_implements`
array class_implements ( mixed $class [, bool $autoload = true ] )

## 尝试加载未定义的类 `__autoload`
void __autoload ( string $class )

## 返回对象或类的父类名 `get_parent_class`
string get_parent_class ([ mixed $obj ] )

## 返回文件路径的信息 `pathinfo`
mixed pathinfo ( string $path [, int $options = PATHINFO_DIRNAME | PATHINFO_BASENAME | PATHINFO_EXTENSION | PATHINFO_FILENAME ] )
$path_parts = pathinfo('/www/htdocs/inc/lib.inc.php');
echo $path_parts['dirname'], "\n";
echo $path_parts['basename'], "\n";
echo $path_parts['extension'], "\n";
echo $path_parts['filename'], "\n"; // since PHP 5.2.0
输出：
	/www/htdocs/inc
lib.inc.php
php
lib.inc

## 检测变量是否为资源类型 `is_resource`
bool is_resource ( mixed $var )

## 检查类是否已定义 `class_exists`
bool class_exists ( string $class_name [, bool $autoload = true ] )

## 返回由已定义类的名字所组成的数组 `get_declared_classes`
array get_declared_classes ( void )

## 返回所有已定义的函数 `get_defined_functions`
array get_defined_functions ( void )

## 设置Cookie `setcookie`
bool setcookie ( string $name [, string $value = "" [, int $expire = 0 [, string $path = "" [, string $domain = "" [, bool $secure = false [, bool $httponly = false ]]]]]] )

## 生成 URL-encode 之后的请求字符串 `http_build_query`
string http_build_query ( mixed $query_data [, string $numeric_prefix [, string $arg_separator [, int $enc_type = PHP_QUERY_RFC1738 ]]] )

## 将整个文件读入一个字符串 `file_get_contents`
string file_get_contents ( string $filename [, bool $use_include_path = false [, resource $context [, int $offset = -1 [, int $maxlen ]]]] )
$homepage = file_get_contents('http://www.example.com/');
$file = file_get_contents('./people.txt', FILE_USE_INCLUDE_PATH);

## 读取文件（可安全用于二进制文件） `fread`
string fread ( resource $handle , int $length )
从文件指针 handle 读取最多 length 个字节。

## 获取或者设置HTTP状态码 `http_response_code`
int http_response_code ([ int $response_code ] )
<?php
http_response_code(404);
?>

## 计算字符串的 MD5 散列值 `md5`
string md5 ( string $str [, bool $raw_output = false ] )

##  获取全部 HTTP 请求头信息 `getallheaders`
array getallheaders ( void )

## 刷新输出缓冲 `flush`
void flush ( void )
flush() 函数不会对服务器或客户端浏览器的缓存模式产生影响。因此，必须同时使用 ob_flush() 和flush() 函数来刷新输出缓冲。

## 获取一个环境变量的值 `getenv`
string getenv ( string $varname )
$ip = getenv('REMOTE_ADDR');

## 设置环境变量的值 `putenv`
bool putenv ( string $setting )
仅存活于当前请求期间。 在请求结束时环境会恢复到初始状态。

## 获取浏览器具有的功能 `get_browser`
mixed get_browser ([ string $user_agent [, bool $return_array = false ]] )

## 检查特定类型的某个变量是否存在 `filter_has_var`
bool filter_has_var ( int $type , string $variable_name )
可用类型：INPUT_GET, INPUT_POST, INPUT_COOKIE, INPUT_SERVER, or INPUT_ENV

## 通过名称获取特定的外部变量，并且可以通过过滤器处理它 `filter_input`
mixed filter_input ( int $type , string $variable_name [, int $filter = FILTER_DEFAULT [, mixed $options ]] )

## 将字符串转换为HTML实体 `htmlspecialchars`
string htmlspecialchars ( string $string [, int $flags = ENT_COMPAT | ENT_HTML401 [, string $encoding = ini_get("default_charset") [, bool $double_encode = true ]]] )

## 将上传的文件移动到新位置 `move_uploaded_file`
bool move_uploaded_file ( string $filename , string $destination )

## 生成一个唯一ID `uniqid`
string uniqid ([ string $prefix = "" [, bool $more_entropy = false ]] )
获取一个带前缀、基于当前时间微秒数的唯一ID

## 启动新会话或者重用现有会话 `session_start`
bool session_start ([ array $options = [] ] )

## 读取/设置会话名称 `session_name`
string session_name ([ string $name ] )

## 读取/设置当前会话的保存路径 `session_save_path`
string session_save_path ([ string $path ] )

## 添加URL重写器的值 `output_add_rewrite_var` 
bool output_add_rewrite_var ( string $name , string $value )
绝对URL(http://example.com/..)不能被重写。
output_add_rewrite_var('var', 'value');
echo '<a href="file.php">link</a>
<a href="http://example.com">link2</a>';
echo '<form action="script.php" method="post">
<input type="text" name="var2" />
</form>';
输出：
	<a href="file.php?var=value">link</a>
<a href="http://example.com">link2</a>

<form action="script.php" method="post">
<input type="hidden" name="var" value="value" />
<input type="text" name="var2" />
</form>

## 设置用户自定义会话存储函数 `session_set_save_handler`
bool session_set_save_handler ( SessionHandlerInterface $sessionhandler [, bool $register_shutdown = true ] )

## 从字符串中去除 HTML 和 PHP 标记 `strip_tags`
string strip_tags ( string $str [, string $allowable_tags ] )
$text = '<p>Test paragraph.</p><!-- Comment --> <a href="#fragment">Other text</a>';
echo strip_tags($text);
echo "\n";
echo strip_tags($text, '<p><a>');		// 允许 <p> 和 <a>
输出：
Test paragraph. Other text
<p>Test paragraph.</p> <a href="#fragment">Other text</a>

## 给一个流添加一个过滤器 `stream_filter_append`
resource stream_filter_append ( resource $stream , string $filtername [, int $read_write [, mixed $params ]] )

## 将一个XML文件解析为对象 `simplexml_load_file`
SimpleXMLElement simplexml_load_file ( string $filename [, string $class_name = "SimpleXMLElement" [, int $options = 0 [, string $ns = "" [, bool $is_prefix = false ]]]] )

## 创建资源流上下文 `stream_context_create`
resource stream_context_create ([ array $options [, array $params ]] )
$opts = array(
  'http'=>array(
    'method'=>"GET",
    'header'=>"Accept-language: en\r\n" .
              "Cookie: foo=bar\r\n"
  )
);

$context = stream_context_create($opts);
$fp = fopen('http://www.example.com', 'r', false, $context);
fpassthru($fp);
fclose($fp);

## 输出文件指针处的所有剩余数据 `fpassthru`
int fpassthru ( resource $handle )

## 初始化一个cURL会话 `curl_init`
resource curl_init ([ string $url = NULL ] )

## 设置一个cURL传输选项 `curl_setopt`
bool curl_setopt ( resource $ch , int $option , mixed $value )

## 执行一个cURL会话 `curl_exec`
mixed curl_exec ( resource $ch )

## 关闭一个cURL会话 `curl_close`
void curl_close ( resource $ch )
$ch = curl_init();	// 创建一个新cURL资源

// 设置URL和相应的选项
curl_setopt($ch, CURLOPT_URL, "http://www.example.com/");
curl_setopt($ch, CURLOPT_HEADER, 0);

curl_exec($ch);	// 抓取URL并把它传递给浏览器
curl_close($ch);	// 关闭cURL资源，并且释放系统资源

## 设置一个流的超时时间 `stream_set_timeout`
bool stream_set_timeout ( resource $stream , int $seconds [, int $microseconds = 0 ] )

## 更新现有的会话ID `session_regenerate_id`
bool session_regenerate_id ([ bool $delete_old_session = false ] )

## 获取一个php配置 `ini_get`
string ini_get ( string $varname )

## 获取所有php配置 `ini_get_all`
array ini_get_all ([ string $extension [, bool $details = true ]] )

## 设置php报错级别 `error_reporting`
int error_reporting ([ int $level ] )

## 设置一个用户自定义的错误处理函数 `set_error_handler`
mixed set_error_handler ( callable $error_handler [, int $error_types = E_ALL | E_STRICT ] )






