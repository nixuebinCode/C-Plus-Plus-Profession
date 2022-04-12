# 剑指offer

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

## 面试题7：重建二叉树

### 题目

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1, 2, 4, 7, 3, 5, 6, 8}和中序遍历序列{4, 7, 2, 1, 5, 3, 8, 6}，则重建出如图所示的二叉树并输出它的头结点。二叉树节点的定义如下：

 ![image-20220226160048561](/images/image-20220226160048561.png)

```c++
struct BinaryTreeNode{
    int m_nValue;
    BinaryTreeNode* m_pLeft;
    BinaryTreeNode* m_pRight;
};
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

## 面试题26：树的子结构

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

## 面试题33：二叉搜索树的后序遍历序列

### 题目

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回true，否则返回false。假设输入的数组的任意两个数字都互不相同。例如，输入数组{5，7，6，9，11，10，8}，则返回true，因为这个整数序列是图中二叉搜索树的后序遍历结果。如果输入的数组是{7，4，6，5}，则由于没有哪棵二叉搜索树的后序遍历结果是这个序列，因此返回false。

 ![image-20220402092933751](images\image-20220402092933751.png)

### 解题思路

在后序遍历得到的序列中，最后一个数字是树的根节点的值。数组中前面的数字可以分为两部分：第一部分是左子树节点的值，它们都比根节点的值小；第二部分是右子树节点的值，它们都比根节点的值大。

例如对于序列{5，7，6，9，11，10，8}，8为根节点，前3个数字5、7、6比8小，是8的左子树节点；后3个数字9、11、10比8大，是8的右子树节点
接下来利用递归的方法分别检测8的左子树和右子树：{5，7，6}的根节点为6，5为其左子树节点，7为其右子树节点。{9，11，10}的根节点为10，9为其左子树节点，11为其右子树节点.

对于序列{7，4，6，5}，5为根节点，我们发现第一个值7就比5大，因此7，4，6均为5的右子树节点，根据二叉搜索树的特性，我们知道7，4，6均应该大于根节点5，其中4小于5，因此该序列不是二叉搜索树的后序遍历序列。

通过以上分析，可以确定程序思路：

对于某一序列，其最后一个值为根节点，从前向后遍历该序列，找到第一个大于根节点的值，那么从该值起到倒数第二个值应该为根节点的右子树节点，若其中有比根节点小的值，则返回false，否则递归地检查根节点地左子树序列，和右子树序列。

### 代码实现

```c++
bool isPostOrderOfBST(int *postOrder, int length){
    if(postOrder == nullptr || length < 0)
        return false;
    if(length == 1 || length == 0)         	// 递归终止条件：序列中只有一个节点或没有节点
        return true;
    int root = postOrder[length - 1];
    int rightTreeStr = 0;
    while(postOrder[rightTreeStr] < root)   // 查找右子树的起始节点
        rightTreeStr++;
    for(int i = rightTreeStr; i < length - 1; i++){ // 检查右子树的节点是否都大于根节点
        if(postOrder[i] < root)
            return false;
    }
    int leftTreeLen = rightTreeStr;
    int rightTreeLen = length - leftTreeLen - 1;
    return isPostOrderOfBST(postOrder, leftTreeLen) &&      // 递归检查其左子树和右子树
            isPostOrderOfBST(postOrder + rightTreeStr, rightTreeLen);

}
```

## 面试题34：二叉树中和为某一值的路径

### 题目

输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。例如输入图中的二叉树和整数22，则打印出两条路径，第一条路径包括节点10、12，第二条路径包括节点10、5、7。二叉树节点的定义如下：

```c++
struct BinaryTreeNode
{
	int m_nValue;
    BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
}；
```

 ![image-20220403095736154](images\image-20220403095736154.png)

### 解题思路

由于路径是由根节点出发的，我们首先要遍历根节点，在树的几种遍历算法中，只有前序遍历是首先访问根节点。

以图中的二叉树为例：

* 先序遍历，每访问一个节点，我们就把当前节点添加到路径中去，当到达节点5时，路径中包括10，5，接下来遍历到节点4，我们把4也添加到路径中去，因为4是叶子节点，我们计算路径的和，为19，不符合要求。
* 我们接着遍历下一个节点，在遍历7之前，**先要从节点4返回节点5**，此时我们需要把4从路径中删除，接下来访问节点7，由于7是叶子节点，计算路径的和，为22，符合要求，则打印出路径。
* 同理，继续遍历下一个节点，在遍历12之前，**先要从节点7返回节点5，然后从节点5返回节点10**，此时需要把路径中的7，5从路径中删除，接下来访问节点12，由于12是叶子节点，计算路径的和，为22，符合要求，则打印出路径。

因此我们可以总结出程序的步骤：

* 先序遍历二叉树，每遍历到一个节点，将其加入路径中去，当加入的节点为叶子节点时，计算路径之和，若符合要求则打印路径。
* 继续遍历，若遍历时从某个节点返回其父亲节点，则把该节点从路径中删除，即当前层递归结束的时候，把它删除。（可以注意到我们每次从路径中删除的节点都是最后加入路径的，因此可以借助栈来完成）。

### 代码实现

```c++
void PrintPath(BinaryTreeNode*, int, int &, vector<BinaryTreeNode*> &);

