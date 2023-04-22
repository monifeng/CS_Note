# LeetCode刷题笔记

> *总有一天你将破蛹而出，成长得比人们期待的还要美丽。*
> *但这个过程会很痛，会很辛苦，有时候还会觉得灰心。*
> *面对着汹涌而来的现实，觉得自己渺小无力。*
> 	*但这，也是生命的一部分。做好现在你能做的，然后，一切都会好的。*
> *我们都将孤独地长大，不要害怕。*

## 1. 数组类

#### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

 

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**示例 2：**

```
输入：nums = [3,2,4], target = 6
输出：[1,2]
```

**示例 3：**

```
输入：nums = [3,3], target = 6
输出：[0,1]
```

 

**提示：**

- `2 <= nums.length <= 104`
- `-109 <= nums[i] <= 109`
- `-109 <= target <= 109`
- **只会存在一个有效答案**

**进阶：**你可以想出一个时间复杂度小于 `O(n2)` 的算法吗？

我的第一次题解：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        //Moni 2022/2/25
        int i = 0;
        int j = 1;
        int[] result = new int[2];
        loop: for (i = 0; i < nums.length; i++){
            for (j = 1; j < nums.length; j++){
                if ((nums[i] + nums[j]) == target && i != j){
                    result[0] = i;
                    result[1] = j;
                    break loop;
                }
            }
        }
        return result;
    }
}
```

![image-20220225232645346](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20220225232645346.png)

基本思路：嵌套循环，暴力破解，但是时间复杂度达到了O（n^2）

**标准答案的暴力解法：**

同样使用了嵌套循环，但是并不用单独定义两个数组，直接返回，并且不用往回寻找，其中关键是： j = i + 1这一行。

看上去赏心悦目，并且十分好用

```java
int n = nums.length;
for (int i = 0; i < n; ++i) {
    for (int j = i + 1; j < n; ++j) {
        if (nums[i] + nums[j] == target) {
            return new int[]{i, j};
        }
    }
}

return new int[0];
```





解法一：首先联想到，两数之和不用往回寻找，即1+2 = 2+1；所以**i**作为第一个标记，可以直接往后寻找，然后联想到，不用每次都做加法，只需要一次减法并记录，然后直接与第二个数比较。

```java
public class TwoSum {
    public static int[] twoSum(int[] nums, int target) {
        //Moni 2022/2/25
        int i = 0;
        int j = 0;
        int num1 = 0;
        int[] result = new int[2];

        loop: for (i = 0; i < nums.length - 1; i++) {
            num1 = target - nums[i];
            j = i + 1;
            while (j < nums.length){
                if (nums[j] == num1){
                    result[0] = i;
                    result[1] = j;
                    break loop;
                }else j++;
            }
        }
        return result;
    }

    public static void main(String[] args) {
        int[] nums = {3, 2, 4};
        int[] result = twoSum(nums, 6);
        for (int i = 0; i < result.length; i++) {
            System.out.println(result[i]);
        }
    }
}

```

解法二：哈希图解法：

基本思路，暴力解法时间复杂度为O（n^2），慢就慢在要比较target - x的值这一步，如果可以直接查找，就能缩短时间，查找想到了哈希图，一个键有唯一的哈希值，键值对的形式，只要构建好哈希图，直接用containsKey就可以查找出唯一的索引。

```java
import java.util.HashMap;

