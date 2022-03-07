# 剑指offer

***



## 面试题4：二维数组中的查找

### 题目

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个 整数，判断数组中是否含有该整数。

例如下面的二维数组就是每行、每列都递增排序。如果在这个数组中查找数字7，则返回true；如果查找数字5，由于数组中不含有该数字，则返回false

```C++
1  2  8 9

2  4  9 12

4  7 10 13

6  8 11 15
```

### 解题思路

首先选取数组中右上角的数字。

如果该数字等于要查找的数字，则查找过程结束。

如果该数字大于要查找的数字，说明要查找的数字只能在该数字左边或者上边，则剔除该数字所在的列。

如果该数字小于要查找的数字，说明要查找的数字只能在该数字右边或者下边，则剔除该数字所在的行。

不断缩小范围，直至找到要查的数字或者查找范围为空。

### 代码实现

```c++
bool findNum(int a[][], int rows, int columns, int number){
    if(a != nullptr && rows > 0 && columns > 0){
        int row = 0, column = columns - 1;
        while(row < rows && column >= 0){
            if(a[row][column] == number){
                return true;
            }
            else if(a[row][column] > number){
                column--;
            }
            else{
                row++;
            }
        }
        return false;
    }
}
```

***

## 面试题5：替换空格

### 题目

请实现一个函数，把字符串中的每个空格替换成"%20"。例如输入“We are happy.”，则输出“We%20are%20happy.”。

### 解题思路

> 最直观的做法是从头到尾扫描字符串，每次碰到空格就进行替换。由于是1个字符替换成3个，每替换一次，必须把空格后面的所有字符向后移动2位。![image-20220223163636530](/images/image-20220223163636530.png)
>
> 假设字符串的长度是n 。对每个空格字符，需要移动后面O(n) 个字符，因此对于含有O(n)个空格字符的字符串而言， 总的时间效率是O(n<sup>2</sup>)

可以发现上图中深色部分向后移动了两次，换一种思路，把从前往后替换改成从后往前替换。

先遍历一遍字符串，统计其中空格的数量，由此来计算新的字符串的长度。

准备两个指针，P1指向原始字符串的末尾，P2指向新字符串的末尾，逐步向前移动两个指针，将P1指向的内容拷贝到P2中，直至遇到一个空格。

遇到一个空格后，P1向前移动一格，在P2位置处插入%20。

重复上述步骤直至P1 = P2。

 ![image-20220223165733077](/images/image-20220223165733077.png)

### 代码实现

```c++
void replaceBlank(char str[], int length){
    if(str == nullptr || length <= 0)
        return;
    int blankCount = 0;
    for(int i = 0; i < length; i++){
        if(str[i] == ' '){
            blankCount++;
        }
    }
    int newLength = length + 2 * blankCount;
    char *p1 = str + length - 1;
    char *p2 = str + newLength - 1;
    while(p1 != p2){
        if(*p1 != ' '){
            *p2 = *p1;
            p1--;
            p2--;
        }
        else{
            *p2 = '0';
            *--p2 = '2';
            *--p2 = '%';
            p1--;
            p2--;
        }
    }
}

int main(){
    char str[100] = "We are happy.";
    replaceBlank(str, 14);
    cout << str << endl;

    return 0;
}
```

***

## 面试题6：从尾到头打印链表

### 题目

输入一个链表的头节点，从尾到头反过来打印出每个节点的值。链表节点定义如下：

```c++
struct ListNode{
    int m_nKey;
    ListNode* m_pNext;
};
```

### 解题思路

第一个遍历到的节点最后一个输出，而最后一个遍历到的节点第一个输出。这是典型的先进后出结构，我们可以利用栈或者递归来实现这种结构。但是值得注意的是递归实现的代码虽然简单，当链表非常长的时候，就会导致函数调用的层级很深，从而有可能导致函数调用栈溢出。

### 代码实现

```c++
void PrintListReversingly_Iteratively(ListNode *pHead){
    stack<ListNode*> stck;
    ListNode *p = pHead;
    while(p != nullptr){
        stck.push(p);
        p = p->m_pNext;
    }
    while(!stck.empty()){
        cout << stck.top()->m_nKey << " ";
        stck.pop();
    }
}

void PrintListReversingly_Recursively(ListNode* pHead){
    ListNode *p = pHead;
    if(p == nullptr) return;
    PrintListReversingly_Recursively(p->m_pNext);
    cout << p->m_nKey << " ";

}
```

***

## 面试题7：重建二叉树

