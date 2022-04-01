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
> ![image-20220304172107092](/images/image-20220304172107092.png)
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

***

## 面试题13：机器人的运动范围

### 题目

地上有一个m行n列的方格。一个机器人从坐标(0, 0)的格子开始移动，它每次可以向左、右、上、下移动一格，但不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入放个(35, 37)，因为3+5+3+7=18。但它不能进入方格(35, 38)，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

### 解题思路

与12题类似，属于典型的回溯法求解问题，不同的是，这里机器人的初始位置是给定了的，即坐标(0, 0)，只需要依次检查每个格子相邻的格子是否满足要求，如果满足要求，则将最终结果+1。

### 代码实现

```c++
int countGridCore(int threshold, int rows, int cols, int row, int col, bool *visited);

int calDigit(int num){
    if(num / 10 == 0)
        return num;
    return num % 10 + calDigit(num / 10);
}

int countGrid(int threshold, int rows, int cols){
    if(rows <=0 || cols <= 0){
        return 0;
    }
    bool *visited = new bool[rows * cols];
    memset(visited, 0, rows * cols);
    int count = countGridCore(threshold, rows, cols, 0, 0, visited);

    delete[] visited;
    return count;
}

int countGridCore(int threshold, int rows, int cols, int row, int col, bool *visited){
    int count = 0;
    if(row >= 0 && row < rows && col >= 0 && col < cols
        && !visited[row * cols + col] && (calDigit(row) + calDigit(col)) <= threshold){
        visited[row * cols + col] = true;
        count = 1 + countGridCore(threshold, rows, cols, row + 1, col, visited)
                + countGridCore(threshold, rows, cols, row - 1, col, visited)
                + countGridCore(threshold, rows, cols, row, col + 1, visited)
                + countGridCore(threshold, rows, cols, row, col - 1, visited);
    }
    return count;
}
```

***

## 面试题14：剪绳子

### 题目

给你一根长度为n的绳子，请把绳子剪成m段(m，n都是整数，n>1并且m>1)，每段绳子的长度记为k[0]，k[1]，...，k[m]。请问k[0]×k[1]×...×k[m]可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2，3，3的三段，此时得到的最大乘积是18。

### 解法思路一：动态规划

> 如果面试题是求一个问题的最优解（通常是求最大值或者最小值），而且该问题能够分解成若干子问题，并且子问题之间还有重叠的更小的子问题，就可以考虑用动态规划来解决这个问题
>
> 可以应用动态规划求解问题的四个特点
>
> * 整体问题可以分解为若干子问题
> * 整体问题的最优解是依赖各个子问题的最优解
> * 这些子问题之间还有相互重叠的更小的子问题
> * 从上往下分析问题，从下往上求解问题，并把已经解决的子问题的最优解存储下来

首先定义函数f(n)为把长度为n的绳子剪成若干段后，各段长度乘积的最大值。

在剪第一刀的时候，剪出来的第一段绳子可能的长度为1,2,3...,n-1，因此f(n) = max( f(i) × f(n-i) )，0<i<n。

根据动态规划问题的第四个特点，我们从下往上求解问题，也就是说我们先得到f(2)，f(3)，再得到f(4)，f(5)，直到得到f(n)：

* n=2时，绳子只可能剪成长度为1的两端，因此f(2)=1
* n=3时，可能把绳子剪成长度分别为1和2的两段，或者长度都为1的三段，由于1×2>1×1×1，因此f(3)=2
* 当计算f(4)时，依次计算f(1)×f(3)，f(2)×f(2)，取最大值
* 当计算f(5)时，依次计算f(1)×f(4)，f(2)×f(3)，取最大值
* 当计算f(6)时，依次计算f(1)×f(5)，f(2)×f(4)，f(3)×f(3)，取最大值
* ...当计算f(i)时，依次计算f(1)×f(i-1)，f(2)×f(i-2)，...，f(i/2)×f(i-i/2)，取最大值

### 代码实现

```c++
int maxProduct(int length){
    if(length < 2)
        return 0;
    if(length == 2)
        return 1;
    if(length == 3)
        return 2;
    int *product = new int[length + 1];     // 数组中第i个元素，表示把长度为i的绳子剪成若干段之后各段长度乘积的最大值，即f(i)
    product[0] = 0;
    product[1] = 1;
    product[2] = 2;                         
    product[3] = 3;
    int max, temp = 0;
    for(int i = 4; i < length + 1; i++){    // 求出f(length)的值，从f(4)开始从下往上算
        max = 0;
        for(int j = 1; j <= i / 2; j++){    // 依次计算f(1)×f(i-1)，f(2)×f(i-2)，...，f(i/2)×f(i-i/2)，取最大值
            temp = product[j] * product[i - j];
            if(max < temp)
                max = temp;
        }
        product[i] = max;
    }
    delete[] product;
    return product[length];
}
```

**这里需要注意的是，对于product[2]，因为题目要求至少剪一刀，因此f(2)应当返回1，但是实际上我们可以不剪，即2>1×1，因此product[2] = 2；同理f(3)也可以不剪，3>f(3)，因此product[3] = 3**

***

## 面试题15：二进制中1的个数

### 题目

请实现一个函数，输入一个整数，输出该数二进制表示中1的个数。例如，把9表示成二进制是1001，有2位是1。因此，如果输入9，则该函数输出2。

### 解题思路

#### 思路一

先判断整数二进制表示中最右边一位是不是1；接着把整数右移一位，此时原来处于从右边数第二位被移到最右边了，再接着判断最右边一位是不是1。
因此需要解决的是如何判断整数最右边1位是不是1，只需把整数与1相与，1除了最右边1位是1，其余高位都是0，若得到的结果是1，则说明整数最右边1位是1。

```c++
int NUmberOf1(int n){
    int count = 0;
    while(n){
        if(n & 1)
            count++;
        n >> 1;
    }
    return count;
}
```

**存在的问题**

若输入的整数n为负数，对负数进行右移操作，最高位会补1，如果一直做右移运算，整数n会变成0xFFFFFFFF，程序进入死循环。

#### 思路二

因为要用整数和1进行与操作，既然右移整数会可能会导致死循环，我们相应地左移1就可以了，依次检测整数的每一位是否为1。

```c++
int NUmberOf1(int n){
    unsigned int flag = 1;
    int count = 0;
    while(flag){
        if(n & flag)
            count++;
        flag << 1;
    }
    return count;
}
```

在这个解法中，循环的次数等于整数n的二进制的位数。

#### 思路三

