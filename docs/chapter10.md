到这里，我们已经讨论过指向数据对象的指针。C 里面也允许声明指针函数的指针。函数指针有很多用途，这里讨论一部分。

考虑一下这个实际问题。你想写一个函数，它能够实际地排序任何数据集合，这些数据存储在数组中。可能会是字符串数组、整型数组、浮点数数组，甚至是结构体数组。排序算法对所有的类型都是一样的。例如，可以是简单的冒泡排序，或者是更复杂的希尔排序、快速排序等。为了演示目的，我们就用一个简单的冒泡排序。

Sedgewick[^1]曾经用 C 通过构建函数来实现冒泡排序，传给函数一个数组的指针，就能被排序。如果我们把这个函数叫做 `bubble()`，排序程序就像下面 `bubble_1.c` 描述的那样：

```
    /*-------------------- bubble_1.c --------------------*/
    /* Program bubble_1.c from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    int arr[10] = { 3,6,1,2,3,8,4,1,7,2};
    void bubble(int a[], int N);
    int main(void)
    {
        int i;
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        bubble(arr,10);
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        return 0;
    }
    void bubble(int a[], int N)
    {
        int i, j, t;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                if (a[j-1] > a[j])
                {
                    t = a[j-1];
                    a[j-1] = a[j];
                    a[j] = t;
                }
            }
        }
    }
    /*---------------------- end bubble_1.c -----------------------*/
```

冒泡排序是一种比较简单的排序。这个算法从第二个扫描到最后一个元素，并把每一个元素都和它前面的元素比较。如果前面的那个比当前的大，就交换它们，使得最大的元素离尾部越来越近。在第一趟排序中，这会让最大的元素移到数组尾部。现在数组仅限于除了最后一个元素的所有元素，重复上面的过程，这会把第二大的元素放到最大的元素之前。这个过程一直重复元素个数减 1 次。最后得到一个排序后的数组。

这里我们的函数用来排序整型数组。因此在循环中的第 1 行我们比较整数大小，第 2 到 4 行用了一个临时整数。现在，我们想要的是看看能不能把这个函数改造成用于任何数据类型，而不只限制在整型。

同时，我们并不想每次使用这个都分析算法与之相关的代码。因此我们着手把比较移出函数 `bubble()`，以便于我们修改比较功能，而不用重写与实际算法相关的部分。这就有了 `bubble_2.c`：

```
    /*---------------------- bubble_2.c -------------------------*/
    /* Program bubble_2.c from PTRTUT10.HTM 6/13/97 */
    /* Separating the comparison function */
    #include <stdio.h>
    int arr[10] = { 3,6,1,2,3,8,4,1,7,2};
    void bubble(int a[], int N);
    int compare(int m, int n);
    int main(void)
    {
        int i;
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        bubble(arr,10);
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        return 0;
    }
    void bubble(int a[], int N)
    {
        int i, j, t;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                if (compare(a[j-1], a[j]))
                {
                    t = a[j-1];
                    a[j-1] = a[j];
                    a[j] = t;
                }
            }
        }
    }
    int compare(int m, int n)
    {
        return (m > n);
    }
    /*--------------------- end of bubble_2.c -----------------------*/
```

如果我们的目标是让排序程序的数据类型独立，一种方法就是使用 void 指针指向这个数据，而不是用整型。为了朝这个方向迈出第一步，让我们修改一些内容使得我们可以使用指针。一开始，我们先使用整型指针：

```
    /*----------------------- bubble_3.c -------------------------*/
    /* Program bubble_3.c from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    int arr[10] = { 3,6,1,2,3,8,4,1,7,2};
    void bubble(int *p, int N);
    int compare(int *m, int *n);
    int main(void)
    {
        int i;
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        bubble(arr,10);
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        return 0;
    }
    void bubble(int *p, int N)
    {
        int i, j, t;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                if (compare(&p[j-1], &p[j]))
                {
                    t = p[j-1];
                    p[j-1] = p[j];
                    p[j] = t;
                }
            }
        }
    }
    int compare(int *m, int *n)
    {
        return (*m > *n);
    }
    /*------------------ end of bubble3.c -------------------------*/
```

请注意变化。我们现在传递了一个整型指针（或者叫整型数组）给 `bubble()`。并且在 `bubble()` 中，我们给比较函数传递了指向数组元素的指针。最后，当然我们在 `compare()` 中解引用了这些指针，使得我们可以做比较。下一步就是要把 `bubble()` 传给 `compare()` 的指针变成 void 指针，使得这个函数变得类型无关。这在 `bubble_4` 中体现：

