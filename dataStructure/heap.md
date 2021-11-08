### **大顶堆**

##### 大顶堆的创建

```c
/* 使用数组创建堆 */

typedef struct HeapStruct *MaxHeap;
struct HeapStruct {
    ElementType *Elements;	/* 存储堆元素的数组 */
    int size;				/* 堆中当前元素的个数 */
    int capacity;			/* 堆的最大容量 */
};

MaxHeap create(int max_size)
{
	MaxHead H = malloc(sizeof(struct HeapStruct));
    H->Elements = malloc((max_size + 1) * sizeof(ElementType));
    H->size = 0;
    H->capacity = max_size;
    H->Elements[0] = MaxData;
    
    return H;
}

void insert(MaxHeap H, ElementType item)
{
    int i;
    if (IsFull(H)) {
        printf("堆已满~\n");
        return;
    }

    i = ++H->size;
    for (; H->Elements[i/2] < item; i/=2) {
        H->Elements[i] = H->Elements[i/2];
    }
    H->Elements[i] = item;
}

ElementType DeleteMax(MaxHeap H)
{
    int parent, child;
    ElementType MaxItem, temp;
    if (IsEmpty(H)) {
        printf("最大堆为空~\n");
        return;
    }
    MaxItem = H->Elements[1];
    temp = H->Elements[H->size--];
    for (parent=1; parent*2 <= H->size; parent=child) {
        child = parent * 2;
        if ((child != H->size) && (H->Elements[child] < H->Elements[child+1])) 
            child++;
        
        if (temp >= H->Elements[child]) break;
        else 
            H->Elements[parent] = H->Elements[child];
    }
    
    H->Elements[parent] = temp;
    
    return MaxItem;
}

/* 堆的调整、构建；对于一个有序数组，对齐进行调整 */
```

### **小顶堆**