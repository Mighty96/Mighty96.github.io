---
title: "Sorting Visualizer"

categories:
  - Project
tags:
  - C++
  - 정렬
toc: true
toc_sticky: true
---

# I. 의도

정렬알고리즘을 정리한지 한달도 지나지 않았는데 벌써 까먹었다.
마침 정렬과정을 눈으로 직접 확인할 수 있는 Sorting Visualizer를 만드는 유튜브를 보고 신기해서 북마크해놨던게 생각났다.

> [유튜브 링크 : Sorting Visualizer Tutorial (software engineering project)](https://youtu.be/pFXYym4Wbkc)  

위 같은 프로그램을 한번 직접 만들어보면 정렬알고리즘이 머릿속에 들어올 것 같았다. 나는 아직 프론트기술을 배우지 못했고, 따라서 동적 웹페이지를 구현할 수 없기 때문에 대신 콘솔로 구현하고자 했다.
<br><br><br>

# II. 해설

## main
```c++
#include <Windows.h> // system사용

int main()
{
    int select = 0;
    int length = 0;
    int arr[51];

    CursorView(false);
    system("mode con: cols=120 lines=55");


    while (true)
    {
        init(select, length);

        makeArr(arr, length);

        for (int i = 1; i <= length; i++)
        {
            draw(arr, i, length);
        }
       
        countDown(length);

        /*
        정렬 분기
        */
        switch (select)
        {
        case 1:
            bubbleSort(arr, length);
            break;
        case 2:
            selectSort(arr, length);
            break;
        case 3:
            insertSort(arr, length);
            break;
        case 4:
            mergeSort(arr, 1, length, length);
            break;
        case 5:
            heapSort(arr, length);
            break;
        case 6:
            quickSort(arr, 1, length, length);
            break;
        }

        gotoxy(0, length);
        std::cout << "정렬완료. 3초후 초기화면으로 돌아갑니다.";
        Sleep(3000);
    }
}
```

`system("mode con: cols=120 lines=55")` : 콘솔의 크기를 고정해준다.

`select`가 가지는 값에 따라 다른 정렬 분기로 들어간다.

## CursorView
```c++
#include <Windows.h> // system 사용

// 커서숨기기
void CursorView(char show)
{
    HANDLE hConsole;
    CONSOLE_CURSOR_INFO ConsoleCursor;

    hConsole = GetStdHandle(STD_OUTPUT_HANDLE);

    ConsoleCursor.bVisible = show;
    ConsoleCursor.dwSize = 1;

    SetConsoleCursorInfo(hConsole, &ConsoleCursor);
}
```

테트리스때에도 사용했던 함수이다.  
커서 구조체를 선언해서 세팅해주는 방법이라고 한다.  
단순히 커서를 숨기는 목적만 가지고 있었기 때문에 이부분은 널리 알려진 코드를 가져왔다.  

## gotoxy
```c++
#include <Windows.h>

void gotoxy(int x, int y)
{
    COORD pos = { x,y };
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
}
```

`COORD`는 단순히 SHORT형 x,y로 이루어진 구조체다.  
아래의 함수를 통해 직접적으로 커서를 핸들링할 수 있게 된다.  
마찬가지로 인터넷에서 쉽게 구할 수 있는 예제이다.

## countDown
```c++
// 카운트다운 출력
void countDown(int length)
{
    int timer = 3;
    while (timer > 0)
    {
        gotoxy(0, length);
        std::cout << timer << "초 후 시작합니다." << '\n';
        Sleep(1000);
        timer--;
    }
    gotoxy(0, length);
    std::cout << "                                        \n";
}
```
가장하단에 시간이 지남에 따라 숫자가 줄어드는 문구를 출력해준다.

## init
```c++
// 초기셋팅
void init(int& select, int& length)
{
    system("cls");
    gotoxy(0, 0);
    Sleep(10);

    std::cout <<
        "--------------------------------------\n" <<
        "| 1. BubbleSort                      |\n" <<
        "| 2. SelectSort                      |\n" <<
        "| 3. InsertSort                      |\n" <<
        "| 4. MergeSort                       |\n" <<
        "| 5. HeapSort                        |\n" <<
        "| 6. QuickSort                       |\n" <<
        "--------------------------------------\n";
    std::cout << "원하는 정렬을 선택해주세요.(1~6)" << '\n';
    std::cin >> select;
    while (select != 1 && select != 2 && select != 3 && select != 4 && select != 5 && select != 6)
    {
        std::cout << "올바른 번호를 입력해주세요.(1~6)" << '\n';
        std::cin >> select;
    }

    std::cout << "배열의 길이를 설정해주세요.(최솟값 : 10 | 최댓값 : 50)" << '\n';
    std::cin >> length;
    while (length < 10 || length > 50)
    {
        std::cout << "올바른 길이를 입력해주세요. (10~50)" << '\n';
        std::cin >> length;
    }
    system("cls");
}
```

초기 셋팅을 실시한다.  
`select`와 `length`는 원하는 값을 받지 못하면 다시 받는다.  
모든 셋팅이 이루어지면 화면을 모두 지워준다.  

## makeArr
```c++
#include <cstdlib> // srand, rand 사용
#include <ctime> // time 사용

// 랜덤배열 생성
void makeArr(int* arr, int length)
{
    int temp;
    srand((unsigned int)time(NULL));

    for (int i = 1; i <= length; i++)
    {
        arr[i] = i;
    }

    for (int i = 1; i <= length * 3; i++)
    {
        int x = rand() % length + 1;
        int y = rand() % length + 1;

        if (x != y)
        {
            temp = arr[x];
            arr[x] = arr[y];
            arr[y] = temp;
        }
    }
}
```

초기 배열을 랜덤하게 생성해준다.
매번 다른 배열을 생성하기 위해 시드에 `time`을 넣었다.

기본적으로 `1 ~ length` 까지 모두 인덱스로 초기화해주고 랜덤한 두 인덱스를 뽑아서 수를 교환해는 작업을 `length * 3`회 만큼 진행했다.

## draw
```c++
void draw(int* arr, int idx, int length)
{
    std::string temp;

    temp = "";

    for (int k = 1; k <= arr[idx]; k++)
    {
        temp += "■";
    }
    for (int k = arr[idx] + 1; k <= length; k++)
    {
        temp += "  ";
    }

    gotoxy(0, idx - 1);
    std::cout << temp << '\n';
}
```

배열과 특정인덱스를 받아 해당 인덱스의 라인만 갱신해준다.

`1 ~ length` 전체를 갱신하는 것 보다 훨씬 효율적이다.

단 `gotoxy`를 이용해서 해당라인으로 이동한 뒤에 덮어씌우는 것이기 때문에 뒤에 `length`까지 모두 지워지도록 빈칸을 더했다.

## Sort

6가지의 `Sorting Algorithm`을 각각 구현했다.  
따로 포스팅한 바가 있기 때문에 생략한다.

> [정렬 알고리즘 보러가기](https://mighty96.github.io/algorithm/sorting/)

<br><br><br>

# III. 결과

## BubbleSort
![bubblesort](https://user-images.githubusercontent.com/68958979/114733386-3ecd1580-9d7e-11eb-98eb-397aadf6fa48.gif)

## SelectSort
![selectsort](https://user-images.githubusercontent.com/68958979/114733388-3ffe4280-9d7e-11eb-9e4c-01309ce8bce4.gif)

## InsertSort
![insertsort](https://user-images.githubusercontent.com/68958979/114733392-4096d900-9d7e-11eb-8a46-a15734010803.gif)

## MergeSort
![mergesort](https://user-images.githubusercontent.com/68958979/114733395-4096d900-9d7e-11eb-9887-351cf3de0f7e.gif)

## HeapSort
![heapsort](https://user-images.githubusercontent.com/68958979/114733397-412f6f80-9d7e-11eb-9540-8763ce5b007f.gif)

## QuickSort
![quicksort](https://user-images.githubusercontent.com/68958979/114733399-412f6f80-9d7e-11eb-9fec-32c3d5eb7786.gif)

# VI. 마무리

단순히 글과 그림만으로 이해하는 것 보다 프로그램을 통해서 만들어보고 실시간으로 과정을 확인하는 것이 훨씬 이해에 도움이 되었다.  

누군가가 정렬 알고리즘에 대해 공부하려 하거든 꼭 내 프로그램으로 공부할 것을 권장하고 싶을 정도이다.

# [github에서 코드 보기](https://github.com/Mighty96/CPP_SortingVisualizer)
