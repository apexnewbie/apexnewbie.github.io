---
layout: post
author: Aki
tags: [Array, バグ]
---

### 二分查找基本步骤

1. 初始化：首先，确定要查找的有序数据集合。确保其中的元素按照升序或者降序排列。

2. 确定查找范围：将整个有序数组集合的查找范围确定为整个数组范围区间，即 `leaf` & `right`

3. 计算中间元素：`mid = (left + right) / 2`

4. 比较中间元素: 将目标元素 `target` 与中间元素 `nums[mid]` 进行比较

    - 如果 target == nums[mid]，说明找到 target，因此返回中间元素的下标位置 mid。
    - 如果 target < nums[mid]，说明目标元素在左半部分[left,mid-1]，更新右边界为中间元素的前一个位置，即 right = mid -1。
    - 如果 target > nums[mid]，说明目标元素在右半部分[mid+1,right]，更新左边界为中间元素的后一个位置，即 left = mid +1。

5. 重复步骤3~4，直到找到目标元素时返回中间元素下标位置，或者查找范围缩小为空(左边界大于右边界)，表示目标元素不存在，此时返回 -1。

### 区间的开闭

`left = 0，right = len(nums) - 1`

区间为 `[left, right]`

### 搜索区间范围的选择

- 思路 1：「直接法」—— 在循环体中找到元素后直接返回结果。
- 思路 2：「排除法」—— 在循环体中排除目标元素一定不存在区间。

#### 直接法

直接法思想：一旦我们在循环体中找到元素就直接返回结果。

**与基本步骤完全相同**

**细节:**
- 这种思路是在一旦循环体中找到元素就直接返回。
- 循环可以继续的条件是 left <= right。
- 如果一旦退出循环，则说明这个区间内一定不存在目标元素。


#### 排除法

排除法思想：在循环体中排除目标元素一定不存在区间。

1. 设定左右边界为数组两端，即left=0，right=len(nums)-1，代表待查找区间为 [left,right] (左闭右闭区间)。
2.取两个节点中心位置 mid，比较目标元素和中间元素的大小，先将目标元素一定不存在的区间排除。
3.然后在剩余区间继续查找元素，继续根据条件排除目标元素一定不存在的区间。
4.直到区间中只剩下最后一个元素，然后再判断这个元素是否是目标元素。

```py
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1
        
        # 在区间 [left, right] 内查找 target
        while left < right:
            # 取区间中间节点
            mid = left + (right - left) // 2
            # nums[mid] 小于目标值，排除掉不可能区间 [left, mid]，在 [mid + 1, right] 中继续搜索
            if nums[mid] < target:
                left = mid + 1 
            # nums[mid] 大于等于目标值，目标元素可能在 [left, mid] 中，在 [left, mid] 中继续搜索
            else:
                right = mid
        # 判断区间剩余元素是否为目标元素，不是则返回 -1
        return left if nums[left] == target else -1

```

**细节:**

- 判断语句是 left < right。这样在退出循环时，一定有left == right 成立，此时只需要判断nums[left] 是否为目标元素即可。

- 关于边界设置可以记忆为：只要看到 `left = mid` 就向上取整。或者记为：
    - `left = mid + 1、right = mid` 和 `mid = left + (right - left) // 2` 一定是配对出现的。
    - `right = mid - 1、left = mid 和 mid = left + (right - left + 1) // 2` 一定是配对出现的。


#### 总结：
1. 直接法：适合简单问题
2. 排除法：适合复杂问题

____

### Reference
1. [LeetCode-Py](https://algo.itcharge.cn/)