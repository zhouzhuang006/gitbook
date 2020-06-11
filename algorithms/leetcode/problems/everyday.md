# 每日打卡一道算法





# 2020-06-08 等式方程的可满足性

https://leetcode-cn.com/problems/satisfiability-of-equality-equations/

## 题目：

给定一个由表示变量之间关系的字符串方程组成的数组，每个字符串方程 equations[i] 的长度为 4，并采用两种不同的形式之一："a==b" 或 "a!=b"。在这里，a 和 b 是小写字母（不一定不同），表示单字母变量名。

只有当可以将整数分配给变量名，以便满足所有给定的方程时才返回 true，否则返回 false。 

 

示例 1：

输入：["a==b","b!=a"]
输出：false
解释：如果我们指定，a = 1 且 b = 1，那么可以满足第一个方程，但无法满足第二个方程。没有办法分配变量同时满足这两个方程。

示例 2：

输入：["b==a","a==b"]
输出：true
解释：我们可以指定 a = 1 且 b = 1 以满足满足这两个方程。

示例 3：

输入：["a==b","b==c","a==c"]
输出：true

示例 4：

输入：["a==b","b!=c","c==a"]
输出：false

示例 5：

输入：["c==c","b==d","x!=z"]
输出：true

 

提示：

    1 <= equations.length <= 500
    equations[i].length == 4
    equations[i][0] 和 equations[i][3] 是小写字母
    equations[i][1] 要么是 '='，要么是 '!'
    equations[i][2] 是 '='



## 方法一：并查集

我们可以将每一个变量看作图中的一个节点，把相等的关系 == 看作是连接两个节点的边，那么由于表示相等关系的等式方程具有传递性，即如果 a==b 和 b==c 成立，则 a==c 也成立。也就是说，所有相等的变量属于同一个连通分量。因此，我们可以使用并查集来维护这种连通分量的关系。

首先遍历所有的等式，构造并查集。同一个等式中的两个变量属于同一个连通分量，因此将两个变量进行合并。

然后遍历所有的不等式。同一个不等式中的两个变量不能属于同一个连通分量，因此对两个变量分别查找其所在的连通分量，如果两个变量在同一个连通分量中，则产生矛盾，返回 false。

如果遍历完所有的不等式没有发现矛盾，则返回 true。

fig1

具体实现方面，使用一个数组 parent 存储每个变量的连通分量信息，其中的每个元素表示当前变量所在的连通分量的父节点信息，如果父节点是自身，说明该变量为所在的连通分量的根节点。一开始所有变量的父节点都是它们自身。对于合并操作，我们将第一个变量的根节点的父节点指向第二个变量的根节点；对于查找操作，我们沿着当前变量的父节点一路向上查找，直到找到根节点。



```java
class Solution {
    public boolean equationsPossible(String[] equations) {

        int[] parent = new int[26];
        for (int i = 0; i < 26; i++) {
            parent[i] = i;
        }
        for (String str : equations) {
            if (str.charAt(1) == '=') {
                int index1 = str.charAt(0) - 'a';
                int index2 = str.charAt(3) - 'a';
                union(parent, index1, index2);
            }
        }
        for (String str : equations) {
            if (str.charAt(1) == '!') {
                int index1 = str.charAt(0) - 'a';
                int index2 = str.charAt(3) - 'a';
                if (find(parent, index1) == find(parent, index2)) {
                    return false;
                }
            }
        }
        return true;
    }

    public void union(int[] parent, int index1, int index2) {
        parent[find(parent, index1)] = find(parent, index2);
    }

    public int find(int[] parent, int index) {
        while (parent[index] != index) {
            parent[index] = parent[parent[index]];
            index = parent[index];
        }
        return index;
    }
}
```







# 2020-06-09 把数字翻译成字符串

https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/

## 题目：

给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

 

示例 1:

输入: 12258
输出: 5
解释: 12258有5种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"

 

## 方法一：动态规划

思路和算法

首先我们来通过一个例子理解一下这里「翻译」的过程：我们来尝试翻译「140214021402」。

