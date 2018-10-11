---
title: PHP根据文件头检测文件类型
date: 2016-10-23 18:30:00
categories:
- PHP
---


文件签名一般都在文件的头部，如果你用十六进制方式查看文件，你就可以看到文件的一些签名信息。如用uestudio以十六进制方式查看zip格式的文件，其文件内容头部有50 4B 03 04这样的十六进制信息。同理jpg文件状况有FF D8 FF E0 xx xx 4A 46这样的十六进制信息，其实这此十六进制都是表示一些特殊字条。

<!-- more -->

php怎么样验证文件类型？先来看一个简单的方法：
```php
<?php
function checkFileType($fileName){  
       $file = fopen($fileName, "rb");  
       $bin = fread($file, 2); //只读2字节  
       fclose($file);  
	// C为无符号整数，网上搜到的都是c，为有符号整数，这样会产生负数判断不正常
       $strInfo  = @unpack("C2chars", $bin);
       $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);  
       $fileType = '';  

	switch( $typeCode )
	{
		case '255216':
			return $typeCode. ' : ' .'jpg';
			break;
		case '7173':
			return $typeCode. ' : ' .'gif';
			break;
		case '13780':
			return $typeCode. ' : ' .'png';
			break;
		case '6677':
			return $typeCode. ' : ' .'bmp';
			break;
		case '7790':
			return $typeCode. ' : ' .'exe';
			break;
		case '7784':
			return $typeCode. ' : ' .'midi';
			break;
		case '8297':
			return $typeCode. ' : ' .'rar';
			break;
		default:
			return $typeCode. ' : ' .'Unknown';
			break;
	}
	//return $typeCode;
   }

$file_name = '11.doc';
echo checkFileType($file_name);
```

下来提供一个类的实现：

```php
<?php
/*通过文件名，获得文件类型*
 *@author chengmo QQ:8292669*
 *@copyright <a href="http://www.cnblogs.com/chengmo">http://www.cnblogs.com/chengmo</a> 2010-10-17
 *@version 0.1
 *$filename="d:/1.png";echo cFileTypeCheck::getFileType($filename); 打印：png
 */
class cFileTypeCheck
{
    private static $_TypeList=array();
    private static $CheckClass=null;
    private function __construct($filename)
    {
        self::$_TypeList=$this->getTypeList();
    }

    /**
     *处理文件类型映射关系表*
     *
     * @param string $filename 文件类型
     * @return string 文件类型，没有找到返回：other
     */
    private function _getFileType($filename)
    {
        $filetype="other";
        if(!file_exists($filename)) throw new Exception("no found file!");
        $file = @fopen($filename,"rb");
        if(!$file) throw new Exception("file refuse!");
        $bin = fread($file, 15); //只读15字节 各个不同文件类型，头信息不一样。
        fclose($file);

        $typelist=self::$_TypeList;
        foreach ($typelist as $v)
        {
            $blen=strlen(pack("H*",$v[0])); //得到文件头标记字节数
            $tbin=substr($bin,0,intval($blen)); ///需要比较文件头长度

            if(strtolower($v[0])==strtolower(array_shift(unpack("H*",$tbin))))
            {
                return $v[1];
            }
        }
        return $filetype;
    }

    /**
     *得到文件头与文件类型映射表*
     *
     * @return array array(array('key',value)...)
     */
    public function getTypeList()
    {
        return array(array("FFD8FFE1","jpg"),
        array("89504E47","png"),
        array("47494638","gif"),
        array("49492A00","tif"),
        array("424D","bmp"),
        array("41433130","dwg"),
        array("38425053","psd"),
        array("7B5C727466","rtf"),
        array("3C3F786D6C","xml"),
        array("68746D6C3E","html"),
        array("44656C69766572792D646174","eml"),
        array("CFAD12FEC5FD746F","dbx"),
        array("2142444E","pst"),
        array("D0CF11E0","xls/doc"),
        array("5374616E64617264204A","mdb"),
        array("FF575043","wpd"),
        array("252150532D41646F6265","eps/ps"),
        array("255044462D312E","pdf"),
        array("E3828596","pwl"),
        array("504B0304","zip"),
        array("52617221","rar"),
        array("57415645","wav"),
        array("41564920","avi"),
        array("2E7261FD","ram"),
        array("2E524D46","rm"),
        array("000001BA","mpg"),
        array("000001B3","mpg"),
        array("6D6F6F76","mov"),
        array("3026B2758E66CF11","asf"),
        array("4D546864","mid"));
    }


    public static function getFileType($filename)
    {
        if(!self::$CheckClass) self::$CheckClass=new self($filename);
        $class=self::$CheckClass;
        return $class->_getFileType($filename);
    }

}

$filename="22.jpg";
echo $filename,"t",cFileTypeCheck::getFileType($filename),"rn";
$filename="11.doc";
echo $filename,"t",cFileTypeCheck::getFileType($filename),"rn";
```
或者可以这么检测：

