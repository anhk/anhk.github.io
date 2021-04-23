---
layout: post
title: 随机密码工具
category: 随机密码
---

随机密码工具
<!--more-->

<div>
	<input type="checkbox" id="english" name="english" checked/>
	<label for="alpha"> 小写字母： a-z </label><br>
	<input type="checkbox" id="ENGLISH" name="ENGLISH" checked/>
	<label for="alpha"> 大写字母： A-Z </label><br>
	<input type="checkbox" id="num" name="num" checked/>
	<label for="alpha"> 数字： 0-9 </label><br>
	<input type="checkbox" id="special" name="special" value="true" />
	<label for="alpha"> 特殊字符： -_!@#$%^&*() </label><br>
	<input type="button" value="生成密码" onclick="RandomPassword()" /><br>
	<label> 密码是： </label><label id="output"></label><br>
	<script type="text/javascript">
		function getOne(arr) {
			return arr[Math.floor(Math.random()*arr.length)];
		}
		function RandomPassword(){
			var num = ["0","1","2","3","4","5","6","7","8","9"];
			var english = ["a","b","c","d","e","f","g","h","i","j","k","l","m","n","o","p","q","r","s","t","u","v","w","x","y","z"];
			var ENGLISH = ["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"];
			var special = ["-","_","!","@","#","$","%","^","&","*","(",")"];
			var alpha = [];
			var arr = [];
			if (document.getElementById('num').checked == true) {
				alpha = alpha.concat(num);
				arr.push(getOne(num));
			}
			if (document.getElementById('english').checked == true) {
				alpha = alpha.concat(english);
				arr.push(getOne(english));
			}

			if (document.getElementById('ENGLISH').checked == true) {
				alpha = alpha.concat(ENGLISH);
				arr.push(getOne(ENGLISH));
			}
			if (document.getElementById('special').checked == true) {
				alpha = alpha.concat(special);
				arr.push(getOne(special));
			}
			for(var i=4; i<16; i++){
				arr.push(alpha[Math.floor(Math.random()*alpha.length)]);
			}
			var newArr = [];
			for(var j=0; j<16; j++){
				newArr.push(arr.splice(Math.random()*arr.length,1)[0]);
			}
			document.getElementById('output').innerHTML=newArr.join("")
		}
	</script>
</div>
