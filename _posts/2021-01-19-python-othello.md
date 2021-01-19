---
title: "파이썬으로 만든 오셀로"

categories:
  - Project
tags:
  - 오셀로
toc: true
toc_sticky: true
---

파이썬을 2주정도의 기간에 걸쳐 배우고,  
내가 배운 문법들로 무언가 결과물을 생산하고 싶었다.

적당한 난이도의 프로그램이 없을까 생각하던 도중,  
마침 교육봉사를 진행하던 센터에서 오셀로를 하고있는 아이들을 보고  
오셀로정도면 구현할 수 있지 않을까? 생각하게 되었다.

사용되는 이미지는 직접 제작하였고,  
GUI구현을 위해 `pygame` 라이브러리를 사용했다.

# 작성코드 의도 및 해설

## 1. 시작메뉴

![123](https://user-images.githubusercontent.com/68958979/105004727-b3423200-5a77-11eb-9879-70acb2cda6a7.png)

```python
def mainmenu():
    menu = True

    while menu:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

        gameDisplay.blit(mainmenu_background, (0, 0))
        Button(mainmenu_start, 405, 250, 150, 80, mainmenu_start_click, 380, 235, game)
        Button(mainmenu_explain, 405, 350, 150, 80, mainmenu_explain_click, 380, 335, explain)
        Button(mainmenu_finish, 405, 450, 150, 80, mainmenu_finish_click, 380, 435, finishgame)

        pygame.display.update()
        clock.tick(15)
```

메인메뉴는

- `시작하기`
- `설명보기`
- `종료하기`

세가지 버튼으로 구성되어 있다.

```python
class Button:  # 버튼
    def __init__(self, img_in, x, y, width, height, img_act, x_act, y_act, action=None):
        mouse = pygame.mouse.get_pos()  # 마우스 좌표
        click = pygame.mouse.get_pressed()  # 클릭여부
        if x + width > mouse[0] > x and y + height > mouse[1] > y:  # 마우스가 버튼안에 있을 때
            gameDisplay.blit(img_act, (x_act, y_act))  # 버튼 이미지 변경
            if click[0] and action is not None:  # 마우스가 버튼안에서 클릭되었을 때
                time.sleep(0.2)
                action()
        else:
            gameDisplay.blit(img_in, (x, y))
```

`Button`클래스는 이런식으로 구성되어 있다.

마우스 커서가 버튼 위에 올라갔을때 효과를 주고싶었다.

사이즈가 작은 글자와 사이즈가 큰 글자 이미지 두개를 만들어서  
마우스를 글자 위에 올려놓았을 때, 큰 글자로 이미지가 바뀌도록 했다.

버튼을 클릭하면 액션을 취하는데,

- 시작하기 -> `game`
- 설명보기 -> `explain`
- 종료하기 -> `finishgame`
  과 같이 함수를 호출한다.

함수를 호출하여 화면을 전환할 때  
즉각적인 이동보다는 텀을 두고싶어 0.2초정도 화면을 멈추고 이동한다.

## 2. 게임화면

![123](https://user-images.githubusercontent.com/68958979/105005623-ec2ed680-5a78-11eb-936a-ed2f3a7b7005.png)

본 게임에 들어갔을 때의 초기화면이다.

### 게임보드 세팅

```python
def game():
    gameexit = False
    player_turn = 1
    there_is[3][3] = 1
    there_is[3][4] = 2
    there_is[4][3] = 2
    there_is[4][4] = 1
```

중앙에 양 팀의 2개씩의 돌을 교차하여 부여하고, 플레이어1의 턴으로 설정한다.

`there_is`라는 의미불명의 2차원 배열이 있는데

```python
  # 돌이 놓여있는가? 누구의 돌인가? 를 판단
there_is = [[0 for i in range(8)] for j in range(8)]
```

총 8 \* 8 크기의 게임보드에서

- 특정 위치에 돌이 없으면 `0`,
- `플레이어 1`의 돌이 놓여있으면 `1`
- `플레이어 2`의 돌이 놓여있으면 `2`  
  로 값이 채워지는 배열이다.

> 지금와서지만 왜 `there_is` 라는 변수명을 썼는지 도저히 모르겠다....

```python
    while not gameexit:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
```

세팅 이후 게임을 종료하기까지의 무한루프로 들어간다.

### 기본화면 출력

```python
        count_player1 = 0
        count_player2 = 0
        gameDisplay.blit(game_background, (0, 0))
        gameDisplay.blit(game_player_turn, (670, 0))
        gameDisplay.blit(game_score, (670, 250))
        gameDisplay.blit(game_player1, (670, 393))
        gameDisplay.blit(game_player2, (670, 495))
```

게임스크린을 세팅한다.  
`count` 는 보드 위에 있는 각 플레이어의 말의 수를 나타낸다.

배경, 버튼, 스코어(SCORE이미지), 그리고  
플레이어들의 말의 모양을 직접 확인할 수 있도록 이미지를 뿌린다.

### 말 그림 출력

```python
         # 말 그림 놓기
        for i in range(8):
            for j in range(8):
                if there_is[i][j] == 1:
                    gameDisplay.blit(game_player1, (53 + (i * 70), 50 + (j * 70)))
                    count_player1 += 1
                elif there_is[i][j] == 2:
                    gameDisplay.blit(game_player2,(53 + (i * 70), 50 + (j * 70)))
                    count_player2 += 1
```

이제 플레이어들의 말 위치를 보드위에 뿌린다.

`there_is` 에 `1`로 기록되어 있다면 `플레이어1`의 말을,  
`there_is` 에 `2`로 기록되어 있다면 `플레이어2`의 말을 뿌린다.

이미지 제작시에 각 칸을 70px \* 70px로 만들었기 때문에,  
이에 맞추어 말이 놓일 x좌표와 y좌표의 수식을 결정했다.

### 스코어 출력

```python
score(count_player1, count_player2)
```

```py
# 현재 점수 표시
def score(player1, player2):
    font = pygame.font.SysFont("a두리둥실", 60)
    player1_score = font.render(str(player1), True, Red)
    player2_score = font.render(str(player2), True, Red)
    gameDisplay.blit(player1_score, (750, 400))
    gameDisplay.blit(player2_score, (750, 500))
```

`score`함수를 호출하여 양측의 스코어를 각각 표시해준다.

### 플레이어 턴 진행

```py
        if player_turn == 1:    # 1P 턴일 때
            gameDisplay.blit(game_player1, (760, 170))
            player1 = Player(game_player1, player_turn)
            player_turn = player1.turn
        else:                   # 2P 턴일 때
            gameDisplay.blit(game_player2, (760, 170))
            player2 = Player(game_player2, player_turn)
            player_turn = player2.turn
        pygame.display.update()
```

`player_turn` 이 1이냐, 2이냐에 따라 각각 플레이어1, 플레이어2가 턴을 갖는다.

턴 플레이어에 맞는 이미지를 Who's turn? 이미지 아래에 보여준다.  
각각 자신의 말을 나타내는 이미지를 가지고 `Player`클래스를 호출한다.

> 지금 다시 리뷰하다보니, player1 과 player2 변수를 굳이
> 따로 두어야 했을까? 싶은 생각이 든다.
> 차후 업데이트하게 된다면 고려해봐야겠다.

### Player 클래스(1)

```py
class Player:  # 플레이어 행동
    def __init__(self, img, turn):
        mouse = pygame.mouse.get_pos()
        click = pygame.mouse.get_pressed()
        self.turn = turn

        gameDisplay.blit(game_pass, (810, 580))
        if 930 > mouse[0] > 810 and 640 > mouse[1] > 580 and click[0] and turn == 1:
            self.turn = 2
            time.sleep(0.5)
        elif 930 > mouse[0] > 810 and 640 > mouse[1] > 580 and click[0] and turn == 2:
            self.turn = 1
            time.sleep(0.5)
```

`Player`클래스는 이렇게 구성했다.

이 때, 중간에 game_pass라는 이미지를 뿌리는데, 이것이 턴넘기기 버튼이다.

> 사실 오셀로는 특정 플레이어가 더이상 말을 둘 곳이 없으면  
> 자동으로 상대방 플레이어에게 턴이 넘어가야 한다.
>
> 그러나 이를 어떻게 구현할 수 있을지 잘 판단이 나지않아  
> 일단 턴넘기기라는 버튼으로 강제 턴이동이 가능하게끔 하였다.  
> 추후 업데이트해야할 사항이다.

### Player 클래스(2)

```py
        for i in range(8):
            for j in range(8):
                if (43 + (i * 70)) < mouse[0] < (113 + (i * 70)) and \
                        (40 + (j * 70)) < mouse[1] < (110 + (j * 70)) and \
                        there_is[i][j] == 0:  # 마우스 올려진 좌표 빈칸 검사
                    gameDisplay.blit(img, (53 + (i * 70), 50 + (j * 70)))  # 빈칸일 시 미리보기
                    if click[0] and turn == 1:  # 1P가 빈자리를 클릭
                        if possible_check(i, j, 1, 2):
                            there_is[i][j] = 1
                            self.turn = 2
                    elif click[0] and turn == 2:  # 2P가 빈자리를 클릭
                        if possible_check(i, j, 2, 1):
                            there_is[i][j] = 2
                            self.turn = 1
```

게임보드 전체를 순차적으로 검색하면서 다음의 행동을 한다.

1. 현재 마우스가 올라가있는 위치가 빈칸인지 검사한다.
2. 해당칸에 플레이어의 말의 미리보기를 보여준다.
3. 만약 해당칸을 클릭시, `possible_check`함수를 호출하여 놓을 수 있는지 판별하고,  
   `true`가 반환되면 현재 위치에 자신의 말을 놓고 턴을 바꾼다.

### possible_check 함수

```py
# 놓으려 하는 자리 주변 체크
def possible_check(x, y, player, opponent):
    check = False  # 놓을 수 없다고 가정
    # 좌상단
    if x > 0 and y > 0 and there_is[x - 1][y - 1] == opponent:
        temp_x = x - 1
        temp_y = y - 1
        while temp_x >= 0 and temp_y >= 0:
            if there_is[temp_x][temp_y] == opponent:
                temp_x -= 1
                temp_y -= 1
            elif there_is[temp_x][temp_y] == player:
                check = True
                temp_x += 1
                temp_y += 1
                while there_is[temp_x][temp_y] == opponent:
                    there_is[temp_x][temp_y] = player
                    temp_x += 1
                    temp_y += 1
                break
            else:
                break
```

위 if문에 대한 설명

1. 좌측 상단에 대하여 상대방의 말이 있는가 판단
2. 혹시 그렇다면, 계속해서 좌측 상단으로 이동하며 상대방이 줄지어있는지 판단
3. 줄지어져 있는 상대방의 말 뒤에 자신의 말이 있는가 판단
4. 모두 만족한다면 다시 되돌아오면서 상대방의 돌을 자신의 돌로 변경
5. 3번의 결과에 따라 `true` / `false` 를 반환.

> 이러한 로직을 사용하였는데, 문제는 놓는 돌을 기준으로  
> 8방향을 모두 판단하여야 한다는 것이다.  
> 그리하여 실제 해당함수의 코드는 8배의 길이다....  
> 이부분은 업데이트의 여지가 충분한데,
>
> ```py
> d_x[8] = {1, 1, 1, -1, -1, -1, 0, 0}
> d_y[8] = {1, -1, 0, 1, -1, 0, 1, -1}
> ```
>
> 과 같이 배열을 선언하여 for문을 통하여 1/8로 압축할 수 있을 것이다.  
> 이것도 업데이트 시 고려해야 할 사항.

### 승패판단

```py
        if count_player1 + count_player2 == 64:     # 총 64개의 돌이 놓이면 종료
            gameDisplay.blit(game_finish, (150, 100))
            if count_player1 > count_player2:
                gameDisplay.blit(game_player1, (450, 300))
            elif count_player2 > count_player1:
                gameDisplay.blit(game_player2, (450, 300))
            else:
                gameDisplay.blit(game_player1, (350, 300))
                gameDisplay.blit(game_player2, (550, 300))
            pygame.display.update()
            time.sleep(5)
            reset()
            mainmenu()
        clock.tick(30)
```

승패판단을 주기적으로 시행한다.

양 플레이어의 돌 갯수를 수시로 업데이트하면서  
돌 갯수의 합이 64(8\*8 보드이므로) 가 된다면  
Win 이미지와 함께 `count` 비교를 통하여 승리플레이어의 말을 띄워준다.

그렇게 5초간 결과창을 보여준 후 다시 메인화면으로 이동한다.

> 이 부분에서도 업데이트 해줘야할 부분이 있는데,  
> 사실 게임이 종료하는 기준은 단순히 보드가 모두 찼을 때 뿐만이 아니다.  
> 두 플레이어가 모두 말을 놓을 수 없게 되었을 때도 게임이 종료되어야 한다.
>
> 하지만 마찬가지로 수시로 이를 판단할려면 내가 만든 로직으로는  
> 수시로 모든 빈칸에 대하여 possible_check 함수를 시행하여  
> 모두 false값을 받아내야 한다.  
> 당장은 이외의 수단이 떠오르지 않아 더 고민해봐야할 것 같다.

## 3. 설명보기

```py
def explain():
    exp = True

    while exp:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
        gameDisplay.blit(explain_background, (0, 0))
        Button(explain_back, 670, 450, 230, 140, explain_back_click, 660, 442, mainmenu)

        pygame.display.update()
        clock.tick(15)
```

설명이 적혀있는 이미지를 뿌려주고,  
메인메뉴로 돌아가는 버튼에는 메인메뉴의 버튼들과 같은 기능들을 넣었다.

## 4. 게임종료

```py
def finishgame():
    pygame.quit()
    sys.exit()
```

게임을 종료한다.

## 5. 마치며

코드를 작성할 때는 정말 시간가는 줄 모르며 열심히 작성했는데,  
지금와서 보니 정말 부족한게 많다는 생각이 든다.

그래도 이렇게 다시 한번 되돌아보는 시간을 가지면서  
어떤 점이 부족했는지 어렴풋이 눈에 들어온다.

중간중간 언급한 것 외에도 변수명이 너무 중구난방하다던가...  
이 코드를 다른사람이 보면 과연 이해할 수 있을지 모르겠다.

앞으로 python을 사용할 일이 얼마나 있을지는 모르겠지만,  
python의 문법적인 측면 뿐 아니라 생각을 구현하는 과정에 대해  
많이 배울 수 있었던 것 같다.

# [github에서 코드 보기](https://github.com/Mighty96/Othello)
