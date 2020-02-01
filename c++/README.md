# C++笔记整理

## STL Functions 重要的STL函数

### max_element, min_element 最大值和最小值

- http://www.cplusplus.com/reference/algorithm/max_element/
- http://www.cplusplus.com/reference/algorithm/minmax_element/

获取container中的最大值和最小值。输入参数是begin and end iterators and optional comparator，返回值是iterator指针指代最大值或最小值的位置。

```c++
int myints[] = {3,7,2,5,6,4,9};

  // using default comparison:
  std::cout << "The smallest element is " << *std::min_element(myints,myints+7) << '\n';
  std::cout << "The largest element is "  << *std::max_element(myints,myints+7) << '\n';

// using function myfn as comp:
  std::cout << "The smallest element is " << *std::min_element(myints,myints+7,myfn) << '\n';
  std::cout << "The largest element is "  << *std::max_element(myints,myints+7,myfn) << '\n';
```

C++11还引入了一个新的方法：minmax_element，该方法可以同时获取最大值和最小值。返回值是一个pair<T,T>的指针，其中first是最小值，second是最大值.

NOTES:
- 显然该方法可以用在所有的container中，并不只是vector。其实，只要是输入是iterator的任意函数都可以用于多种container。


### kth element 第k小（或者大）的值

http://www.cplusplus.com/reference/algorithm/nth_element/

使用<algorithm>中的`nth_element()`，该方法默认得到container中**第k小**的元素。不过注意，它还对数组进行了in-place排序，把 <= 该元素的数放在左边，>= 该元素的放在右边。因此，它需要修改该container。整个过程其实就是类似于quicksort中的partition过程。注意，同样类似partition，这里的左边元素和右边元素内部并没有进行排序。函数的平均时间是O(n)。

```c++
// using function as comp
void std::nth_element (myvector.begin(), myvector.begin()+5, myvector.end(),myfunction);
```

这里找到了数组中第6小的数（因为begin()是第1小），放在`myvector[5]`中，同时还把前第1-5小的数字放在myvector[0...4]中，后面从第7小开始的数字在右边的myvector[6...]中。注意，该函数默认是找第nth小的值，并且n是从0开始的。

当然，nth_element()也支持自定义比较函数，即上面例子中的最后一个参数myfunction。例如，用std::greater<int>()的话，找到的就是第k大的元素。当然也可以用其它的custom comparator，例如绝对值距离、欧氏距离等。

#### 应用
nth_element()可以在O(N)时间内找到一个数组中前K大或前K小的元素。这要比普通的O(NlogN)的排序法以及O(NlogK)的priority_queue都要快很多。它在LC中的一些应用题目有: 
- 347: Top K Frequent Elements
- 215: Kth Largest Element in an Array
- 692: Top K Frequent Words
- 973: K Closest Points to Origin


### lower_bound, upper_bound 上界和下界

- http://www.cplusplus.com/reference/algorithm/lower_bound/
- http://www.cplusplus.com/reference/set/set/lower_bound/

获取container某个范围内的针对某个值val的上界和下界。注意，对于某个输入val来说，下界lower_bound返回的是：**第一个不小于**val的元素（>=val）的位置；上界upper_bound的定义是：**第一个大于**val的元素（>val）的位置。显然，当container中没有元素等于val时，上下界返回值相同。

用法：

```c++
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,
                               const T& val);
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,
                               const T& val, Compare comp);
```
举例：

```c++
auto low = std::lower_bound (v.begin(), v.end(), 20);
auto up = std::upper_bound (v.begin(), v.end(), 20); 
int interval = std::distance(low, up);
```

这里的distance()参见下面介绍。

#### 注意
- 返回值是**指针**，指向上下界元素。如果不存在上下界（即数组所有元素都小于target），那么会返回last指针。因此，返回的指针 return_iterator 不要直接使用 * 来获取其元素，而是建议使用类似这样的方法来获取该指针在数组中的位置：

```c++
auto low = std::lower_bound (v.begin(), v.end(), 50);
int pos = low - v.begin();
if (pos < v.size()) // pos will be v.size() if no lower bound
{
    // With lower bound. Do sth. 
}
```

当然，对于所有类似的返回指针的STL函数，都要注意这一点。
- std::map和multimap, 以及std::set 和 multiset 中已经有这两个成员函数了，名字完全相同，返回值也是位置指针，当然输入就只有input value而已，它会寻找整个map或set。例如，下面例子从set中删除某下界和某上界之间的所有元素。

```c++
itlow=myset.lower_bound (30);
itup=myset.upper_bound (60);
myset.erase(itlow,itup);  
```

### distance 两个iterator之间的距离

http://www.cplusplus.com/reference/iterator/distance/

使用std::distance(first, second)可以获取两者之间的距离，即相隔的元素个数。该函数在计算set或者map中的两个iterator的距离时非常有用，因为**set/map中的iterators之间不能直接相减**，但是vector中的iterator之间可以（因为vector数据是连续存储的，iterator相减就是内存地址相减）。

具体例子可以参见上面lower_bound()中给出的例子。

#### 注意
distance()的返回值可以是负数。例如，distance(first, last) = 3，而distance(last, first) = -3。但是，通常是vector中的iterators才会这样。

