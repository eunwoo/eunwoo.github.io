---
title: cpp explicit keyword
---

컴파일러는 gcc를 사용하였다.  
gcc ex_class.cpp -std=c++17 -lstdc++ -fno-elide-constructors  
  
-fno-elide-constructors는 컴파일러가 최적화를 통해 생성자 호출을 생략하지 않도록 지시한다.  
구체적으로는 return value optimization이 수행되지 않도록 한다. 아래 위키를 참고한다.
https://en.wikipedia.org/wiki/Copy_elision  
  
```C++
File: ex_class.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   SomeClass() { cout << "default" << endl; }
08:   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
09:   SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
10:   void setValue(double d) { mValue = d; }
11:   double getValue() const { return mValue; }
12: private:
13:   double mValue;
14: };
15: SomeClass foo(SomeClass me)
16: {
17:   cout << "foo" << endl;
18:   return me;
19: }
20: int main()
21: {
22:   SomeClass someClass(10.0);
23:   cout << "1" << endl;
24:   SomeClass cooClass = someClass;
25:   cout << "2" << endl;
26:   SomeClass newClass(someClass);
27:   cout << "3" << endl;
28:   SomeClass anotherClass = foo(someClass);
29: }
```
실행결과는 아래와 같다.  
```
init
1
copied
2
copied
3
copied
foo
copied
```
복사생성자는 8번째 줄과 같이 이미 생성된 다른 클래스를 참조하여 객체를 생성하는 생성자이다.  
24번째, 26번째 줄에서 한 번, 28번째 줄에서 2번 호출이 되었다.  
24번째 줄에서 복사생성자가 실행된 경우를 explicit(명시적)호출이라고 하고,  
28번째 줄에서는 복사생성자가 2번 실행되는데 foo함수의 인자인 me 객체는 foo함수의 스택에 메모리가 생성되고 someClass객체에서 me객체로 객체가 복사될 때 복사생성자가 한 번 실행되고, 리턴값으로 객체를 반환할 때 me객체를 anotherClass객체로 복사할 때 복사생성자가 호출되는데, implicit(암시적)호출이라고 한다.  
26번째 줄은 대입연산자가 아니고 복사생성자에 의해서 처리된다. 하지만 implicit으로 분류된다.  
explicit인지 implicit인지의 구분은 생성자의 인수가 주어졌을 때, 인수 바로 앞에 객체의 이름이 있는 경우에 explicit이라고 한다.(참고 1)

참고로 복사생성자의 인수에 참조연산자(&)가 없을 경우에는 에러가 발생한다.
```
ex_class.cpp:9:32: error: invalid constructor; you probably meant ‘SomeClass (const SomeClass&)’
    9 |   SomeClass(const SomeClass src) : mValue(src.mValue) { cout << "copied" << endl; }
      |
```

복제생성자 앞에 explicit을 붙이면 컴파일러에서 명시적호출만 허용하기 때문에 에러가 발생한다.  
```C++
File: ex_class1.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   SomeClass() { cout << "default" << endl; }
08:   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
09:   explicit SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
10:   void setValue(double d) { mValue = d; }
11:   double getValue() const { return mValue; }
12: private:
13:   double mValue;
14: };
15: SomeClass foo(SomeClass me)
16: {
17:   cout << "foo" << endl;
18:   return me;
19: }
20: int main()
21: {
22:   SomeClass someClass(10.0);
23:   cout << "1" << endl;
24:   SomeClass cooClass = someClass;
25:   cout << "2" << endl;
26:   SomeClass newClass(someClass);
27:   cout << "3" << endl;
28:   SomeClass anotherClass = foo(someClass);
29: }
```

