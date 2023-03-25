---
title: "RAII-Thread"
date: 2023-03-24T18:57:38-05:00
repo: "https://github.com/matthew-hennefarth/RAII-Thread"
languages: ["C++"]
---

Header only file for a C++ RAII thread that joins upon destruction. 
It wraps the interface of std::thread.
Feel free to download or copy the file into your own project.

## Example
To create a RAII Thread with a lambda, we can do the following:
```cpp
#include "RAIIThread"
#include <iostream>
int main()
{
  {
    RAIIThread thread([](){
      std::cout << "Hello from RAII Thread" << '\n';
    });
  }
  /* Thread is auto joined upon leaving inner scope */
  return 0;
}
```

Similarly for functions:
```cpp
#include "RAIIThread"
#include <iostream>
void threadUnsafeFunction( int i )
{
  std::cout << "This function is not thread safe because of lack of mutex" << '\n';
  std::cout << "The number is " << i << '\n';
}
int main()
{
  {
    RAIIThread thread(threadUnsafeFunction, 5);
  }
  /* Thread is auto joined upon leaving inner scope */
  return 0;
}
```