如果是set/map中，一定要提前确定first的位置一定<=second位置。例如，前面的set例子中，first是lower_bound位置，second是upper_bound位置，distance(first, second)正确，但是distance(second, first)会报错，或者直接Time Limit Exceeded (TLE)。这通常是因为，set/map中的内存并不是连续的，因此无法从second开始不断向后找到first。容易想到，一种解决办法是，在set/map中，可以计算

```c++
distance(set.begin(), iter1) - distance(set.begin(), iter2)。
```
即，两个iterators和begin()之间的距离再相减即可。

### equal_range 等于某个target的range的上下界

- http://www.cplusplus.com/reference/algorithm/equal_range/
- http://www.cplusplus.com/reference/set/set/equal_range/

找出container中等于输入val的元素上下界。返回一个pair代表上下界的位置。显然它只适用于container中元素已经排好序的情况。

```c++
std::sort (v.begin(), v.end(), mygreater);                   // 30 30 20 20 20 10 10 10
auto bounds=std::equal_range (v.begin(), v.end(), 20, mygreater);
```

和上下界函数类似的，std::map和multimap, 以及std::set 和 multiset 中也已经有equal_range()了，返回值也是位置指针的pair，当然输入就只有input value而已，它会寻找整个map或set。

### <a name="custom-comp"></a>Custom comparators 自定义比较方法

自定义比较方法通常用于STL的比较函数和container中。

#### In STL functions

传统的定义函数方法：

```c++
bool mycmp(Interval& a, Interval& b){
   return a.start < b.start || (a.start == b.start && a.end < b.end);
}
std::sort(nums.begin(), nums.end(), mycmp);
```

使用C++14的Lambda expression定义函数会更加简单：

```c++
auto mycmp = [](Interval& a, Interval& b){return a.start < b.start; };
std::sort(nums.begin(), nums.end(), mycmp);
```

**注意：**
STL中已经为常见类型定义了比较方法。例如，对int或浮点数来说，它默认按从小到大排序，即默认使用的是`std::less<int>()`或`std::less<float>()`。如果想要按从大到小排序，可使用C++已经定义好的std::greater<int>()，无需自定义函数了。不过一定注意后面的**括号**，即：

```c++
std::sort(nums.begin(), nums.end(), std::greater<int>());
```

这是因为，`std::less, std::greater`等是一个struct template type，加上括号后才是函数指针。


#### In Containers

在container中使用自定义比较函数要稍微麻烦一些。传统的函数定义方法需要定义一个struct并且重载operator ()，例如：

```c++
// Interval is some custom class defined previously
struct Cmp{
	bool operator() (const Interval& a, const Interval& b) const{
        return a.start < b.start;
	}
};
// Use Cmp in containers
set<Interval, Cmp> mySet; // usage in set
priority_queue<Interval, vector<Interval>, Cmp> myQueue; // usage in max-heap
```

类似上面在STL函数中的**注意**部分：如果是C++中自带的container，可以使用std::less, std::greater等template struct type，不过注意此时**不需要**在后面加上括号。这是因为它本身是template struct type，加上括号后是function pointer。例如：

```c++
set<float, std::greater<float>> mySet; // max-to-min set
priority_queue<int, vector<int>, std::greater<int>> myQueue; // min-heap
```

这里注意，priority_queue默认是**最大堆**，但是其比较函数和它的排序方式是相反的。例如，比较函数是'大于'，对应则是最小堆；反之则是最大堆。

如果使用C++14 lambda expression将会简洁很多：

```C++
// Interval is some custom class defined previously
auto Cmp = [](Iterval& a, Iterval& b){ return a.start < b.start;}; // custom comparator

set<Interval, decltype(Cmp)> mySet(Cmp); // min-to-max set
priority_queue<Interval, vector<Interval>, decltype(Cmp)> myQueue(Cmp); // max-heap
```

**注意两点：**
- 使用decltype()来获取比较函数的类型;
- 变量名后要使用(Cmp)，即把函数指针放入括号


#### An example

下面是一个完整例子。输入一个二维数组，其中每一个一维vector均是排好序的。要求将所有的vector按照从小到大的顺序放入到set中，其中两个vector之间的比较方法就是把它看做一个串起来的字符串，按照字典顺序 (lexicographic order) 比较。例如{1,2} < {3,5}，{1,2} < {1, 2, 4}，{1, 3} > {1, 2, 5}等。


```C++
vector<vector<int>> nums = {{3, 5, 7}, {1, 2, 4}, {3, 6}, {1, 5}, {4, 4, 5}, {2, 6}};
using iter = vector<int>::iterator;
auto comp = [](pair<iter, iter> it1, pair<iter, iter> it2) {
    while (it1.first != it1.second && it2.first != it2.second){
        if (*it1.first > *it2.first)
            return false;
        else if (*it1.first < *it2.first)
            return true;
        else{
            it1.first++;
            it2.first++;
        }
    }
    if (it1.first == it1.second)  // length of it1's vector is <= it2's
        return true;
    if (it2.first == it2.second)  // vector by it1 is shorter than that of it2
        return false;
    return true;
};
set<pair<iter, iter>, decltype(comp)> st(comp);
for (auto& vec : nums)
    st.insert({vec.begin(), vec.end()});
```