public class TwoSum {
    public static int[] twoSum(int[] nums, int target) {
        HashMap<Integer,Integer> sites = new HashMap<Integer,Integer>();
        for (int i = 0; i < nums.length; i++) {
            int n = target - nums[i];
            sites.put(nums[i],i);
            if (sites.containsKey(n)){
                return new int[]{i, sites.get(target - nums[i])};
            }
        }

        return new int[0];
    }

```

注意这里会报错，原因是例如，target值为6，第一个元素为3时，由于先将元素压进了哈希图，所以会直接找到哈希。即i = index = 0；所以解决方案有两种，第一种是先用一次循环将全部元素压进哈希图中，再用一个循环遍历数组，在哈希表中查找target - x。第二种是，先查找一次，然后再压入哈希图，这样就避免了重复使用第一个元素**（？）**

```java
public class TwoSum {
    public static int[] twoSum(int[] nums, int target) {
        HashMap<Integer,Integer> sites = new HashMap<Integer,Integer>();
        for (int i = 0; i < nums.length; i++) {
            sites.put(nums[i],i);
        }
        for (int i = 0; i < nums.length; i++) {
            int n = target - nums[i];
            //注意，另一个下标不能等于i
            if(sites.containsKey(n)  && sites.get(target - nums[i]) != i){
                return new int[]{i, sites.get(n)};
            }
        }

        return new int[0];
    }
```

我更偏爱第一种解决方式，便于理解。



**总结**

查找最快的办法是构建哈希图，然后用哈希图的特性，键值对一一对应，并且利用了hash方法，一个键有唯一的下标，这也是快速找到containsKey的原因。时间复杂度大大降低了。



## 2. 字符串

### 9. [回文数](https://leetcode-cn.com/problems/palindrome-number/)

**第一次解答：**

```java
public static boolean isPalindrome(int x) {
        //moni 2022/2/26
        //回文数一定是正数
        int count = 0;
        int num = x;
        while (num > 0){
            num /= 10;
            count++;
        }
        int[] nums = new int[count];
        if (x < 0){
            return false;
        }
        if (x < 10){
            return true;
        }
        for(int i = 0; i < count; i++){
            nums[i] = (int) (x / Math.pow(10,i) % 10);
        }
        for(int i = 0; i < count; i++){
            if (nums[i] != nums[count -1 -i]){
                return false;
            }
        }

        return true;
    }
```

基本思路，计算出位数（计算位数使用的是整型除法的性质，当除以10的位数次方时，该数变成0，同时我的理解是，需要一个数来存储原来x的值）并且存入数组中，在数组中我使用了补数的性质，当下标为补数时，若是回文数，则应该相等，并且以此做判断，不相等时返回false。同时注意，个位数一定是回文数，而负数一定不是回文数。



**解法二：**

```java
class Solution {
    public boolean isPalindrome(int x) {
        //moni 2022/2/27
        String str = Integer.toString(x);
        //用StringBuilder需要用toString（）转回字符串
         String str1 = new StringBuilder(str).reverse().toString();
         if (str.equals(str1)){
             return true;
         }

         return false;
    }
}
```

利用字符串类型，先转换为字符串，方法为Intger的toString方法，然后转换转换为StringBuilder（原因：String无法更改，而StringBuffer虽然线程安全，但是略微慢一点），再使用StringBuilder的reverse（）方法和toString方法，再与原字符串进行比较，如果相等，则判断为true，反之则为false。

> 其中探索了一条弯路，就是尝试将String类型转回int类型，但是当反转字符串时，遇到问题，就是如123456789这种数，反转后会报错，因为超出了int类型的范围， 也许可以尝试转换为long类型，但是我觉得这样可能反而浪费内存。



官方解法一：整型半转

```
class Solution {
    public boolean isPalindrome(int x) {
        int reNum = 0;
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }
        
