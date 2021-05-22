---
title: cpp explicit keyword
---

컴파일러는 gcc를 사용하였다.  
gcc ex_class.cpp -std=c++17 -lstdc++ -fno-elide-constructors  
  
-fno-elide-constructors는 컴파일러가 최적화를 통해 생성자 호출을 생략하지 않도록 지시한다.  
구체적으로는 return value optimization이 수행되지 않도록 한다. 아래 위키를 참고한다.
https://en.wikipedia.org/wiki/Copy_elision  
  
```
File: ex_class.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   SomeClass(double d) : mValue(d) { cout << "init" << endl; }
08:   SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
09:   void setValue(double d) { mValue = d; }
10:   double getValue() const { return mValue; }
11: private:
12:   double mValue;
13: };
14: SomeClass foo(SomeClass me)
15: {
16:   cout << "foo" << endl;
17:   return me;
18: }
19: int main()
20: {
21:   SomeClass someClass(10.0);
22:   cout << "1" << endl;
23:   SomeClass cooClass = someClass;
24:   cout << "2" << endl;
25:   SomeClass newClass(someClass);
26:   cout << "3" << endl;
27:   SomeClass anotherClass = foo(someClass);
28: }
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
23번째, 25번째 줄에서 한 번, 27번째 줄에서 2번 호출이 되었다.  
23번째 줄에서 복사생성자가 실행된 경우를 explicit(명시적)호출이라고 하고,  
24번째 줄에서는 복사생성자가 2번 실행되는데 foo함수의 인자인 me 객체는 foo함수의 스택에 메모리가 생성되고 someClass객체에서 me객체로 객체가 복사될 때 복사생성자가 한 번 실행되고, 리턴값으로 객체를 반환할 때 me객체를 anotherClass객체로 복사할 때 복사생성자가 호출되는데, implicit(암시적)호출이라고 한다.  
explicit인지 implicit인지의 구분은 생성자의 인수가 주어졌을 때, 인수 바로 앞에 객체의 이름이 있는 경우에 explicit이라고 한다.(참고 1)

참고로 복사생성자의 인수에 참조연산자(&)가 없을 경우에는 에러가 발생한다.
```
ex_class.cpp:8:32: error: invalid constructor; you probably meant ‘SomeClass (const SomeClass&)’
    8 |   SomeClass(const SomeClass src) : mValue(src.mValue) { cout << "copied" << endl; }
      |
```

복제생성자 앞에 explicit을 붙이면 컴파일러에서 명시적호출만 허용하기 때문에 에러가 발생한다.  
```
File: ex_class1.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   SomeClass(double d) : mValue(d) {}
08:   <span style="color:red">explicit</span> SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
09:   void setValue(double d) { mValue = d; }
10:   double getValue() const { return mValue; }
11: private:
12:   double mValue;
13: };
14: SomeClass foo(SomeClass me)
15: {
16:   cout << "foo" << endl;
17:   return me;
18: }
19: int main()
20: {
21:   SomeClass someClass(10.0);
22:   cout << "1" << endl;
23:   SomeClass cooClass = someClass;
24:   cout << "2" << endl;
25:   SomeClass newClass(someClass);
26:   cout << "3" << endl;
27:   SomeClass anotherClass = foo(someClass);
28: }

```

에러메시지  
```
ex_class1.cpp: In function ‘SomeClass foo(SomeClass)’:
ex_class1.cpp:17:10: error: cannot convert ‘SomeClass’ to ‘double’
   17 |   return me;
      |          ^~
ex_class1.cpp:7:20: note:   initializing argument 1 of ‘SomeClass::SomeClass(double)’
    7 |   SomeClass(double d) : mValue(d) {}
      |             ~~~~~~~^
ex_class1.cpp: In function ‘int main()’:
ex_class1.cpp:22:24: error: cannot convert ‘SomeClass’ to ‘double’
   22 |   SomeClass cooClass = someClass;
      |                        ^~~~~~~~~
      |                        |
      |                        SomeClass
ex_class1.cpp:7:20: note:   initializing argument 1 of ‘SomeClass::SomeClass(double)’
    7 |   SomeClass(double d) : mValue(d) {}
      |             ~~~~~~~^
ex_class1.cpp:24:32: error: cannot convert ‘SomeClass’ to ‘double’
   24 |   SomeClass anotherClass = foo(someClass);
      |                                ^~~~~~~~~
      |                                |
      |                                SomeClass
ex_class1.cpp:7:20: note:   initializing argument 1 of ‘SomeClass::SomeClass(double)’
    7 |   SomeClass(double d) : mValue(d) {}
      |             ~~~~~~~^
ex_class1.cpp:14:25: note:   initializing argument 1 of ‘SomeClass foo(SomeClass)’
   14 | SomeClass foo(SomeClass me)
      |               ~~~~~~~~~~^~
```

아래와 같이 수정하면 에러를 해결할 수 있다.  
foo함수의 인자는 SomeClass이지만, 함수를 호출할 때는 double을 인자로 사용해도 에러가 발생하지 않는 이유는  
double 인자를 갖는 생성자가 호출되기 때문이다. 참조연산자(&)를 제거해야 에러가 발생하지 않는다.  
```
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
```
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