一个巧妙之处是，这里的set中保存的并非数组的reference，而是数组的范围指针。这是因为，在comp中使用指针操作显得清楚易懂。

### inplace_merge 归并排序

https://en.cppreference.com/w/cpp/algorithm/inplace_merge

inplace_merge()用来合并两个已经排好序的区间，即merge sort中的最后一步的merge过程。用法是：Merges two consecutive sorted ranges [first, middle) and [middle, last) into one sorted range [first, last).

显然，它可以用在merge sort中。另外一个应用是可以和 `nth_element()`一起使用。nth_element用来获取第k大的值，并且将数组分成了左右两部分，左边都是<=第k大的值（但是本身无序），右边都是>第k大的值（但是本身也无序）。

举例：

```c++
template<class Iter>
void merge_sort(Iter first, Iter last)
{
    if (last - first > 1) {
        Iter middle = first + (last - first) / 2;
        merge_sort(first, middle);
        merge_sort(middle, last);
        std::inplace_merge(first, middle, last);
    }
}
```

### Erase-remove idiom 删除container中的元素

https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Erase-Remove

有关 `std::remove(first, last, val)`的使用：
- 它并不是真正从数组中删除了元素，而是把数组中等于val的元素都放在了数组后面，不等于val的放在数组前面，然后返回第一个满足条件（等于val）的值的位置指针`first_iter`。
- 因此，如果需要真正删除数组中等于val的所有元素，需要接着使用 `vector.erase(first_iter, last)`；
- `std::remove_if` 的用法完全一样。

About std::remove:
- `std::remove` algorithm does not eliminate elements from the container! It simply moves the elements not being removed to the front of the container, leaving the contents at the end of the container undefined. 
- `std::remove` algorithm returns an iterator to the beginning of the range of "unused" elements. It does not change the end() iterator of the container nor does the size of it. Member erase function can be used to really eliminate the members from the container in the following idiomatic way.
- So is `std::remove_if`.

```c++
template<typename T>
inline void remove(std::vector<T> & v, const T & item)
{
    v.erase(std::remove(v.begin(), v.end(), item), v.end());
}

std::vector<int> v;
// fill it up somehow
remove(v, 99);// really remove all elements with value 99
```

### Hash自定义方法，以及使用pair作为hashmap的key值

- https://en.cppreference.com/w/cpp/utility/hash
- https://stackoverflow.com/questions/4870437/pairint-int-pair-as-key-of-unordered-map-issue/4870467

C++中默认不支持使用std::pair作为hashmap的key值。如果想要做到这点，需要为你想用作key值的类型自定义hash。

以std::pair为例，自定义hash的标准方法是，先用std::hash函数分别生成这个pair中两个元素的hash值h1和h2，然后，有两种方法生成这个pair的hash：
- 使用 `h1 ^ (h2 << 1)`，这个结果就是最终的hash，直接返回即可；
- 使用`boost::hash_combine()`函数，参见下面代码。

感觉上，使用boost要更加general一些，适用于几乎所有类型，参见下面代码。

```c++
namespace std {
    
template <class T, class U>
struct hash< std::pair<T, U> > {
    size_t operator()( std::pair<T, U> const &p) const {
        std::size_t _1 = std::hash<T>()(p.first);
        std::size_t _2 = std::hash<T>()(p.second);
        std::size_t retval = 0;
        boost::hash_combine(retval, _1);
        boost::hash_combine(retval, _2);
        return retval;
    }
};
```

上面代码是直接在namespace std中定义这个hash结构体。这样的一个好处是，使用该hash定义hashmap时和普通时候完全相同：
```
unordered_map<std::pair<int, int>, double> mp;
```

即它省略了定义hashmap时的第三个参数：struct类型名称，因为C++默认会为pair类型寻找std::hash函数，而我们已经定义了它。如果不是这样，则第三个参数则不能省略，例如如果在其它的namespace A中（或者直接没有在任何namespace中）定义了hash结构体，那么使用时就必须是：
```
unordered_map<std::pair<int, int>, double, A::hash_struct_name> mp;
```
如果没有namespace，则去掉上面的`A::`这几个字符。


## STL Containers 容器

### Vector 数组

这里记录几个特别的函数：back() 和 pop_back()，前者用于获取数组末尾元素的值，后者用于踢出数组末尾元素。不过，vector并没有 front() 或者 pop_front() 函数，如果有需要，可以使用std::deque 或者 std::list。

#### assign() vs resize()

`assign(n, val)` 将数组大小设置为n，且**所有**元素设置为val。而 `resize(n, val)` 也是把数组大小设置为n，但是它仅仅把**新增加的**元素设置为val，而不会影响vector已有的元素值。因此，显然resize通常用于扩大一个数组并且不影响当前元素，而assign则通常用于修改数组大小并且重新初始化。

当然，如果一个vector本身为空，两者自然就没有区别。


### List 链表

和vector不同的是，list包含插入或者踢出头部和尾部的方法：push_front(), push_back(), pop_front(), pop_back()，以及获取头部和尾部数值的引用的方法front()和back()，操作都是O(1)时间。

https://www.geeksforgeeks.org/linked-list-vs-array/

LinkedList相对于Array的优点：
- Dynamic size
- Ease of insertion/deletion


#### splice(): 移动链表元素

http://www.cplusplus.com/reference/list/list/splice/

