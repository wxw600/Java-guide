# JAVA算法题之字符串处理
### 一、找出字符串中出现最多的字符
###### 对于这个题目我给大家提供两个思路，一个就是常规的思路双重for循环遍历，另一个就是利用map的特性 。
###### 思路一：时间复杂度高n^2,遍历n*n次
```java
public class Stringways {
    @Test//出现最多的字符
    public void test1() {
        String str = "fbasfbaihsbbdiasaaaa";
        //统计每一个字符出现了多少次
        char res = str.charAt(0);
        int max = 0;//最多出现了多少次
        for (int i = 0; i < str.length(); i++) {
            char temp = str.charAt(i);
            int count = 0;
            for (int j = 0; j < str.length(); j++) {
                if (str.charAt(j) == temp) {
                    count++;//计算temp出现了多少次
                }
            }
            if (count >= max) {
                    max = count;
                    res = temp;
                }
        }
        System.out.println(res + "出现次数最多：" + max);
    }
   ```
###### 思路二：有关次数的问题我们可以利用map的键值对特性（值，次数），想法的话也不是特别难。
  ```java
  @Test
    public void test2() {//统计每个字符出现的字数，出现最多的，出现最少的。
        String str = "fbasfbaihsbbdiasaaaa";
        char res = str.charAt(0);
        int max = 0;//最多出现了多少次
        Map<Character, Integer> map = new HashMap<>();
        for (int i = 0; i < str.length(); i++) {
            char c =  str.charAt(i);
            Integer count = map.get(c);
            if(count == null ){
                //字符没有出现过
                count = 1;
            }else {
                count++;
            }
            map.put(c, count);
            if (count>max){
                res = c;
                max = count;
            }
        }
        System.out.println(res + "出现次数最多：" + max);
    }
 ```
 ##### 统计每个字符出现的字数，出现最多的，出现最少的都可以使用这种方法，不一一举例，理解思路。
  
 ### 二、找出字符串中第一次重复出现的字符。
 ######  对于这个题目大家可以从<font color=#0033FF >重复</font>入手，在集合框架Set集合的特点：一个无序（存入和取出顺序有可能不一致）容器，<font color=#0033FF >不可以存储重复元素</font>，只允许存入一个null元素，必须保证元素唯一性。（推荐）
  ```java
   public void test2_1(){
        String str = "asdngfjssd";
        Set<Character> set = new HashSet<>();
        for (int i = 0; i < str.length(); i++) {
            if(!set.add(str.charAt(i))){
                //set里的add方法返回boolean类型值，
                // 没有重复set返回true证明没有 重复了set返回false已经存在了
                System.out.println(str.charAt(i));
                break;
            }
        }
    }
  ```


 
  ######  题目也可以用map实现，Hashset底层其实使用Hashmap实现，我们只需要保证不出现重复的键名就可以，map.containsKey()方法判断是否包含指定的键名。
  ### 三、找出字符串中第一个只出现一次的字符。
  ######  思路一：考虑使用map，统计每个字符出现的次数，然后从前到后遍历找到第一个次数为一的字符就是所要找的。
   ```java
  @Test
    public void test3(){
        String str = "asdnfsdfssd";
        Map<Character, Integer> map = new HashMap<>();
        for (int i = 0; i < str.length(); i++) {
            char c =  str.charAt(i);
            Integer count = map.get(c);
            if(count == null ){
                //字符没有出现过
                count = 1;
            }else {
                count++;
            }
            map.put(c, count);
        }
        for (int i = 0; i < str.length(); i++) {
            if (map.get(str.charAt(i))==1){
                System.out.println(str.charAt(i));
                break;
            }
        }
    }
  ```
  ######  思路二：指定字符第一次出现的位置和最后一次出现的位置一样就是第一个只出现一次的字符，使用string给我们提供的api，indexOf()方法：查找一个字符串中，第一次出现指定字符串的位置。lastIndexOf()方法：返回指定字符在此字符串中最后一次出现处的索引。
  ```java
  @Test
    public void test3_2(){
        String str = "asdnfsdfssd";
        for (int i = 0; i < str.length(); i++) {
            char m =  str.charAt(i);
            if(str.indexOf(m)==str.lastIndexOf(m)){
                System.out.println(m);
                break;
            }
        }
    }      
```
#### 今天就分享到这里，之后会继续更新，路过的小伙伴感觉不错的可以帮忙点赞，有其他想法的同学可以评论留言互相交流，本人看到后会第一时间回复的。
