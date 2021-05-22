---
title: cpp explicit keyword
---

컴파일러는 gcc를 사용하였다.  
gcc ex_class.cpp -std=c++17 -lstdc++ -fno-elide-constructors  

```
File: ex_class.cpp
01: #include <iostream>
02: using namespace std;
03: 
04: class SomeClass
05: {
06: public:
07:   SomeClass(double d) : mValue(d) {}
08:   SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
09:   void setValue(double d) { mValue = d; }
10:   double getValue() const { return mValue; }
11: private:
12:   double mValue;
13: };
14: SomeClass& foo(SomeClass &me)
15: {
16:     cout << "foo" << endl;
17:     return me;
18: }
19: int main()
20: {
21:   SomeClass someClass(10.0);
22:   SomeClass newClass(someClass);
23:   SomeClass anotherClass = foo(someClass);
24: }

```
에러 없이 컴파일 됨.  
실행결과  
```
copied
foo
copied
```
복사생성자는 8번째 줄과 같이 이미 생성된 다른 클래스를 참조하여 객체를 생성하는 생성자임. 인자로 const reference 타입을 가진다.  
22번째 줄은 객체를 생성할 때 이전에 생성된 객체를 복사하여 생성하는 코드이며 복사생성자가 호출된다.  
실행결과 3째줄을 보면 foo함수에서 객체를 리턴할 때 복사생성자가 호출됨을 알 수 있다. 22번째 줄에서 복사생성자가 실행된 경우를 explicit(명시적)호출이라고 하고,  
17번째 줄에서 함수의 리턴값으로 객체를 반환할 때 임시객체(이름없는 객체)가 생성될 때 복사생성자가 호출되는데, implicit(암시적)호출이라고 한다.  

복제생성자 앞에 explicit을 붙이면 컴파일러에서 명시적호출만 허용하기 때문에 에러가 발생한다.  
```
#include <iostream>
using namespace std;

class SomeClass
{
public:
  SomeClass(double d) : mValue(d) {}
  explicit SomeClass(const SomeClass& src) : mValue(src.mValue) { cout << "copied" << endl; }
  void setValue(double d) { mValue = d; }
  double getValue() const { return mValue; }
private:
  double mValue;
};
SomeClass& foo(SomeClass &me)
{
    cout << "foo" << endl;
    return me;
}
int main()
{
  SomeClass someClass(10.0);
  SomeClass anotherClass = foo(someClass);
}
```

에러메시지  
```
ex_class.cpp: In function ‘int main()’:
ex_class.cpp:21:31: error: cannot convert ‘SomeClass’ to ‘double’
   21 |   SomeClass anotherClass = foo(someClass);
      |                            ~~~^~~~~~~~~~~
      |                               |
      |                               SomeClass
ex_class.cpp:7:20: note:   initializing argument 1 of ‘SomeClass::SomeClass(double)’
    7 |   SomeClass(double d) : mValue(d) {}
      |             ~~~~~~~^
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