void PrintPath(BinaryTreeNode* root, int sumOfPath){
    if(root == nullptr || sumOfPath < 0)
        return;
    vector<BinaryTreeNode*> path;   // 用curSum记录当前路径之和，避免每次重新遍历路径，计算路径和
    int curSum = 0;
    PrintPath(root, sumOfPath, curSum, path);
}
// 注意这里curSum和path使用引用类型，这样在下层递归中修改curSum和path时，能够影响上层递归。
void PrintPath(BinaryTreeNode* pNode, int sumOfPath, int &curSum, vector<BinaryTreeNode*> &path){									if(pNode == nullptr)    // 递归终止条件
        return;    
    path.push_back(pNode);   // 将当前节点加入路径
    curSum += pNode->m_nValue;
    if(pNode->m_pLeft == nullptr && pNode->m_pRight == nullptr){  // 当前节点为叶子节点
        if(curSum == sumOfPath){
            for(auto item : path)
                cout << item->m_nValue << '\t';
            cout << endl;
        }
    }
    PrintPath(pNode->m_pLeft, sumOfPath, curSum, path);
    PrintPath(pNode->m_pRight, sumOfPath, curSum, path);
    // 当前递归结束，返回上层
    path.pop_back();
    curSum -= pNode->m_nValue;
}
```

## 面试题35：复杂链表的复制

### 题目

请实现函数 ComplexListNode* Clone(ComplexListNode* pHead)，复制一个复杂链表。在复杂链表中，每个节点除了有一个 m_pNext 指针指向下一个节点，还有一个 m_pSibling 指针指向链表中的任意节点或者 nullptr，如图为一个含有5个节点的复杂链表。实线箭头表示 m_pNext 指针，虚线箭头表示 m_pSibling 指针。节点的定义如下：

```c++
struct ComplexListNode{
	int m_nValue;
    ComplexListNode* m_pNext;
    ComplexListNode* m_pSibling;
}
```

 ![image-20220404093244501](images\image-20220404093244501.png)

### 解题思路

>最直观的做法是把复制过程分成两步：第一步是复制原始链表上的每个节点，并用 m_pNext 链接起来；第二步是设置每个节点的 m_pSibling 指针，由于查找每个节点的 m_pSibling 指针所指向的节点都需要从表头沿着 m_pNext 遍历，因此这种方法的时间复杂度为 O(n^2)
>
>由于上述方法的时间主要花费在定位节点的 m_pSibling 上面，我们试着在这方面优化：第一步仍然是复制原始链表上的每个节点 N 创建 N‘，并用 m_pNext 链接起来，同时我们把<N, N'>的配对信息放到一个哈希表中；第二步仍然是设置节点的 m_pSibling 指针，对于节点 N‘ ，假设原始节点的 m_pSibling 为 S，则 N’ 的 m_pSibling 应为对应的 S‘，可以由第一步保存的哈希表中由 S 直接得出。这种方法虽然减少了时间复杂度，但是利用了哈希表，空间复杂度为O(n)

最优解法：

* 根据原始链表的每个节点N创建对应的N'，这一次，我们把N’链接到N的后面：

   ![image-20220404094904211](images\image-20220404094904211.png)

* 设置复制出来的节点的 m_pSibling 。假设原始链表上的 N 的 m_pSibling 指向节点 S ，那么对应的 N‘ 是 N 的m_pNext，同样 S’ 也是 S 的 m_pNext ，因此我们可以利用 S 的 m_pSibling 的 m_pNext 找到 N‘ 的 m_pSibling 

 ![image-20220404095222282](images\image-20220404095222282.png)

* 将这个长链表拆分为两个链表：把奇数位置的节点用 m_pNext 链接起来就是原始链表。偶数位置的节点用 m_pNext 链接起来就是复制出来的链表。

### 代码实现

```C++
ComplexListNode* Clone(ComplexListNode* pHead){
    if(pHead == nullptr)
        return nullptr;
    // Step1: 把复制的节点接在原始节点后面
    ComplexListNode *pNode = pHead;
    while(pNode != nullptr){
        // 创建复制节点
        ComplexListNode* pNodeCopy = new ComplexListNode();
        pNodeCopy->m_nValue = pNode->m_nValue;

        pNodeCopy->m_pNext = pNode->m_pNext;
        pNode->m_pNext = pNodeCopy;
        pNode = pNodeCopy->m_pNext;
    }
    // Step2：设置复制的节点的 m_pSibling 指针
    pNode = pHead;
    ComplexListNode *pNodecpy = nullptr;
    while(pNode != nullptr){
        pNodecpy = pNode->m_pNext;
        if(pNode->m_pSibling != nullptr)        // 有些节点的 m_pSibling 可能指向nullptr
            pNodecpy->m_pSibling = pNode->m_pSibling->m_pNext;
        pNode = pNode->m_pNext->m_pNext;
    }
    // Step3：将复制链表从长链表中拆下来
    pNode = pHead;
    ComplexListNode *pRet = pNode->m_pNext;
    while(pNode != nullptr){
        pNodecpy = pNode->m_pNext;
        pNode->m_pNext = pNodecpy->m_pNext;
        if(pNode->m_pNext == nullptr)
            pNodecpy->m_pNext = nullptr;
        else
            pNodecpy->m_pNext = pNode->m_pNext->m_pNext;
        pNode = pNode->m_pNext;
    }
    return pRet;
}
```

## 面试题36：二叉搜索树与双向链表

### 题目

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。比如输入图中左边的二叉搜索树，则输出转换之后的排序双向链表。二叉树节点的定义如下：

```c++
struct BinaryTreeNode
{
	int m_nValue;
    BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
}；
```

 ![image-20220406095418017](images\image-20220406095418017.png)

### 解题思路

我们在将二叉搜索树转换成排序双向链表时， 原先指向左子节点的指针调整为链表中指向前一个节点的指针， 原先指向右子节点的指针调整为链表中指向后一个节点的指针。

很容易想到利用二叉树的中序遍历来从小到大遍历序列，这里的难点是在中序遍历过程中，需要想办法记录当前遍历的节点pNodeCur和它的前驱节点pNodePre。前者在当前递归记录即可，因此采用本地变量记录，后者需要在上层递归中记录，因此采用引用形参的方式记录。

### 代码实现

```c++
void ConvertBSTtoLst(BinaryTreeNode* pNode, BinaryTreeNode* &pNodePre);

