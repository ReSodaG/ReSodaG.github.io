---
title: LettCode 0004 题解
date: 2022-09-17 18:04:20
tags:
  - LettCode
---

## 从 How Hard Can It Be 到真的 Hard

首先，第四题 [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) 是 LeetCode 的第一道 Hard 难度的题目，求两个有序数组的中位数。

首先题目看到这里，相信大家的反应是 How Hard Can It Be，只要从最小的数字开始，数到中间就完事了，毕竟数组是有序的。但是真正的难点是下一个要求，时间复杂度为 O(log (m+n))。当加上这个限制条件的时候，相信很多刚开始刷题的朋友们都会感觉非常的突兀和不习惯，而想解决这个题目也就默认给了你一些前置条件，就是你至少要知道二分查找。

## 二分查找

为什么二分查找的时间复杂度是 O(logn)？假设一个有序的整数数组 A，数组 A 有 N 个元素，寻找数组 A 中有没有数字 j。对这个数组运用二分查找，同时假设数组 A 中没有数字 j。

```
// 数组 A 中的内容
[a, b, c, d, e, f, g, h, i, k, l, m, n, o, p, q]

// 进行第一次二分，因为元素 h 小于 j，元素 j 可能在从 元素 i 到 r 的区间中
[a, b, c, d, e, f, g, h] [i, k, l, m, n, o, p, q, r]

// 对从 i 开始 到 r 结束的这段数组再次进行二分查找，j 可能在 i 到 n这个区间中
[a, b, c, d, e, f, g, h] [i, k, l, n] [m, o, p, q]

// 对区间进行二分查找，元素 j 可能在区间 i 到 k 之间
[a, b, c, d, e, f, g, h] [i, k] [l, n] [m, o, p, q]

// 对 i 到 k 的区间进行二分
[a, b, c, d, e, f, g, h] [i] [k] [l, n] [m, o, p, q]

// 此时发现二分后两个区间中都只有一个元素，且都不为数字 j，得出结论，数组 A 中不存在数字 j

```

通过上面演示的二分查找的过程可以看出来，二分就是通过不断的将要查找的值可能存在的区间对半切开，直到只剩一个元素或者找到要查找的数字为止。我们假设每次进行查找最多进行 X 次二分比较行为，二分的过程回退到原始数组，可以发现从一个元素开始，通过若干次乘以二，数组的长度将会符合原始数组的长度。而这个乘以二的次数正好等于数组进行二分的次数 X。得到以下的式子：

N = 2^X^ => X = log~2~(N)

这就是为什么二分查找的时间复杂度是 O(logn) 的一个简单推导。

```JavaScript

const binarySearch = (nums, i) => {
    const m = nums.length;
    let left = 0, right = m - 1;
    while (left < right) {
        let mid = Math.floor((left + right) / 2);
        if (nums[mid] < i) left = mid + 1;
        else right = mid;
    }
    if (nums[left] == i)
        return left + 1;
    else return false
}

```

上面是一个二分的例子，要注意的地方是，二分的结束条件和如何决定下一次二分的区间。这里可以看到，循环的结束条件是首位相遇，也就是区间中只有一个元素的情况，因为二分的特性，最后区间中的元素一定是逐步减少的，同时这个区间也是一直符合我们设置的判定条件，此时区间中唯一的元素就是我们要找的元素了。也可以在每次二分时判断中间选择的点是不是符合要查找的条件，这样当运气好的时候，可以一次就把元素挑选出来。

## 题目分析

既然是求两个数组的中位数，那么两个数组之间一定是有关联的，而从中位数所在的位置，一定是可以把两个数组所有的元素分为两块，分别是小于中位数的和大于中位数的，且两边的数量应该是刚好相等或左侧数字比右侧多一个。此时的中位数应该就是左侧最大数字和右侧最小数字的平均数或左侧最大的数字。

