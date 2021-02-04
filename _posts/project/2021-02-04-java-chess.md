---
title: "자바로 만든 체스(1~2일차)"

categories:
  - Project
tags:
  - 체스
  - JAVA
toc: true
toc_sticky: true
---

자바의 문법을 익숙하게 하기위한 수단으로 체스 프로젝트를 선택했다.

swing을 통하여 GUI를 구현할 것이고, 한 대의 PC로 두명이 진행하는 로컬플레이부터  
가능하다면 두명의 플레이어가 원격으로 진행하는 멀티플레이까지 구현해보고 싶다.

프로젝트 진행 도중에 스스로 코드작성의도를 잊을까봐, 블로그에 따로 정리하려 한다.

# 오늘까지의 진행상황

## 기본UI구성

프로그램 테두리를 제거하여 깔끔하게 만들고, 이에 따라 창이동이 불가능함을 고려하여  
상단바를 만들고 드래그 앤 드롭으로 창을 이동할 수 있도록 했다.

```java
        menuBar.setBounds(0,0,1280,30);
        menuBar.addMouseListener(new MouseAdapter() {
            @Override
            public void mousePressed(MouseEvent e) {
                mouseX = e.getX();
                mouseY = e.getY();
            }
        });
        menuBar.addMouseMotionListener(new MouseMotionAdapter() {
            @Override
            public void mouseDragged(MouseEvent e) {
                int x = e.getXOnScreen();
                int y = e.getYOnScreen();
                setLocation(x - mouseX, y - mouseY);
            }
        });
```

상단바는 이렇게 구성되어 있다.
상단바를 클릭하는 순간 `mouseX`와 `mouseY`가 결정된다.
이 둘은 각각 마우스를 클릭한 순간 프로그램 스크린의 가장좌측상단의 좌푯값과 클릭한 좌푯값의 상대값을 가져온다.

마우스를 드래그하는 동안은 `x`와 `y`가 수시로 변경된다.
이 둘은 각각 마우스를 드래그하는 동안 (프로그램이 아닌) 출력화면의 좌측상단을 기준으로 절대좌표를 가져온다.

이 둘을 활용하여 `setLocation(x - mouseX, y - mouseY)`로 설정함으로써
프로그램 스크린을 수시로 내 마우스를 따라오도록 만든다.