### 题目

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1, 2, 4, 7, 3, 5, 6, 8}和中序遍历序列{4, 7, 2, 1, 5, 3, 8, 6}，则重建出图所示的二叉树并输出它的头结点。二叉树节点的定义如下：

 ![image-20220226160048561](/images/image-20220226160048561.png)

```c++
struct BinaryTreeNode{
    int m_nValue;
    BinaryTreeNode* m_pLeft;
    BinaryTreeNode* m_pRight;
}
```

### 解题思路

与树相关的题目大多数可以往递归上靠

在二叉树的前序遍历中，第一个数字总是树的根节点的值。但在中序遍历中，根节点的值在序列的中间，其左边是左子树的值，右边是右子树的值。接着可以用同样的方法分别构建左、右子树，也就是说，接下来的事情可以用递归的方法去完成。

### 代码实现

```c++
BinaryTreeNode* construct(int *startPreorder, int *endPreorder, int *startInorder, int *endInorder){
    // 前序遍历的一个数字为根节点，构造根节点
    int rootValue = startPreorder[0];
    BinaryTreeNode *root = new BinaryTreeNode();
    root->m_nValue = rootValue;
    root->m_pLeft = nullptr;
    root->m_pRight = nullptr;
    if(startPreorder == endPreorder){
        if(startInorder == endInorder && *startPreorder == *startInorder)
            return root;
        else{
            cerr << "Invalid Input." << endl;
            return nullptr;
        }
    }
    // 在中序遍历中寻找根节点的位置
    int *rootInorder = startInorder;
    while(*rootInorder != rootValue && rootInorder <= endInorder)
        rootInorder++;
    if(rootInorder > endInorder){   //在中序遍历中未找到根节点
        cerr << "Invalid Input." << endl;
        return nullptr;
    }
    // 计算左、右子树的长度
    int leftLength = rootInorder - startInorder;
    int rightLength = endInorder - rootInorder;
    // 构建左子树
    if(leftLength > 0)
        root->m_pLeft = construct(startPreorder+1, startPreorder+leftLength, startInorder, rootInorder-1);
    // 构建右子树
    if(rightLength > 0)
        root->m_pRight = construct(startPreorder+leftLength+1, endPreorder, rootInorder+1, endInorder);

    return root;

}

BinaryTreeNode* constructBtree(int *preorder, int *inorder, int length){
    if(preorder == nullptr || inorder == nullptr || length <= 0){
        return nullptr;
    }
    return construct(preorder, preorder + length - 1, inorder, inorder + length - 1);
}

int main(){
    const int length = 8;
    int preorder[length] = {1, 2, 4, 7, 3, 5, 6, 8};
    int inorder[length] = {4, 7, 2, 1, 5, 3, 8, 6};
    BinaryTreeNode* root = constructBtree(preorder, inorder, length);
    PrintTree(root);
    DestroyTree(root);
}
```

## 面试题8：二叉树的下一个节点

### 题目

给定一棵二叉树和其中的一个节点，如何找出中序遍历序列的下一个节点？树中的节点除了有两个分别指向左、右子节点的指针，还有一个指向父节点的指针。

### 解题思路

 ![image-20220227164117831](/images/image-20220227164117831.png)

考虑下面三种情况：

如果一个节点有右子树，则它的下一个节点就是它右子树中的最左子节点。

如果一个节点没有右子树，并且它是其父节点的左子节点，则它的下一个节点就是它的父节点。

如果一个节点没有右子树，并且它是其父节点的右子节点（如图中的i，g），则可以沿着指向父节点的指针一直向上遍历，直到找到一个节点是它父节点的左子节点，则这个节点的父节点就是我们要找的下一个节点。

### 代码实现

```c++
BinaryTreeNode* getNexeNode(BinaryTreeNode *pNode){
    if(pNode == nullptr) return nullptr;
    if(pNode->m_pRight == nullptr){                     // 该节点不具有右子树
        if(pNode->m_pParent == nullptr){                // 该节点没有父节点，即为根节点
            return nullptr;
        }
        else if(pNode->m_pParent->m_pLeft == pNode){    // 该节点为父节点的左子节点
            return pNode->m_pParent;
        }
        else{                                           // 该节点为父节点的右子节点
            BinaryTreeNode *parentNode = pNode->m_pParent;
            while(parentNode->m_pParent != nullptr && parentNode->m_pParent->m_pRight == parentNode){
                parentNode = parentNode->m_pParent;
            }
            return parentNode->m_pParent;
        }

    }
    else{   											//该节点有右子树
        BinaryTreeNode *nextNode = pNode->m_pRight;
        while(nextNode->m_pLeft != nullptr){
            nextNode = nextNode->m_pLeft;
        }
        return nextNode;
    }
}
```