에러메시지  
```
ex_class1.cpp: In function ‘SomeClass foo(SomeClass)’:
ex_class1.cpp:18:10: error: no matching function for call to ‘SomeClass::SomeClass(SomeClass&)’
   18 |   return me;
      |          ^~
ex_class1.cpp:8:3: note: candidate: ‘SomeClass::SomeClass(double)’
    8 |   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
      |   ^~~~~~~~~
ex_class1.cpp:8:20: note:   no known conversion for argument 1 from ‘SomeClass’ to ‘double’
    8 |   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
      |             ~~~~~~~^
ex_class1.cpp:7:3: note: candidate: ‘SomeClass::SomeClass()’
    7 |   SomeClass() { cout << "default" << endl; }
      |   ^~~~~~~~~
ex_class1.cpp:7:3: note:   candidate expects 0 arguments, 1 provided
ex_class1.cpp: In function ‘int main()’:
ex_class1.cpp:24:24: error: no matching function for call to ‘SomeClass::SomeClass(SomeClass&)’
   24 |   SomeClass cooClass = someClass;
      |                        ^~~~~~~~~
ex_class1.cpp:8:3: note: candidate: ‘SomeClass::SomeClass(double)’
    8 |   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
      |   ^~~~~~~~~
ex_class1.cpp:8:20: note:   no known conversion for argument 1 from ‘SomeClass’ to ‘double’
    8 |   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
      |             ~~~~~~~^
ex_class1.cpp:7:3: note: candidate: ‘SomeClass::SomeClass()’
    7 |   SomeClass() { cout << "default" << endl; }
      |   ^~~~~~~~~
ex_class1.cpp:7:3: note:   candidate expects 0 arguments, 1 provided
ex_class1.cpp:28:41: error: no matching function for call to ‘SomeClass::SomeClass(SomeClass&)’
   28 |   SomeClass anotherClass = foo(someClass);
      |                                         ^
ex_class1.cpp:8:3: note: candidate: ‘SomeClass::SomeClass(double)’
    8 |   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
      |   ^~~~~~~~~
ex_class1.cpp:8:20: note:   no known conversion for argument 1 from ‘SomeClass’ to ‘double’
    8 |   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
      |             ~~~~~~~^
ex_class1.cpp:7:3: note: candidate: ‘SomeClass::SomeClass()’
    7 |   SomeClass() { cout << "default" << endl; }
      |   ^~~~~~~~~
ex_class1.cpp:7:3: note:   candidate expects 0 arguments, 1 provided
ex_class1.cpp:15:25: note:   initializing argument 1 of ‘SomeClass foo(SomeClass)’
   15 | SomeClass foo(SomeClass me)
      |               ~~~~~~~~~~^~
```

아래와 같이 수정하면 에러를 해결할 수 있다.  
foo함수의 인자는 SomeClass이지만, 함수를 호출할 때는 double을 인자로 사용해도 에러가 발생하지 않는 이유는  
double 인자를 갖는 생성자가 호출되기 때문이다. 참조연산자(&)를 제거해야 에러가 발생하지 않는다.  
```C++
File: ex_class.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   SomeClass(double d) : mValue(d) {}
08:   // SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
09:   explicit SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
10:   void setValue(double d) { mValue = d; }
11:   double getValue() const { return mValue; }
12: private:
13:   double mValue;
14: };
15: // SomeClass& foo(SomeClass &me)
16: // {
17: //     cout << "foo" << endl;
18: //     return me;
19: // }
20: double foo(SomeClass me)
21: {
22:     cout << "foo" << endl;
23:     return me.getValue();
24: }
25: int main()
26: {
27:   SomeClass someClass(10.0);
28:   SomeClass newClass(someClass);
29:   // SomeClass anotherClass = foo(someClass);
30:   SomeClass anotherClass = foo(someClass.getValue());
31: }

```

생성자 앞에 다시 explicit을 사용하면, 다시 에러가 발생한다.
```C++
File: ex_class.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   // SomeClass(double d) : mValue(d) {}
08:   explicit SomeClass(double d) : mValue(d) {}
09:   // SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
10:   explicit SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
11:   void setValue(double d) { mValue = d; }
12:   double getValue() const { return mValue; }
13: private:
14:   double mValue;
15: };
16: // SomeClass& foo(SomeClass &me)
17: // {
18: //     cout << "foo" << endl;
19: //     return me;
20: // }
21: double foo(SomeClass me)
22: {
23:     cout << "foo" << endl;
24:     return me.getValue();
25: }
26: int main()
27: {
28:   SomeClass someClass(10.0);
29:   SomeClass newClass(someClass);
30:   // SomeClass anotherClass = foo(someClass);
31:   SomeClass anotherClass = foo(someClass.getValue());
32: }
```
에러메시지는 다음과 같다.
```
ex_class.cpp: In function ‘int main()’:
ex_class.cpp:31:50: error: could not convert ‘someClass.SomeClass::getValue()’ from ‘double’ to ‘SomeClass’
   31 |   SomeClass anotherClass = foo(someClass.getValue());
      |                                ~~~~~~~~~~~~~~~~~~^~
      |                                                  |
      |                                                  double
```

참고  
1. https://www.foonathan.net/2017/10/explicit-assignment/

