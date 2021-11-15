### LeetCode算法题答案(php)


```
1. 两数之和
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。
示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]

class Solution {

    /**
     * @param Integer[] $nums
     * @param Integer $target
     * @return Integer[]
     */
    function twoSum($nums, $target) {
        foreach($nums as $k=>$v){
            $re = $target - $v;
            if(in_array($re,$nums)){
                $k1 = array_search($re,$nums);
                if($k != $k1){
                    return [$k,$k1];
                }
            }
        }
    }
}
```


```
2,两链表相加
给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：

输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807


/**
 * Definition for a singly-linked list.
 * class ListNode {
 *     public $val = 0;
 *     public $next = null;
 *     function __construct($val) { $this->val = $val; }
 * }
 */
class Solution {
    //核心思想 两个相同位的数相加 大于10的进位即可
    /**
     * @param ListNode $l1
     * @param ListNode $l2
     * @return ListNode
     */
    function addTwoNumbers($l1, $l2) {
        $obj = null;
        $addition = 0;
        do{
            $v = $l1->val + $l2->val + $addition;
            
            if($v>=10){
                $v -= 10;
                $addition = 1;
            }else{
                $addition = 0;
            }
            
            $tmp = new ListNode($v);
            
            if($obj == null){
                $obj = $tmp;
            }else{
                $self->next = $tmp;
            }
            
            $self = $tmp;
            $l1 = $l1->next;
            $l2 = $l2->next;
        }while($l1 || $l2 || $addition);
            
        return $obj; 
    }
}
```

```
3,反转一个单链表。
示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL


/**
 * Definition for a singly-linked list.
 * class ListNode {
 *     public $val = 0;
 *     public $next = null;
 *     function __construct($val) { $this->val = $val; }
 * }
 */
class Solution {

    /**
     * @param ListNode $head
     * @return ListNode
     */
    function reverseList($head) {
        if(null == $head || null == $head->next) return $head;
        $reverHead = $this->reverseList($head->next);
        $head->next->next = $head;
        $head->next = null;
        return $reverHead;
    }

}


/**
 * Definition for a singly-linked list.
 * class ListNode {
 *     public $val = 0;
 *     public $next = null;
 *     function __construct($val) { $this->val = $val; }
 * }
 */
class Solution {

    /**
     * @param ListNode $head
     * @return ListNode
     */
    function reverseList($head) {
        if(null == $head) return $head;
        $pre = $head;
        $cur = $head->next;
        $next = null;
        while($cur != null){
            $next = $cur->next;
            $cur->next = $pre;
            $pre = $cur;
            $cur = $next;
        }
        $head->next = null;
        $head = $pre;
        return $head;
    }

}
```


```
4,删除排序链表中的重复元素

给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。

示例 1:

输入: 1->1->2
输出: 1->2
示例 2:

输入: 1->1->2->3->3
输出: 1->2->3



/**
 * Definition for a singly-linked list.
 * class ListNode {
 *     public $val = 0;
 *     public $next = null;
 *     function __construct($val) { $this->val = $val; }
 * }
 */
class Solution {

    /**
     * @param ListNode $head
     * @return ListNode
     */
    function deleteDuplicates($head) {
        if(null == $head || null == $head->next){
            return $head;
        }
        
        if($head->val == $head->next->val){
            return $this->deleteDuplicates($head->next);
        }else{
            $head->next = $this->deleteDuplicates($head->next);
        }
        return $head;
    }
}
```

```
5,删除排序链表中的重复元素 II
给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 没有重复出现 的数字。
示例 1:

输入: 1->2->3->3->4->4->5
输出: 1->2->5
示例 2:

输入: 1->1->1->2->3
输出: 2->3

/**
 * Definition for a singly-linked list.
 * class ListNode {
 *     public $val = 0;
 *     public $next = null;
 *     function __construct($val) { $this->val = $val; }
 * }
 */
class Solution {

    /**
     * @param ListNode $head
     * @return ListNode
     */
    function deleteDuplicates($head) {
        if(null == $head){
            return $head;
        }
        if(null != $head->next && $head->val == $head->next->val){
            while(null != $head && null != $head->next &&  $head->val == $head->next->val){
                $head = $head->next;
            }
            return $this->deleteDuplicates($head->next);
        }else{
            $head->next = $this->deleteDuplicates($head->next);
        }
        return $head;
    }
}
```

```
6,三数之和

给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]


class Solution {

    /**
     * @param Integer[] $nums
     * @return Integer[][]
     */
    function threeSum($nums) {
        
        $count = count($nums);
        sort($nums);
        $result = [];
        
        for($i=0;$i<$count;$i++){

            $start = $i + 1;
            $end = $count - 1;
            while($start < $end){
                
                $sum = $nums[$i] + $nums[$start] + $nums[$end];
                
                if($sum < 0){
                    $start +=  1;
                }
                
                if($sum > 0){
                    $end -= 1;
                }
                
                if($sum == 0){
                     
                    if(!in_array([$nums[$i] , $nums[$start] , $nums[$end]],$result)){
                        $result[] = [$nums[$i] , $nums[$start] , $nums[$end]];
                    }
                    $end -= 1;
                    $start += 1;
                }
            }
        }
        return $result;
    }
}
```
```
7,LRU缓存机制
运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。



示例:

LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4




class LRUCache {
    private $capacity;
    private $list = [];
    private $map = [];
    
    /**
     * @param Integer $capacity
     */
    function __construct($capacity) {
        $this->capacity = $capacity;
    }
  
    /**
     * @param Integer $key
     * @return Integer
     */
    function get($key) {
        if(array_key_exists($key,$this->map)){
            unset($this->list[array_search($key,$this->list)]);
            array_push($this->list,$key);
            return $this->map[$key];
        }else{
            return -1;
        }
    }
  
    /**
     * @param Integer $key
     * @param Integer $value
     * @return NULL
     */
    function put($key, $value) {
        if($this->get($key) == -1){
            if(count($this->list) == $this->capacity){
                $dkey = array_shift($this->list);
                unset($this->map[$dkey]);
            }
            array_push($this->list,$key);
        }
        $this->map[$key] = $value;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * $obj = LRUCache($capacity);
 * $ret_1 = $obj->get($key);
 * $obj->put($key, $value);
 */
```