***

## 面试题9：用两个栈实现队列

### 题目

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数appendTail和deleteHead，分别完成在队列尾部插入节点和在队列头部删除节点的功能。

```c++
template<typename T> class CQueue{
public:
    CQueue(void);
    ~CQueue(void);
    
    void appenTail(const T& node);
    T deleteHead();
private:
    stack<T> stack1;
    stack<T> stack2;
};
```

### 解题思路

从队列的声明中可以看出，一个队列包含了两个栈stack1和stack2，因此这道题的意图是要求我们操作这两个“先进后出”的栈实现一个“先进先出”的队列。

插入一个元素的步骤：
将插入的元素压入stack1中，则stack1中最底端的元素就是队列的头部。

删除一个元素的步骤：
当stack2不为空时，在stack2中的栈顶元素是最先进入队列的元素，可以弹出。
当stack2为空时，我们把stack1中的元素逐个弹出并压入stack2，由于先进入队列的元素被压入stack1的底端，经过弹出和压入操作后就储与stack2的顶端，可以弹出。

 ![image-20220302211220257](/images/image-20220302211220257.png)

### 代码实现

```c++
template<typename T>void CQueue<T>::appenTail(const T& node){
    stack1.push(node);
}

template<typename T>T CQueue::deleteHead(){
    if(stack2.empty()){
        while(!stack1.empty()){
            T &obj = stack1.top();
            stack1.pop();
            stack2.push(obj);
        }
    }
    if(stack2.empty()){
        throw new exception("queue is empty");
    }
    T ret = stack2.top();
    stack2.pop();
    return ret;
}
```

***

## 面试题10：求斐波那契数列的第n项

### 题目

写一个函数，输入n，求斐波那契数列的第n项。斐波那契数列的定义如下：
$$
f(n) =
\begin{cases}
0,& \text n=0\\
1,& \text n=1\\
f(n-1)+f(n-2),& \text n>2
\end{cases}
$$

### 解题思路