考虑任何一个整数的二进制减去1，会导致其从右往左的第一个1变为0，这个1右边的0全部变为1，这个1左边的不变。例如：10100 - 1 = 10011

我们将该整数与其减1后得到的结果相与，从右往左的第一个1左边的不变，这个1右边(包括这个1)将变为0。例如10100 & 10011 = 10000

上述操作消除了整数二进制表示中从右往左的第一个1，继续对整数进行上述操作，直至整数变为0，循环的次数为整数中1的个数。

```c++
int NUmberOf1(int n){
    int count = 0;
    while(n){
        count++;
        n = n & (n - 1);
    }
    return count;
}
```

***

## 面试题16：数值的整数次方

### 题目

实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

### 解题思路

> 这道题目看似简单，但是首先要考虑清除各种可能的情况，并不是直接exponent个base相乘就可以。如果exponent是负数怎么办？能想到的解决方法是先对exponent取绝对值，计算完之后再取倒数，那么又很自然地想到如果要对0取倒数怎么办，因此如果base为0，exponent为负数，程序就会出错。
>
> 接下来要考虑的是如何处理错误，一般有以下三种方式
>
> * 函数用返回值来告知调用者是否出错
> * 当错误发生时设置一个全局变量，调用者可以分析这个全局变量，从而得知出错的原因
> * 当函数运行出错时，我们就抛出一个异常，还可以根据不同的出错原因定义不同的异常类型
>
> ![image-20220312100827464](images\image-20220312100827464.png)
>
> 根据上述思路写出代码：
>
> ```c++
> bool equal(double num1, double num2){       // 注意：由于double精度问题，不能直接用==比较两个double
>     if ((num1 - num2 > -0.0000001) && (num1 - num2 < 0.0000001))
>         return true;
>     else
>         return false;
> }
> 
> double PowerPositive(double base, int exponent){
>     double result = 1.0;
>     while(exponent--){
>         result *= base;
>     }
>     return result;
> }
> 
> double Power(double base, int exponent){
>     g_InvalidInput = false;
>     if(equal(base, 0.0) && exponent < 0){   // 底数为0，指数为负数
>         g_InvalidInput = true;
>         return 0.0;
>     }
>     if(exponent < 0){
>         return 1.0 / PowerPositive(base, -exponent);
>     }
>     else{
>         return PowerPositive(base, exponent);
>     }
> }
> ```

题目做到这里，思考一个问题：假如输入的exponent为32，那么我们需要做31次乘法。但是如果我们已经知道了一个数的16次方，那么只需要在16次方的基础上再平方一次就可以了，同理16次方是在8次方的基础上再平方一次就可以了...即有下面的公式：
$$
a^n =\begin{cases}a^{n/2}·a^{n/2},& \text a为偶数\\a·a^{(n-1)/2}·a^{(n-1)/2},& \text a为奇数\end{cases}
$$

### 代码实现

```c++
double PowerPositive(double base, int exponent){
    double result = 1.0;
    if(exponent == 0)
        return 1.0;
    if(exponent == 1)
        return base;
        
    if(exponent % 2){   // exponent为奇数
        result *= base * PowerPositive(base, exponent - 1);
    }
    else{               // exponent为偶数
        result *= PowerPositive(base, exponent / 2);
        result *= result;
    }
    return result;
}
```

另外注意两个细节，当代码中出现%，/的时候可以考虑能否用位运算代替它们，例如可以用exponent >> 1代替exponent / 2。再例如可以利用exponent & 1来判断exponent是奇数还是偶数。

```c++
double PowerPositive(double base, int exponent){
    double result = 1.0;
    if(exponent == 0)
        return 1.0;
    if(exponent == 1)
        return base;

    if(exponent & 1){   // exponent二进制位最后一位是1，即exponent为奇数
        result *= base * PowerPositive(base, exponent - 1);
    }
    else{               // exponent为偶数
        result *= PowerPositive(base, exponent >> 1);
        result *= result;
    }
    return result;
}
```

***

## 面试题17：打印从1到最大的n位数

### 题目

输入数字n，按顺序打印出从1到最大的n位十进制数。比如输入3，则打印出1、2、3一直到最大的3位数999。

### 解题思路一

注意：这里题目中没有给出n的范围，需要考虑n很大，int和long long无法表示的情况，即<font color='red'>大数问题</font>。

解决大数问题，最常用也是最容易的方法是用字符串或数组表示大数，即字符串中的每个字符是'0'~'9'之间的某一个字符，用来表示数字中的一位，当实际数字不够n位的时候，在字符串的前半部分补0。

接下来最重要的是在字符串表达的数字上模拟加法，即每次加1，并且判断该数字什么时候到达了最大数字"99...9"，有以下两种方法：

* 利用库函数strcmp直接比较加1后的字符串和"99...9"，每次比较的时间复杂度为O(n)
* 我们在模拟字符串加法时，需要记录每一位的进位，可以发现只有当字符串表示的数字是"99..99"的时候，再增加1，才会导致第1位（数组下标为0）产生进位，我们可以利用这一点来判断，比较的时间复杂度则为O(1)

### 代码实现

```c++
bool Increment(char *numStr, int n){
    int takeover = 1;   // 进位
    for(int i = n - 1; n != -1; --i){
        

        int sum = numStr[i] - '0' + takeover;
        if(sum < 10){
            numStr[i] = '0' + sum;
            break;
        }
        else{               // 产生进位
            if(i == 0){     // 当第一位产生进位时，说明字符串表示的数字已达99...999
                return false;
            }
            takeover = 1;
            numStr[i] = '0' + sum - 10;
        }
    }
    return true;
}

void PrintToMaxOfNDigits(int n){
    if(n <= 0){
        return;
    }

    // 创建n+1位的字符串以表示n为整数
    char *numStr = new char[n + 1];
    memset(numStr, '0', n);
    numStr[n] = '\0';

    // 每次给数字加1，并打印，直至到达最大的数字
    while(Increment(numStr, n)){
        bool isBeginning0 = true;
        for(int i = 0; i < n; i++){ // 打印字符串表示的数字
            if(numStr[i] == '0' && isBeginning0){
                continue;
            }
            else{
                isBeginning0 = false;
                cout << numStr[i];
            }
        }
        cout << endl;
    }
    delete[] numStr;
}
```

### 解题思路二

如果我们在数字前面补0，就会发现n位所有十进制数其实就是n个从0到9的全排列。也就是说我们把数字的每一位都从0到9排列一遍，就得到了所有的十进制数。

全排列用递归很容易表达，数字的每一位都可能是0~9中的一个数，然后设置下一位。递归结束的条件是我们已经设置了数字的最后一位。