```c++
splice(iterator position, list& x);
splice(iterator position, list& x, iterator i);
splice(iterator position, list& x, iterator first, iterator last);
```

list中的splice成员函数用于将一个链表**全部或者部分**元素插入到另一个链表中（也可以是同一个链表）。例如：

```C++
std::list<int> mylist1, mylist2;
std::list<int>::iterator it;

// set some initial values:
for (int i = 1; i <= 4; ++i)
    mylist1.push_back(i);  // mylist1: 1 2 3 4

for (int i = 1; i <= 3; ++i)
    mylist2.push_back(i * 10);  // mylist2: 10 20 30

it = mylist1.begin();
++it;  // points to 2

// mylist1: 1 10 20 30 2 3 4
// mylist2 (empty)
// "it" still points to 2 (the 5th element)
mylist1.splice(it, mylist2); 

// mylist1: 1 10 20 30 3 4
// mylist2: 2
// "it" is now invalid.
mylist2.splice(mylist2.begin(), mylist1, it);
```

**splice可以移动链表中某些元素到同一个链表的其它位置**

list1.splice(list2) 在把list2中元素插入到list1中后，也会把list2中这部分插入的元素删除。因此，list1.splice(list1) 就相当于把list1本身的部分元素移动到了其它位置。这种操作非常有用。

### list vs deque 比较

https://stackoverflow.com/questions/1436020/whats-the-difference-between-deque-and-list-stl-containers

deque的底层类似普通数组vector，而list的底层是链表linked list，因此这两者的特点和它们底层容器的特点是一致的。

#### 访问元素
- Deque manages its elements with a dynamic array, provides random access, and has almost the same interface as a vector. 双向队列deque类似于一个动态数组。类似于std::vector，它也可以使用index访问任意元素，本质上就相当于std::vector的扩展，两者的API函数也很类似，只是deque多了push_front和pop_front()操作而已。
- List manages its elements as a doubly linked list and does not provide random access. 链表list就是双向链表，无法通过index访问元素(当然如果通过interator是可以的)。

注意：deque是可以通过index访问元素的，虽然用处不多。

#### 相关操作的时间
- Deque provides Fast insertions and deletions at both the end and the beginning. Inserting and deleting elements in the middle is relatively slow because all elements up to either of both ends may be moved to make room or to fill a gap. 双向队列deque可以在Amortized O(1)时间(平均时间)进行头尾元素的增加和删除，但是需要O(N)时间在中部插入或删除元素。
- In List, inserting and removing elements is fast at each position, including both ends. 双向链表list可以在O(1)时间在任意部位插入删除元素。