当了解到中位数可以通过分割两个数组实现的话，我们就可以把问题提炼成，两个有序数组 A 和 B，将两个数组分割为左右元素个数相同或者左边多一个的情况，两侧元素满足左侧所有的元素小于右侧所有的元素的情况。因为元素的总数是确定的，我们只要确定数组 A 中的位置，就能推出数组 B 中元素的位置，所以这个问题又可以变成，在数组 A 中寻找一个元素，使得这个元素满足以下条件：通过该元素可以确定一个数组 B 中的元素，把两个数组分成左右两部分，这两部分中，左边的最大值小于右边的最小值。

为什么二分查找可以用来解决这个问题呢？主要原因还是，二分是在一个有序的列表中查找符合某一个条件的元素，那么不论是和某个值相等，还是像这个题目一样，要求左边元素全都小于右边的元素，都是可以实现的。当然，在实际解体中，还要注意数值的取值范围，防止出现因为各种边界条件导致的错误。

## 题解

### 采用二分解法

主要是利用二分查找的方式在较短的数组中寻找符合要求的分割位置，当满足要求时，返回中位数；不满足的时候，根据左侧应该小于右侧的原则，去确定答案在那个区间中。

```JavaScript
var findMedianSortedArrays = function (nums1, nums2) {

    // 对元素最少的数组进行二分，减少计算
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);

    const m = nums1.length, n = nums2.length;
    // left 和 right 是要二分的数组的范围，第一个数组可以分割成 m + 1 种可能的情况。
    let left = 0, right = m;
    // midL 是左侧最大的数，midR是右侧最小的数
    let midL = 0, midR = 0;

    // left 和 right 相等是为了处理某一个数组为空的特殊情况
    while (left <= right) {
        // i 指向的是二分后数组开始的元素，
        let i = Math.floor((right + left) / 2);
        // 推导和化简如何计算另一个数组要选择的分割点，根据左侧元素和右侧相等或多一个的原则
        // let j = n - (Math.floor((m + n) / 2) - (m - i));
        // 2 + 3 - Math.floor((2 + 3) / 2) === 3 === Math.floor((2 + 3 + 1) / 2) === Math.cell((2 + 3) / 2)
        let j = Math.ceil((n + m) / 2) - i;
        // il 和 ir 是第一个数组的二分的数值，由于数组是有序的，那么 il 就是第一个数组左边最大的数，ir 就是右边最小的数。jl 和 jr 同理。
        // Infinity 和 -Infinity 分别是 JavaScript 里面的最大数和最小数，用来适配当二分位置旁边元素为空的时候的处理方法。
        const il = i === 0 ? -Infinity : nums1[i - 1];
        const ir = i === m ? Infinity : nums1[i];
        const jl = j === 0 ? -Infinity : nums2[j - 1];
        const jr = j === n ? Infinity : nums2[j];
        midL = Math.max(il, jl);
        midR = Math.min(ir, jr);
        // 根据二分查找，可以得到下面的解法
        if (midL <= midR) {
            return (m + n) % 2 ? midL : (midL + midR) / 2;
        } else {
            // jl 应该小于 ir，此时证明了 ir 过小，所以要找到数在右侧区间
            if (jl > ir) {
                left = i + 1;
            } else {
                right = i - 1;
            }
        }
    }
};

```

### LeetCode 中文官方题解

上面的解法比较容易理解，就是非常明显的二分查找的方式，但是官方的题解，就有一点难度了。

首先需要分析正确分割数组后的情况，当 i、j 作为两个数组的分界点在正确的位置的时候，分割点两边的数一定符合特定的状态：il <= ir; il <= jr; jl <= jr; jl <= ir。而且根据两个数组递增的特性，只有在正确分割数组时，才能符合上面的状态。也就是说，符合题解的 i 一定是符合条件 il <= ir 的最大值，j 一定是符合条件 jl <= ir 的最大值。