### 代码实现

```c++
void PrintNumber(char* number, int n)
{
    bool isBeginning0 = true;
    for(int i = 0; i < n; i++){ // 打印字符串表示的数字
        if(number[i] == '0' && isBeginning0){
            continue;
        }
        else{
            isBeginning0 = false;
            cout << number[i];
        }
    }
    cout << endl;
}

void Print1ToMaxOfNDigitsRecursively(char* number, int n, int index)    // index表示当前位
{
    if(index == n - 1){             // 如果当前位为最后一位，则打印数字
        PrintNumber(number, n);
        return;
    }
    // 如果当前位不是最后一位，则设置下一位
    for(int i = 0; i < 10; i++){    // 设置数字的当前位为0~9
        number[index + 1] = '0' + i;    
        Print1ToMaxOfNDigitsRecursively(number, n, index + 1); // 设置下一位的下一位
    }
}

void Print1ToMaxOfNDigits_2(int n)
{
    if(n <= 0)
        return;
    char *num = new char[n + 1];

    for(int i = 0; i < 10; i++){    // 设置数字的最高位为0~9
        num[0] = '0' + i;
        Print1ToMaxOfNDigitsRecursively(num, n, 0); // 设置下一位
    }

    delete[] num;
}
```

***

## 面试题18-1：在O(1)时间内删除链表节点

### 题目

给定单向链表的头指针和一个节点指针，定义一个函数在O(1)时间内删除该节点。链表节点与函数的定义如下：

```c++
struct ListNode
{
	int m_nValue;
	ListNode* m_pNext;
};
void DeleteNode(ListNode** pListHead, ListNode* pToBeDeleted);
```

### 解题思路

在单向链表中删除一个节点，常规的做法无疑是从链表的头节点开始，顺序遍历查找要删除的节点，并在链表中删除该节点。这种思路由千需要顺序查找，时间复杂度自然就是O(n ) 了。

换一种思路，我们要删除节点i , 先把 i 的下一个节点 j 的内容复制到 i , 然后把 i 的指针指向节点 j 的下一个节点。此时再删除节点 j 其效果刚好是把节点 i 删除了。

另外需要考虑特殊情况：

* 如果要删除的节点位于链表的尾部， 那么它就没有下一个节点，我们只能从头节点开始顺序遍历
* 如果链表中只有一个节点，而我们又要删除链表的头节点（ 也是尾节点），那么，此时我们在删除节点之后，还需要把链表的头节点设置为nullptr

### 代码实现

```c++
void DeleteNode(ListNode** pListHead, ListNode* pToBeDeleted){
	if(pListHead == nullptr || pToBeDeleted == nullptr)
		return;
	if(pToBeDeleted->m_pNext != nullptr){	// 待删除的节点不是尾节点
		ListNode* next = pToBeDeleted->m_pNext;
		pToBeDeleted->m_nValue = next->m_nValue;
		pToBeDeleted->m_pNext = next->m_pNext;
		delete next;
		next = nullptr;
	}
	else if(*pListHead == pToBeDeleted){	// 待删除的节点是尾节点,且是头节点，即链表只有一个节点
		delete pToBeDeleted;
		pToBeDeleted = nullptr;
		*pListHead = nullptr;
	}	
	else{									// 待删除的节点是尾节点,且不是头节点
		ListNode* pre = *pListHead;
		while(pre->m_pNext != pToBeDeleted)
			pre = pre->m_pNext;
		pre->m_pNext = pToBeDeleted->m_pNext;
		delete pToBeDeleted;
		pToBeDeleted = nullptr;
	}
}
```

***

## 面试题18-2：删除链表中重复的节点

### 题目

在一个排序的链表中，如何删除重复的节点？例如

输入: 1->2->3->3->4->4->5
输出: 1->2->5

输入: 1->1->1->2->3
输出: 2->3

### 解题思路

在处理链表时，有两种情况要考虑进去：第一，需不需要修改头节点；第二待处理节点为链尾节点。**如果需要修改头节点，那么就不能传入pHead*，而要传入pHead****，因为我们要修改的是指向头节点的指针，既然要修改指针，传递的参数必须是指针的指针才行。

我们可以从头遍历链表，用pre指针记录当前遍历节点的前继节点，当当前节点的值与下一个节点的值相等时，就可以让pre指向下一个大于当前节点的节点。

### 代码实现

```c++
void deleteDuplication(ListNode **pHead){
    if(pHead == nullptr)
        return;
    ListNode* pPre = nullptr;   			// 前继指针初始置为nullptr，可以用来判断当前遍历的节点是否为头节点
    ListNode* pNode = *pHead;
    while(pNode != nullptr){
        ListNode* pNext = pNode->m_pNext;
        if(pNext != nullptr && pNext->m_nValue == pNode->m_nValue){ //如果该节点为重复节点
            int dupliVal = pNode->m_nValue; 						// 重复的值
            while(pNode != nullptr && pNode->m_nValue == dupliVal){ // 删除后面所有等于重复值的节点
                delete pNode;
                pNode = nullptr;
                pNode = pNext;
                if(pNext != nullptr)        // 注意这里加非空判断，否则当链表所有节点的值都相同时，pNext可能为空，再取pNext->m_pNext时会报错
                    pNext = pNext->m_pNext;
            }
            if(pPre == nullptr){            // 重复的节点位于头节点
                *pHead = pNode;
            }
            else{
                pPre->m_pNext = pNode;
            }
        }
        else{								// 如果当前节点非重复节点，同时前进pNode和pPre
            pPre = pNode;
            pNode = pNext;
        }
    }
}
```

***

## 面试题19：正则表达式匹配

### 题目

请实现一个函数用来匹配包含'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'\*'表示它前面的字符可以出现任意次(包含0次)。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串“aaa”与模式”a.a“和”ab\*ac\*a“匹配，但与”aa.a“和”ab\*a“均不匹配。

### 解题思路

普通的比较两个字符串是否相等，本质上也是利用了划分的思想，比如比较”abc“和”abde“，先比较两者的第一个字符，相等，则继续比较两个子字符串”bc“和”bde“...**直到两者有一个为空或者都为空**(递归终止条件)。

此题只是在普通的比较上增加了特殊规则而已，假设i和j分别指向字符串和模式串当前待比较的字符。

* 如果模式串的下一个字符P[j+1]不是*，则情况比较简单，只需要比较当前字符串和模式串即可
  * 若当前比较的字符匹配，即S[i] == P[j] || P[j] == .
    * 可以把S[i],P[j]砍掉，继续比较下一个子串
  * 否则
    * 返回false
  
