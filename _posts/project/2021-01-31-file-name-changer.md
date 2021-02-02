---
title: "자바로 만든 파일 이름 일괄 변경프로그램"

categories:
  - Project
tags:
  - 자바
  - 파일 이름 바꾸기
toc: true
toc_sticky: true
---

인터넷에서 사진을 수집하곤 하는데  
누구나 그렇듯, 굉장히 파일명에 대한 강박관념이 있어서  
사진을 모아놓은 폴더를 볼 때마다 속이 답답했다.

수백 수천장의 사진을 하나하나 번호붙여가며 이름을 바꿀 수 없기에,  
항상 프로그래밍을 공부한다면 꼭 내손으로 이걸 만들자 생각했었는데  
드디어 만들어볼 수 있게 되었다.

# 기능

![예시1](https://user-images.githubusercontent.com/68958979/106376867-5e3dde80-63dc-11eb-944a-9885cdbd953f.png)

프로젝트 내에 `data`폴더를 만들고, 그 안에 변경하고자 하는 파일들을 넣는다.  
현재 txt확장자 파일 3개와 bmp확장자 파일 2개가 들어있다.

![예시2](https://user-images.githubusercontent.com/68958979/106376868-5f6f0b80-63dc-11eb-995a-3e11c792f14a.png)

프로그램을 실행하면 `data`폴더 내에 몇 개의 파일이 존재하는지 알려주고,  
사용자에게 리스트를 보여줄지에 대한 응답을 요구한다.

![예시3](https://user-images.githubusercontent.com/68958979/106376869-5f6f0b80-63dc-11eb-9ab3-9ec1969c4aa7.png)

리스트를 확인하였을 때, `data`폴더 내에 있는 파일들의 리스트를 출력한다.  
사용자에게 특정 확장자를 제외할지에 대한 응답을 요구한다.

![예시4](https://user-images.githubusercontent.com/68958979/106376870-6007a200-63dc-11eb-9c67-6f892a4f3780.png)

특정 확장자를 제외하고자 하였을 때, 공백을 두고 제외하고자 하는 확장자를 입력받는다.  
예시에서는 bmp확장자 파일을 제외하였다.  
사용자에게 일괄변경할 파일들의 접두사를 입력받는다.

![예시5](https://user-images.githubusercontent.com/68958979/106376871-6007a200-63dc-11eb-9762-1bb49ac18e44.png)

`2021년에 모은 텍스트파일`이라는 접두사를 두고 txt확장자 파일들을 정리하고자 하였다.  
다음으로 일괄변경된 파일들의 뒤에 붙을 번호의 자릿수를 정한다.  
현재 3개의 파일밖에 없어서 1자리로 충분하지만, 시연을 위해 5를 입력했다.

![예시6](https://user-images.githubusercontent.com/68958979/106376872-60a03880-63dc-11eb-92a9-f0b38d55e580.png)

사용자에게 일괄변경을 위하여 최종 확정된 리스트를 보여줄지에 대한 응답을 요구한다.  
리스트를 확인하였을 때, 제외하지 않은 확장자들에 대해서 다시 리스트를 보여준다.
사용자에게 일괄변경을 시행할지에 대한 최종 확인을 요구한다.

![예시7](https://user-images.githubusercontent.com/68958979/106376873-60a03880-63dc-11eb-9434-17147a1cc7f5.png)

시행과정에서 bmp확장자 파일들은 리스트에서 제외하였기 때문에  
3개의 txt확장자 파일과 2개의 bmp확장자 파일 중 txt확장자 파일들만 파일명이 일괄변경된 모습을 볼 수 있다.

이 때, 사용자가 입력한 `2021년에 모은 텍스트파일`을 접두사로 갖고  
빈칸에 0을 채운 5자리수로 번호가 매겨진걸 볼 수 있다.

# 작성코드 해설

## 1. 파일리스트 불러오기

```py
        File dir = new File("data");
        File[] fileList = dir.listFiles();
```

`data`디렉토리를 지정하여 `fileList`에 디렉토리 내 파일들을 모두 넣어준다.

```py
     if (fileList != null && fileList.length > 0) {
            System.out.println("------------------------------------------------------");
            System.out.println("| 파일들의 이름을 숫자로 정리해주는 프로그램입니다.        |");
            System.out.println("| 총 " + fileList.length + "개의 파일이 발견되었습니다.                        |");
            System.out.println("------------------------------------------------------");
```

파일이 하나 이상 들어있는지 확인한 후에 그 뒤의 과정들을 시행한다.

```py
        } else {
            System.out.println("해당 폴더에 파일이 없습니다.");
```

만약 파일이 없다면, 해당 메세지를 출력하고 프로그램을 종료한다.

## 2. 파일리스트 출력

```py
            System.out.println("리스트를 확인하시겠습니까? (Y/N)");
            String judge = scanner.nextLine();
            if (judge.equals("Y") || judge.equals("y")) {
                for (File file : fileList) {
                    System.out.println(file.getName());
                }
            } else if (judge.equals("N") || judge.equals("n")) {
                ;
            } else {
                System.out.println("Y 또는 N 을 입력해주세요.");
            }
```

사용자의 응답에 따라 파일리스트를 출력한다.

`fileList`에 파일에 대한 정보가 담겨있기 때문에,  
for문을 통하여 모든 파일들의 `getName`메소드를 호출하여 출력했다.

`N` 또는 `n`을 입력하면 패스하고, 그 외의 입력이 들어오면 정확한 입력을 요구한다. (이 과정은 이하 생략)

## 3. 특정 확장자 제외

```py
            System.out.println("제외하고자 하는 파일 확장자가 있으십니까? (Y/N)");
            while (true) {
                judge = scanner.nextLine();
                if (judge.equals("Y") || judge.equals("y")) {
                    System.out.println("\n");
                    System.out.println("공백을 두고 대상에서 제외하고자 하는 확장자 명을 입력해주세요.");
                    temp = scanner.nextLine();
                    excludeExtension = temp.split(" ");
                    break;
                } else if (judge.equals("N") || judge.equals("n")) {
                    break;
                } else {
                    System.out.println("Y 또는 N 을 입력해주세요.");
                }
            }
```

사실 혼자 사용할 것이면 별로 필요없는 기능이긴 한데,  
그래도 뭔가 기능이 하나라도 더 있으면 좋을 것 같아 함께 구현했다.

여러가지 파일들이 모여있을 때, 특정 확장자를 리스트에서 제외할 수 있다.
이 과정에서는 단순히 사용자에게 입력받은 확장자들을 `excludeExtension`이라는 `String`배열에 저장해두기만 한다.

## 4. 접두사 및 번호자리수 입력

```py
            System.out.println("숫자 앞에 붙을 접두사를 입력해주세요.");
            head = scanner.nextLine();
            System.out.println("\n\n");
            System.out.println("몇자리 번호를 부여하시겠습니까?");
            System.out.println("예) 3 입력 시 a001.jpg a002.jpg ...");
            number = scanner.nextInt();
            scanner.nextLine();
```

파일명을 구성하는 접두사와 번호자리수를 입력받는다.

## 5. 최종리스트 확인

```py
            System.out.println("최종 리스트를 확인하시겠습니까? (Y/N)");
            while (true) {
                judge = scanner.nextLine();
                if (judge.equals("Y") || judge.equals("y")) {
                    for (File file : fileList) {
                        temp_ = file.getName().split("\\.");
                        if (Arrays.asList(excludeExtension).contains(temp_[1])) {
                            ;
                        } else {
                            System.out.println(file.getName());
                            count++;
                        }
                    }
                    break;
                } else if (judge.equals("N") || judge.equals("n")) {
                    for (File file : fileList) {
                        temp_ = file.getName().split("\\.");
                        if (Arrays.asList(excludeExtension).contains(temp_[1])) {
                            ;
                        } else {
                            count++;
                        }
                    }
                    break;
                } else {
                    System.out.println("Y 또는 N 을 입력해주세요.");
                }
            }
```

사용자의 요구에 따라 최종 리스트를 출력한다.

이 때, `temp_`는 String배열로 이루어져 있는데, 이는 각각의 파일들에 있어서  
`temp_[0]`에는 파일명을, `temp_[1]`에는 확장자 명을 `.`으로 구분하여 입력받는다.

이 때, 만약 기존에 제외하고자 했던 확장자명들을 모아놓은 `excludeExtension`에  
현재 선택된 파일의 확장자명이 포함되어 있다면, 출력되지 않도록 하였다.

사용자가 출력을 원하지 않더라도 최종 확인을 받는 과정에서의 `count`를 얻기 위하여 출력을 제외하고 같은 과정을 진행한다.

## 6. 파일 이름 일괄 변경

```py
     System.out.println(fileList.length + "개의 파일 중 " + count + "개의 파일을 정리하시겠습니까? (Y/N)");
            count = 1;
            while (true) {
                judge = scanner.nextLine();
                if (judge.equals("Y") || judge.equals("y")) {
                    for (File file : fileList) {
                        temp_ = file.getName().split("\\.");
                        if (Arrays.asList(excludeExtension).contains(temp_[1])) {
                            ;
                        } else {
                            File rename = new File("data/" + String.format("%s%0" + number + "d.%s", head, count, temp_[1]));
                            file.renameTo(rename);
                            count++;
                        }
                    }
                    System.out.println();
                    break;
                } else if (judge.equals("N") || judge.equals("n")) {
                    System.out.println("정리하지 않고 프로그램을 종료합니다.");
                    break;
                } else {
                    System.out.println("Y 또는 N 을 입력해주세요.");
                }
```

최종적으로 파일 이름 변경에 대한 사용자의 확인을 받는다.

사용자가 `Y` 또는 `y`를 입력했다면,  
다시 한 번 전체파일들을 불러와 `temp_`배열에 이름과 확장자를 나누어 저장하고,  
`excludeExtension`에 포함되지 않는 확장자의 파일이라면  
새로운 파일 인스턴스의 이름을 구성한다.

`head`와 `number`는 각각 사용자에게 입력받은 접두사와 번호자리수이다.

기존의 `file`의 파일명을 새로 만든 파일 인스턴스 `rename`의 파일명으로 바꾼다.

# 마치며

사실 프로그램을 작성할 당시에는 막연히 내가 사용할 것이라는 생각만 했다.  
그래서 그런지 사용자의 입력에 대한 예외처리가 하나도 되어있지 않아서,  
만약 누군가가 사용하게 된다면 완전 엉터리 프로그램이 되겠구나 생각했다.

그래도 실용성이라는 부분에 있어서 처음으로 만들어본 프로그램이었기 떄문에 굉장히 만족스럽다.

다음에도 무언가 필요한 것이 있다면 간단한 수준 내에서 스스로 만들 수 있다는 자신감을 얻을 수 있던 것 같다.

# [github에서 코드 보기](https://github.com/Mighty96/FileNameChanger)