详细解释：假设此时的 i 和 j 都是符合最终条件的值，那么当 i = i + 1 时，需要满足条件 ++il <= --jr 才能成立,因为 ++il = ir 和 --jr = jl 条件可以转换成 ir <= jl，这显然与分割点左边全部小于等于右边的条件相悖。所以一定没有比符合条件的 i 值更大的。对于条件 jl <= ir 中的 j 是最小值，同理可以转换成 jr <= il 也是相悖的。

理清上面的思路后，就可以进行解题：

```JavaScript

var findMedianSortedArrays = function (nums1, nums2) {
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);

    const m = nums1.length, n = nums2.length;
    let left = 0, right = m;
    let midL = 0, midR = 0;

    while (left <= right) {

        let i = Math.floor((right + left) / 2);
        let j = Math.ceil((n + m) / 2) - i;
        const il = i === 0 ? -Infinity : nums1[i - 1];
        const ir = i === m ? Infinity : nums1[i];
        const jl = j === 0 ? -Infinity : nums2[j - 1];
        const jr = j === n ? Infinity : nums2[j];

        // 此时的二分查找条件变成了寻找最大的满足 il <= jr 条件的 i 的值，并且由于为了防止进入死循环，对 left 进行了强制右移
        // 所以需要将每个满足条件的两端的最值进行记录，这样即便由于 i 刚好在在过程中变成中值导致的无解的情况进行了规避
        if (il <= jr) {
            midL = Math.max(il, jl);
            midR = Math.min(ir, jr);
            left = i + 1;
        } else {
            right = i - 1;
        }

        // jl <= ir 作为条件的题解

        // if (jl <= ir) {
        //     midL = Math.max(il, jl);
        //     midR = Math.min(ir, jr);
        //     right = i - 1;
        // } else {
        //     left = i + 1;
        // }
    }

    return (m + n) % 2 ? midL : (midL + midR) / 2;
};
```

### 改进过的二分解法

不过上面的解题方式和逻辑多少有些绕弯，其实我们只要运用二分查找每次都一定可以排除一半的不符合要求的数组元素就可以了。当发生条件 jl > ir 时，一定证明了此时 i 选的过小，那么正确的 i 一定在区间 [i + 1, right] 之间，对 left 进行修改就可以了。

同理，当 il > jr 发生时，一定是 i 选的过大了，此时就在区间 [left, i - 1] 之中，但是要注意的是，由于对中位数 i 的区间变化变成了 - 1 的情况，为了防止死循环和 i 出现负值的情况，此时计算中位数时应该使用 left + right + 1 的方式计算，或者直接使用 `Math.ceil()` 函数

```JavaScript

var findMedianSortedArrays = function (nums1, nums2) {
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);

    const m = nums1.length, n = nums2.length;
    let left = 0, right = m;
    let midL = 0, midR = 0;
    while (left < right) {
        let i = Math.floor((right + left) / 2);
        let j = Math.ceil((n + m) / 2) - i;

        const il = i === 0 ? -Infinity : nums1[i - 1];
        const ir = i === m ? Infinity : nums1[i];
        const jl = j === 0 ? -Infinity : nums2[j - 1];
        const jr = j === n ? Infinity : nums2[j];

        if (jl > ir) {
            left = i + 1;
        } else {
            right = i;
        }
    }
    let i = left;
    let j = Math.ceil((n + m) / 2) - i;

    const il = i === 0 ? -Infinity : nums1[i - 1];
    const ir = i === m ? Infinity : nums1[i];
    const jl = j === 0 ? -Infinity : nums2[j - 1];
    const jr = j === n ? Infinity : nums2[j];
    midL = Math.max(il, jl);
    midR = Math.min(ir, jr);

    return (m + n) % 2 ? midL : (midL + midR) / 2;
};

```

## 感悟

在这个题目中，最大的感受是二分的用法扩展。二分不只是单纯的在一个数组中查找某个数或者字符串，而是通过尝试深刻的理解到了，只要有有序数组和分割条件，就能进行二分查找。

## 参考连接

- [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

- [寻找两个有序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/solution/xun-zhao-liang-ge-you-xu-shu-zu-de-zhong-wei-s-114/)