BinaryTreeNode* ConvertBSTtoLst(BinaryTreeNode* pRoot){
    if(pRoot == nullptr)
        return nullptr;
    BinaryTreeNode* pNodePre = nullptr;	// 用于递归中记录前驱节点
    
    ConvertBSTtoLst(pRoot, pNodePre);
    pNodePre->m_pRight = nullptr;   // 处理尾节点
    // 找到链表的头节点
    BinaryTreeNode* pHeadOfLst = pRoot;
    while(pHeadOfLst->m_pLeft != nullptr)
        pHeadOfLst = pHeadOfLst->m_pLeft;
    return pHeadOfLst;
}

void ConvertBSTtoLst(BinaryTreeNode* pNode, BinaryTreeNode* &pNodePre){
    if(pNode == nullptr)
        return;
    BinaryTreeNode* pNodeCur = pNode;   // 记录当前遍历节点
    // 遍历左子树
    if(pNode->m_pLeft != nullptr)
        ConvertBSTtoLst(pNode->m_pLeft, pNodePre);
    
    // 处理指针
    if(pNodePre != nullptr)
        pNodePre->m_pRight = pNodeCur;
    pNodeCur->m_pLeft = pNodePre;
    // 当前节点处理完毕，更新pNodePre
    pNodePre = pNodeCur;

    // 遍历右子树
    if(pNode->m_pRight != nullptr)
        ConvertBSTtoLst(pNode->m_pRight, pNodePre);
}
```

## 面试题37：序列化二叉树

### 题目

请实现两个函数，分别用来序列化和反序列化二叉树。

### 解题思路

回忆面试题7，我们可以根据二叉树的前序遍历和中序遍历结果来重建一棵二叉树，因此最简单的序列化方法就是得到这棵二叉树的前序遍历和后序遍历。但是这种方法有一个问题，就是要求树中不能有数值重复的节点，且只有当两个序列中的所有数据都读出后才能开始反序列化。

因此我们考虑根据前序遍历的顺序来序列化二叉树，遍历二叉树碰到 nullptr 指针时，把这些 nullptr 指针序列化为一个特殊的字符（如'$'）。例如图中的二叉树序列化成字符串“1，2，4，\$，\$，\$，3，5，\$，\$，6，\$，\$”

 ![image-20220407103900706](images\image-20220407103900706.png)

接着来分析如何反序列化二叉树，第一个读出来的数字是1，由于是先序遍历，1为跟节点。接下来读入数字2，根据先序遍历，2应该为1的左子节点。同样接下来的4是2的左子节点，接着两个\$，意味着4的左、右子节点均为 nullptr，因此4是一个叶节点。接下来回到2节点，重建它的右子节点，由于下一个字符是 \$，着表面2的右子节点为 nullptr，再接下来回到根节点，构造根节点的右子树...

### 代码实现

```c++
struct BinaryTreeNode
{
	int m_nValue;
    BinaryTreeNode* m_pLeft;
	BinaryTreeNode* m_pRight;
};

