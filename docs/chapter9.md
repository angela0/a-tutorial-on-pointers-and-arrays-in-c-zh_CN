有些时候，在运行时用 `malloc()`、`calloc()` 或者其他分配内存的函数来分配内存是很方便的。使用这种方法不用事先就给出存储一个数组所需内存块的大小信息，直到运行的时候。或者，它可以允许你在一个时间点使用这部分内存去存储一个整型数组，而当你不再使用它的时候可以释放它给别的代码使用，例如再去存储一个结构体。

当内存被分配的时候，分配函数（如 `malloc()`, `calloc()` 等）会返回一个指针，它的类型取决于你是在用 K&R 编译器，还是新的 ANSI 类型编译器。在老的编译器中会返回 `char` 类型指针，而 ANSI 编译器则是 `void`。

如果你使用老的编译器，并要给整型数组分配空间，那你必须把 `char` 指针强制转换为 `int` 指针。如为 10 个整数分配空间：

```
    int *iptr;
    iptr = (int *)malloc(10 * sizeof(int));
    if (iptr == NULL)
    { .. ERROR ROUTINE GOES HERE .. }
```

而如果你用的是符合 ANSI 的编译器，`malloc()` 返回 `void` 指针。但 `void` 指针是可以赋值给任何类型的指针变量的，因此你不再需要像上面那样显式类型转换。数组的大小会在运行时计算，而在编译时是不需要的。所以上面的 10 可以是一个从数据文件或者键盘读到的变量，也可以在运行时根据需要计算得到。

由于数组符号和指针符号的等价，`iptr` 像上面那样被赋值之后，你是可以使用数组符号的。例如你可以这样写：

```
    int k;
    for (k = 0; k < 10; k++)
        iptr[k] = 2;
```

来把所有元素都设置为 2。

即使对指针和数组有了相当好的理解，一些初学者也可能一开始对多维数组的动态分配感到困惑。一般而言，我们想要通过数组符号访问这种数组的元素，而不是指针符号，不管怎样。根据应用的不同，我们在运行时可能知道也可能不知道数组的大小。这就让我们有不同的方式完成我们的任务。

就像我们看到的，在动态分配一个一维数组的时候，它的大小可以在运行时计算到。而动态分配多维数组时，编译时我们不再需要知道第一维。是否需要知道高维，这要看你写的代码了。这里我们将讨论一下动态分配二维整数数组的不同方法。

首先我们先考虑在编译时第二维已知的情况。

### 方法 1 { #Method1 }

处理这个问题的一种方法是使用 `typedef` 关键字。为了分配二维整数数组，你应该想起下面 2 种写法能生成相同的目标代码：

```
    multi[row][col] = 1;    *(*(multi + row) + col) = 1;
```

下面 2 中写法也能生成同样的代码：

```
    multi[row];             *(multi + row);
```

既然右边的那个会计算成指针，那么左边的数组符号也会计算成指针。事实上，`multi[0]` 会返回第一行第一个数的地址，`multi[1]` 得到第二行第一个数的地址，等等。实际上，`multi[n]` 计算整型数组的指针，即二维数组的第n行。也就是说，`multi` 可以看成数组的数组，`multi[n]` 则是这个数组的数组的第n个数组的指针。这里的**指针**这个词代表的是地址值（不是我们通常说的指针）。然而这在文献中十分常见，所以当你读到这样的语句时要注意区分它到底是数组的常量地址还是一个指针变量（指针变量本身就是一个数据对象）。

看看这个程序：

```
    --------------- Program 9.1 --------------------------------
    /* Program 9.1 from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <stdlib.h>
    #define COLS 5
    typedef int RowArray[COLS];
    RowArray *rptr;
    int main(void)
    {
        int nrows = 10;
        int row, col;
        rptr = malloc(nrows * COLS * sizeof(int));
        for (row = 0; row < nrows; row++)
        {
            for (col = 0; col < COLS; col++)
            {
                rptr[row][col] = 17;
            }
        }
        return 0;
    }
    ------------- End of Prog. 9.1 --------------------------------
```

这里假设用的 ANSI 类型的编译器，所以 `malloc()` 返回的 void 指针不必强制类型转换。如果你用的是 K&R 编译器，你需要这样写：

```
    rptr = (RowArray *)malloc(.... etc.
```

使用这种方法，`rptr` 就有了一个数组名的所有特征值（除了 `rptr` 是可以改变的），并且在下面的代码中全都使用数组符号。也就意味着，如果你要写一个函数去修改数组中的内容，你必须要在函数的形参中使用 COLS，就像我们在讨论传递一个二维数组给函数时做的那样。