>最直观的解法是采用递归的方法
>
>```c++
>long long Fibonacci(unsigned int n){
>    if(n <= 0)
>        return 0;
>    if(n == 1)
>        return 1;
>    return Fibonacci(n - 1) + Fibonacci(n - 2);
>}
>```
>
>这种解法存在严重的效率问题，以求解f(10)为例，可以用树形结构来表示这种依赖关系，如图
>
> ![image-20220304172107092](https://github.com/nixuebinCode/C-Study-Note/blob/main/%E5%89%91%E6%8C%87Offer%E5%88%B7%E9%A2%98%E7%AC%94%E8%AE%B0/images/image-20220304172107092.png)
>
>我们不难发现，在这棵树中有很多节点是重复的，而且重复的节点数会随着n的增大而急剧增加

重复的计数太多，只要想办法避免重复计算就行了，可以把已经得到的数列中间项保存起来。

### 算法实现

```c++
long long Fabonacci(unsigned n){
    long long prePre = 0, pre = 1, ret = 0;
    if(n == 0)
        return prePre;
    if(n == 1)
        return pre;
    for(unsigned i = 0; i != n - 1; i++){
        ret = prePre + pre;
        prePre = pre;
        pre = ret;
    }
    return ret;
}
```

***

## 面试题11：旋转数组的最小数字

### 题目

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组{3，4，5，1，2}为{1，2，3，4，5}的一个旋转，该数组的最小值为1。

### 解题思路

旋转之后的数组实际上可以划分为两个排序的子数组，最小的元素刚好是这两个子数组的分界线，可以用二分查找法的思路来寻找这个最小的元素。

我们用两个指针分别指向数组的第一个元素和最后一个元素，如果发生了旋转，第一个元素应该大于等于最后一个元素。接着我们可以利用两个指针找到数组中间的元素

* 如果该元素大于或者等于第一个指针指向的元素，则说明该中间元素位于前面的递增序列，可以让第一个指针指向这个中间元素。
* 如果该元素小于或者等于第二个指针指向的元素，则说明该中间元素位于后面的递增序列，可以让第二个指针指向这个中间元素。

按照上述思路，第一个指针总是指向前面的递增序列，第二个指针总是指向后面的递增序列，最终第一个指针将指向前面的递增序列的最后一个元素，第二个指针将指向后面的递增序列的第一个元素。也就是它们最终会指向两个相邻的元素，而第二个指针指向的刚好是最小的元素。

 ![image-20220306203814878](/images/image-20220306203814878.png)

按照定义还有一个特例：如果把排序数组的前面的0个元素搬到最后面，即排序数组本身，这仍是数组的一个旋转，此时第一个元素小于最后一个元素，直接返回第一个元素即可。

此外，如果第一个指针和第二个指针指向的元素，以及它们中间的元素都相等，我们则无法确定中间的元素是位于第一个递增序列，还是第二个递增序列，也就无法移动两个指针来缩小查找的范围，此时，我们不得不采用顺序查找的方法。

 ![image-20220306204542755](/images/image-20220306204542755.png)

### 代码实现

```c++
int MinInorder(int *numbers, int low, int high){
    int minVal = numbers[low];
    for(int i = low + 1; i != high + 1; i++){
        if(numbers[i] < minVal)
            minVal = numbers[i];
    }
    return minVal;
}

int Min(int *numbers, int length){
    if(numbers == nullptr || length <=0)
        cerr << "Invalid parameters";
    int low = 0, high = length - 1, mid;
    if(numbers[0] < numbers[length - 1]){   // 旋转数组是排序数组本身
        return numbers[0];
    }
    while(low + 1 != high){
        mid = (low + high) / 2;
        if(numbers[mid] == numbers[low] &&
        numbers[mid] == numbers[high]){
            return MinInorder(numbers, low, high); // 当无法判断中间的元素位于哪一个递增序列时，对剩余序列进行顺序查找
        }
        else if(numbers[mid] >= numbers[low]){
            low = mid;
        }
        else if(numbers[mid] <= numbers[high]){
            high = mid;
        }
    }
    return numbers[high];
}
```

***

## 面试题12：矩阵中的路径

### 题目

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3x4 的矩阵中包含一条字符串" bfce" 的路径（路径中的字母用下画线标出） 。但矩阵中不包含字符串"abtb" 的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子

a  <u>b</u>  t g
c  <u>f</u>  <u>c</u>  s
j  d  <u>e</u> h

### 解题思路

这是一个可以用回溯法解决的典型题。首先，在矩阵中任选一个各自作为路径的起点。假设矩阵中某个格子的字符为ch， 并且这个各自将对应于路径上的第i个字符。

* 如果路径上的第i个字符不是ch，那么这个格子不符合条件。
* 如果路径上的第i个字符正好是ch，那么到相邻的格子寻找路径上的第i+1个字符，除矩阵边界上的格子之外，其他格子都有4个相邻的格子

重复上述过程，直到路径上的所有字符都在矩阵中找到相应位置。此外由于路径中不能有重复的格子，还需要定义和字符矩阵大小一样的布尔值矩阵，用来标识路径是否已经进入了每个格子。

### 代码实现

```c++
bool hasPathCore(const char *matrix, int rows, int cols, int row, int col,
                    const char *str, int &pathLength, bool *visited);

bool hasPath(const char *matrix, int rows, int cols, const char *str){
    if(matrix == nullptr || rows < 1 || cols < 1 || str == nullptr)
        return false;
    int pathLength = 0;                         // 已找到的路径长度
    bool *visited = new bool[rows * cols];      // 记录该格子是否已经在路径中
    memset(visited, 0, rows * cols);           

    for(int row = 0; row < rows; row++){
        for(int col = 0; col < cols; col++){
            if(hasPathCore(matrix, rows, cols, row, col, str, pathLength, visited)){
                return true;
            }
        }
    }
    delete[] visited;
    return false;
}

bool hasPathCore(const char *matrix, int rows, int cols, int row, int col,
                    const char *str, int &pathLength, bool *visited){
    if(pathLength == strlen(str)){              // 设置递归终止条件：已找到所有路径
        return true;
    }
    bool hasPath = false;
    // 如果当前格子的字符与对应的路径上的字符相等，则找下一个格子
    if(row >= 0 && row < rows && col >= 0 && col < cols 
        && matrix[row * cols + col] == str[pathLength] && !visited[row * cols + col])
    {
        visited[row * cols + col] = true;   // 设置当前格子已经访问过
        pathLength++;
        // 开始寻找相邻的格子是否满足条件                  
        hasPath = hasPathCore(matrix, rows, cols, row - 1, col, str, pathLength, visited)
                    || hasPathCore(matrix, rows, cols, row + 1, col, str, pathLength, visited)
                    || hasPathCore(matrix, rows, cols, row, col - 1, str, pathLength, visited)
                    || hasPathCore(matrix, rows, cols, row, col + 1, str, pathLength, visited);
        if(!hasPath){            // 相邻的四个格子都不符合下一个字符，则回溯到前一个字符，重新定位
            --pathLength;
            visited[row * cols + col] = false;
        }        
    }
    return hasPath;
}
```