void Serialize(const BinaryTreeNode* pRoot, ostream& stream){
    if(pRoot == nullptr){
        stream << "$,";
        return;
    }
    stream << pRoot->m_nValue << ',';
    Serialize(pRoot->m_pLeft, stream);
    Serialize(pRoot->m_pRight, stream); 
}

// 每次从输入流中读入一个字符，反序列化二叉树
void DeSerialize(BinaryTreeNode **pRoot, istream& stream){
    int number;
    char ch;
    stream >> ch;
    while(ch == ',')
        stream >> ch;
    
    if(ch >= '0' && ch <= '9'){
        stream.putback(ch);
        stream >> number;
        *pRoot = new BinaryTreeNode();
        (*pRoot)->m_nValue = number;
        (*pRoot)->m_pLeft = nullptr;
        (*pRoot)->m_pRight = nullptr;

        DeSerialize(&((*pRoot)->m_pLeft), stream);
        DeSerialize(&((*pRoot)->m_pRight), stream);
    }
}
```

## 面试题38：字符串的排列

### 题目

输入一个字符串，打印出该字符串中字符的所有排列。例如，输入字符串 abc，则打印出由字符 a、b、c 所能排列出来的所有字符串 abc、acb、bac、bca、cab 和 cba。

### 解题思路

我们把一个字符串看成由两部分组成：第一部分是它的第一个字符；第二部分是后面的所有字符。

第一步求所有可能出现在第一个位置的字符，即对于给定的字符串，把第一个字符和后面所有的字符（包括它自身）进行交换。

第二步固定第一个字符，求后面所有字符的排列，这个时候我们仍然把后面的所有字符分成两部分：后面字符的第一个字符，以及这个字符之后的所有字符。然后把第一个字符逐一和它后面的字符交换。

 ![image-20220409161021486](images\image-20220409161021486.png)

### 代码实现

```c++
void Permutation(char* pStr, char* pBegin);