        while (reNum < x){
            reNum = reNum * 10 + x % 10;
            x /= 10;
        }
        return reNum == x || reNum / 10 == x;
    }
}
```

思路：如果全部反转可能会溢出，那就只转一半，比较转换后的整型与原来的整型，如果相同，则为回文数。同时值得注意的是，最好先排除边界，这道题比较明显，那就是x<0 或者 x 最后一位为0 的一定不是回文数。具体方法如下：

- 利用 /10 和 %10的方法可以获得每一位数字；
- 利用 *10 和 += 可以得到反转的数字；
- 每次 *10的时候，原数/10；
- 判断到一半的方法就是当反转数字大于或等于缩小原数时，可以判断为到一半了，注意：奇数时，会出现反转数字大于缩小原数的情况，可以直接把反转数字/10，这样不会影响比较结果，因为除掉的那个数字是中间的数，反而排出了比较时的异常结果。

虽然分奇数偶数两种情况，但是并不用讨论， 在返回时，直接使用一个 || 判断，有真为真，全假为假。



### [13. 罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer/)

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
例如， 罗马数字 2 写做 II ，即为两个并列的 1 。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。
给定一个罗马数字，将其转换成整数。

 

示例 1:

输入: s = "III"
输出: 3
示例 2:

输入: s = "IV"
输出: 4
示例 3:

输入: s = "IX"
输出: 9
示例 4:

输入: s = "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
示例 5:

输入: s = "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.


提示：

1 <= s.length <= 15
s 仅含字符 ('I', 'V', 'X', 'L', 'C', 'D', 'M')
题目数据保证 s 是一个有效的罗马数字，且表示整数在范围 [1, 3999] 内
题目所给测试用例皆符合罗马数字书写规则，不会出现跨位等情况。
IL 和 IM 这样的例子并不符合题目要求，49 应该写作 XLIX，999 应该写作 CMXCIX 。
关于罗马数字的详尽书写规则，可以参考 罗马数字 - Mathematics 。



**暴力解法一：**

```java
    public static int romanToInt(String s) {
        int result = 0;
        String[] str = s.split("");
        for (int i = 0; i < str.length; i++) {
            switch (str[i]){
                case "I":
                    if(i+1 < str.length && str[i+1].equals("X")){
                        result += 9;
                        i++;
                    }else if(i+1 < str.length && str[i+1].equals("V")){
                        result += 4;
                        i++;
                }else {result += 1;}
                    break;

                case "X":
                    if(i+1 < str.length && str[i+1].equals("L")){
                        result += 40;
                        i++;
                    }else if(i+1 < str.length && str[i+1].equals("C")){
                        result += 90;
                        i++;
                    }else {result += 10;}
                    break;

                case "C":
                    if(i+1 < str.length && str[i+1].equals("D")){
                        result += 400;
                        i++;
                    }else if(i+1 < str.length && str[i+1].equals("M")){
                        result += 900;
                        i++;
                    }else {result += 100;}
                    break;

                case "V":
                    result += 5;
                    break;

                case "L":
                    result += 50;
                    break;

                case "D":
                    result += 500;
                    break;

                case "M":
                    result += 1000;
                    break;
            }
        }

        return result;
    }
```

解法思路，列出特殊情况，并用switch方法，触发特殊情况时，直接加特殊情况的值，然后使下标+1，跳过下一个数字。非常暴力，但是非常笨重。



**解法二：**

```java
    public static int romanToInt(String s) {
        int result = 0;
        HashMap<Character, Integer> hp = new HashMap<>();
        hp.put('I',1);
        hp.put('V',5);
        hp.put('X',10);
        hp.put('L',50);
        hp.put('C',100);
        hp.put('D',500);
        hp.put('M',1000);

        for (int i = 0; i < s.length(); i++){
            int x = hp.get(s.charAt(i));
            if(i < s.length()-1 && x < hp.get(s.charAt(i+1))){
                result -= x;
            }else {
                result += x;
            }
        }

        return result;
    }
```

思路：利用哈系树，存储键值对。本解法关键在于，当



### [14. 最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix/)

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

 

示例 1：

输入：strs = ["flower","flow","flight"]
输出："fl"
示例 2：

输入：strs = ["dog","racecar","car"]
输出：""
解释：输入不存在公共前缀。


提示：

1 <= strs.length <= 200
0 <= strs[i].length <= 200
strs[i] 仅由小写英文字母组成

**解法一：字符串比较法**

首先检测出最短字符串长度，然后要做的是，排除一些特殊情况，例如，最短字符串长度为0时，可以直接返回一个空字符串，同时当字符串数组长度为1时，直接返回那个字符串。

然后采用两层循环，第一层循环代表的是目标字符串长度，从1开始，小于等于最短字符串长度，第二层循环代表检验数组中每个字符串，截取长度为目标字符串长度的的子字符串并比较，当发现不等时，立刻返回上次循环的目标字符串。

**但这个做法，需要非常小心字符串长度越界的问题！如果有j+1，就需要让循环条件改为j < length - 1;如果有 i -1 ；就需要排除掉 i  =  0的情况，也就是 目标字符串长度为0 的时候**。

同时，该做法用了一个循环和一个嵌套循环，时间复杂度达到了惊人的（n^2 + n），太过于浪费资源，需要改进。

```java
public class LongestCommonPrefix {
    public static String longestCommonPrefix(String[] strs) {
        //moni 2022/3/2
        if(strs.length == 1 || strs[0].length() == 0){
            return strs[0];
        }
        int min = 200;
        for (String x : strs) {
            if(x.length() < min){
                min = x.length();
            }
        }
        int i = 1;
        int j = 0;
        for(i = 1;i <= min; i++){
            for (j = 0; j < strs.length - 1; j++) {
                String targetStr = strs[j].substring(0, i);
                if(!targetStr.equals(strs[j + 1].substring(0, i))){
                    return strs[j].substring(0, i-1);
                }
            }
        }
        return strs[j].substring(0, i-1);
    }
}

```



**解法二：优化倒置比较法**

该方法也是采用比较字符串的方式，但优化了过程，首先求出最短字符串长度，并排除特殊情况（最短字符串长度为0），然后注意到，如果是公共前缀，那么第一个字符串一定是最先比较的，也就是说，可以直接把第一个字符串暂时视作公共字符串，然后把他的子字符串（长度< 最短字符串）逐个存入哈希图中，键为长度，值为子字符串。

然后遍历字符串数组，再用一个循环，让每个字符串切割成长度为min的子字符串，并与第一个字符串的长度为min的子串作比较（注意：字符串比较用equals（）方法），**如果两者相等，打破该次循环**，开始进入下一个字符串的比较。

> 相等打破循环的原因：如果匹配，说明该字符串的子串也是匹配的，不需要继续浪费时间比较。如果不匹配，min-1，后面的字符串比较是公用这个min的，所以节省了很多时间。

```java
public class LongestCommonPrefix {
    public static String longestCommonPrefix(String[] strs) {
        //moni 2022/3/2
        int min = 200;
        for(String x : strs){
            min = Math.min(x.length(), min);
        }
        if(min == 0){
            return "";
        }
        String targetStr = "";
        for(String x : strs){
            for(; min >= 0; min--){
                targetStr = x.substring(0, min);
                if(targetStr.equals(strs[0].substring(0,min))){
                    break;
                }
            }
        }
        return targetStr;
    }