```php
<?php
$filename = '22.jpg';

$extname = strtolower(substr($filename, strrpos($filename, '.') + 1));
echo $extname.'<br />';
$file = @fopen($filename, 'rb');
    if ($file)
    {
        $str = @fread($file, 0x400); // 读取前 1024 个字节
		echo substr($str, 0, 4);
        @fclose($file);
    }
 	if (substr($str, 0, 4) == 'MThd' && $extname != 'txt')
        {
            $format = 'mid';
        }
        elseif (substr($str, 0, 4) == 'RIFF' && $extname == 'wav')
        {
            $format = 'wav';
        }
        elseif (substr($str ,0, 3) == "/xFF/xD8/xFF")
        {
            $format = 'jpg';
        }
        elseif (substr($str ,0, 4) == 'GIF8' && $extname != 'txt')
        {
            $format = 'gif';
        }
        elseif (substr($str ,0, 8 ) == "/x89/x50/x4E/x47/x0D/x0A/x1A/x0A")
        {
            $format = 'png';
        }
        elseif (substr($str ,0, 2) == 'BM' && $extname != 'txt')
        {
            $format = 'bmp';
        }
        elseif ((substr($str ,0, 3) == 'CWS' || substr($str ,0, 3) == 'FWS') && $extname != 'txt')
        {
            $format = 'swf';
        }
        elseif (substr($str ,0, 4) == "/xD0/xCF/x11/xE0")
        {   // D0CF11E == DOCFILE == Microsoft Office Document
            if (substr($str,0x200,4) == "/xEC/xA5/xC1/x00" || $extname == 'doc')
            {
                $format = 'doc';
            }
            elseif (substr($str,0x200,2) == "/x09/x08" || $extname == 'xls')
            {
                $format = 'xls';
            } elseif (substr($str,0x200,4) == "/xFD/xFF/xFF/xFF" || $extname == 'ppt')
            {
                $format = 'ppt';
            }
        } elseif (substr($str ,0, 4) == "PK/x03/x04")
        {
            $format = 'zip';
        } elseif (substr($str ,0, 4) == 'Rar!' && $extname != 'txt')
        {
            $format = 'rar';
        } elseif (substr($str ,0, 4) == "/x25PDF")
        {
            $format = 'pdf';
        } elseif (substr($str ,0, 3) == "/x30/x82/x0A")
        {
            $format = 'cert';
        } elseif (substr($str ,0, 4) == 'ITSF' && $extname != 'txt')
        {
            $format = 'chm';
        } elseif (substr($str ,0, 4) == "/x2ERMF")
        {
            $format = 'rm';
        } elseif ($extname == 'sql')
        {
            $format = 'sql';
        } elseif ($extname == 'txt')
        {
            $format = 'txt';
        }

		echo $format;
```


[转自：http://www.nowamagic.net/librarys/veda/detail/836](http://www.nowamagic.net/librarys/veda/detail/836)