#### 内存处理
- Deque: Any insertion or deletion of elements other than at the beginning or end invalidates all pointers, references, and iterators that refer to elements of the deque. 双向队列deque会在删除中间元素时，释放所有相关的指向这些元素的指针的内存(其实就是对应vector::erase()操作，它会把待删除的元素后面的元素全部移动到刚刚删除的元素位置）。
- List: Inserting and deleting elements does not invalidate pointers, references, and iterators to other elements. 相反的，双向链表list在删除元素时，并不会释放相关内存。


### set/unordered_set, multiset/unordered_multiset

- set和multiset的构成是tree结构（为了排序需要），因此检索和删除的最坏时间是O(logn)（即at worst），它们的底层实现类似BST，即平衡二叉树。而unordered_set和unordered_multiset的底层是hash table结构，故检索和删除的平均时间是O(1)（虽然最坏时间是O(n)）。
- 对于multiset和unordered_multiset来说，erase(key)会删除所有的值为key的元素;
- 注意：multiset 和 multimap 不存在 [] 操作符。如果想要插入数据，可以使用成员函数insert({key, value}).

### priority-queue 优先队列/堆

priority_queue默认是**最大堆**，它只需O(1)时间来获取最大值，并且插入和删除元素的时间是O(logn)。

A priority queue is a container adaptor that provides constant time lookup of the largest (by default) element, at the expense of logarithmic insertion and extraction.

```c++
// min-heap with standard comparator std::greater<T>
std::priority_queue<int, std::vector<int>, std::greater<int>> q2;

// Using lambda to compare elements.
auto cmp = [](int left, int right) { return (left ^ 1) < (right ^ 1); };
std::priority_queue<int, std::vector<int>, decltype(cmp)> q3(cmp);
```

关于自定义比较函数，参见前面的"[Custom Comparator自定义比较函数](#custom-comp)"中的具体内容。

## Smart Pointers (unique_ptr, shared_ptr, weak_ptr) 智能指针

自己写的测试用代码: https://github.com/chaowang15/test_code/blob/master/cpp_test/smart_pointer/smart_pointer.cpp

### unique_ptr

Official Ref: https://en.cppreference.com/w/cpp/memory/unique_ptr

对于一个object来说，最多只有一个unique_ptr能指向它。因此，unique_ptr无法复制。

For an object/instance, at most one `unique_ptr` can point at it.

```c++
unique_ptr<A> p0 = make_unique<A>(); // use 'make_unique' to create a pointer
unique_ptr<A> p1 (new A); // use new to create an unique pointer
// unique_ptr<A> p2(p1); // ERROR: only 1 unique pointer is allowed to point to an object
// unique_ptr<A> p2 = p1; // ERROR: same as above
unique_ptr<A> p2 = std::move(p1); // OK: now p1 is nullptr and holds nothing, while p2 holds the object
A* p3 = p1.get(); // get() returns the stored pointer
```

Pointing to a C-array: https://docs.microsoft.com/en-us/cpp/cpp/how-to-create-and-use-unique-ptr-instances?view=vs-2019

```c++
// Create a unique_ptr to an array of 5 integers.
auto p = make_unique<int[]>(5);

// Initialize the array.
for (int i = 0; i < 5; ++i)
{
    p[i] = i;
    wcout << p[i] << endl;
}
```

仅有一个例外情况下，unique_ptr可以复制：作为函数返回值：

```c++
unique_ptr<int> clone(int p)
{
    unique_ptr<int> ptr = make_unique<int>(p);
    return ptr;
}
```

In this case, the compiler knows the object being returned is about to be destroyed. So, it does a special kind of copy.


### shared_ptr

Official Ref: 
- https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
- https://docs.microsoft.com/en-us/cpp/cpp/how-to-create-and-use-shared-ptr-instances?view=vs-2019

多个shared_ptr可以指向同一个object。当最后一个shared_ptr被清理后，它们所指向的object才会被删除。

Multiple shared_ptr can points at one object/resource.

```c++
shared_ptr<A> p0 = make_shared<A>(); // use 'make_shared' to create a pointer
shared_ptr<A> p1 (new A); // use new to create a shared pointer
shared_ptr<A> p2(p1); // OK: multiple shared pointers can point to a same object
A* p3 = p1.get(); // get stored pointer
int num = p1.use_count(); // 'use_count()': get number of shared_ptr copies
```

### weak_ptr

Ref:
- Official ref: https://docs.microsoft.com/en-us/cpp/cpp/how-to-create-and-use-weak-ptr-instances?view=vs-2019
- Typical weak_ptr case in cyclic reference: https://thispointer.com/shared_ptr-binary-trees-and-the-problem-of-cyclic-references/

Features:
- weak_ptr指向shared_ptr所指向的object，但是它不会更改shared_ptr的个数（即use_count()返回值不变）。
- weak_ptr指向的Object的删除依然同shared_ptr相关，和该weak_ptr毫无关系。即：有可能object已经删除了，但是weak_ptr依然存在（当然，此时它指向的位置是不合理的）。这点和shared_ptr不同：所有的shared_ptr都清理掉后，它们所指向的object才会被删除。因此，每次都需要首先使用lock()函数来判断它所指向的对象是否存在：

```c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);
...
if (shared_ptr<int> np = wp.lock()) // if the object p points to still exists, then np is not nullptr
{
    // access the object with np
}
```

- weak_ptr doesn't control the lifetime of the object to which it points to. Instead, it points to an object managed by a `shared_ptr`. 
- Binding a weak_ptr to a shared_ptr does NOT change the reference count of that shared_ptr (that is, result of `use_count()` stays the same). 
- The object which weak_ptr points to will be deleted as long as the last shared_ptr goes away, no matter if weak_ptr exists or not. This is why it is named as `weak`.


当两个object中存在shared_ptr相互指向对方的object时，这两个object将无法被release，即形成了“环”。原因是，对于每个object，系统只会在它的reference count（即指向它的shared_ptr个数）等于1时才会release。如果有环，那么每个object都会被2个shared_ptr所指向。此时，可以使用weak_ptr来替代其中一个shared_ptr以去掉环结构。

Given a special cyclic example that object A has `shared_ptr` pointing to object B, while object B has `shared_ptr` pointing to A. A typical example is a binary Tree where each node has children pointers and a parent pointer, then a parent node and child node will point to each other. Then, both A and B will not be released since each shared_ptr will have `use_count() > 1` (since an object will only be released/destroyed when its shared_ptr's count is equal to 1). 

To break this, we can utilize a weak_ptr to avoid cycles of pointers (such as replacing the parent pointer with a weak_ptr in the above binary tree case).

### Don't assign an existing object to a smart pointer

https://stackoverflow.com/questions/28502067/how-to-assign-an-address-of-an-existing-object-to-a-smart-pointer

In this case, use a common pointer instead. Seems a smart pointer is only created from a new object.


### Custom deleter for unique_ptr and shared_ptr

See the code link for details: https://github.com/chaowang15/test_code/blob/master/cpp_test/smart_pointers/smart_pointers.cpp


## Comparison between unique_ptr and shared_ptr

- https://www.geeksforgeeks.org/auto_ptr-unique_ptr-shared_ptr-weak_ptr-2/
- https://stackoverflow.com/questions/6876751/differences-between-unique-ptr-and-shared-ptr

#### Same

都是智能指针：当指针不再使用时，系统会自动release/deallocate该指针指向的对象。

Both of these classes are **smart pointers**, which means that they automatically (in most cases) will **deallocate** the object that they point at when that object can no longer be referenced. The difference between the two is how many different pointers of each type can refer to a resource. 

#### Difference

- `unique_ptr`：仅有1个该指针指向同一个对象，即该指针无法复制。
- `shared_ptr`：多个指针都可以指向同一个对象。当最后一个指针不再使用时，该对象才会被release。

In short:

- Use `unique_ptr` when you want a single pointer to an object that will be reclaimed when that single pointer is destroyed.
- Use `shared_ptr` when you want multiple pointers to the same resource.

## Structured binding declaration (Since C++ 17)

Ref:
- Official: https://en.cppreference.com/w/cpp/language/structured_binding
- Another one: https://stackoverflow.com/questions/40673080/stdignore-with-structured-bindings

Quick and easy unpacking in C++ with structured bindings.

### Binding to an array

```c++
int a[2] = {1,2};
 
auto [x,y] = a; // creates e[2], copies a into e, then x refers to e[0], y refers to e[1]
auto& [xr, yr] = a; // xr refers to a[0], yr refers to a[1]
```

### Binding to a tuple

```c++
auto tuple = std::make_tuple(1, 'a', 2.3);
std::array<int, 3> a{1, 2, 3};

// unpack the tuple into individual variables declared above
const auto [i, c, d] = tuple;
// same with an array
auto[x,y,z] = a; 
```

### Binding to a data members like struct

```c++
struct S {
    int x1 : 2;
    volatile double y1;
};
S f();
 
const auto [x, y] = f(); // x is a const int lvalue identifying the 2-bit bit field
                         // y is a const volatile double lvalue
```

#### Note: Partial extraction/unpacking is not allowed so far

http://dominikberner.ch/structured-bindings/

That is, for instance of the **Binding to a tuple** case, you have to use `[i,c,d]=tuple` even though you may not want to use some variable of `i,c,d` (I remember in python or matlab this is supported by using `-` or `*` to replace the variable you don't want).

## Function type and function pointer 函数类型和函数指针

- Official reference about pointers: https://en.cppreference.com/w/cpp/language/pointer
- Code about function pointer: https://github.com/chaowang15/test_code/blob/master/cpp_test/function_pointer/function_pointers.cpp

**Function type** is determined by both return type and input parameters. Two function types are the same if both return type and input parameters are the same.

```c++
int square(int a); // this function type is 'int(int)'
```

**Function pointer** is a pointer to a function:

```c++
int (*pf)(int); // pf is a function pointer which points to a function type 'int(int)'
```

注意："函数类型"和"函数指针"是不同的。和函数类型不同的是，函数指针可以被复制，拷贝或者放在数组中，因此应用很广。

### Non-member function name is a function pointer by default 函数名其实就是函数指针

Let's firstly look at C-array type as a function parameter:

```c++
//! C-array as function parameter.
void sum(int *nums);
void sum(int nums[]);    // Same as above: array name is actually a pointer to an array
void sum(int nums[10]);  // Same as above: dimension will be ignored. But this is good for documentation purposes.
```

数组名其实是数组指针，这点和"函数名也是函数指针"类似。对于普通的非类成员函数，函数名就是函数指针。地址操作符（即引用符号'&'）加不加都是一样的。

For instance, the above function name `square` is actually a function pointer with exactly the same type as `pf`. So we can do this:

```c++
pf = square; // function name is function pointer by default
pf = &square; // OK: exactly the same as above. The reference sample is optional.
pf(2); // OK: this runs 'square(2)'
int pf2(int);
pf2 = square; // ERROR: 'pf2' is a function type but 'square' is a function pointer
```

### Function pointer is used as a function parameter

**Array name is actually a pointer to an array**. Function pointer is similar: function name is actually a function pointer to a function. 

函数类型和函数指针都可以作为一个函数的输入参数。只是如果是函数类型作为输入参数时，它会被自动转化为函数指针。

A function type as a function parameter will be converted to a function pointer automatically. 

```c++
void test(int pf(int), int a); // pf will be converted to a function pointer automatically
void test(int (*pf)(int), int a); // Same as above.
```
The above two functions are actually exactly the same. Actually the compiler will show error about 'redefinition of the same function'.

Some usage:

```c++
typedef int f1(int);
typedef decltype(square) f2; // f2 is same as f1: a function type
f2 = square; // ERROR: square is a function pointer
typedef int (*f3)(int); 
typedef decltype(square) *f4; // f4 is same as f3: a function pointer type
test(square, 2); // OK
test(f4, 2); // OK: same as above
void test(f1, int);  // OK: f1 will be converted to a function pointer automatically
void test(f3, int);  // OK
```

### Function pointer is used as a return type

函数类型无法作为函数的返回值：只有函数指针可以。
We cannot return a function type. We must return a function pointer.

```
f2 func(int); // ERROR: f2 is a function type and cannot be a return type
f3 func(int); // OK: f3 is a function pointer
f2* func(int); // OK: now it returns a pointer to a function type, which is a function pointer
```

### Pointer to member function

和和普通的非成员函数指针不同，类成员函数指针必须写成 `&C::f` 形式（即，必须加上取地址符号address-of operator，并且不能有括号），其中 `f` 是函数名。类成员变量指针也是完全相同的: `&C::m`，其中 `m` 是成员变量。

Pointer to class member function is exactly the same as the pointer to data member: `&C::m`, where `C` is class name and `m` is function name or a member variable name. Note that `&(C::m)` or `&m` are both WRONG expression.

```c++
struct Base
{
    int add(int n)
    {
        int sum = val + n;
        return sum;
    }
    int val = 5;
};

void testMemberPointer()
{
    int (Base::*p)(int) = &Base::add; // define and initialize a member function pointer type
    decltype(&Base::add) r = &Base::add; // same as above
    auto q = &Base::add; // this is much simpler

    Base base;
    (base.*p)(2); // NOTE: this is the only correct usage of calling a pointer to member function
    // base.*p(2); // ERROR
    // (base.(*p))(2); // ERROR
    // base.p(2); // ERROR

    Base *bptr = &base;
    (bptr->*p)(3); // ONLY correct way
    // bptr->*p(2); // ERROR
    // (bptr->(*p))(2); // ERROR
    // bptr->p(2); // ERROR
}
```

### std::bind, std::ref, std::cref 函数参数绑定和参数引用

Official Ref: 
- https://en.cppreference.com/w/cpp/utility/functional/bind
- https://en.cppreference.com/w/cpp/utility/functional/ref

特点：
- bind()用来绑定一个函数的某些输入参数，以建立新的函数。
- `std::ref()`和`std::cref`用来获取参数的引用
- bind()的输入参数默认全都不是引用（即便你用了引用符号 & 也不是），除非使用`std::ref()`或者 `std::cref`(reference to const)

Features:
- Used as a function wrapper. Calling this wrapper is equivalent to invoking f with some of its arguments bound to args.
- The arguments to bind are copied or moved, and are never passed by reference unless wrapped in std::ref or std::cref.

```c++
void f(int n1, int n2, int& n3, const int& n4, int& n5)
{
	n3++;
	//n4++; // compile error
	n5++;
}
int main()
{
    using namespace std::placeholders;  // for _1, _2, _3...
    int m = 5, n = 7;
    // _1, _2, _3 are from std::placeholders, and represent the first and second future arguments of f1
    auto f1 = std::bind(f, _2, _3, std::ref(m), std::cref(n), n);
    n = 10;

    // Call f(2, 1001, 5, 10, 7). The last parameter is 7 since it is not a reference.
    f1(9, 2, 1001); // 9 is unused

    // After the call, m = 6, n = 10
    // NOTE that m is added by 1 in f(), but n is unchanged, because bind() will ignore a common reference (&)
    // unless using std::ref()
}
```

## Resource Acquisition is Initialization (RAII) 

### 原理说明

- Wiki page with lock_guard example: https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization

- cppreference page about RAII with lock_guard example, and C++ library classes supporting RAII: https://en.cppreference.com/w/cpp/language/raii


RAII翻译成“资源获取即初始化”。首先，这个名字翻译的很烂。RAII是C++ 的一种机制。它想做的事情是，对于那种在使用之前必须获取（acquire）的资源（例如分配的内存、socket、打开的文档handles、数据库链接等），RAII将会把这种资源的生命周期绑定到一个基于RAII机制的object的生命周期（即两者生命周期相同）。即，当这个被绑定的object被release时，它所绑定的资源也会被release。

例如，我们想用一个文件的handle，例如file，来读取某个大文件或者将一个数据写入文件。如果不用RAII，那么，如果在读取或者写入出错的时候，它会throw an exception，然后提前终止了程序。此时，这个handle以及它的资源其实并没有被release，因为程序还没有读取或者写入完毕，因此根本没有运行到release资源这个部分。这显然不好。

如果使用RAII，例如用lock_guard(mutex)，然后再进行读写文件，那么，即使出现exception，RAII机制会保证这个lock_guard的scope范围内的资源都被release掉，而不管有没有出现exception。

STL标准库中遵循RAII的类有很多，例如std::string, std::vector, std::thread，以及其他一些类似的containers等，它们默认都会在constructor中获取资源，然后在destructor中释放资源。即，它们**不需要显示的释放或清理（explicit cleanup)**。

另外还有 std::unique_ptr和std::shared_ptr等用来管理动态资源的智能指针，以及std::lock_guard, std::unique_lock, std::shared_lock等用来管理mutex。


### 另一个解释以及例子

下面是来自stackoverflow上的解释以及一个例子：https://stackoverflow.com/questions/2321511/what-is-meant-by-resource-acquisition-is-initialization-raii

It's a really terrible name for an incredibly powerful concept, and perhaps one of the number 1 things that C++ developers miss when they switch to other languages. There has been a bit of a movement to try to rename this concept as Scope-Bound Resource Management, though it doesn't seem to have caught on just yet.

When we say 'Resource' we don't just mean memory - it could be file handles, network sockets, database handles, GDI objects... In short, things that we have a finite supply of and so we need to be able to control their usage. The 'Scope-bound' aspect means that the lifetime of the object is bound to the scope of a variable, so when the variable goes out of scope then the destructor will release the resource. A very useful property of this is that it makes for greater exception-safety. For instance, compare this:

```c++
RawResourceHandle* handle=createNewResource();
handle->performInvalidOperation();  // Oops, throws exception
...
deleteResource(handle); // oh dear, never gets called so the resource leaks
```

With the RAII one
```c++
class ManagedResourceHandle {
public:
   ManagedResourceHandle(RawResourceHandle* rawHandle_) : rawHandle(rawHandle_) {};
   ~ManagedResourceHandle() {delete rawHandle; }
   ... // omitted operator*, etc
private:
   RawResourceHandle* rawHandle;
};

ManagedResourceHandle handle(createNewResource());
handle->performInvalidOperation();
```

In this latter case, when the exception is thrown and the stack is unwound, the local variables are destroyed which ensures that our resource is cleaned up and doesn't leak.


## Multi-threading 多线程

- Official Ref of mutex: https://en.cppreference.com/w/cpp/thread/mutex
- Official Ref of lock_guard: https://en.cppreference.com/w/cpp/thread/lock_guard
- Locking mutex and RAII: http://kayari.org/cxx/antipatterns.html#locking-mutex

```c++
std::map<std::string, std::string> g_pages;
std::mutex g_pages_mutex;
 
void save_page(const std::string &url)
{
    // simulate a long page fetch
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::string result = "fake content";
 
    std::lock_guard<std::mutex> guard(g_pages_mutex);
    g_pages[url] = result;
}
 
int main() 
{
    std::thread t1(save_page, "http://foo");
    std::thread t2(save_page, "http://bar");
    t1.join();
    t2.join();
 
    // safe to access g_pages without lock now, as the threads are joined
    for (const auto &pair : g_pages) {
        std::cout << pair.first << " => " << pair.second << '\n';
    }
}
```

## Class

### Initialization of constant class members 初始化const成员变量

类中的const成员变量显然是不能在成员函数内更改的，它甚至**也不能在构造函数内部初始化**。那么如何初始化？有几种方法：
- 直接在定义处赋值。这个是最简单的。例如：
```c++
class Foo{
private:
    const int num_ = 100;
};
```
显然，该方法适用于对所有的该类的objects全部使用相同的constant变量。

- 可以在初始化列表（initialization list）中初始化：
```c++
class Foo{
    Foo (int x, int y): x_(x), y_(y){
        // Do sth. else
    }
private:
    const int x_, y_;
};
```
该方法成立的原因是，初始化列表**不被认为是进入了构造函数的内部**。这种方法的好处是，定义该类的一个object后，便将传入的参数赋值到了const成员变量中，将来就无法改变了。这更适用于初始化不同，但是又想要将来固定住这些成员变量的情况，例如从一个外部文件读入的原始数据。
- 直接将成员设定成static const，这样就可以在类的外部初始化了：
```C++
class A
{
private:
    static const int a; //declaration
};
const int A::a = 10; //defining the static member outside the class
```
不过这显然有点投机取巧的感觉，并不适用于有大量的const members的情况。

### Concrete class Vs. Abstract class 实体类 Vs. 抽象类

https://www.sanfoundry.com/cpp-program-differentiate-concrete-class-abstract-class/

- 实体类是没有任何virtual functions的类，即它的instance可以被初始化的类。
- 抽象类是某些或全部function都是Virtual的类。它的instance不能被初始化，故它只能作为子类的父类使用。

- An abstract class is meant to be used as a base class where some or all functions are declared purely virtual and hence can not be instantiated. 
- A concrete class is an ordinary class which has no purely virtual functions and hence can be instantiated.

### Virtual functions 虚函数

虚函数的作用通常是，强制派生类定义该虚函数，否则编译出错。

#### override关键字

C++11的新关键字：在子类的函数声明中使用override来表示该函数是继承自基类的虚函数。方法很简单： 派生类中的`void func() override;` 表示基类中有虚函数`virtual func();`。

In a member function declaration or definition, override ensures that the function is virtual and is overriding a virtual function from a base class. 

#### Better use virtual destructor for base class 在基类中最好使用虚析构函数

原因是，基类的函数会被派生类的相同名字的函数所覆盖。因此，如果基类的析构函数不是虚函数，那么在delete一个派生类的object时，基类的析构函数不会被调用。这样有可能会造成内存泄露。因此，最好在基类中使用virtual destructor。当然也要对其定义，即便内容是空。

#### Pure virtual functions 纯虚函数

- 如果要定义一个抽象类Abstract Class（抽象类是指，无法使用该类创建object），那么至少有一个函数必须是纯虚函数，即在虚函数的声明最后使用`= 0`关键字。
- 可以无需定义纯虚函数。但是有一个例外：纯虚析构函数。它必须要定义，参见下面内容。

#### Virtual destructors or pure virtual destructors must be defined 虚析构函数必须要定义 

Ref: https://www.studytonight.com/cpp/virtual-destructors.php

虚析构函数必须要定义，无论它是不是纯虚函数。原因很简单：派生类的object被销毁时，也将会调用base类的析构函数。因此，析构函数必须要定义（否则会出现compile error）。因此，一个好的习惯是，**在任何情况下都定义析构函数**，无论它是不是虚函数。

This is because the destructor of a base class is always called when a derived object is destroyed. Failing to define it will cause a link error.

```c++
class Base
{
public:
    Base(){}
    virtual ~Base() = 0; // pure function
};
Base::~Base()
{}
```

如果析构函数是纯虚函数，即`virtual ~Base() = 0;`，那么需要在类的外部定义该虚构函数。如果只是普通的虚函数，在类的内部定义也可以。例如`virtual ~Base() {}` ，或者使用关键字 `=default` 以使用系统默认的析构函数：`virtual ~Base() = default;`

#### Virtual functions and polymorphism 虚函数和多态

有至少一个虚函数的classes才能形成多态。

### Pointers to base class or derived class 指向基类或者派生类的指针