    public static void main(String[] args) {
        String[] strs = {"flower","flow","flight"};
        System.out.println(longestCommonPrefix(strs));
    }
}

```



**解法三：横向比较法：**

该解法的核心在于横向比较，也就是先比较s1与s2的公共前缀，然后用这个公共前缀去比较s3，一直比较下去，如果途中出现空串，立马返回空串，否则返回最后的公共前缀。

注意：

排除特殊情况，例如空字符串数组，字符串数组长度为1。

```
public static String longestCommonPrefix(String[] strs) {
    //moni 2022/3/3
    if(strs.length == 0 || strs[0].length() == 0){
        return "";
    }
    String prefix = strs[0];
    for(int i = 0;i < strs.length - 1;i++){
        prefix = commonPrefix(prefix,strs[i+1]);
        if (prefix.length() == 0){
            return "";
        }
    }
    return prefix;
}

public static String commonPrefix(String s1,String s2){
    int prefix = 0;
    int min = Math.min(s1.length(), s2.length());
    while (prefix < min && s1.charAt(prefix) == s2.charAt(prefix)){
        prefix++;
    }

    return s2.substring(0,prefix);
}
```



### [20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。


示例 1：

输入：s = "()"
输出：true
示例 2：

输入：s = "()[]{}"
输出：true
示例 3：

输入：s = "(]"
输出：false
示例 4：

输入：s = "([)]"
输出：false
示例 5：

输入：s = "{[]}"
输出：true


提示：

1 <= s.length <= 104
s 仅由括号 '()[]{}' 组成

**解法一：出栈入栈法**：

本题很容易想到栈的做法，遍历字符串。遇到左括号则入栈，遇到右括号，首先判断是不是空栈，如果是，立刻返回false，因为一定不匹配；如果不是空栈，则判断前一个字符是不是相对应的左括号。最后判断是否为空栈，如果一一对应，则一定是空栈（右括号不入栈）。

```java
import java.util.Stack;

public class IsValid {
    public static boolean isValid(String s) {
        //Moni 2022/3/4
         Stack sk = new Stack<Character>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if(sk.empty() && "})]".indexOf(c) >= 0){
                return false;
            }else if("([{".indexOf(c) >= 0){
                sk.push(c);
            }
            switch (c){
                case ')':
                    if(!sk.pop().equals('(')) return false;
                    break;
                case ']':
                    if(!sk.pop().equals('[')) return false;
                    break;
                case '}':
                    if(!sk.pop().equals('{')) return false;
                    break;
            }
        }
        if(!sk.empty()){
            return false;
        }
        return true;
    }