void Permutation(char* pStr){
    if(pStr == nullptr)
        return;
    Permutation(pStr, pStr);
}

// pBegin 指向当前操作的字符串的第一个字符
void Permutation(char* pStr, char* pBegin){
    if(*pBegin == '\0'){	// 当 pBegin 指向的是 pStr 的末尾时，则说明前面的所有字符都已经交换完毕，递归终止，并打印出当前字符串
        printf("%s\n", pStr);
        return;
    }
    // 交换当前操作的字符串的第一个字符和其余字符
    for(char* pCh = pBegin; *pCh != '\0'; pCh++){
        char temp = *pBegin;
        *pBegin = *pCh;
        *pCh = temp;

        //交换完成后，对字串执行递归操作
        Permutation(pStr, pBegin + 1);

        // 因为每次交换是交换第一个字符和剩余字符，在进行下一次交换前，要把最开始的第一个字符交换回来
        temp = *pBegin;
        *pBegin = *pCh;
        *pCh = temp;
    }
}
```

## 面试题39：数组中出现次数超过一半的数字

### 题目

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如，输入一个长度为9的数组 {1，2，3，2，2，2，5，4，2}。由于数字 2 在数组中出现了5次，超过数组长度的一半，因此输出 2。

### 解题思路一

考虑题目中数组的特性：有一个数字出现的次数超过数组长度的一半。如果把这个数组排序，那么**排序之后位于数组中间的数字一定就是那个出现次数超过数组长度一半的数字**。利用快速排序中 partition 算法的启发，我们先在数组中随机选择一个数字，然后以它为 pivot 对数组进行一次划分。如果这个选中的数字的下标刚好是 n/2，那么这个数字就是数组的**中位数**；如果它的下标大于 n/2，那么中位数应该位于它的左边，我们可以接着在它的左边部分的数组中查找；如果它的下标小于 n/2，那么中位数应该位于它的右边，我们可以接着在它的右边部分的数组中查找。

### 代码实现

```c++
bool CheckMoreThanHalf(int *numbers, int length, int number);
int partition(int* numbers, int low, int high);

int MoreThanHalfNum(int* numbers, int length){
    if(numbers == nullptr || length <= 0)
        return 0;
    
    int middle = length / 2;
    // 选取数组中的第一个数字对数组进行划分
    int low = 0;
    int high = length - 1;
    int pos = partition(numbers, low, high);
    while(pos != middle){
        if(pos < middle){   
            low = pos + 1;
            pos = partition(numbers, low, high);
        }
        else{
            high = pos - 1;
            pos = partition(numbers, low, high);
        }
    }
    int result = numbers[pos];

    // 检查该数字是否出现了一半以上
    if(!CheckMoreThanHalf(numbers, length, result))
        result = 0;
    return result;
}

int partition(int* numbers, int low, int high){
    int pivot = numbers[low];
    while(low < high){
        while(low < high && numbers[high] >= pivot) high--;
        numbers[low] = numbers[high];
        while(low < high && numbers[low] <= pivot)  low++;
        numbers[high] = numbers[low];
    }
    numbers[low] = pivot;
    return low;
}