* 如果模式串的下一个字符P[j+1]是*
  * 首先因为*可以表示前面的字符出现0次，那么相当于把P[j]和P[j+1]砍掉，再继续比较字符串与剩余的模式串，比如abc和d\*abd，可以把d\*砍掉，继续比较abc和abd
  
  * 如果不砍掉P[j]和P[j+1]，即*前面的字符至少出现一次，比如abc和a\*bc
    * 如果S[i] == P[j] || P[j] == .
      
      因为*前面的字符可以出现任何次，则将字符串往前前进一个，继续和模式比较，即比较bc和a\*bc
      
    * 否则返回false
  
* 最后考虑递归终止条件：即s或p为空时
  * p为空，s为空
    
    返回true
    
  * p为空，s不为空
    
    返回false

### 代码实现

```c++
bool regularMatch(const char* str, const char* pattern){
    if(str == nullptr || pattern == nullptr)
        return false;

    if(*pattern == '\0')
        return (*str == '\0');

    bool firstMatch = *str != '\0' && (*str == *pattern || *pattern == '.');
    // bool firstMatch = (*str == *pattern || *pattern == '.');			// 注意这里需要加一个*str != '\0'。否则当比较""和".*a"时
    																	// firstMatch会等于true，导致执行regularMatch(str + 1, 																			// pattern)，此时str就会发生越界
    if(*(pattern + 1) != '*'){   // 模式的下一个字符不是*时
        if(firstMatch)
            return regularMatch(str + 1, pattern + 1);
        else
            return false;
    }
    else{                       // 模式的下一个字符是*时
        return (regularMatch(str, pattern + 2))   // *前面的字符出现0次，砍掉pattern的前两个字符
                || (firstMatch && regularMatch(str + 1, pattern));  
    }
}
```

***

## 面试题20：表示数值的字符串

### 题目

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2" 、"-123" 、"3.1416" 及"-1E-16"都表示数值，但"12e" 、"la3.14" 、"l.2.3" 、"+-5"及"12e+5.4"都不是。

### 解题思路

表示数值的字符串应该符合模式 A[ . [B] ]\[ e|E C \]或者[ . [B] ]\[ e|E C \]。其中A为数值的整数部分，B紧跟着小数点为数值的小数部分，C紧跟着e或者E为数值的指数部分，且不能是小数。A和C都可能以'+'或'-'开头，后面跟0~9的数字串，B则是0~9的数字串，不能以正负开头。**在小数里可能没有数值的整数部分。例如， 小数123 等于0.123 ，因此A 部分不是必需的；同理小数里也可能没有数值的小数部分，例如123.等于123.0，因此.B部分也不是必须的。**

判断一个字符串是否符合上述模式时，首先尽可能多地扫描A部分；如果遇到小数点，则开始扫描B部分；如果遇到e或者E，则开始扫描C部分

### 代码实现

```c++
bool scanUnsigned(const char **str);
bool scanInteger(const char **str);

bool isNum(const char *str){
    if(str == nullptr){
        return false;
    }
    // 扫描A部分
    bool numeric = scanInteger(&str);   // true表示包含A部分，false表示不包含A部分
    // 如果出现'.'，则开始扫描B部分
    if(*str == '.'){
        ++str;
        // 下面一行代码用|的原因：
        // 1. 小数可以没有整数部分，如.123等于0.123:
        // 2. 小数点后面可以没有数字，如233.等于233.0
        // 3. 当然，小数点前面和后面可以都有数字，如233.666
        numeric = scanUnsigned(&str) || numeric;
    }
    // 如果出现'e'或者'E'，则开始扫描B部分
    if(*str == 'e' || *str == 'E'){
        ++str;
        // 下面一行代码用＆＆的原因：
        // 1.当e或E前面没有数字时，整个字符串不能表示数宇，如.e1、el;
        // 2.当e或E后面没有整数时，整个字符串不能表示数宇，如12e、12e+5.4
        numeric = scanInteger(&str) && numeric;
    }
    // 当A，B，C部分都没有问题，且已经扫描倒字符串末尾时，返回true
    return numeric && *str == '\0';
}

// 扫描一个字符串里是否包含0~9的数字串（不能以正负开头）
// 由于需要更改实参的指针，因此形参的类型需要为const char**
// 调用完后实参的指针指向字符串中第一个不是数字的字符
bool scanUnsigned(const char **str){
    const char *before = *str;
    while(**str != '\0' && **str >= '0' && **str <= '9')
        (*str)++;     // 当当前字符为0~9时，使指针*str指向下一个字符
    return *str > before;
}

// 扫描一个字符串里是否包含0~9的数字串（可以以正负开头）
bool scanInteger(const char **str){     
    if(**str == '+' || **str == '-')
        (*str)++;           // 不要写成*str++了，记得带括号
    return scanUnsigned(str);
}
```

***

## 面试题21：调整数组顺序使奇数位于偶数前面

### 题目

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

### 解题思路

可以参照快速排序中划分的思想，设置两个指针，初始化时第一个指针指向数组的第一个数字，它只向后移动；第二个指针指向数组的最后一个数字，它只向前移动。在两个指针相遇之前，如果第一个指针指向的数字是偶数，并且第二个指针指向的是奇数，**则交换这两个数字**。

进一步思考，可以将判断数字是否是奇数/偶数当成一个**函数指针**传递给我们的函数，这样今后若有类似的处理，比如把数组中的数字分为两部分，能被3整除的放在不能被3整除的前面，或是所有负数在非负数前面，只需要在调用我们的函数时，修改函数指针的参数即可。

### 代码实现

```c++
bool isEven(int);
void Reorder(int *, unsigned int, bool(*)(int));

void ReorderOddEven(int *pData, unsigned int length){
    Reorder(pData, length, isEven);
}

void Reorder(int *pData, unsigned int length, bool(*func)(int)){
    if(pData == nullptr || length == 0)
        return;
    int *low = pData;
    int *high = pData + length - 1;
    while(low < high){
        while(low < high && !(func(*low))) low++;     // 注意此处要加low < high
                                                        // 否则对于序列1、3、5、7、2、4、6会出问题
        while(low < high && func(*high)) high--;
        if(low < high){
            int temp = *low;
            *low = *high;
            *high = temp;
        }
    }
}

bool isEven(int number){     // 判断一个数字是否是偶数
    return !(number & 1);
}
```

***

## 面试题22：链表中倒数第k个节点

### 题目

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。链表节点定义如下：

