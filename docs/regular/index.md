<div align='center' ><font size='5'>常用正则表达式</font></div>

​	1:校验金额是否正确

 ```java
/*
*只能由数字和小数点组成
*不能为负数
*小数点后最多两位
*整数部分两位以上时首位不能为 0
*/
import java.util.regex.*;
class RegexExample1{
   public static void main(String[] args){
      String content = "00.1";
      String regex = "(^[1-9]([0-9]+)?(\.[0-9]{1,2})?$)|(^(0){1}$)|(^[0-9]\.[0-9]([0-9])?$)";
 
      boolean isMatch = Pattern.matches(regex, content);
      System.out.println(isMatch);//false
   }
}
 ```

