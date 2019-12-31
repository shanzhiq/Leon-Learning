<center> c++ 常见格式转换</center>
```c++
//字符串转数字

```

<center ><font color='red'>map的问题</font></center>

map赋值相比于insert函数来说，insert函数更好。

```c++
int main()

{

    map<int, string> mapMy;

    mapMy[0] = "My"; // 第一种赋值方式：[ ]下标操作符

 

    map<int, string> mapYou;

    mapYou[0] = "a";

    mapYou[1] = "b";

    mapYou[2] = "c";

 

    // 第二种方式：insert方法

    // map的insert函数其中的一种，将一个map指针指向的区间内的值，赋值到自己当中

    mapMy.insert(mapYou.begin(), mapYou.end()); 

 

    map<int, string>::iterator iter2;

    iter2 = mapMy.begin();
	 for (auto i=mapYou.begin(); i != mapYou.end(); ++i)

    {

        cout<< i->second << endl;

    }
    for (; iter2 != mapMy.end(); ++iter2)

    {

        cout << iter2->second << endl;

    }
	

    return 0;

}
```

这里的运行结果是：

```
第一种情况：

a

b

c

第二种情况：

My

b

c
```

