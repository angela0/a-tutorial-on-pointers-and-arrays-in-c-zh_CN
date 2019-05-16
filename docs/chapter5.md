你可能知道，我们可以声明包含不同数据类型的一块数据，被称作结构体声明。例如，一个人事档案包含的结构可能像这样：

```
    struct tag {
        char lname[20]; /* last name */
        char fname[20]; /* first name */
        int age; /* age */
        float rate; /* e.g. 12.75 per hour */
    };
```

假设在磁盘文件中有一堆这种结构，我们想把每一个都读出来并打印出姓名，以便于我们在文件中生成一个列表。其余的信息并不打印出来。打印的时候用一个函数调用，并传给这个函数一个指向结构体的指针。为了演示方便，这里我只用一个结构体。注意，我们的目标是写出这个函数，而不是读取文件，所以假设你已经知道该怎么读。

回顾一下，我们可以通过 `.` 操作符访问结构体成员，像这样：

```
    //--------------- program 5.1 ------------------
    /* Program 5.1 from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <string.h>
    struct tag {
        char lname[20]; /* last name */
        char fname[20]; /* first name */
        int age; /* age */
        float rate; /* e.g. 12.75 per hour */
    };
    struct tag my_struct; /* declare the structure my_struct */
    int main(void)
    {
        strcpy(my_struct.lname,"Jensen");
        strcpy(my_struct.fname,"Ted");
        printf("\n%s ",my_struct.fname);
        printf("%s\n",my_struct.lname);
        return 0;
    }
    //-------------- end of program 5.1 --------------
```

注意到，这个结构比 C 中常用到的小很多，我们可以给它加好多：

```
    date_of_hire;           (没有给出数据类型)
    date_of_last_raise;
    last_percent_increase;
    emergency_phone;
    medical_plan;
    Social_S_Nbr;
    etc.....
```

如果我们有很多员工，处理结构体数据的话肯定是要用函数。例如，我们可能要打印出传到函数的结构体里面的员工姓名。然而，在原始的 C 里面（Kernighan & Ritchie, 1st Edition），不允许传递结构，只能传递结构体的指针。但在 ANSI C 里，是可以传递一个完整的结构体。既然我们是学习指针，所以不再讨论这个。

不管怎样，如果我们传递一个完整的结构体，那就意味着我们必须从调用函数拷贝这个结构体的全部内容到被调函数里面。在使用栈的系统中，这将会把整个结构体放进栈中。如果结构体过大，这将会遇到麻烦。但传递一个指针过去，便能最小化使用栈空间。

无论如何，既然是讨论指针，那我们就来谈谈怎样传递一个结构体指针，并在函数里面使用它。

看看刚才我们说的一种情况：想要一个函数，接受一个结构体指针作为参数，并在函数里面访问该结构体的成员。例如，在我们的示例结构体中打印员工姓名。

好了，既然知道我们的指针是要指向使用 `struct tag` 声明的结构体，我们可以这样声明一个结构体指针：

```
    struct tag *st_ptr;
```

然后把它指向我们的结构体：

```
    st_ptr = &my_struct;
```

现在我们可以通过解引用来访问既定的成员。但是，应该怎么样解逆引用一个结构体指针呢？好吧，假设我们要用这个指针去设置员工的年龄，可以这样写：

```
    (*st_ptr).age = 63;
```

仔细看看这个语句，也就是用括号里面的东西代替 `st_ptr` 指向的东西，即结构体 `my_struct`。因此，这其实是和 `my_struct.age` 一样的。

然而，这是一个十分常用的表达式，因此，C 语言的设计者创建了一个替代语法来做这件事：

```
    st_ptr->age = 63;
```

想想这个，看一下下面的程序：

```
    //------------ program 5.2 ---------------------
    /* Program 5.2 from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <string.h>
    struct tag{ /* the structure type */
        char lname[20];     /* last name */
        char fname[20];     /* first name */
        int age;            /* age */
        float rate;         /* e.g. 12.75 per hour */
    };
    struct tag my_struct;           /* define the structure */
    void show_name(struct tag *p);  /* function prototype */
    
    int main(void)
    {
        struct tag *st_ptr;     /* a pointer to a structure */
        st_ptr = &my_struct;    /* point the pointer to my_struct */
        strcpy(my_struct.lname,"Jensen");
        strcpy(my_struct.fname,"Ted");
        printf("\n%s ",my_struct.fname);
        printf("%s\n",my_struct.lname);
        my_struct.age = 63;
        show_name(st_ptr);      /* pass the pointer */
        return 0;
    }
    
    void show_name(struct tag *p)
    {
        printf("\n%s ", p->fname); /* p points to a structure */
        printf("%s ", p->lname);
        printf("%d\n", p->age);
    }
    -------------------- end of program 5.2 ----------------
```

再次声明，这里有很多东西需要一次理解，所以读者应该编译运行不同的代码片段，单步调试 main 函数并观察一下 `my_struct` 和 `p` 这些变量。并且要进入函数里面看看到底发生了什么。