```c++
struct ListNode{
    int m_nValue;
    ListNode* m_pNext;
};
```

### 解题思路

设置两个快慢两个指针p，q；q指针始终指向p指针前k个节点，同时移动p指针和q指针，当q指针到达链尾的下一个节点，即nullptr时，此时p指针所指的即为倒数第k个节点。

### 代码实现

```c++
ListNode* FindKthToTail(ListNode *pHead, int k){
    if(pHead == nullptr || k <= 0)
        return nullptr;
    ListNode* pSlow = pHead;
    ListNode* pFast = pSlow;
    // 将pFast指向pSlow后k个位置，注意不能直接写pFast = pSlow + k，这不是数组！
    for(int i = 0; i < k - 1; i++){
        pFast = pFast->m_pNext;
        if(pFast == nullptr)        // 需额外判断当链表长度为3时，要求返回倒数第4个节点的情况
            return nullptr;
    }
    pFast = pFast->m_pNext;

    while(pFast != nullptr){
        pSlow = pSlow->m_pNext;
        pFast = pFast->m_pNext;
    }
    return pSlow;
}
```

***

## 面试题23：链表中环的入口节点

### 题目

如果一个链表中包含环，如何找出环的入口节点？例如，在如图所示的链表中，环的入口节点是节点3.

 ![image-20220320130847830](images\image-20220320130847830.png)

### 解题思路

* 确定一个链表中是否包含环

  定义两个指针，同时从链表的头节点出发，一个指针一次走一步，另一个指针一次走两步。如果走得快的指针追上了走得慢的指针那么链表就包含环；如果走得快的指针走到了链表的末尾都没有追上走得慢的指针，那么链表就不包含环。

* 找到环的入口

  定义两个指针P1和P2指向链表的头节点。如果链表中的环有n个节点，则指针P1先在链表上向前移动n步，然后两个指针以相同的速度向前移动，直到它们相遇，相遇的节点正好是环的入口节点。

   ![](images\微信截图_20220320132314.png)

* 确定环中节点的数目

  在我们确定链表中是否包含环的时候，两个指针相遇的节点一定在环中。可以从这个节点出发，一遍继续向前移动一遍计数，当再次回到这个节点时，就可以得到环中节点数了。

### 代码实现

```c++
struct ListNode
{
	int m_nValue;
	ListNode* m_pNext;
};

// 返回链表中环的节点的数目，若不存在环，则返回-1
int numOfLoop(ListNode* pHead){
    if(pHead == nullptr)
        return -1;
    ListNode* pSlow = pHead;
    ListNode* pFast = pHead;
    ListNode* pNodeOfLoop = nullptr;
    int loopCount = 1;
    while(pFast != nullptr && pFast->m_pNext != nullptr){
        pSlow = pSlow->m_pNext;
        pFast = pFast->m_pNext->m_pNext;
        if(pSlow == pFast){
            pNodeOfLoop = pSlow;
            break;
        }
    }
    if(pNodeOfLoop == nullptr){     // 链表中不存在环
        return -1;
    }
    ListNode *counter = pNodeOfLoop->m_pNext;
    while(counter != pNodeOfLoop){
        loopCount++;
        counter = counter->m_pNext;
    }
    return loopCount;
}

ListNode* entryOfLoop(ListNode* pHead){
    if(pHead == nullptr)
        return nullptr;
    int nodeOfLoopCnt = numOfLoop(pHead);
    if(nodeOfLoopCnt == -1)
        return nullptr;
    ListNode* P1 = pHead;
    ListNode* P2 = pHead;
    for(int i = 0; i < nodeOfLoopCnt; i++){
        P1 = P1->m_pNext;
    }
    while(P1 != P2){
        P1 = P1->m_pNext;
        P2 = P2->m_pNext;
    }
    return P1;
}
```

***

## 面试题24：反转链表

### 题目

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。链表节点定义如下：

```c++
struct ListNode{
    int m_nKey;
    ListNode* m_pNext;
};
```

### 解题思路

为了正确地反转一个链表，需要调整链表中指针的方向。需要注意的是，我们在调整当前节点的next指针时，需要把它指向它的前一个节点，同时需要保存它的下一个节点，否则我们在修改其next指针后，会发生断链。

### 代码实现

```c++
ListNode* reverseLst(ListNode* pHead){
    if(pHead == nullptr){
        return nullptr;
    }
    ListNode* pPre = nullptr;
    ListNode* pNode = pHead;
    ListNode* pNext = pHead->m_pNext;
    while(pNode != nullptr){
        pNode->m_pNext = pPre;
        pPre = pNode;
        pNode = pNext;
        if(pNode == nullptr){   // 当前节点为尾节点
            return pPre;
        }
        pNext = pNode->m_pNext;
    }
    return nullptr;
}
```

***

## 面试题25：合并两个排序的链表

### 题目

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。链表节点定义如下：

```c++
struct ListNode{
    int m_nKey;
    ListNode* m_pNext;
};
```

 ![](images\微信截图_20220322111101.png)

### 解题思路一

使用两个指针分别指向两个链表的头节点，每次将值较小的节点加入最终合并的链表。需要注意两个事情：

* 头节点的处理
* 需要额外用一个指针记录合并链表的尾节点，用以每次加入新的节点

### 代码实现

```c++
ListNode* mergeLst(ListNode* pHead1, ListNode* pHead2){
    if(pHead1 == nullptr)
        return pHead2;
    if(pHead2 == nullptr)
        return pHead1;
    ListNode* ret = nullptr;
    ListNode* tail = nullptr;       // 记录新链表的尾节点
    ListNode* p1 = pHead1;
    ListNode* p2 = pHead2;
    while(p1 != nullptr && p2 != nullptr){
        if(p1->m_nValue < p2->m_nValue){
            if(ret == nullptr)
                ret = p1;
            else
                tail->m_pNext = p1;
            tail = p1;
            p1 = p1->m_pNext;
        }
        else{
            if(ret == nullptr)
                ret = p2;
            else
                tail->m_pNext = p2;
            tail = p2;
            p2 = p2->m_pNext;
        }
    }
    
    if(p1 != nullptr)
        tail->m_pNext = p1;
    if(p2 != nullptr)
        tail->m_pNext = p2;
    return ret;
}
```

### 解题思路二

当我们得到两个链表中值较小的头节点并把它链接到已经合并的链表之后，两个链表剩余的节点依然是排序的，因此合并的步骤和之前的步骤是一样的。这就是典型的递归过程，我们可以定义递归函数完成这一合并过程。

### 代码实现

