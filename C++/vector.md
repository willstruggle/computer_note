# vector

使用vector，需添加头文件#include<vector>，
   要使用sort或find，则需要添加头文件#include<algorithm>。
   为了简化书写，需在.h中增加using namespace std;

1.vector的初始化及赋值
```
　　std::vector<int> nVec;　　　　 // 空对象
　　std::vector<int> nVec(5,-1);　 // 创建了一个包含5个元素且值为-1的vector
　　std::vectorstd::string strVec{"a", "b", "c"};　　// 列表初始化  　
　　要注意“（）”和“{}”这样的初始化情况，比如：
　　std::vector<int> nVec(10，1);　　　　// 包含10个元素，且值为1
　　std::vector<int> nVec{10，1};　　　　// 包含2个元素，值分别为10,1
　　然而，一般在程序中，并不会知道vector的元素个数，故使用以上方式倒显得繁琐，所以可以使用push_back，它会负责将一个值当成vector对象的尾元素“压到（push）”vector对象的“尾端(back)”。比如：
　　std::vector<int> nVec;
　　for(int i = 0; i < 5; ++i)
　　　　nVec.push_back(i);　　　　// 压入元素
　　for(size_t i = 0; i < nVec.size(); ++i)
　　　　std::cout << nVec[i] << std::endl;　// 输出元素
　　其中size()是获取vector元素的个数，另外vector中可使用empty()来返回vector中是否存在元素，如果为空，则返回true，否则返回false。同时，针对nVec[i]是通过下标运算符来获取对应的vector数值的，千万注意，针对于空的vector，万不可通过下标运算符来添加元素，比如：
　　std::vector<int> nVec;
　　for(int i = 0; i < 5; ++i)
　　　　nVec[i] = i;　　　　// error
```
　　这样编写代码是错误的，nVec是空的，不包含任何对象。当然也就不可能通过下标来添加或访问任何元素。若要添加请使用push_back。
　　当然，针对于输出，可使用迭代器iterator来表示，比如上面的例子可写成：
```
　　std::vector<int>::iterator itr = nVec.begin();
　　for(; itr != nVec.end(); ++itr)
　　　　std::cout << (*itr)  << std::endl;
```
　　针对于iterator有两种标准库类型： iterator 和 const_iterator。
　　两者的区别主要是后者类似于常量指针，只能读取不能修改。如果vector对象不是常量，两者均可使用。

2.vector中插入元素
　　vector,deque,list和string都支持insert成员，使用insert可在容器的任意位置插入0个或多个元素。
一般insert函数将元素插入到迭代器所指定的位置之前，比如：
```
　　slist.insert(iter,"hello");　　　　　　// 将Hello添加到iter之前的位置
　　要注意，将元素插入到vector,deque和string中的任何位置都是合法的，但是这样做会很耗时。
　　c.insert(pos,num);　　　　// 在pos位置插入元素num
　　c.insert(pos,n,num);　　　// 在pos位置插入n个元素num
　　c.insert(pos,beg,end);　　// 在pos位置插入区间为[beg,end)的元素
```

