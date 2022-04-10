# 第一课

## STL 六大组件

### 六大组件内容

* 容器（Containers）
* 分配器（Allocators）
* 算法（Algorithms）
* 迭代器（Iterators）
* 适配器（Adaptors）
* 仿函数（Functors）

### 六大组件之间的关系

Container 通过 Allocator 取得数据储存空间

Algorithm 通过 Iterator 存取 Container  的内容，完成算法

Functor 可以协助 Algorithm 完成不同的策略变化

Adapter 可以修饰 Container，Iterator，Functor

![image-20220409222340068](images\image-20220409222340068.png)

```c++
#include <vector>
#include <algorithm>
#include <functional>
#include <iostream>

using namespace std;

int main(){
    int ia[6] = {27, 210, 12, 47, 109, 83};
    vector<int, allocator<int>> vi(ia, ia + 6);	// allocator<int>指明这个容器使用哪一个分配器，可以不声明，会默认选择
    
    cout << count_if(vi.begin(), vi.end(), 
                    not1(bind2nd(less<int>(), 40)));	// less<int>()：仿函数，比较两数大小，a有没有小于b
    													// bind2nd： 函数适配器，把less<int>()第二个参数绑定为40，a有没有小于40
    													// not1：函数适配器，a有没有大于等于40
    
    return 0;
}
```