分成两种情况：

    首先我们可以把每一位单独翻译，即 [1,4,0,2][1, 4, 0, 2][1,4,0,2]，翻译的结果是 beac
    然后我们考虑组合某些连续的两位：
        [14,0,2][14, 0, 2][14,0,2]，翻译的结果是 oac。
        [1,40,2][1, 40, 2][1,40,2]，这种情况是不合法的，因为 404040 不能翻译成任何字母。
        [1,4,02][1, 4, 02][1,4,02]，这种情况也是不合法的，含有前导零的两位数不在题目规定的翻译规则中，那么 [14,02][14, 02][14,02] 显然也是不合法的。

那么我们可以归纳出翻译的规则，字符串的第 iii 位置：

    可以单独作为一位来翻译
    如果第 i−1i - 1i−1 位和第 iii 位组成的数字在 101010 到 252525 之间，可以把这两位连起来翻译

到这里，我们发现它和「198. 打家劫舍」非常相似。我们可以用 f(i)f(i)f(i) 表示以第 iii 位结尾的前缀串翻译的方案数，考虑第 iii 位单独翻译和与前一位连接起来再翻译对 f(i)f(i)f(i) 的贡献。单独翻译对 f(i)f(i)f(i) 的贡献为 f(i−1)f(i - 1)f(i−1)；如果第 i−1i - 1i−1 位存在，并且第 i−1i - 1i−1 位和第 iii 位形成的数字 xxx 满足 10≤x≤2510 \leq x \leq 2510≤x≤25，那么就可以把第 i−1i - 1i−1 位和第 iii 位连起来一起翻译，对 f(i)f(i)f(i) 的贡献为 f(i−2)f(i - 2)f(i−2)，否则为 0。我们可以列出这样的动态规划转移方程：

f(i)=f(i−1)+f(i−2)[i−1≥0,10≤x≤25]f(i) = f(i - 1) + f(i - 2)[i - 1 \geq 0, 10 \leq x \leq 25] f(i)=f(i−1)+f(i−2)[i−1≥0,10≤x≤25]

边界条件是 f(−1)=0f(-1) = 0f(−1)=0，f(0)=1f(0) = 1f(0)=1。方程中 [c][c][c] 的意思是 ccc 为真的时候 [c]=1[c] = 1[c]=1，否则 [c]=0[c] = 0[c]=0。

有了这个方程我们不难给出一个时间复杂度为 O(n)O(n)O(n)，空间复杂度为 O(n)O(n)O(n) 的实现。考虑优化空间复杂度：这里的 f(i)f(i)f(i) 只和它的前两项 f(i−1)f(i - 1)f(i−1) 和 f(i−2)f(i - 2)f(i−2) 相关，我们可以运用「滚动数组」思想把 fff 数组压缩成三个变量，这样空间复杂度就变成了 O(1)O(1)O(1)。



```java
class Solution {
    public int translateNum(int num) {
        String src = String.valueOf(num);
        int p = 0, q = 0, r = 1;
        for (int i = 0; i < src.length(); ++i) {
            p = q;
            q = r;
            r = 0;
            r += q;
            if (i == 0) {
                continue;
            }
            String pre = src.substring(i - 1, i + 1);
            if (pre.compareTo("25") <= 0 && pre.compareTo("10") >= 0) {
                r += p;
            }
        }
        return r;
    }
}
```



# 2020-06-10 回文数

https://leetcode-cn.com/problems/palindrome-number/

## 题目：

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

示例 1:

输入: 121
输出: true

示例 2:

输入: -121
输出: false
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。

示例 3:

输入: 10
输出: false
解释: 从右向左读, 为 01 。因此它不是一个回文数。

进阶:

你能不将整数转为字符串来解决这个问题吗？



## 方法一：字符串

```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x > 0 && x < 10) {
          return true;
        }
        String ss = String.valueOf(x);
        int len = ss.length();
        for (int i = 0; i < ss.length() / 2 ; i ++) {
            if(ss.charAt(i) != ss.charAt(len - 1 - i)) {
                return false;
            }
        }
        return true;
    }
}

```



## 方法二：反转一半数字

思路

映入脑海的第一个想法是将数字转换为字符串，并检查字符串是否为回文。但是，这需要额外的非常量空间来创建问题描述中所不允许的字符串。

第二个想法是将数字本身反转，然后将反转后的数字与原始数字进行比较，如果它们是相同的，那么这个数字就是回文。
但是，如果反转后的数字大于 int.MAX\text{int.MAX}int.MAX，我们将遇到整数溢出问题。