### 方法 2 { #Method2 }

在上面的方法 1 中，`rptr` 是一个指向“有 COLS 个整数的一维数组类型”的指针。已经知道有一种语法可以不使用 `typedef` 而定义这种类型的指针。这样写：

```
    int (*xptr)[COLS];
```

变量 `xptr` 拥有所有和方法 1 中一样的特征值，并且不需要使用 `typedef`。这里的 `xptr` 指针，它指向一个整型数组，并且数组的大小通过 `#defined COLS` 定义。那个括号让指针符号有了高优先级，尽管原来数组符号比指针符号优先级高。但如果这样写：

```
    int *xptr[COLS];
```

那我们只是定义了 `xptr` 数组，这个数组存放指针，大小为通过 `#defined` 定义的 COLS。他们根本是不一样的。然而指针数组在动态分配二维数组也是有用的，我们将在下面 2 中方法中看到。

### 方法 3 { #Method3 }

考虑一下这种情况：我们在编译时并不知道每一行的元素个数，也就是行数和列数都是在运行时定义的。一种方法就是创建一个 `int` 类型的指针数组，然后给每一行分配空间，最后用指针数组指向每一行。看看代码:

```
    -------------- Program 9.2 ------------------------------------
    /* Program 9.2 from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <stdlib.h>
    int main(void)
    {
        int nrows = 5; /* Both nrows and ncols could be evaluated */
        int ncols = 10; /* or read in at run time */
        int row;
        int **rowptr;
        rowptr = malloc(nrows * sizeof(int *));
        if (rowptr == NULL)
        {
            puts("\nFailure to allocate room for row pointers.\n");
            exit(0);
        }
        printf("\n\n\nIndex Pointer(hex) Pointer(dec) Diff.(dec)");
        for (row = 0; row < nrows; row++)
        {
            rowptr[row] = malloc(ncols * sizeof(int));
            if (rowptr[row] == NULL)
            {
                printf("\nFailure to allocate for row[%d]\n",row);
                exit(0);
            }
            printf("\n%d %p %d", row, rowptr[row],rowptr[row]);
            if (row > 0)
                printf(" %d",(int)(rowptr[row] - rowptr[row-1]));
        }
        return 0;
    }
    --------------- End 9.2 ------------------------------------
```

在上面的代码中，`rowptr` 是一个指针的指针。这个例子中，它指向 `int` 指针数组的第一个元素，想想调用了多少次 `malloc()`：
    
```
    To get the array of pointers      1       call
    To get space for the rows         5       calls
                                    -----
                    Total             6       calls
```

如果你选择这种方法的话，需要注意，尽管你可以使用数组符号来访问数组中的每一个元素，如 `rowptr[row][col] = 17;`，但它并不意味着二维数组中的元素在内存中是连续的。

但是你却可以使用数组符号就像它们是连续的一样。例如，你可以这样写时：

```
    rowptr[row][col] = 176;
```

`rowptr` 就像是编译时创建的二维数组一样。当然 `row` 和 `col` 必须在你创建的数组范围内，这也和编译时创建的数组一样。

如果你想用连续的内存块来存储数组的元素，你可以像下面那样做。

### 方法 4 { #Method4 }

这个方法中，我们先分配一块内存来存放整个数组。然后再创建指针数组指向每一行。因此，尽管我们使用了指针数组，实际内存中的数组还是连续的。代码像是这样：

