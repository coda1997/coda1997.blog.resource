---
title: Array of function pointers
date: 2017-9-19
categories: c
---



### The function pointer:

- In C, we can have pointers to functions. The following is a simple example showing declaration and function call using the function pointer:

  ```c
  void fun(int a){
    printf("Value of a is %d\n",a);
  }

  int main(){
    //fun_ptr is a pointer to fun
    void (*fun_ptr)(int) = &fun;
    
    //Invoking fun() by fun_ptr
    int x = 1;
    (*fun_ptr)(x);
    
    /*
    **The function's name can be used to get function's address without &
    **the pointer's name can also be used to invoke the function without *
    */
    void (*fun_ptr2)(int) = fun ;//also works
    fun_ptr2(10);
    return 0;
    
  }
  ```

  Output:

  ```
  Value of a is 1
  Value of a is 10
  ```


- It is easy to understand a function pointer is a pointer that holds the **start address** of the specific function executable code.

- A function pointer can be passed to another function as a parameter,can be returned from a function or can be stored in an array of function pointers.

  ```c
  // A simple c program to show function pointers as a parameter and can be returned
  #include<stdio.h>

  void foo1(){printf("foo1\n");}
  void foo2(){printf("foo2\n");}

  void fun1(void (*foo)()){
      foo();
  }
  /*
  **fun2 is a function, void (*)() is the type of return value and void(*foo)() is a 
  **function pointer as a parameter.
  */
  void (*fun2(void(*foo)()))(){ return foo;}

  //In Unix, signal function is a tradition example
  void (*signal (int signo, void (*func)(int))) (int);
  ```
  #### Advantage:

- This pointer is very useful to avoid code redundancy. When we use a function like `qsort()` in C, what we need is a compare function and pass the pointer of it to `qsort()`.

  ```c
  #include<stdio.h>
  #include<stdlib.h>
  int compare(const void *a, const void *b){
      return (*(int*)a - *(int*)b );
  }

  int main(){
    int array[] = {11,2,3,5,15};
    int n = sizeof(array)/sizeof(array[0]);
    qsort(array[],n,sizeof(int),compare);
    return 0;
  }
  ```

- Programmers want the OS execute their own handle function when the system handles a specified event. Usually, we can implement a handle function (CallBack function), giving system a pointer of the function for the specific handing.

  ```c
  typedef void (*event_handler)(unsigned int para1,unsigned int para2);
  struct event{
    unsigned int event_id;
    event_handler handler;// a function pointer
  };
  ```
  #### Disadvantage:

  - When the function type is more complicated, the more difficulty we may face without using `#define` or `typedef`.

### Array of function pointers

- `(*foo[])()` is an array holding function pointers, for easy access to each function which is pointed to by some function pointers.

  ```c
  #include <stdio.h>
  void add(int a, int b){
      printf("Addition is %d\n", a+b);
  }
  void subtract(int a, int b){
      printf("Subtraction is %d\n", a-b);
  }
  void multiply(int a, int b){
      printf("Multiplication is %d\n", a*b);
  }
   
  int main(){
    
      // fun_ptr_arr is an array of function pointers
      void (*fun_ptr_arr[])(int, int) = {add, subtract, multiply};
      unsigned int ch, a = 15, b = 10;
      printf("Enter Choice: 0 for add, 1 for subtract and 2 "
              "for multiply\n");
      scanf("%d", &ch);
      if (ch > 2) return 0;
   
      (*fun_ptr_arr[ch])(a, b);
   
      return 0;
  } 
  ```

- Many C compiler will attempt to convert switch statement into a jump table, which also called branch table. Jump Table is an efficient means of handling similar events in software like event handling or delegate, which is a typical example of array of function pointers.

  ```c
  void (*pf[])(void) = {fun1,fun2,fun3,...,funn};

  void test(const int jump_index){
      if(jump_index<sizeof(pf)/sizeof(*pf))
        pf[jump_index]();
  }
  ```

  #### Advantage:

  - Easy for maintenance. If we want to change specific behavior of a event, which has its own default function, we can just modify customer function used for the event

    ```c
    //FUN is a pointer to the function with type void (*)()
    typedef void (*FUN)();

    enum EVENT_TYPE{
        EVENT_TYPE_1=0,
      	EVENT_TYPE_2,
      	EVENT_TYPE_3,
      	EVENT_TYPE_END
    };
    //According different events, we just modify the eFunMap for easy maintenance
    EVENT_FUN_MAP eFunMap[EVENT_TYPE_END] = {
        eventFun1,
        eventFun2,
        eventFun3
    };

    void doAction(EVENT_TYPE eventType)
    {
        (eFunMap[eventType].pFun)();
    }
    ```

  - Dynamic override. In Unix, when we handle signals, we often use arrays of function pointers. Also, in some event handling, we often use it to replace default function dynamic.

    ```c
    void doAction(EVENT_TYPE eventType)
    {
        (eFunMap[eventType].pFun)();
    }
     
    void reloadEventFun(EVENT_TYPE eventType)
    {
        eFunMap[eventType].pFun = &MyEventFun1;
      //*(eFunMap[eventType].pFun) = MyEventFun1; also works
    }
    ```


###Conclusion

The array of function pointers is used so universally, like delegate in c# or java, `sort()` in Python, event handling in C or C++ and nearly everywhere in Unix kernel, that it is significant to use it expertly and comprehendingly.