按照第二个想法，为了避免数字反转可能导致的溢出问题，为什么不考虑只反转 int\text{int}int 数字的一半？毕竟，如果该数字是回文，其后半部分反转后应该与原始数字的前半部分相同。

例如，输入 1221，我们可以将数字 “1221” 的后半部分从 “21” 反转为 “12”，并将其与前半部分 “12” 进行比较，因为二者相同，我们得知数字 1221 是回文。

算法

首先，我们应该处理一些临界情况。所有负数都不可能是回文，例如：-123 不是回文，因为 - 不等于 3。所以我们可以对所有负数返回 false。除了 0 以外，所有个位是 0 的数字不可能是回文，因为最高位不等于 0。所以我们可以对所有大于 0 且个位是 0 的数字返回 false。

现在，让我们来考虑如何反转后半部分的数字。

对于数字 1221，如果执行 1221 % 10，我们将得到最后一位数字 1，要得到倒数第二位数字，我们可以先通过除以 10 把最后一位数字从 1221 中移除，1221 / 10 = 122，再求出上一步结果除以 10 的余数，122 % 10 = 2，就可以得到倒数第二位数字。如果我们把最后一位数字乘以 10，再加上倒数第二位数字，1 * 10 + 2 = 12，就得到了我们想要的反转后的数字。如果继续这个过程，我们将得到更多位数的反转数字。

现在的问题是，我们如何知道反转数字的位数已经达到原始数字位数的一半？

由于整个过程我们不断将原始数字除以 10，然后给反转后的数字乘上 10，所以，当原始数字小于或等于反转后的数字时，就意味着我们已经处理了一半位数的数字了。

![fig1](9_fig1.png)

```java
class Solution {
    public boolean isPalindrome(int x) {
        // 特殊情况：
        // 如上所述，当 x < 0 时，x 不是回文数。
        // 同样地，如果数字的最后一位是 0，为了使该数字为回文，
        // 则其第一位数字也应该是 0
        // 只有 0 满足这一属性
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }

        int revertedNumber = 0;
        while (x > revertedNumber) {
            revertedNumber = revertedNumber * 10 + x % 10;
            x /= 10;
        }

        // 当数字长度为奇数时，我们可以通过 revertedNumber/10 去除处于中位的数字。
        // 例如，当输入为 12321 时，在 while 循环的末尾我们可以得到 x = 12，revertedNumber = 123，
        // 由于处于中位的数字不影响回文（它总是与自己相等），所以我们可以简单地将其去除。
        return x == revertedNumber || x == revertedNumber / 10;
    }
}
```



# 2020-06-11 每日温度

https://leetcode-cn.com/problems/daily-temperatures/

## 题目:

根据每日 气温 列表，请重新生成一个列表，对应位置的输出是需要再等待多久温度才会升高超过该日的天数。如果之后都不会升高，请在该位置用 0 来代替。

例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。

提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的均为华氏度，都是在 [30, 100] 范围内的整数。



```java
public class Solution {

    public static void main(String[] args) {

        int[] array = new int[]{73, 74, 75, 71, 69, 72, 76, 73};
        int[] result = new Solution().dailyTemperatures(array);
        System.out.println(Arrays.toString(result));
    }

    public int[] dailyTemperatures(int[] T) {

        int len = T.length;
        if (len == 1) {
            return new int[]{0};
        }
        int[] result = new int[len];
        for (int a = 0; a < len; a++) {
            int up = 0;
            boolean find = false;
            for (int b = a + 1; b < len; b++) {
                if (T[b] > T[a]) {
                    up++;
                    find = true;
                    break;
                }
                up++;
            }
            if (!find) {
                up = 0;
            }
            result[a] = up;
        }
        return result;
    }
}
```



## 方法一：暴力

对于温度列表中的每个元素 T[i]，需要找到最小的下标 j，使得 i < j 且 T[i] < T[j]。

由于温度范围在 [30, 100] 之内，因此可以维护一个数组 next 记录每个温度第一次出现的下标。数组 next 中的元素初始化为无穷大，在遍历温度列表的过程中更新 next 的值。