```c++
ListNode* mergeRecursive(ListNode* pHead1, ListNode* pHead2){
    if(pHead1 == nullptr)
        return pHead2;
    if(pHead2 == nullptr)
        return pHead1;
    ListNode* pMergeNode = nullptr;
    if(pHead1->m_nValue < pHead2->m_nValue){
        pMergeNode = pHead1;
        pMergeNode->m_pNext = mergeRecursive(pHead1->m_pNext, pHead2);
    }
    else{
        pMergeNode = pHead2;
        pMergeNode->m_pNext = mergeRecursive(pHead1, pHead2->m_pNext);
    }
    return pMergeNode;
}
```

***

## 面试题26：

### 题目

输入两棵二叉树A和B，判断B是不是A的子结构。二叉树节点的定义如下：

```c++
struct BinaryTreeNode
{
	double m_dbValue;
	BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
};
```

例如图中的两棵二叉树，由于A 中有一部分子树的结构和B 是一样的，因此B 是A 的子结构。

 ![image-20220324094125264](images\image-20220324094125264.png)

### 解题思路

* 第一步，在树A中找到和树B的根节点的值一样的节点R

  利用递归遍历树A，，若找到节点R，则调用第二步的方法

* 第二步，判断树A中以R为根节点的子树是不是包含和树B一样的结构

  利用递归的思想：递归地判断节点R和树B的左右子节点的值是否相同，递归的终止条件是我们达到了树A或树B的叶子节点。

一个细节值得我们注意：题目中节点的值的类型为double。在判断两个节点的值是不是相等时，不能直接写`pRoot1->m_dbValue == pRoot2->m_dbValue`，这是因为在计算机内表示小数时都有误差。判断两个小数是否相等，只能判断它们之差的绝对值是不是在一个很小的范围内(通常0.0000001之内)

### 代码实现

```c++
bool HasSubTree2(BinaryTreeNode* pRoot1, BinaryTreeNode* pRoot2);

struct BinaryTreeNode
{
	double m_dbValue;
	BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
};

bool equal(double d1, double d2){		// 判断两个double是否相等
    if((d1 - d2) >= -0.0000001 && (d1 - d2) <= 0.0000001){
        return true;
    }
    else{
        return false;
    }
}

bool HasSubTree(BinaryTreeNode* pRoot1, BinaryTreeNode* pRoot2){
    if(pRoot1 == nullptr || pRoot2 == nullptr)
        return false;
    bool ret = false;
    if(equal(pRoot1->m_dbValue, pRoot2->m_dbValue)){	// 如果在A中找到节点R，则进行第二步判断
        ret = HasSubTree2(pRoot1, pRoot2);
    }
    if(!ret){   										// 如果没找到，则递归地查找子树
        ret = HasSubTree(pRoot1->m_pLeft, pRoot2) ||
                 HasSubTree(pRoot1->m_pRight, pRoot2);
    }

    return ret;
}

bool HasSubTree2(BinaryTreeNode* pRoot1, BinaryTreeNode* pRoot2){
    if(pRoot2 == nullptr)
        return true;
    if(pRoot1 == nullptr)
        return false;
    if(equal(pRoot1->m_dbValue, pRoot2->m_dbValue)){
        return HasSubTree2(pRoot1->m_pLeft, pRoot2->m_pLeft) &&
                HasSubTree2(pRoot1->m_pRight, pRoot2->m_pRight);
    }
    else{
        return false;
    }
}
```

***

## 面试题27：二叉树的镜像

### 题目：

请完成一个函数，输入一棵二叉树，该函数输出它的镜像。如图，两棵二叉树互为镜像。二叉树节点定义如下：

```c++
struct BinaryTreeNode
{
	int m_nValue;
	BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
};
```

 ![image-20220325124823049](images\image-20220325124823049.png)

### 解题思路

观察上图，可以发现根节点保持不变，但它的左右两个子节点发生了改变，我们不妨先交换其左右字节点，如图a所示。之后我们注意到节点值为10，6的子节点仍需要交换顺序，交换之后分别是图中第三棵和第四棵树，最后交换完所有非叶子节点的子节点之后，我们就得到了最终的镜像二叉树

 ![image-20220325125301051](images\image-20220325125301051.png)

因此获得一棵二叉树的镜像的过程为：前序遍历这棵树的每个节点，如果遍历到的节点非叶子节点，就交换它的两个子节点。

### 代码实现

```c++
void MirrorRecursively(BinaryTreeNode* pNode){
    if(pNode == nullptr)
        return;
    if(pNode->m_pLeft == nullptr && pNode->m_pRight == nullptr){
        return;
    }

    BinaryTreeNode *temp = pNode->m_pLeft;
    pNode->m_pLeft = pNode->m_pRight;
    pNode->m_pRight = temp;

    if( pNode->m_pLeft != nullptr)
        MirrorRecursively(pNode->m_pLeft);
    if( pNode->m_pRight != nullptr)
        MirrorRecursively(pNode->m_pRight);
}
```

***

## 面试题28：对称的二叉树

### 题目

请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。例如图中所示的3棵二叉树中，第一棵二叉树是对称的，而另外两棵不是。

 ![image-20220327093747080](images\image-20220327093747080.png)

### 解题思路

对于树的题目，多从**遍历**入手，常见的三种遍历算法中都是先遍历左节点，再遍历右节点。我们是否可以定义一种遍历算法，先遍历右节点，再遍历左节点？观察图中对称的二叉树，如第一课其先序遍历序列为{8,6,5,7,6,7,5}，若定义一种遍历顺序为根右左，则其序列为{8,6,5,7,6,7,5}。可以发现两者序列一致。因此我们只要设置两个指针同时从根出发，一个按照根左右的顺序遍历，一个按照根右左的顺序遍历，依次比较两个指针所指向的节点的值是否相等即可，注意在获取两者指针指向的节点的值之前，要确保二者都不为空指针。

### 代码实现

```c++
struct BinaryTreeNode
{
	int m_nValue;
	BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
};

bool isSymmetric(BinaryTreeNode*, BinaryTreeNode*);

bool isSymmetric(BinaryTreeNode* pNode){
    return isSymmetric(pNode, pNode);
}

// 对pNode1进行根左右遍历，pNode2进行根右左遍历
bool isSymmetric(BinaryTreeNode* pNode1, BinaryTreeNode* pNode2){
    // 对根进行判断
    if(pNode1 == nullptr && pNode2 == nullptr)  // 二者均为空
        return true;
    if(pNode1 ==nullptr || pNode2 == nullptr)   // 二者有一个为空
        return false;
    if(pNode1->m_nValue == pNode2->m_nValue){   // 二者均不为空且相等时，比较子节点
        return isSymmetric(pNode1->m_pLeft, pNode2->m_pRight) &&    
                isSymmetric(pNode1->m_pRight, pNode2->m_pLeft);
    }                                           // 二者均不为空且不相等
    else{                                       
        return false;
    }
}
```