```
    /*------------------ bubble_4.c ----------------------------*/
    /* Program bubble_4.c from PTRTUT10,HTM 6/13/97 */
    #include <stdio.h>
    int arr[10] = { 3,6,1,2,3,8,4,1,7,2};
    void bubble(int *p, int N);
    int compare(void *m, void *n);
    int main(void)
    {
        int i;
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        bubble(arr,10);
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        return 0;
    }
    void bubble(int *p, int N)
    {
        int i, j, t;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                if (compare((void *)&p[j-1], (void *)&p[j]))
                {
                    t = p[j-1];
                    p[j-1] = p[j];
                    p[j] = t;
                }
            }
        }
    }
    int compare(void *m, void *n)
    {
        int *m1, *n1;
        m1 = (int *)m;
        n1 = (int *)n;
        return (*m1 > *n1);
    }
    /*------------------ end of bubble_4.c ---------------------*/
```

现在要注意，这样做的时候我们必须把 `compare()` 中的 void 指针转换成实际排序的类型。但如你看到的，这样是可以的。既然我们传给 `bubble()` 的仍然是整型数组的指针。那么当我们把它们作为参数传给 `compare()` 的时候，就必须强制转换为 void 指针。

现在我们把问题定位到该传给 `bubble()` 什么。我们想把 `bubble()` 的第一个参数也是 void 指针。但这意味着，我们得改变变量 `t`，因为它现在是整型。另外，在使用 `t = p[j-1];` 的地方我们需要知道 `p[j-1]` 的类型，以便于我们知道要拷贝给变量 `t` （或者是代替 `t` 的不管什么东西）多少字节。

现在在 `bubble_4.c` 中的 `bubble()`，一些信息，如被存储的数据的类型（即每个元素的大小）可以通过第一个参数是一个整型指针获取到。如果我们想让 `bubble()` 排序任何类型的数据，我们得把这个指针都转换为 void 指针。但是这样做，我们就会把与之相关的数组元素的大小这些信息丢失掉。因此，我们在 `bubble_5.c` 中加入额外的参数来处理这一信息。

相比前面的改进，从 `bubble4.c` 到 `bubble5.c` 的改变可能比较多，因此仔细比较一下这 2 个模块的差异。

```
    /*---------------------- bubble5.c ---------------------------*/
    /* Program bubble_5.c from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <string.h>
    long arr[10] = { 3,6,1,2,3,8,4,1,7,2};
    void bubble(void *p, size_t width, int N);
    int compare(void *m, void *n);
    int main(void)
    {
        int i;
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%d ", arr[i]);
        }
        bubble(arr, sizeof(long), 10);
        putchar('\n');
        for (i = 0; i < 10; i++)
        {
            printf("%ld ", arr[i]);
        }
        return 0;
    }
    void bubble(void *p, size_t width, int N)
    {
        int i, j;
        unsigned char buf[4];
        unsigned char *bp = p;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                if (compare((void *)(bp + width*(j-1)), (void *)(bp + j*width))) /* 1 */
                {
                    /* t = p[j-1]; */
                    memcpy(buf, bp + width*(j-1), width);
                    /* p[j-1] = p[j]; */
                    memcpy(bp + width*(j-1), bp + j*width , width);
                    /* p[j] = t; */
                    memcpy(bp + j*width, buf, width);
                }
            }
        }
    }
    int compare(void *m, void *n)
    {
        long *m1, *n1;
        m1 = (long *)m;
        n1 = (long *)n;
        return (*m1 > *n1);
    }
    /*--------------------- end of bubble5.c ---------------------*/
```

注意到现在我们把数组的类型从 `int` 换成了 `long`，来展示 `compare()` 需要做出的改变。但在 `bubble()` 中我们去掉了变量 `t`（本应该是从 `int` 换成 `long` 的）。我加上了一下 4 个无符号字符长的 buffer，它可以存放一个 long（这会在在后面修改）。无符号字符指针 `*bp` 用于指向要排序的数组的基址，就是数组的第一个元素。

我们也必须修改传给 `compare()` 的参数以及交换 2 个元素的方式。使用 `memcpy()` 和指针符号而不是数组符号能够降低类型敏感。

再强调一遍，细心比较 `bubble5.c` 和 `bubble4.c` 能让你更加理解到底发生了什么以及为什么。

我们现在看看 `bubble6.c`，它用和 `bubble5.c` 一样的 `bubble()` 函数来排序字符串数组。因为字符串的比较和 long int 的比较不一样，所以我们必须修改比较函数。另外，我删除了 `bubble5.c` 里面的注释。