bool CheckMoreThanHalf(int *numbers, int length, int number){
    int cnt = 0;
    for(int i = 0; i < length; i++){
        if(numbers[i] == number)
            cnt++;
    }
    return (cnt > length / 2);
}
```

### 解题思路二

数组中有一个数字出现的次数超过数组长度的一半，也就是说这个数字出现的次数比其他所有数字出现的次数的和还要多。因此我们在遍历数组时保存两个值：当前遍历的数字，出现的次数。当我们遍历到下一个数字的时候：

* 如果下一个数字和我们之前保存的数字相同，则次数+1
* 如果下一个数字和我们之前保存的数字不同，则次数-1，如果减1后次数为0，那么我们就更新保存的数字，并把次数设为1。

遍历到数组末尾，我们保存的数字就是要找的数字。

### 代码实现

```c++
int MoreThanHalfNum_Sol2(int* numbers, int length){
    if(numbers == nullptr || length <= 0)
        return 0;
    int number = numbers[0];
    int cnt = 1;
    for(int i = 1; i < length; i++){
        if(numbers[i] == number)
            cnt++;
        else
            cnt--;
            if(cnt == 0){
                number = numbers[i];
                cnt = 1;
            }
    }
    int result = number;

    // 检查该数字是否出现了一半以上
    if(!CheckMoreThanHalf(numbers, length, result))
        result = 0;

    return result;
}
```

## 面试题40：最小的 K 个数

### 题目

输入 n 个整数，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8 这 8 个数字，则最小的 4 个数字是1、2、3、4。

### 解题思路一

时间复杂度为 O(n)，只有当我们可以修改输入的数组时可用

同面试题39，可以利用 partition 来解决这个问题，找出其中最小的 k 个数，我们只需找到第 k 个小的数，那么经过 partition 之后，在这个数左边的数字都比它小，即数组中左边的 k 个数字就是所求。

### 代码实现

```c++
int partition(int *numbers, int low, int high);

void GetLeastNumbers(int *numbers, int length, int *output, int k){
    if(numbers == nullptr || k <= 0)
        return;
    int low = 0;
    int high = length - 1;
    int pos = partition(numbers, low, high);
    while(pos != k){
        if(pos < k){
            low = pos + 1;
            pos = partition(numbers, low, high);
        }
        else{
            high = pos - 1;
            pos = partition(numbers, low, high); 
        }
    }
    for(int i = 0; i <= pos; i++)
        output[i] = numbers[i];
}

int partition(int *numbers, int low, int high){
    int pivot = numbers[low];
    while(low < high){
        while(low < high && numbers[high] >= pivot)
            high--;
        numbers[low] = numbers[high];
        while(low < high && numbers[low] <= pivot)
            low++;
        numbers[high] = numbers[low];
    }
    numbers[low] = pivot;
    return low;
}
```

### 解题思路二

时间复杂度为 O(nlogk)，特别适合处理海量数据，对于海量的数据，由于内存的大小是有限制的，可能不能一次性把海量的数据读入内存，这个时候就可以从硬盘中每次读入一个数字，判断该数字是不是需要放入容器 leastNumbers 即可。

我们可以先创建一个大小为 k 的数据容器来存储最小的 k 个数字，接下来每次从输入的 n 个整数中读入一个数：

* 如果容器中已有的数字少于 k 个，则直接把此次读入的数放入容器之中。
* 如果容器中已有 k 个数字了，我们要在 k 个数字中找出最大数，然后比较此次读入的数据和最大数，如果比最大数大，抛弃这个数；如果比最大数小，则用这个数替换容器中的最大数。

因此我们需要设计一种数据结构来实现该容器，该数据结构需要三件事情：

* 找出最大值
* 删除最大数
* 插入一个节点

为了减少时间复杂度，我们可以采用红黑树，其查找、插入、删除操作都能保证在 O(logk) 内完成，而一共有 n 个数，因此总的时间复杂度为  O(nlogk)。

STL 中的 set 和 multiset 都是基于红黑树实现的，因此我们可以直接使用。

### 代码实现

```c++
typedef multiset<int, greater<int>> intSet; // 指明 multiset 按从大到小排序
typedef multiset<int, greater<int>>::iterator setIterator;