```
8,买卖股票的最佳时机

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意你不能在买入股票前卖出股票。

示例 1:

输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
示例 2:

输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。


class Solution {

    /**
     * @param Integer[] $prices
     * @return Integer
     * 思路:最大利益就是数组中的最大差值,所以用一个变量存储当前数组中最小值,并计算键值与最小值的差(利益),取最大差,遍历一遍数组就可以了
     */
    function maxProfit($prices) {
        $profit = 0;
        $min = $prices[0];
        foreach($prices as $k => $v){
            $min = ($min > $v) ? $v : $min;
            $profit = ($v - $min) > $profit ? ($v - $min) : $profit;
        }
        return $profit;
    }
}
```

```
9,无重复字符的最长子串
给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1:

输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
示例 2:

输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
示例 3:

输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。



{

    /*
    假如有字符串dababc
    滑动窗口开始时i=j=0,得到d是所求的
    i=0,j=1 da是所求
    i=0,j=2 dab是所求
    i=0,j=3 daba不是所求，a重复了，此时第一个a的下标为index=1,那么滑动窗口的i此时应该修改成index
    i=1,j=3 ab才是所求
    ....
    所以归根到底就是怎么更有效地得出i、j、index
    j很简单，拿到字符串长度，然后循环时j++就是所求
    i初始值为0，后续都是根据index获取
    可以定义一个关联数组arr,键名为字符，键值为字符所在位置(index，index需要大于i)，判断字符重复时直接用isset/array_key_exists即可(或者也可以用字符串形式，查找字符串中字符位置)
    */
    /**
     * @param String $s
     * @return Integer
     */
    function lengthOfLongestSubstring($s) {
        $count = strlen($s);
        $max = $len = 0;
        $i = $j = 0;
        $arr = [];
        while($j<$count){
            if(isset($arr[$s[$j]]) && $arr[$s[$j]] >= $i){
                $i = $arr[$s[$j]];
                $len = $j - $i;
            }else{
                $len++;
            }
            if($max < $len) $max = $len;
            $arr[$s[$j]] = $j;
            $j++;
        }
        return $max;
    }
    
    
}
```

```
10,数组中重复的数据
给定一个整数数组 a，其中1 ≤ a[i] ≤ n （n为数组长度）, 其中有些元素出现两次而其他元素出现一次。

找到所有出现两次的元素。

你可以不用到任何额外空间并在O(n)时间复杂度内解决这个问题吗？

示例：

输入:
[4,3,2,7,8,2,3,1]

输出:
[2,3]


class Solution {

    /**
     * @param Integer[] $nums
     * @return Integer[]
     */
    function findDuplicates($nums) {
        $array = [];
        $uniqueArray = [];
        foreach($nums as $k => $v){
            if(in_array($v,$uniqueArray)){
                $array[] = $v;
            }else{
                $uniqueArray[] = $v;
            }
            
        }
        return $array;
    }
}
```


```
11,最大子序和
给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
示例:

输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
进阶:

如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。


class Solution {

    /**
     * @param Integer[] $nums
     * @return Integer
     */
    function maxSubArray($nums) {
        for($i=1;$i<count($nums);$i++)
            $nums[$i] = max($nums[$i],$nums[$i]+$nums[$i-1]);
        return max($nums);
    }
}
```

```
12,最长回文子串

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
示例 2：

输入: "cbbd"
输出: "bb"



class Solution {

    /**
     * @param String $s
     * @return String
     */
    function longestPalindrome($s) {
        
        $count = strlen($s);
        $result = $s[0];
        $tmpA = null;
        $tmpB = null;
        
        for($i=0;$i<$count;$i++){
            $tmpA = null;
            $tmpB = null;
            for($j=$count;$j>$i;$j--){
                if($j - $i < strlen($result))  break;
                $tmpA = substr($s,$i,$j-$i);
                $tmpB = strrev($tmpA);
                if($tmpA == $tmpB && strlen($result)<strlen($tmpA) ){
                    $result = $tmpA;
                }
            }
            
        }
        return $result;
        
    }
}
```



```
13,数字范围按位与

给定范围 [m, n]，其中 0 <= m <= n <= 2147483647，返回此范围内所有数字的按位与（包含 m, n 两端点）。

示例 1: 

输入: [5,7]
输出: 4
示例 2:

输入: [0,1]
输出: 0

class Solution {

    /**
     * @param Integer $m
     * @param Integer $n
     * @return Integer
     */
    function rangeBitwiseAnd($m, $n) {
        while($n>$m){
            $n = $n & ($n - 1);
        }
        return $n;
    }
}

```



题目来源：力扣（LeetCode）
自己提交的PHP答案记录（备忘）