![mouse](https://user-images.githubusercontent.com/68958979/106884551-cbc37500-6724-11eb-88b1-123c9347adac.png)
그림으로 표현하면 이렇게 볼 수 있다.

## 메인화면 구성

![1](https://user-images.githubusercontent.com/68958979/106881430-e09e0980-6720-11eb-9780-00d6554c69ed.gif)

메인화면은 세개의 버튼으로 구성했다.

각각 포토샵으로 하이라이트이미지를 만들어 마우스가 올라오면 기존의 이미지가 하이라이트 이미지로 바뀌고,  
마우스가 버튼밖으로 나가면 다시 기존의 이미지로 바뀌도록 만들었다.

`Local`버튼을 클릭하면 로컬플레이 화면으로 전환된다.
기존의 버튼들을 모두 감추고 미리 배치하여놓은, 로컬플레이를 하는데에 필요한 컴포넌트들이 보여지도록 했다.

`Exit`버튼을 클릭하면 프로그램이 종료된다.
`Multi`버튼은 아직 구현되지 않았다.

사실 버튼그림을 만드는데 많은 시간을 소비했다...

## 닉네임설정

![로컬1](https://user-images.githubusercontent.com/68958979/106884936-4a201700-6725-11eb-802f-80e08615e7c4.png)

메인화면에서 `Local`버튼을 클릭하여 로컬플레이에 진입했을 시,  
양 플레이어의 닉네임을 정할 수 있는 텍스트에어리어와 `Start`버튼을 보여준다.

## 게임보드 세팅

![로컬2](https://user-images.githubusercontent.com/68958979/106884940-4b514400-6725-11eb-96c0-6095926a1159.png)

```java
        player1 = new Player(p1.getName(), 1);
        player2 = new Player(p2.getName(), 2);
        for (int i = 0 ; i < 16 ; i++)
        {
            p1bt[i].setBounds(-23 + 75 * player1.getPieces()[i].getPos_x(), -15 + 75 * player1.getPieces()[i].getPos_y(), 75, 75);
            p2bt[i].setBounds(-23 + 75 * player2.getPieces()[i].getPos_x(), -15 + 75 * player2.getPieces()[i].getPos_y(), 75, 75);
            p1bt[i].setVisible(true);
            p2bt[i].setVisible(true);
        }
```

이름을 정하고 `Start`버튼을 누르면 텍스트에어리어와 버튼이 사라지고  
좌측 게임보드에 양측의 기물이 16피스씩 배치된다.

`Player`클래스는 `name`과, 16개 피스의 위치정보 및 종류가 담긴 `pieces[16]`을 필드로 갖는다.

`p1bt` 및 `p2bt`는 `JButton`을 담고 있는 배열이다.
각각 player1, player2의 피스 16개씩을 담고있다.

위 과정을 통하여 각각의 초기 위치를 지정해준다. 각 피스간의 간격은 75픽셀이다.

## 기물의 드래그 앤 드랍

![2](https://user-images.githubusercontent.com/68958979/106881436-e1cf3680-6720-11eb-8d88-e5bd649ff08a.gif)

마우스이벤트를 사용하여 각 기물들의 드래그 앤 드랍기능을 구현하였다.

```java
            p1bt[i].addMouseListener(new MouseAdapter() {
                int previous_x;
                int previous_y;
                @Override
                public void mousePressed(MouseEvent e) {
                    previous_x = (23 + (e.getXOnScreen() - getLocation().x)) / 75;
                    previous_y = (15 + (e.getYOnScreen() - getLocation().y)) / 75;
                }
```

특정 기물이 클릭되었을 때, 그 위치를 각각 `previous_x`와 `previous_y`에 담는다.

`getLocation().x`와 `getLocation().y`는 각각 출력화면을 기준으로  
현재 내 프로그램 스크린 가장 좌측상단의 좌표를 가져온다.  
즉, `e.getXOnScreen() - getLocation().x` 와 `e.getYOnScreen() - getLocation().y`는  
프로그램 스크린 가장 좌측상단을 원점으로 하였을 때, 내 마우스의 좌표를 나타내게 된다.

이 때, 위의 수식에 의하여 기물의 위치에 따라 0~7까지의 값을 갖게된다.

```java
                @Override
                public void mouseReleased(MouseEvent e) {
                    p1bt[temp].setBounds(-23 + 75 * previous_x, -15 + 75 * previous_y, 75, 75);
                }
```

이는 `mouseReleased`에서, 마우스로 드래그중인 기물이 드랍되었을 때 실행된다.
기존에 구해놓았던 위치값을 역산하여 다시 기존의 위치로 돌려놓는 역할을 한다.

만약 나눗셈을 통한 버림이 이루어지는 수식을 사용하지 않을 경우, 기물을 집을 때 기물의 75\*75픽셀 중  
어디를 잡았느냐에 따라 내려놓을 때의 위치가 변하게 된다.

이는 내가 선택한 기물이 이동할 수 없는 곳으로 드랍하였을 때, 기존의 위치로 돌아오는 기능을 수행하게 될 것 같다.

```java
            p1bt[i].addMouseMotionListener(new MouseMotionAdapter() {
                @Override
                public void mouseDragged(MouseEvent e) {

                    int x = e.getXOnScreen() - getLocation().x;
                    int y = e.getYOnScreen() - getLocation().y;
                    p1bt[temp].setBounds(x - 37, y - 37, 75, 75);
                }
            });
```

기물의 드래그는 현재 마우스의 위치를 `x`와 `y`에 담고, 드래그가 일어날 때마다 선택한 기물의 중앙을 현재 마우스 위치로 변경하도록 하였다.
