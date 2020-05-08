# JAVA算法之字符串处理（二）
### 一、按字节数截取字符串。

 ######   核心思路：输入一个字符串和字节数，输出按字节数截取字符串， 其中要保证汉字不能被截半个字节，设计思路就是判断汉字占几个字节，通过string.valueof.getBytes()方法获取字符的字节数，之后遍历判断不要超过指定的字节数就可以了



```java
 @Test
    public void test3_3(){
        String str="人MVP生SVP";
        int count = 4;
        int sum = 0;//储存已经截取的字符长度
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < str.length(); i++) {
            //每个字符的字节数
            int len = String.valueOf(str.charAt(i)).getBytes().length;
            if (sum+len<=count){
                sum+=len;
                builder.append(str.charAt(i));
            }else{
                break;
            }
        }
        System.out.println(builder.toString());


    }

   ```
 
扩展：通常一个汉字在计算机中是占两个字节但是在Java中根据编码格式不同所占字节数也是不同的，所以大家不要怀疑运行的结果，不要怀疑自己。（自信点）GBK编码，一个汉字占两个字节。 UTF-16编码，通常汉字占两个字节，CJKV扩展B区、扩展C区、扩展D区中的汉字占四个字节UTF-8编码是变长编码，通常汉字占三个字节，扩展B区以后的汉字占四个字节。        
   
   
### 二、截取目标字符串。



###### 核心思路一：使用string提供的api帮助我们寻找，substring方法截取，并使用indexOf()方法：查找一个字符串中，第一次出现指定字符串的位置。找到索引位置（普及：lastIndexOf()方法：返回指定字符在此字符串中最后一次出现处的索引）
  ```java
@Test
    public void test3_5(){
        //string 里的api substring（）
        String str="<p id=\"text\">wangmouren</p>";
        String recpet="mou";
        int index = str.indexOf(recpet);
        String tar = str.substring(index,index + recpet.length());
        System.out.println(tar);

    }
 ```
 
   ######  核心思路二：使用正则表达式。
  ```java
  @Test
    public void test3_6(){

        String str="<p id=\"text\">wangmouren</p>";
      String res=".*(mou).*";
        Pattern pattern = Pattern.compile(res);
        Matcher matcher = pattern.matcher(str);
        if (matcher.find()) {
            System.out.println(matcher.group(1));
        }
    }
```
Pattern和Matcher是正则表达式对象，将String类型的正则表达式传入Pattern.compile（）中，生成Pattern对象，Pattern对象表示编译后的正则表达式。

Pattern pattern = Pattern.compile("wang"); 传入需要的参数

利用Pattern对象的matcher()生成Matcher对象，利用Matcher对象来进行一系列操作
 Matcher matcher = pattern.matcher("wangmou");参数为用于匹配的字符串
### 三、字符串反转。
 ######  对于这个题目，实现的算法有很多可以自己编写也可以使用自带的工具类。
 ##### 核心思路一：平常的话可以使用工具类，方便我们的操作
  ```java
  @Test
    public void test6(){
        String str="abcdefg";
        StringBuilder builder = new StringBuilder(str);
        System.out.println(builder.reverse());
    }

  ```
 ##### 核心思路二：定义一个空字符串，然后遍历给定的字符串从后面遍历到第一个，然后拼接遍历的字符串。
  ```java
      @Test
  public void test7(){
        String str="abcdefg";
        String fin = "";
        for (int i = str.length()-1; i >=0; i--) {
            fin+= str.charAt(i);
        }
        System.out.println(fin);
    }


  ```
 ##### 核心思路三：首先把字符串转换成数组，然后头尾互换知道换到中间位置停止，字符串就完成了互换。其实就是数组学习的最简单循环互换。
  ```java
   @Test
    public void test8(){
        String str="abcdefg";
       char[] chars = str.toCharArray();
      for (int  i=0;i<chars.length/2;i++){
          char temp = chars[i];
          chars[i] = chars[chars.length-i-1];
          chars[chars.length-i-1] = temp;
      }
        System.out.println(new String(chars));
    }


  ```
### 四、对换句子中单词的位置
  ##### 核心思路:逆向遍历数组，每次遍历都把第i个元素拼接进去，最后注意空格分隔符，最好就是截取一下。
  ```java
   @Test
    public void test8(){
        System.out.println(Exchange("I love you","" ));
    }

    String Exchange(String in,String out) {
        String[] arrs = in.split(out);
        StringBuilder builder = new StringBuilder();
        for (int i = arrs.length - 1; i >= 0; i--) {
            builder.append(arrs[i]);
            builder.append(out);

        }
        return builder.substring(0, builder.length() - out.length());
    }


  ```
#### 今天就分享到这里，之后会继续更新，路过的小伙伴感觉不错的可以帮忙点赞，有其他想法的同学可以评论留言互相交流，本人看到后会第一时间回复的。