```
    ----------------- Program 9.3 -----------------------------------
    /* Program 9.3 from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <stdlib.h>
    int main(void)
    {
        int **rptr;
        int *aptr;
        int *testptr;
        int k;
        int nrows = 5; /* Both nrows and ncols could be evaluated */
        int ncols = 8; /* or read in at run time */
        int row, col;
        
        /* we now allocate the memory for the array */
        aptr = malloc(nrows * ncols * sizeof(int));
        if (aptr == NULL)
        {
            puts("\nFailure to allocate room for the array");
            exit(0);
        }
        /* next we allocate room for the pointers to the rows */
        rptr = malloc(nrows * sizeof(int *));
        if (rptr == NULL)
        {
            puts("\nFailure to allocate room for pointers");
            exit(0);
        }
        /* and now we 'point' the pointers */
        for (k = 0; k < nrows; k++)
        {
            rptr[k] = aptr + (k * ncols);
        }
        /* Now we illustrate how the row pointers are incremented */
        printf("\n\nIllustrating how row pointers are incremented");
        printf("\n\nIndex Pointer(hex) Diff.(dec)");
        for (row = 0; row < nrows; row++)
        {
            printf("\n%d %p", row, rptr[row]);
            if (row > 0)
                printf(" %d",(rptr[row] - rptr[row-1]));
        }
        printf("\n\nAnd now we print out the array\n");
        for (row = 0; row < nrows; row++)
        {
            for (col = 0; col < ncols; col++)
            {
                rptr[row][col] = row + col;
                printf("%d ", rptr[row][col]);
            }
            putchar('\n');
        }
        puts("\n");
        /* and here we illustrate that we are, in fact, dealing with
        a 2 dimensional array in a contiguous block of memory. */
        printf("And now we demonstrate that they are contiguous in memory\n");
        testptr = aptr;
        for (row = 0; row < nrows; row++)
        {
            for (col = 0; col < ncols; col++)
            {
                printf("%d ", *(testptr++));
            }
            putchar('\n');
        }
        return 0;
    }
    ------------- End Program 9.3 -----------------
```

在看看我们调用了几次 `malloc()`：

```
    To get room for the array itself        1   call
    To get room for the array of ptrs       1   call
                                          ----
                        Total               2   calls
```

现在，每次 `malloc()` 调用都会花费额外的空间开销，因为 `malloc()` 通常都是被操作系统执行，生成一个包含与块大小有关的数据的链表。但更重要的是，对于很大的数组（数百行）记录什么时间需要释放什么是一件很麻烦的事。另外，联合在一起的连续数据块可以使用 `memset()` 全部初始化为 0，这样看起来，这种方法更好。

作为多维数组的最后一个示例，我们将展示动态分配一个三维数组。它还会给你展示的是观察这种分配过程。由于这些原因，我们会使用上面说到的第二种方法（分配整块内存）。看看下面的代码：

```
    ------------------- Program 9.4 -------------------
    /* Program 9.4 from PTRTUT10.HTM 6/13/97 */
    #include <stdio.h>
    #include <stdlib.h>
    #include <stddef.h>
    int X_DIM=16;
    int Y_DIM=5;
    int Z_DIM=3;
    int main(void)
    {
        char *space;
        char ***Arr3D;
        int y, z;
        ptrdiff_t diff;
        /* first we set aside space for the array itself */
        space = malloc(X_DIM * Y_DIM * Z_DIM * sizeof(char));
        
        /* next we allocate space of an array of pointers, each
        to eventually point to the first element of a
        2 dimensional array of pointers to pointers */
        Arr3D = malloc(Z_DIM * sizeof(char **));
        
        /* and for each of these we assign a pointer to a newly
        allocated array of pointers to a row */
        for (z = 0; z < Z_DIM; z++)
        {
            Arr3D[z] = malloc(Y_DIM * sizeof(char *));
            /* and for each space in this array we put a pointer to
            the first element of each row in the array space
            originally allocated */
            for (y = 0; y < Y_DIM; y++)
            {
                Arr3D[z][y] = space + (z*(X_DIM * Y_DIM) + y*X_DIM);
            }
        }
        /* And, now we check each address in our 3D array to see if
        the indexing of the Arr3d pointer leads through in a
        continuous manner */
        for (z = 0; z < Z_DIM; z++)
        {
            printf("Location of array %d is %p\n", z, *Arr3D[z]);
            for ( y = 0; y < Y_DIM; y++)
            {
                printf(" Array %d and Row %d starts at %p", z, y, Arr3D[z][y]);
                diff = Arr3D[z][y] - space;
                printf(" diff = %d ",diff);
                printf(" z = %d y = %d\n", z, y);
            }
        }
        return 0;
    }
    ------------------- End of Prog. 9.4 -------------------
```

如果你跟随教程到这个地方，那在注释的基础上独自理解上面这段代码应该不是问题。然而还是有几个点需要指出。让我们从这行代码开始：

```
    Arr3D[z][y] = space + (z*(X_DIM * Y_DIM) + y*X_DIM);
```

注意到这里的 `space` 是一个字符指针，和 `Arr3D[z][y]` 的类型是一样的。要明白当给指针加上一个整数,就如计算 `(z*(X_DIM * Y_DIM) + y*X_DIM)` 得到的整数一样，结果是一个新的指针值，这很重要。并且给指针变量赋一个指针值的时候，类型一定要匹配。