```
    /*--------------------- bubble6.c ---------------------*/
    /* Program bubble_6.c from PTRTUT10.HTM 6/13/97 */
    
    #include <stdio.h>
    #include <string.h>
    #define MAX_BUF 256
    char arr2[5][20] = { "Mickey Mouse",
                         "Donald Duck",
                         "Minnie Mouse",
                         "Goofy",
                         "Ted Jensen" };
    void bubble(void *p, int width, int N);
    int compare(void *m, void *n);
    
    int main(void)
    {
        int i;
        putchar('\n');
        for (i = 0; i < 5; i++)
        {
            printf("%s\n", arr2[i]);
        }
        bubble(arr2, 20, 5);
        putchar('\n\n');
        for (i = 0; i < 5; i++)
        {
            printf("%s\n", arr2[i]);
        }
        return 0;
    }
    void bubble(void *p, int width, int N)
    {
        int i, j, k;
        unsigned char buf[MAX_BUF];
        unsigned char *bp = p;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                k = compare((void *)(bp + width*(j-1)), (void *)(bp +
                j*width));
                if (k > 0)
                {
                    memcpy(buf, bp + width*(j-1), width);
                    memcpy(bp + width*(j-1), bp + j*width , width);
                    memcpy(bp + j*width, buf, width);
                }
            }
        }
    }
    int compare(void *m, void *n)
    {
        char *m1 = m;
        char *n1 = n;
        return (strcmp(m1,n1));
    }
    /*------------------- end of bubble6.c ---------------------*/
```

`bubble()` 函数没有改变说明它能够排序不同类型的数据。剩下要做的就是传给 `bubble()` 我们要使用的比较函数的名字，以使得 `bubble()` 有真正的通用性。就像数组的名字是数组第一个元素在数据段的地址，函数名也是这个函数在代码段的地址。因此我们需要用一个函数指针，这个例子中是比较函数。函数指针必须要和所指函数的参数数量、参数类型以及返回值类型匹配。这里，我们这样声明函数指针：

```
    int (*fptr)(const void *p1, const void *p2);
```

注意，如果我们这样写：

```
    int *fptr(const void *p1, const void *p2);
```

只是声明了一个函数原型，它返回一个整型指针。这是因为在 C 里面 `()` 比 `*`（指针符号）的优先级高。通过把 `()` 放在字符串 `*fptr`两侧，表明我们声明了一个函数指针。

现在我们修改 `bubble()` 的声明，给它加上第四个参数--合适类型的函数指针。就变成了这样：

```
    void bubble(void *p, int width, int N,
                int(*fptr)(const void *, const void *));
```

当我们调用 `bubble()` 的时候，我们加入要用的比较函数名，`bubble7.c` 展示了怎么用相同的 `bubble()` 函数来排序不同的数据类型。

```
    /*------------------- bubble7.c ------------------*/
    /* Program bubble_7.c from PTRTUT10.HTM 6/10/97 */
    #include <stdio.h>
    #include <string.h>
    #define MAX_BUF 256
    long arr[10] = { 3,6,1,2,3,8,4,1,7,2};
    char arr2[5][20] = { "Mickey Mouse",
                         "Donald Duck",
                         "Minnie Mouse",
                         "Goofy",
                         "Ted Jensen" };
    void bubble(void *p, int width, int N,
    int(*fptr)(const void *, const void *));
    int compare_string(const void *m, const void *n);
    int compare_long(const void *m, const void *n);
    
    int main(void)
    {
        int i;
        puts("\nBefore Sorting:\n");
        for (i = 0; i < 10; i++) /* show the long ints */
        {
            printf("%ld ",arr[i]);
        }
        puts("\n");
        for (i = 0; i < 5; i++) /* show the strings */
        {
            printf("%s\n", arr2[i]);
        }
        bubble(arr, 4, 10, compare_long); /* sort the longs */
        bubble(arr2, 20, 5, compare_string); /* sort the strings */
        puts("\n\nAfter Sorting:\n");
        for (i = 0; i < 10; i++) /* show the sorted longs */
        {
            printf("%d ",arr[i]);
        }
        puts("\n");
        for (i = 0; i < 5; i++) /* show the sorted strings */
        {
            printf("%s\n", arr2[i]);
        }
        return 0;
    }
    void bubble(void *p, int width, int N,
    int(*fptr)(const void *, const void *))
    {
        int i, j, k;
        unsigned char buf[MAX_BUF];
        unsigned char *bp = p;
        for (i = N-1; i >= 0; i--)
        {
            for (j = 1; j <= i; j++)
            {
                k = fptr((void *)(bp + width*(j-1)), (void *)(bp +
                j*width));
                if (k > 0)
                {
                    memcpy(buf, bp + width*(j-1), width);
                    memcpy(bp + width*(j-1), bp + j*width , width);
                    memcpy(bp + j*width, buf, width);
                }
            }
        }
    }
    int compare_string(const void *m, const void *n)
    {
        char *m1 = (char *)m;
        char *n1 = (char *)n;
        return (strcmp(m1,n1));
    }
    int compare_long(const void *m, const void *n)
    {
        long *m1, *n1;
        m1 = (long *)m;
        n1 = (long *)n;
        return (*m1 > *n1);
    }
    /*----------------- end of bubble7.c -----------------*/
```

[^1]: "Algorithms in C"

    Robert Sedgewick

    Addison-Wesley

    ISBN 0-201-51425-7
