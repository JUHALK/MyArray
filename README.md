# MyArray

## struct Address{ int level; void* next;};
>
레벨 0인 Address는 레벨 1짜리의 Address 배열의 주소이다. arr[2][3][4] 가있다면 2는 레벨 1, 3은 레벨2, 레벨2의 next 에서는 int 4개짜리 배열을 가르키고 있어야한다. 즉 n차원 배열일때 n-1레벨의 Address의 next 에 int 배열을 가지고있어야 한다.

## void InitializeAddress(Address* curr) 
>
```
void InitializeAddress(Address* curr)
  {
    if(curr->level == dim - 1) //재귀함수 종료 조건
    {
      curr->next = new int[size[dim - 1]];
      return;
    }
    curr->next = new Address[size[curr->level]];  //재귀함수 타고 내려갈때
    for(int i = 0; i < size[curr->level]; i++)    //재귀함수 호출 부분
    {
      static_cast<Address*>(curr->next)[i].level = curr->level + 1;
      InitializeAddress(static_cast<Address*>(curr->next) + i);
    }
  }
```

## void FreeAddress(Address* curr)
>
```
void FreeAddress(Address* curr)
{
  if(curr->level == dim - 1)                     //재귀함수 종료조건
  {
      delete[] static_cast<int*>(curr->next);
      return;
  }
  for(int i = 0; i < size[curr->level]; i++)               //재귀함수 호출부분
  {
      static_cast<Address*>(curr->next)[i].level = curr->level + 1;
      FreeAddress(static_cast<Address*>(curr->next) + i);
  }
   delete[] static_cast<Address*>(curr);           //재귀함수호출 이후에 있으므로 올라갈때
}
  ```
  어떤 코드가 재귀함수 호출 부분 기준으로 위에있냐 아래있느냐에 따라서 올라갈때 실행되는지 내려갈때 실행되는지 바뀐다.

## 재귀함수 특징
>
1. 재귀함수 종료조건 + 명령문
2. 재귀함수 호출부분
3. 명령문
>
재귀함수 종료조건은 대부분 함수 처음에 등장한다. 재귀함수 호출부분을 다음에 어떤 명령문이 쓰여있다면 그 명령문은 바닥을 찍고 올라가는 도중에 실행된다. 재귀함수 호출부분 전에 어떤 명령문이 쓰여 있다면 그명령문은 바닥을 향해 내려가면서 실행된다. 예를들어 
```
int ary[9] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
void PrintArray(int index)
{
  if(index == 8){cout << ary[8]; return;}
  PrintArray(index+1);
  cout << ary[index];
} 
```
라면 987654321이 출력되고
```
int ary[9] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
void PrintArray(int index)
{
  if(index == 8){cout << ary[8]; return;}
  cout << ary[index];
  PrintArray(index+1);
} 
```
라면 123456789 가 출력된다.
```
int ary[9] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
void PrintArray(int index)
{
  if(index == 8){cout << ary[8]; return;}
  PrintArray(index+1);
  cout << ary[index];
  PrintArray(index+1);
} 
```
라면 12345678987654321 이출력된다. 헷갈리면 재귀함수 호출부분에 재귀함수 종료조건의 명령문을 넣어보자.

## class Int{};
>
```
class Int {
  void* data;
  int level;
  Array* array;
};
```
Array의 [] 연산자만 가지고는 제한이 많다 왜냐하면 다중배열의 경우 반환값을 그때마다 다르게 해야하기때문이다. Int 클래스를통해서 현재 어떤배열의 몇번째 배열인지 저장을 하고 최종적으로 int 값을 리턴할수있게 도와준다. 즉, 여러번의 []연산끝에는 data에 int 배열을 가르키는 주소가 남아있게되고 operator int(), Int& operator=(const int& a) 를통해 언제든지 int 형으로 읽히고 쓸준비가 된다.

## Array의 []연산자, Int의 []연산자
>
```
Int operator[](const int& n)
  {
    Int num(0, top); //스택에 저장된Int
    return Int(n);   //스택에 저장된 Int를 그대로 받고 임시객체를 생성하고 삭제함 그이후에는 하나의 같은 임시객체로 처리된다.
  }
```
무조건 Int 객체를 생성해야하니 반환형 Int으로한다. 그리고 임시객체의 []연산자가 아래에 수행된다.
```
  Int& operator[](const int& n) //const int& n을통해 불필요한 연산막기
  {
    if(level = MyArray->dim - 1)
      data = static_cast<int*>(data->next) + n;
    else
      data = static_cast<Address*>(data->next) + n;
    level++;
    return *this;
  }
```
여기서는 임시객체를 받고 또다른임시객체를 생성할필요없이 계속 같은 임시객체를 반환한다.
간편하게 Int의 생성자에 data를 받을수잇는 매개변수를 설정하고
```
Int operator[](const int& n) //const int& n을통해 불필요한 연산막기
  {
    return Int(data, level + 1, ary);
  }
```
와같이 바꿀수있지만 그 과정에서 임시객체가 계속 생성되기에 성능저하가 발생한다. 같은 main 함수기준 전자는 0.000145, 후자는 0.000178초 기록햇다. Int&를 사용하면 한 배열당 단하나의 Int가 필요하다.

## const int& a 를 인자로받는 함수에 대한 고찰
>복사를 안하기 때문에 더 빠를것이라 생각했지만 레퍼런스로 받고 역참조를할때에도 추가적인 연산이 필요하다. const T&a 는 T대신에 사용할수있다 객체처럼 덩치가 큰것들은 const&로 해주는 것이 좋지만 int 와같은 기본자료형은 그냥 복사해주는것이 좋다.  Passing by "const & is generally more efficient because it accepts a bigger value than passing by "const value" but passing an "int" to a function is only 4 bytes no mater the size of the number.So passing by reference does not have any noticeable advantage. It probably costs more when de-referencing. 그리고 일반적인 템플렛을 만들떄는 const&를 써주는게좋다. 객체도 받고 일반형도 받을수있기때문이다!



## 자료를 읽고 쓰게해주는 int operator=(const int& n), operator int()
>
```
int operator=(const int& n)
{
  *static_cast<int*>(data) = n;
  return n;
}
operator int() //읽을때만 사용가능.
{
return *static_cast<int *>(data);
}
```
여기서 = 연산자는 값을 쓸수있게 도와준다. 예를들면 arr[3][2] = 4 가있을떄
[]의 두번의 연산으로 &Int를 받고 = 연산자를 통해 직접 값을 Array에 대입한다.
operator int()의 경우는 cout << arr[3][2]와 같은 연산을 처리할때 작동한다.