    public static void main(String[] args) {
        String s = "()";
        System.out.println(isValid(s));
    }
}
```





## 3. 链表 

#### [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

 ![image-20220301015847819](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20220301015847819.png)

示例 1：

输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
示例 2：

输入：l1 = [0], l2 = [0]
输出：[0]
示例 3：

输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]


提示：

每个链表中的节点数在范围 [1, 100] 内
0 <= Node.val <= 9
题目数据保证列表表示的数字不含前导零



由于本人对于数据结构掌握的不够牢靠，看到链表就感觉很懵，不知道怎么办。所以见到本题完全无法下笔。其中我觉得最难的点在于，如何创建链表（本题貌似用不到）和如何遍历列表。现在终于明白，遍历列表，需要 用 l1 = l1.next；

**解法一：**

同时遍历链表， 创建一个新链表，然后对遍历出来的数字逐步相加，并且要**非常注意进位**，如果某一个链表遍历空了，当作0处理。遇到的难题有：

1. 返回值返回什么东西；
2. head 和 tail 如何连接。
3. 分配head和tail的地址空间。

值得注意的是，当l1和l2遍历完后，如果还有进位，需要表示出来，也就多了一行代码，是当carry最后不为0时，为tail new一个空间出来，表示这个进位操作。

```java
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode head = null,tail = null;
        int carry = 0;
        while(l1 != null || l2 != null){
            int n1 = l1 != null ? l1.val : 0;
            int n2 = l2 != null ? l2.val : 0;

            int sum = n1 + n2 + carry;
            carry = sum / 10;
            if(head == null){
                //初始化，连接head 和 tail 两个结点
                head = tail = new ListNode(sum % 10);
            }else {
                //先让tail.next 赋值，然后让tail = tail.next ,这样可以保证新链表的创建。
                tail.next = new ListNode(sum % 10);
                tail = tail.next;
            }
            if(l1 != null){
                l1 = l1.next;
            }
            if(l2 != null) {
                l2 = l2.next;
            }
        }
        //当l1 ，l2都遍历完，若有进位，需要以下代码来进位操作。
        if (carry > 0) {
            tail.next = new ListNode(carry);
        }

        return head;
    }
```

**解法二：递归**

```java
public class AddTwoNumbers {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode resList = new ListNode();
        return add(l1,l2,resList,0);
    }

    public static ListNode add(ListNode l1, ListNode l2, ListNode resList,int carry){
        if (l1 == null && l2 == null && carry != 1){
            return resList;
        }else if(l1 == null && l2 == null && carry == 1){
            resList.next = new ListNode(carry);
            return resList;
        }
        int n1 = l1 != null ? l1.val : 0;
        int n2 = l2 != null ? l2.val : 0;
        int sum = n1 + n2 + carry;
        carry = sum / 10;
        if(resList == null){
            resList = new ListNode(sum % 10);
        }else {
            resList.next = new ListNode(sum % 10);
            resList = resList.next;
        }
        l1 = l1 != null ? l1.next : null;
        l2 = l2 != null ? l2.next : null;
        add(l1,l2,resList,carry);

        return resList;
    }
}

```

运用递归的思想，但是需要注意的点是：

- 进位符最好单独作为一个变量列出来；

- 注意递归结束时，如果carry不为0，需要额外创建一个结点，存储进位值。

- 初始化链表，可以直接使用resList = new ListNode(),然后后面的接入链表时，需要使用

- ```java
              resList.next = new ListNode(sum % 10);
              resList = resList.next;
  ```

  在链表中，直接控制next = 新值，然后让指针**（？）**移动到next位置。



## 递归算法

> **递归算法，简单来讲即调用自己。**
>
> *人类理解迭代，上帝理解递归*

### 1. 方法

首先，需要一个递归出口，也就是不能无限循环。

调用自己，就需要使用至少一个在改变的参数，否则也会出现死循环。

### 2. 难点

#### 2.1 如何解决重复初始化的问题？

在递归方法外初始化，递归结束后输出该变量，就得到了结果。

### 3. 什么情况用递归

需要循环，且每次循环有值改变时。
