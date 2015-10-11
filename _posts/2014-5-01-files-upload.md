---
layout: post
title: php文件上传
category: 读书笔记
tags: 文件上传
---


###什么是文件上传

把`浏览器`所在电脑的文件存放到`服务器`上, 这个过程称之为文件上传

上传分为两个部分:
 
- 将浏览器所在电脑的文件选中,提交给服务器,服务器要接收文件.

- 服务器必须要接收文件: apache不能接收文件,PHP负责接收.

###前期准备

- 设置php.ini

- 确保上传功能被打开


    file_uploads = On


- 设置上传文件的临时存放目录


    upload_tmp_dir ="E:\PHP\upload"  #;设置文件上传临时保存路径,默认是在c:/windows/temp


- 设置文件表单域


    <input type=’file’ name=’myfile’>


- 设置post提交二进制

	`enctype=”multipart/form-data”`

####`$_FILES获取的数据`
{% highlight php startinline linenos %} 
array(1) {
  ["myfile"]=>
  array(5) {
    ["name"]=>											#上传文件的文件名
    string(10) "123123.jpg"
    ["type"]=>											#上传文件的类型:大类型/格式  称为MIME类型
    string(10) "image/jpeg"
    ["tmp_name"]=>										#上传到服务器时临时存储文件路径
    string(38) "E:\PHP\upload\php7FC2.tmp"
    ["error"]=>											#错误描述码
    int(0)
    ["size"]=>											#上传文件大小
    int(6340)
  }
}
{% endhighlight %}
###使用的函数
- `Move_uploaded_file(文件所在目录[文件名], 文件要存储的目录[带文件名])`移动文件-->原文件不存在
- `Copy(文件所在目录,文件要存储的目录`复制文件-->原文件存在

###使用时注意的问题
- `move_uploaded_file()`会事先检测源文件是否通过php上传过来的，否则不予移动(`只能用post`)
- `move_uploaded_file()`这个函数虽然`多次调用但是只能执行一次`，原因就在于`move_uploaded_file()`只支持post提交的信息，也就是用户只提交了一个post，第一次调用被执行后，之后就不是post的值也就是说不在执行了，解决办法就是`用copy代替move_uploaded_file()`

###封装一个上传文件函数
{% highlight php startinline linenos %} 
	/*
	 * 上传文件
	 * @param1 array $file,要上传的文件信息
	 * @param2 string $path,文件要上传的路径
	 * @param3 string &$error,记录错误信息
	 * @return string 文件的名字(新名字)
	*/
	function upload($file,$path,&$error){
		//定义错误信息: 专门记录错误: 在函数外部能访问
		//验证$file的合法性
		if(!is_array($file)){
			//不是数组
			$error = '上传文件不合法!';
			return false;
		}
		//验证系统错误处理
		switch($file['error']){
			case 1:
				$error = '文件过大,超过服务器允许的大小!';
				return false;
			case 2:
				$error = '文件超过浏览器允许的大小!';
				return false;
			case 3:
				$error = '文件只上传部分!';
				return false;
			case 4:
				$error = '用户没有选中要上传的文件!';
				return false;
			case 6:
			case 7:
				$error = '服务器错误!';
				return false;
		}
		//都没有错误: 移动文件
		$filename = getRandomName($file['name']);
		if(@move_uploaded_file($file['tmp_name'],$path . '/' . $filename))
		{
			//成功
			return $filename;
		}else{
			//失败
			$error = '文件移动失败!';
			return false;
		}
	}
	/*
	 * 生成文件名: YYYYmmddHHIISS + 随机6位字符串
	 * @param1 string $name,文件原始名字: 在浏览器所在电脑的名字
	 * @return string 新的名字
	*/
	function getRandomName($name){
		//构造时间日期部分
		$newname = date('YmdHis');
		//获取随机部分
		$str = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890';
		//随机取6位
		for($i = 0; $i < 6;$i++){
			//每次随机取一位
			$newname .= $str[mt_rand(0,strlen($str) - 1)];
		}
		//构造后缀名
		$newname .= strrchr($name,'.');
		//返回
		return $newname;
	}
{% endhighlight %}