反向遍历温度列表。对于每个元素 T[i]，在数组 next 中找到从 T[i] + 1 到 100 中每个温度第一次出现的下标，将其中的最小下标记为 warmerIndex，则 warmerIndex 为下一次温度比当天高的下标。如果 warmerIndex 不为无穷大，则 warmerIndex - i 即为下一次温度比当天高的等待天数，最后令 next[T[i]] = i。

为什么上述做法可以保证正确呢？因为遍历温度列表的方向是反向，当遍历到元素 T[i] 时，只有 T[i] 后面的元素被访问过，即对于任意 t，当 next[t] 不为无穷大时，一定存在 j 使得 T[j] == t 且 i < j。又由于遍历到温度列表中的每个元素时都会更新数组 next 中的对应温度的元素值，因此对于任意 t，当 next[t] 不为无穷大时，令 j = next[t]，则 j 是满足 T[j] == t 且 i < j 的最小下标。

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int length = T.length;
        int[] ans = new int[length];
        int[] next = new int[101];
        Arrays.fill(next, Integer.MAX_VALUE);
        for (int i = length - 1; i >= 0; --i) {
            int warmerIndex = Integer.MAX_VALUE;
            for (int t = T[i] + 1; t <= 100; ++t) {
                if (next[t] < warmerIndex) {
                    warmerIndex = next[t];
                }
            }
            if (warmerIndex < Integer.MAX_VALUE) {
                ans[i] = warmerIndex - i;
            }
            next[T[i]] = i;
        }
        return ans;
    }
}
```



复杂度分析

    时间复杂度：O(nm)O(nm)O(nm)，其中 nnn 是温度列表的长度，mmm 是数组 next 的长度，在本题中温度不超过 100100100，所以 mmm 的值为 100100100。反向遍历温度列表一遍，对于温度列表中的每个值，都要遍历数组 next 一遍。
    
    空间复杂度：O(m)O(m)O(m)，其中 mmm 是数组 next 的长度。除了返回值以外，需要维护长度为 mmm 的数组 next 记录每个温度第一次出现的下标位置。

## 方法二：单调栈

可以维护一个存储下标的单调栈，从栈底到栈顶的下标对应的温度列表中的温度依次递减。如果一个下标在单调栈里，则表示尚未找到下一次温度更高的下标。

正向遍历温度列表。对于温度列表中的每个元素 T[i]，如果栈为空，则直接将 i 进栈，如果栈不为空，则比较栈顶元素 prevIndex 对应的温度 T[prevIndex] 和当前温度 T[i]，如果 T[i] > T[prevIndex]，则将 prevIndex 移除，并将 prevIndex 对应的等待天数赋为 i - prevIndex，重复上述操作直到栈为空或者栈顶元素对应的温度小于等于当前温度，然后将 i 进栈。

为什么可以在弹栈的时候更新 ans[prevIndex] 呢？因为在这种情况下，即将进栈的 i 对应的 T[i] 一定是 T[prevIndex] 右边第一个比它大的元素，试想如果 prevIndex 和 i 有比它大的元素，假设下标为 j，那么 prevIndex 一定会在下标 j 的那一轮被弹掉。

由于单调栈满足从栈底到栈顶元素对应的温度递减，因此每次有元素进栈时，会将温度更低的元素全部移除，并更新出栈元素对应的等待天数，这样可以确保等待天数一定是最小的。

以下用一个具体的例子帮助读者理解单调栈。对于温度列表 [73,74,75,71,69,72,76,73][73,74,75,71,69,72,76,73][73,74,75,71,69,72,76,73]，单调栈 stack\textit{stack}stack 的初始状态为空，答案 ans\textit{ans}ans 的初始状态是 [0,0,0,0,0,0,0,0][0,0,0,0,0,0,0,0][0,0,0,0,0,0,0,0]，按照以下步骤更新单调栈和答案，其中单调栈内的元素都是下标，括号内的数字表示下标在温度列表中对应的温度。

    当 i=0i=0i=0 时，单调栈为空，因此将 000 进栈。
    
        stack=[0(73)]\textit{stack}=[0(73)]stack=[0(73)]
    
        ans=[0,0,0,0,0,0,0,0]\textit{ans}=[0,0,0,0,0,0,0,0]ans=[0,0,0,0,0,0,0,0]
    
    当 i=1i=1i=1 时，由于 747474 大于 737373，因此移除栈顶元素 000，赋值 ans[0]:=1−0ans[0]:=1-0ans[0]:=1−0，将 111 进栈。
    
        stack=[1(74)]\textit{stack}=[1(74)]stack=[1(74)]
    
        ans=[1,0,0,0,0,0,0,0]\textit{ans}=[1,0,0,0,0,0,0,0]ans=[1,0,0,0,0,0,0,0]
    
    当 i=2i=2i=2 时，由于 757575 大于 747474，因此移除栈顶元素 111，赋值 ans[1]:=2−1ans[1]:=2-1ans[1]:=2−1，将 222 进栈。
    
        stack=[2(75)]\textit{stack}=[2(75)]stack=[2(75)]
    
        ans=[1,1,0,0,0,0,0,0]\textit{ans}=[1,1,0,0,0,0,0,0]ans=[1,1,0,0,0,0,0,0]
    
    当 i=3i=3i=3 时，由于 717171 小于 757575，因此将 333 进栈。
    
        stack=[2(75),3(71)]\textit{stack}=[2(75),3(71)]stack=[2(75),3(71)]
    
        ans=[1,1,0,0,0,0,0,0]\textit{ans}=[1,1,0,0,0,0,0,0]ans=[1,1,0,0,0,0,0,0]
    
    当 i=4i=4i=4 时，由于 696969 小于 717171，因此将 444 进栈。
    
        stack=[2(75),3(71),4(69)]\textit{stack}=[2(75),3(71),4(69)]stack=[2(75),3(71),4(69)]
    
        ans=[1,1,0,0,0,0,0,0]\textit{ans}=[1,1,0,0,0,0,0,0]ans=[1,1,0,0,0,0,0,0]
    
    当 i=5i=5i=5 时，由于 727272 大于 696969 和 717171，因此依次移除栈顶元素 444 和 333，赋值 ans[4]:=5−4ans[4]:=5-4ans[4]:=5−4 和 ans[3]:=5−3ans[3]:=5-3ans[3]:=5−3，将 555 进栈。
    
        stack=[2(75),5(72)]\textit{stack}=[2(75),5(72)]stack=[2(75),5(72)]
    
        ans=[1,1,0,2,1,0,0,0]\textit{ans}=[1,1,0,2,1,0,0,0]ans=[1,1,0,2,1,0,0,0]
    
    当 i=6i=6i=6 时，由于 767676 大于 727272 和 757575，因此依次移除栈顶元素 555 和 222，赋值 ans[5]:=6−5ans[5]:=6-5ans[5]:=6−5 和 ans[2]:=6−2ans[2]:=6-2ans[2]:=6−2，将 666 进栈。
    
        stack=[6(76)]\textit{stack}=[6(76)]stack=[6(76)]
    
        ans=[1,1,4,2,1,1,0,0]\textit{ans}=[1,1,4,2,1,1,0,0]ans=[1,1,4,2,1,1,0,0]
    
    当 i=7i=7i=7 时，由于 737373 小于 767676，因此将 777 进栈。
    
        stack=[6(76),7(73)]\textit{stack}=[6(76),7(73)]stack=[6(76),7(73)]
    
        ans=[1,1,4,2,1,1,0,0]\textit{ans}=[1,1,4,2,1,1,0,0]ans=[1,1,4,2,1,1,0,0]

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int length = T.length;
        int[] ans = new int[length];
        Deque<Integer> stack = new LinkedList<Integer>();
        for (int i = 0; i < length; i++) {
            int temperature = T[i];
            while (!stack.isEmpty() && temperature > T[stack.peek()]) {
                int prevIndex = stack.pop();
                ans[prevIndex] = i - prevIndex;
            }
            stack.push(i);
        }
        return ans;
    }
}
```



复杂度分析

    时间复杂度：O(n)O(n)O(n)，其中 nnn 是温度列表的长度。正向遍历温度列表一遍，对于温度列表中的每个下标，最多有一次进栈和出栈的操作。
    
    空间复杂度：O(n)O(n)O(n)，其中 nnn 是温度列表的长度。需要维护一个单调栈存储温度列表中的下标。



















