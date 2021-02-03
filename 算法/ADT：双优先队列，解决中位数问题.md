# ADT：双优先队列，解决中位数问题

> 相关算法题：[480. 滑动窗口中位数](https://leetcode-cn.com/problems/sliding-window-median/)

[TOC]

### 1 思路

需要维护一个数据结构，能够支持

* `void insert(int num)`
* `void remove(int num)`
* `double getMedian()`

因为是中位数，这里用两个优先队列来实现，示例图如下：

![image-20210203220205052](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203220205052.png)

为了方便获取中位数，我们需要指定规则：

1. 总元素个数为偶数时，大根堆的元素个数==小根堆的元素个数
2. 总元素个数为奇数时，大根堆的元素个数==小根堆的元素个数+1

在实现的时候为了方便，我们抽取一个`void balance()`来调整

> 这里设计到的背景知识有：c++中优先队列的使用
>
> [c++标准库 std::priority_queue 优先队列的使用](https://blog.csdn.net/u014779536/article/details/111314643)
>
> [C++优先队列自定义排序总结](https://blog.csdn.net/qq_40691051/article/details/102874220)

### 2 插入

因为是基于优先队列的实现，插入很简单

插入时，只需要比较大根堆的根，决定插入到大根堆还是小根堆即可，然后调用`void balance()`来进行平衡

### 3 删除

因为是基于优先队列的实现，没有提供删除接口，只能删除堆顶的元素，所以使用**懒删除**

在数据结构中维护一个map，`unordered_map<int, int> delay_delete;`其中key为元素，value为应删除的次数

插入时，只需要

1. 比较大根堆的根，决定删除元素在大根堆还是小根堆
2. 如果删除元素在堆顶，直接pop，否则懒删除，`delay_delete[num]++;`
3. 调用`void balance()`来进行平衡

因为使用懒删除，实现起来需要增加一些细节：

* 使用small、big来记录堆的元素个数，而不是使用`size()`
* `balance()`不仅需要保证两个堆的元素平衡，还需要保证顶上元素是有效的，即顶上的元素不能是懒删除的元素

### 4 完整实现

```c++
class Windows {
private:
    // 大顶堆，存小一半的
    priority_queue<int, vector<int>, less<int>> big_heap;
    // 小顶堆，存大一半的
    priority_queue<int, vector<int>, greater<int>> small_heap;
    // 延迟删除
    unordered_map<int, int> delay_delete;
    // 总元素个数
    int n = 0;
    int small = 0;
    int big = 0;

    template<typename T>
    void clear_top(T &heap) {
        if(heap.empty()){
            return;
        }
        while (delay_delete[heap.top()] != 0) {
            delay_delete[heap.top()]--;
            heap.pop();
            if (heap.empty()) {
                break;
            }
        }
    }

    // 保证balance以后是平衡、且顶上元素是有效的
    void balance(){
        // 调整后需要满足 big==small 或者 big==small+1
        while (big>small+1){
            clear_top(big_heap);
            int tmp = big_heap.top();
            big_heap.pop();
            small_heap.push(tmp);
            big--;
            small++;
        }
        while (big<small){
            clear_top(small_heap);
            int tmp = small_heap.top();
            small_heap.pop();
            big_heap.push(tmp);
            big++;
            small--;
        }
        clear_top(small_heap);
        clear_top(big_heap);
    }

public:
    void insert(int num) {
        n++;
        if (big_heap.empty()||num <= big_heap.top()) { // 情况1：直接加在大顶堆
            big_heap.push(num);
            big++;
        } else { // 情况2：加在小顶堆
            small_heap.push(num);
            small++;
        }
        balance();
    }

    void remove(int num) {
        n--;
        delay_delete[num]++;
        if (num <= big_heap.top()) { // 情况1：大顶堆里删
            big--;
            if(big_heap.top()==num){
                delay_delete[num]--;
                big_heap.pop();
            }
        } else { // 情况2：小顶堆里删
            small--;
            if(small_heap.top()==num){
                delay_delete[num]--;
                small_heap.pop();
            }
        }
        balance();
    }

    double getMedian() {
        if(n%2==1){
            return big_heap.top();
        }else{
            return ((double)big_heap.top()+(double)small_heap.top())/2;
        }
    }
};
```

### 5 解决leetcode-408

```c++
class Solution {
public:
    vector<double> medianSlidingWindow(vector<int> &nums, int k) {
        vector<double> res;
        Windows windows;
        int left=0,right=k;
        for (int i = 0; i < k; ++i) {
            windows.insert(nums[i]);
        }
        res.push_back(windows.getMedian());
        while (right<nums.size()){
            windows.remove(nums[left++]);
            windows.insert(nums[right++]);
            res.push_back(windows.getMedian());
        }
        return res;
    }
};
```