3. vector删除元素
　　针对于非array容器有多种删除方式，以erase为例，比如：
```
　　c.erase(p);　　　　　　// 删除迭代器p所指定的元素，返回一个指向被删除元素之后的迭代器。
　　c.erase(begin,end);　  // 删除b,e所指定范围内的元素，返回一个指向被删除元素之后的迭代器。
　　c.clear();　　　　　　　// 删除所有元素
```
　　注意，删除元素，会导致迭代器无效。故下面的编写方式是错误的，比如：
```
　　std::vector<int> nVec;
　　for(int i = 0; i < 5; ++i)
　　　　nVec.push_back(i);
　　std::vector<int>::iterator iter = nVec.begin();
　　for(; iter != nVec.end(); ++iter)
　　{
　　　　if(*iter == 1)
　　　　　　nVec.erase(iter);
　　}
```
　　正确的方式是(删除特定元素)：
```
　　std::vector<int>::iterator iter = nVec.begin();
　　for(; iter != nVec.end();)
　　{
　　　　if(*iter == 0)
　　　　　　iter = nVec.erase(iter);
　　　　else
　　　　　　iter++;
　　}
```
　　删除容器内某一个特定的元素，编写方式可为：
```
　　std::vector<int>::iterator iter = std::find(nVec.begin(),nVec.end(),5);
　　if(iter != nVec.end())
　　　　nVec.erase(iter);
```
　　删除容器内某一段范围内的元素，编写方式可为：
```
　　first = std::find(nVec.begin(),nVec.end(), value1);
　　last = std::find(nVec.begin(),nVec.end(), value2);
　　if(first != nVec.end() && last != nVec.end())　　　　// 判断有效性
　　{
　　　　nVec.erase(first,last);
　　}
```
　　删除容器内所有元素，当然可以这样：
```
　　nVec.erase(nVec.begin(),nVec.end());
```
　　也可以 nVec.clear();
　　
4. vector的容量与大小
　　vector并非随着每个元素的插入而增长自己，它总是分配一些额外的内存容量，这种策略使得vector的效率更高些。若要获取当前vector的大小，可调用size()函数，而获取当前vector的容量，可调用capcity()。
　　注意,list不需要容量，是由于它的每次增长，只是简单的链接新元素而已。
　　
5. 自定义类的排序
如果vector保存的内容为class，通过重写 <, ()或自定义的比较函数 compare_index均可。根据容器中保存内容不同，略有差异。
a.如果容器中是对象时，用操作符<或者比较函数，比较函数的参数是引用；
b.如果容器中是对象指针时，用（）或比较函数排序，比较函数的参数是指针；
c.排序使用std::sort
```
class TestIndex{
public:
int index;
TestIndex(){
}
TestIndex(int _index):index(_index){
}
bool operator()(const TestIndex* t1,const TestIndex* t2){
printf("Operator():%d,%d/n",t1->index,t2->index);
return t1->index < t2->index;
}
bool operator < (const TestIndex& ti) const {
printf("Operator<:%d/n",ti.index);
return index < ti.index;
}
};
bool compare_index(const TestIndex* t1,const TestIndex* t2){
printf("CompareIndex:%d,%d/n",t1->index,t2->index);
return t1->index < t2->index;
}
int main(int argc, char** argv) {
list<TestIndex*> tiList1;
list<TestIndex> tiList2;
vector<TestIndex*> tiVec1;
vector<TestIndex> tiVec2;
TestIndex* t1 = new TestIndex(2);
TestIndex* t2 = new TestIndex(1);
TestIndex* t3 = new TestIndex(3);
tiList1.push_back(t1);
tiList1.push_back(t2);
tiList1.push_back(t3);
tiList2.push_back(*t1);
tiList2.push_back(*t2);
tiList2.push_back(*t3);
tiVec1.push_back(t1);
tiVec1.push_back(t2);
tiVec1.push_back(t3);
tiVec2.push_back(*t1);
tiVec2.push_back(*t2);
tiVec2.push_back(*t3);
printf("tiList1.sort()/n");
tiList1.sort();//无法正确排序
printf("tiList2.sort()/n");
tiList2.sort();//用<比较
printf("tiList1.sort(TestIndex())/n");
tiList1.sort(TestIndex());//用()比较
printf("sort(tiVec1.begin(),tiVec1.end())/n");
sort(tiVec1.begin(),tiVec1.end());//无法正确排序
printf("sort(tiVec2.begin(),tiVec2.end())/n");
sort(tiVec2.begin(),tiVec2.end());//用<比较
printf("sort(tiVec1.begin(),tiVec1.end(),TestIndex())/n");
sort(tiVec1.begin(),tiVec1.end(),TestIndex());//用()比较
printf("sort(tiVec1.begin(),tiVec1.end(),compare_index)/n");
sort(tiVec1.begin(),tiVec1.end(),compare_index);//用compare_index比较
return 0;
｝
```

版权声明：本文为CSDN博主「xian_wwq」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/tpriwwq/article/details/80609371