## 面试题29：顺时针打印矩阵

### 题目

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。例如，如果输入如下矩阵：

```c++
1  2  3  4
5  6  7  8
9  10 11 12
13 14 15 16
```

则依次打印出数字1，2，3，4，8，12，16，15，14，13，9，5，16，7，1

### 解题思路

我们可以把矩阵看成一圈圈的，根据题意可知道是由外圈到内圈顺序依次打印，我们可以用一个循环来打印矩阵，每次打印矩阵中的一个圈：

假设矩阵为1×1的，打印第一圈的左上角坐标为（0，0）
假设矩阵为2×2的，打印第一圈的左上角坐标为（0，0）
假设矩阵为3×3的，打印第一圈的左上角坐标为（0，0），第二圈的左上角坐标为（1，1）
假设矩阵为4×4的，打印第一圈的左上角坐标为（0，0），第二圈的左上角坐标为（1，1）
假设矩阵为5×5的，打印第一圈的左上角坐标为（0，0），第二圈的左上角坐标为（1，1），第三圈的左上角坐标为（2，2）
...
可以看出，每一圈的起始坐标横坐标与纵坐标相等，假设矩阵为rows×columns，每一圈的起始坐标（start，start），均满足start \* 2 < rows, start * 2 < columns

```c++
void PrintMatrixClockwise(int **numbers, int rows, int columns){
    if(numbers == nullptr || rows <= 0 || columns <=0)
        return;
    for(int start = 0; start * 2 < rows && start * 2 < columns; start++){
        PrintMatrixCircle(start);
    }
}
```

接着考虑如何实现打印一圈的功能，我们可以把打印一圈分为四步：从左到右打印一行；从上到下打印一列；从右到左打印一行；从下到上打印一列。规定好每一步的起始行号，列号，终止行号，列号，即可完成四步的打印。

需要注意的是最后一圈有可能只有一列，一行，或者单独一个元素。如图所示

 ![image-20220328205229608](images\image-20220328205229608.png)

第一步总是需要的。

如果只有一行，那就不用第二步了，也就是需要第二步的前提是终止行号大于起始行号。

在第二步的基础上，需要第三步的前提条件是至少有两列，即终止列号大于起始列号。

在第三步的基础上，需要第四步的前提条件是至少有三行，即终止行号要比起始行号大2.

### 代码实现

```c++
void PrintMatrixCircle(int start, int rows, int columns, int **numbers);

void PrintMatrixClockwise(int **numbers, int rows, int columns){
    if(numbers == nullptr || rows <= 0 || columns <=0)
        return;
    for(int start = 0; start * 2 < rows && start * 2 < columns; start++){
        PrintMatrixCircle(start, rows, columns, numbers);
    }
}

void PrintMatrixCircle(int start, int rows, int columns, int **numbers){
    int rowStr = start;
    int rowEnd = rows - 1 - start;
    int colStr = start;
    int colEnd = columns - 1 - start;
    // 从左到右打印一行
    for(int i = colStr; i <= colEnd; i++){
        cout << numbers[rowStr][i] << " ";
    }

    // 从上到下打印一列
    if(rowEnd == rowStr){   // 只有一行
        return;
    }
    for(int i = rowStr + 1; i <= rowEnd; i++){
        cout << numbers[i][colEnd] << " ";
    }

    // 从右到左打印一行
    if(colStr == colEnd){   // 只有一列
        return;
    }
    for(int i = colEnd - 1; i >= colStr; i--){
        cout << numbers[rowEnd][i] << " ";
    }

    // 从下到上打印一列
    if(rowEnd <= rowStr + 1){   // 小于两行
        return;
    }
    for(int i = rowEnd - 1; i >= rowStr + 1; i--){
        cout << numbers[i][colStr] << " ";
    }
}
```

## 面试题30：包含min函数的栈

### 题目

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的min函数。在该栈中，调用min、push及pop的时间复杂度都是O(1)。

### 解题思路

要返回栈中最小的元素，我们可以在栈中添加一个成员变量记录栈中的最小值，每次有新元素进栈的时候，如果该元素比当前最小元素小，则更新最小元素。

但是如果栈中当前的最小元素被弹出了，我们必须找到栈中的次小元素来更新这个最小值，同理，次小元素被弹出了，我们必须找到次次小元素。因此我们必须记录每次加入新元素后的最小值，并在每次有元素弹出后，取得上一次最小值，因此可以用一个栈来记录每次的最小值。

例如我们往栈里压入3、4、2、1：

| 步骤 | 操作  | 数据栈     | 最小值栈   | 最小值 |
| ---- | ----- | ---------- | ---------- | ------ |
| 1    | 压入3 | 3          | 3          | 3      |
| 2    | 压入4 | 3、4       | 3、3       | 3      |
| 3    | 压入2 | 3、4、2    | 3、3、2    | 2      |
| 4    | 压入1 | 3、4、2、1 | 3、3、2、1 | 1      |
| 5    | 弹出  | 3、4、2    | 3、3、2    | 2      |
| 6    | 弹出  | 3、4       | 3、3       | 3      |
| 7    | 弹出  | 3          | 3          | 3      |

### 代码实现

```c++
#include <stack>
#include <cassert>
#include <iostream>

using namespace std;

template<typename T>
class StackWithMin{
public:
    void push(const T& value);
    void pop();
    const T& min() const;
private:
    stack<T> m_data;
    stack<T> m_min;
};

template<typename T>
void StackWithMin<T>::push(const T& value){
    m_data.push(value);
    if(m_min.empty() || value < min()){
        m_min.push(value);
    }
    else{
        m_min.push(min());
    }
}

template<typename T>
void StackWithMin<T>::pop(){
    assert(m_data.size() > 0 && m_min.size() > 0);
    m_data.pop();
    m_min.pop();
}

template<typename T>
const T& StackWithMin<T>::min() const{
    assert(m_data.size() > 0 && m_min.size() > 0);
    return m_min.top();
} 
```

## 面试题31：栈的压入、弹出序列

### 题目

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列{1，2，3，4，5}是某栈的压栈顺序，序列{4，5，3，2，1}是该压栈序列对应的一个弹出序列，但{4，3，5，1，2}就不可能是该压栈序列的弹出序列。

