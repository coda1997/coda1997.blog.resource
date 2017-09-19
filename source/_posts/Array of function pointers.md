## Array of function pointers

### The function pointer:

- It is easy to understand a function pointer is a pointer that holds the address of the specified function.

- A function pointer can be transferred to another function as a parameter, return from a function or can be stored in an array of function pointers.

  #### Example

  Programmers want the OS execute their own handle function when the system handles a specified event. Usually, we can implement a handle function (CallBack function), giving system a pointer of the function for the specific handing.

  ```c
  typedef void (*event_handler)(unsigned int para1,unsigned int para2);
  struct event{
    unsigned int event_id;
    event_handler handler;// a function pointer
  };
  ```

### Array of function pointers

- `(*foo[])()` is an array holding function pointers, for easy access to each function which is pointed to by some function pointers.

- The array is used universally in event handling.

- For example, if we want to execute a different Call Back function when we sent a different signal, the array is used.

  ```c
  void foo1(){}
  void foo2(){}
  void foo3(){}

  void (*foo_ptr[3])() = {foo1,foo2,foo3};

  int
  main(){
    int option;
    scanf("%d",&option);
    if((option>=0)&&(option<3))
      (*foo_ptr[option])();
    return 0;
  }
  ```

  ​

###Conclusion

The array of function pointers is used so universally, like delegate in c# or java, event handling and so on, that it is significant to use it expertly and comprehendingly.

