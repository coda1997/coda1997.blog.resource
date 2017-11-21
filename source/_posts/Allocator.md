---
title: Allocator
date: 2017-11-20
categories: c
---



### Build a simple malloc

In Linux, a program asks sbrk or brk for space, which increase the heap size and return to a pointer to the start of the new region on the heap. Also, when a program calls malloc(0), it just return NULL. Thus, we can build a simple malloc, to realize how heap works.

```c
void* malloc(size_t size){
  void *p = sbrk(0);
  void *request = sbrk(size);
  if(request==NULL){
    return NULL;
  }else{
    assert(p==request);
    return p;
  }
}
```

It is the easiest malloc function in Linux we build. But it can tell us how heap works. If we need to call a memory block dynamically, we call malloc to allocate some memory from the heap. So, there is a question, how does free work or how do we free it.

<!--more-->

### Mate-information

We can just return the allocated memory to the system, but it may be going into trouble if we need to call malloc to reserve space each time. Thus, we need to reserve the freed memory for reuse. A common method is to store meta-information about a memory region in some space that we can know which block we can reuse or which is allocated. We need a lightly more space to store some of meta-information like size, isFreed and so on. Also, need a linked list to reserve it. Here is the structure of meta-information containing a pointer to the payload.

```c
struct block_meta{
  size_t size;
  struct block_meta *next;
  int free;
}
```

But, there is a more significant thing we haven't thought before. What strategy we use to choose a probe memory block for reuse. The common ways are to return the first memory free block which is larger than or equal to what we want and to return the a memory block with the most suitable size. Each of them has advantages and shortcomings. But we can choose one of them, which is easy to implement, the former one.

Here is the code:

```c
struct block_meta *find_free_block(struct block_meta**last,size_t size){
  struct block_meta *current = global_base;
  while(current&&!(current->free&&current->size>=size)){
    *last = current;
    current = current->next;
  }
  return current;
}
```

If we can't find a free block(Maybe it's the first time to malloc memory space), we have to request space from the system.

```c
struct block_meta *request_space(struct block_meta* last, size_t size) {
  struct block_meta *block;
  block = sbrk(0);
  void *request = sbrk(size + META_SIZE);
  assert((void*)block == request); // Not thread safe.
  if (request == (void*) -1) {
  return NULL; // sbrk failed.
  }
  if (last) { // NULL on first request.
    last->next = block;
  }
  block->size = size;
  block->next = NULL;
  block->free = 0;
  return block;
}

```

With the preparation, we can implement our `malloc` and `free` without any extra functions.

```c
void *malloc(size_t t){
  struct block_meta* block;
  if(size<=0){
    return NULL;
  }
  if(!global_base){
    block=request_space(NULL,size);
  }else{
    struct block_meta* last = global_base;
    block = find_free_block(&last,size);
    if(!block){
      block=request_space(last,size);
      if(!block){
        return NULL;
      }
    }else{
      block->free=0;
    }
  }
  return block+1;
}

void free(void* ptr){
  if(!ptr){
    return;
  }
  struct block_meta *block = get_block_ptr(ptr);
  assert(block->free==0);
  block->free = 1;
}
```

 ### Realloc and calloc

If we want to add some useful functions to `malloc`, we just wrap it and add some functions like initializing memory block or resize a specified memory block. Thus, we build `realloc` to resize memory block and `calloc` to allocate a memory block with initialization.

#### Realloc

the prototype of realloc is:

```c
void* realloc(void* ptr,size_t size);
```

If the size is larger than the size of ptr, we malloc a new memory block with enough space, or just return itself if the size is less then the original.

```c
void *realloc(void* ptr,size_t size){
  if(!ptr){
    return malloc(size);
  }
  struct block_meta* block = get_block_ptr(ptr);
  if(block->size>=size){
    return ptr;
  }
  void* new_ptr;
  new_ptr = malloc(size);
  if(!new_ptr){
    return NULL;
  }
  memcpy(new_ptr,ptr,block->size);//only copy the payload.
  free(ptr);
  return new_ptr;
}
```

#### Calloc

We just initialize all the new memory with 0. It is easy to do but very important for programs.

```c
void *calloc(size_t size){
  void* ptr = malloc(size);
  if(ptr){
    memset(ptr,0,size);
    return ptr;
  }else{
    return NULL;
  }
}
```

### Merge and Split

The original `malloc` and `free` will ignore some "extra" space if the size is less than that of the memory block. It wastes a lot of space especially when we use a greedy or extravagant way to distribute memory blocks. Thus, we need a `split` function to short some space for other uses and a `merge` function to combine two continuous memory block to a larger one.

#### Merge

Because the continuous memory we request from the heap are stored in the linked list sequentially, we can merge two adjacent element in the list if they are free. Here are codes:

```c
void merge(struct block_meta *current){
  struct block_meta* front,*end;
  if(current==global_base){
    front==NULL;
  }else{
    front = global_base;
    while(front->next==current){
      front=front->next;
    }
    if(front->free==1&&current->free==1){
      front->size += META_SIZE+current->size;
      front->next = current->next;
      current=front;
    }
  }
  end=current.next;
  if(!end){
    return;
  }else{
    if(current->free==1&&end->free==1){
      current->size +=META_SIZE+end->size;
      current->next = end->next;
      end=current;
    }
  }
}
```

#### Split

We split a whole memory block into two continuous memory block in the situation where the whole block is quite larger than we need. Thus, we short the size of previous one and construct a new meta structure for the remains. Also, inert it into the list sequentially.

```c
void split(struct block_meta *current,size_t size){
  if(current->size<size+META_SIZE+1){
    return;
  }
  struct block_meta *new_ptr = (struct block_meta*)(&current+(META_SIZE+size)*8);
  memcpy(new_ptr,ptr,META_SIZE);
  new_ptr->size=current->size- META_SIZE-current->size -size;
  current->size=size;
  current->free=0;
  new_ptr->free=1;
  new_ptr->next=current->next;
  current->next=new_ptr;
}
```

Then, the last step is to modify our `malloc` and `free` functions to go them into effect.

```c
void *malloc(size_t t){
  struct block_meta* block;
  if(size<=0){
    return NULL;
  }
  if(!global_base){
    block=request_space(NULL,size);
  }else{
    struct block_meta* last = global_base;
    block = find_free_block(&last,size);
    split(block,size); // split one whole block
    if(!block){
      block=request_space(last,size);
      if(!block){
        return NULL;
      }
    }else{
      block->free=0;
    }
  }
  return block+1;

}

void free(void* ptr){
  if(!ptr){
    return;
  }
  struct block_meta *block = get_block_ptr(ptr);
  assert(block->free==0);
  block->free = 1;
  merge(block); merge it with adjacent blocks
}
```