### 解题思路

首先我们需要一个辅助栈，按照第一个序列的顺序将数字入栈，遍历第二个序列，即弹出序列，如果下一个要弹出的元素位于栈顶，则直接弹出；若不在栈顶，则按照入栈顺序将还没有入栈的数字压入栈中，直到下一个要弹出的数字被压入栈顶为止；如果所有数字全都压入栈中仍然没有找到下一个要弹出的数字，则该序列就不可能是一个弹出序列。	

### 代码实现按

```c++
bool isPopOrder(const int* pPush, const int* pPop, int nLength){
    if(pPush == nullptr || pPop == nullptr || nLength <= 0)
        return false;
    stack<int> stk;
    int pushIndex = 0;
    for(int i = 0; i < nLength; i++){               // 遍历弹出序列
        if(!stk.empty() && stk.top() == pPop[i]){   // 如果下一个要弹出的元素刚好是栈顶元素，则直接弹出
            stk.pop();
            continue;
        }
        if(stk.empty() || stk.top() != pPop[i]){    // 如果栈为空或者下一个要弹出的元素不是栈顶元素
            while(pushIndex != nLength && pPush[pushIndex] != pPop[i]){     // 按照入栈顺序将元素依次压入，直到遇到下一个要弹出的元素
                stk.push(pPush[pushIndex++]);
            }
            if(pushIndex == nLength){               // 如果所有元素都入栈完还没有找到下一个要弹出的元素，则返回false
                return false;
            }
            pushIndex++;                            // 跳过下一个要弹出的元素，继续遍历pPop
        }
    }
    return true;
}
```

## 面试题32-1：不分行从上到下打印二叉树

### 题目

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。例如图中的二叉树，依次打印出8，6，10，5，7，9，11。二叉树的节点的定义如下：

```c++
struct BinaryTreeNode
{
	int m_nValue;
    BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
}；
```

 ![image-20220331193313827](images\image-20220331193313827.png)

### 解题思路

考查二叉树的层序遍历。每次打印一个节点的时候，如果该节点有子节点，则把该节点的子节点放到一个队列的末尾。接下来到队列的头部取出最早进入队列的节点，重复前面的打印操作，直至队列中所有的节点都被打印出来。

### 代码实现

```C++
void PrintBST(BinaryTreeNode* root){
    if(root == nullptr)
        return;
    deque<BinaryTreeNode*> bstque;
    bstque.push_back(root);
    while(!bstque.empty()){
        BinaryTreeNode* tmp = bstque.front();
        cout << tmp->m_nValue << '\t';
        bstque.pop_front();
        if(tmp->m_pLeft != nullptr)
            bstque.push_back(tmp->m_pLeft);
        if(tmp->m_pRight != nullptr)
            bstque.push_back(tmp->m_pRight);
    }
}
```

## 面试题32-2：分行从上到下打印二叉树

### 题目

从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。例如图中的打印结果为：

```c++
8
6	10
5	7	9	11
```

 ![image-20220331202515244](images\image-20220331202515244.png)

### 解题思路

参考二叉树的层序遍历，为了区分每一层，我们需要额外两个变量：

* 一个变量表示在当前曾中还没有打印的节点数
* 一个变量表示下一层节点的数目

### 代码实现

```c++
void PrintBST(BinaryTreeNode* root){
    if(root == nullptr)
        return;
    deque<BinaryTreeNode*> bstque;
    bstque.push_back(root);
    int curLayerCnt = 0;
    int nextLayerCnt = 1;
    while(!bstque.empty()){
        curLayerCnt = nextLayerCnt; // 开始打印当前层时，把之前记录的下一层的节点数赋给当前层的节点数
        nextLayerCnt = 0;

        while(curLayerCnt--){      // 打印当前层
            BinaryTreeNode* tmp = bstque.front();
            cout << tmp->m_nValue << '\t';
            bstque.pop_front();
            if(tmp->m_pLeft != nullptr){
                bstque.push_back(tmp->m_pLeft);
                nextLayerCnt++;
            }
            if(tmp->m_pRight != nullptr){
                bstque.push_back(tmp->m_pRight);
                nextLayerCnt++;
            }
        }

        cout << '\n';        
    }
}
```

## 面试题32-3：之字形打印二叉树

### 题目

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行依次类推。例如图中二叉树的打印结果为：

```c++
1
3	2
4	5	6	7
15	14	13	12	11	10	9	8
```

 ![image-20220401145413825](images\image-20220401145413825.png)

### 解题思路

分析图中打印的具体步骤：

* 打印节点1，并把它的子节点2，3放到一个容器里，由于2，3的输出顺序为3，2，因此这个容器可以是一个栈。
* 打印节点3，2，依次把3，2的子节点放到一个容器里，由于其子节点的输出顺序为4，5，6，7，即我们入栈的顺序应该为7，6，5，4，即需要先将其右子节点入栈，再将其左子节点入栈。
* 打印节点4，5，6，7，同步骤1，依次将各节点的子节点按左子节点、右子节点的顺序入栈。

因此总结打印过程：

* 当打印奇数层时，我们依次将各节点的子节点按左子节点、右子节点的顺序入栈
* 当打印偶数层时，我们依次将各节点的子节点按右子节点、左子节点的顺序入栈

### 代码实现

```c++
void PrintBST(BinaryTreeNode* root){
    if(root == nullptr)
        return;
    stack<BinaryTreeNode*> oddStack;    // 用于打印奇数层
    stack<BinaryTreeNode*> evenStack;   // 用于打印偶数层
    oddStack.push(root);
    while(!oddStack.empty() || !evenStack.empty()){
        if(!oddStack.empty()){                  		// 打印奇数层
            while(!oddStack.empty()){                
                BinaryTreeNode* temp = oddStack.top();
                cout << temp->m_nValue << '\t';
                oddStack.pop();
                if(temp->m_pLeft != nullptr)
                    evenStack.push(temp->m_pLeft);
                if(temp->m_pRight != nullptr)
                    evenStack.push(temp->m_pRight);
            }
            cout << '\n';
        }
        else{                           				// 打印偶数层
            while(!evenStack.empty()){                
                BinaryTreeNode* temp = evenStack.top();
                cout << temp->m_nValue << '\t';
                evenStack.pop();
                if(temp->m_pRight != nullptr)
                    oddStack.push(temp->m_pRight);
                if(temp->m_pLeft != nullptr)
                    oddStack.push(temp->m_pLeft); 
            }
            cout << '\n';
        }
    }
}
```