void GetLeastNumbers(const vector<int>& data, intSet& leastNumbers, int k){
    leastNumbers.clear();
    if(k <= 0 || data.size() < k)
        return;
    vector<int>::const_iterator iter = data.begin();
    for(; iter != data.end(); iter++){
        if(leastNumbers.size() < k)
            leastNumbers.insert(*iter);
        else{
            setIterator iterGreatest = leastNumbers.begin();
            if(*iter < *iterGreatest){
                leastNumbers.erase(iterGreatest);
                leastNumbers.insert(*iter);
            }
        }
    }
}
```

## 面试题41：数据流中的中位数

### 题目

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

### 解题思路

遇到从数据流中不断读入数值这种题，首先考虑的是我们需要一种容器来保存数据流读出的数据。重点是考虑用哪种数据结构来实现这种容器，并针对题目中的操作能获得最低的时间复杂度：插入和查找中位数。

* 没有排序的数组，插入一个数据的时间复杂度是 O(1) ，查找中位数我们可以利用 Partition 算法，在 O(n)

* 排过序的数组，即每插入一个元素，保持数组有序，插入数据的时间复杂度为 O(n) ， 查找中位数的时间复杂度是 O(1)

* 排序的链表，插入数据的时间复杂度为 O(n) ， 可以定义两个指针，指向链表的中间节点，那么查找中位数的时间复杂度是 O(1)

* 二叉搜索树，插入新数据的时间复杂度为 O(logn)，为了得到中位数，可以在二叉搜索树中添加一个表示子树节点数目的字段，利用这个字段，可以在 O(logn) 时间里找到中位数

* 显然二叉搜索树存在不平衡的情况，因此考虑使用平衡二叉树或者红黑树，但是面试中先要实现带特定结构的AVL树或者红黑树（在每个节点添加一个表示子树节点数目的字段），显然是比较困难的

* 考虑容器内数据是奇数和偶数的两种情况，我们用指针P1和P2指向数据的中间节点：

    ![image-20220412160330568](images\image-20220412160330568.png)

  可以发现指针P1，P2将数据分成两个部分，左边部分的数据比右边部分的数据小，如果我们能快速获得左边数据的最大值，以及右边数据的最小值，那么就可以得到中位数。很自然地想到左边用大根堆来存储数据，右边用小根堆来存取数据。我们可以利用 vector 以及 algorithm头文件里地push_heap() 和 pop_heap() 方法来实现堆。

### 代码实现

```c++
template<typename T>
class DynamicArray{
public:
    // 从数据流中读取一个数字，并插入到我们的数据结构中
    void Insert(T value){   
        if(((maxHeap.size() + minHeap.size()) & 1) == 0){ // 当前插入的数字是第奇数个（已经插入了偶数个），插入大根堆
            // 如果待插入的数字比小根堆中最小值大
            if(!minHeap.empty() && value > minHeap[0]){
                // 先将该数字插入到小根堆中
                minHeap.push_back(value);
                push_heap(minHeap.begin(), minHeap.end(), greater<T>());
                // 弹出小根堆中的最小值插入大根堆中
                value = minHeap[0];     // 小根堆中的最小值
                pop_heap(minHeap.begin(), minHeap.end(), greater<T>()); // pop_heap 会将堆顶元素（即为数组第一个位置）和数组最后一个位置对																			// 调
                minHeap.pop_back();     // 删除交换后的堆顶元素
            }
            // 将数字插入大根堆中
            maxHeap.push_back(value);
            push_heap(maxHeap.begin(), maxHeap.end(), less<T>());
        }
        else{   // 把当前数字插入小根堆
            if(!maxHeap.empty() && value < maxHeap[0]){
                maxHeap.push_back(value);
                push_heap(maxHeap.begin(), maxHeap.end(), less<T>());
                value = maxHeap[0];
                pop_heap(maxHeap.begin(), maxHeap.end(), less<T>());
                maxHeap.pop_back();
            }
            minHeap.push_back(value);
            push_heap(minHeap.begin(), minHeap.end(), greater<T>());
        }
    }

    T GetMedian(){
        if(minHeap.size() == 0 && maxHeap.size() == 0)
            throw exception();
        // 因为我们是先插入大根堆，再插入小根堆，如此循环
        // 因此当容器内数字是奇数个时，中位数应该在大根堆里
        if(((maxHeap.size() + minHeap.size()) & 1) == 1){
            return maxHeap[0];
        }
        else{
            return (maxHeap[0] + minHeap[0]) / 2;
        }
    }


private:
    vector<T> maxHeap;  // 大根堆，存取数组中较小的值，用来快速获得其中的最大值
    vector<T> minHeap;  // 小根堆，存取数组中较大的值，用来快速获得其中的最小值
};
```

