View source chúng ta sẽ có code:
```
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
    $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
    if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
        $mydata['showpassword'] = $tempdata['showpassword'];
        $mydata['bgcolor'] = $tempdata['bgcolor'];
        }
    }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}

$data = loadData($defaultdata);

if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}

saveData($data);



?>

```
...
```
<?
if($data["showpassword"] == "yes") {
    print "The password for natas12 is <censored><br>";
}

?>
```

Để print được password thì `($data["showpassword"] == "yes" 

Theo đoạn code thì `$data = loadData($defaultdata);`

Cùng nhìn vào fucntion `$loadData` thì ta thấy giá data sẽ bị phụ thuộc vào một biến `$templete`, giá trị của `$templete` sẽ được tính bằng cách 
`base64_decode($_COOKIE["data"])`, sau đó thực hiện xor với `$key` bằng fucntion `or_encrypt` sau đó thực hiện json_decode giá trị trên. 

Việc chúng ta cần làm bây giờ là phải thay đổi giá trị trong biến `$templete` sao cho `$templete['showpassword']="yes"` từ đó tạo ra một cookie mới gửi lên 
thì chúng ta có thể lấy được password.

Muốn làm được như vậy thì chúng ta cần biết được `$key` mà phép xor sử dụng.

Áp dụng tính chất của phép XOR: `A Xor B = C => A Xor C = B => B Xor C = A`, chúng ta sẽ tìm được `$key`:
```
<?php

function xor_encrypt($in) {
    $key = base64_decode('ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=');
    $text = $in;
    $outText = '';
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$json = json_encode(array('showpassword' =>'no', 'bgcolor' => '#ffffff'), true);

$key = (xor_encrypt($json));

echo $key;
```
Kết quả:
```
qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq
```
=> $key = 'qw8J'

Cuối cùng solve code tìm cookie:
```
<?php

function xor_encrypt($in) {
    $key = 'qw8J';
    $text = $in;
    $outText = '';
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$templete = json_encode(array('showpassword' => 'yes' ,'bgcolor' => '#ffffff' ));

$cookie = base64_encode(xor_encrypt($templete));
echo $cookie;
?>
```
Kết quả:
```
ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK
```
Thay đổi cookie data thành giá trị cookie mới ta sẽ được password cho level 12:

![](/Image/img.png)

## Thank you for your reading!
# End.
