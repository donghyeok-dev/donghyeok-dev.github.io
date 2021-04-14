---
layout: customPost
title:  "Git Rebase로 Commit 합치기"
categories: 
  - Tools
  - Git
---

## Git Rebase로 Commit 합치기

feature 브랜치를 작업한 후 commit을 정리한 다음 push 또는 merge 등을 하고 싶을 때가 있습니다.



- 다음과 같은 commit을 합치는 작업입니다.

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210414164133970.png" alt="image-20210414164133970" style="zoom:150%;" />

- commit이 합쳐질 범위의 마지막 커밋을 확인하고 다음과 같은 명령을 실행합니다.

  ```
  git rebase -i 08c3c05^
  ```

- git edit설정에 따라 vim이나 지정된 edit에 rebase할 커밋 리스트가 나옵니다.

  여기서 rebase 대상이 아닌 커밋의 라인은 삭제합니다.

  ![image-20210414164458652](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210414164458652.png)

- rebase 대상 커밋 중 첫번째를 제외한 나머지 커밋들은 pick에서 s로 변경하고 저장, 종료합니다.

  <img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210414164634892.png" alt="image-20210414164634892" style="zoom:150%;" />

- 그럼 rebase 커밋 메시지를 입력하는 edit가 한번 더 열리는데 불필요한 커밋 메시지는 삭제하고 최종 커밋 메시지만 입력하고 저장, 종료합니다.

  ![image-20210414164750638](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210414164750638.png)

- 최종적으로 rebase가 완료됩니다.

  <img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210414164829151.png" alt="image-20210414164829151" style="zoom:150%;" />

- 마지막으로 로그를 확인합니다.

  <img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210414164910842.png" alt="image-20210414164910842" style="zoom:150%;" />



- 기타 사항

  - rebase 도중 작업을 취소하고 싶을 때

    ```
    git rebase --abort
    ```

  - rebase 작업이 완료되고 난 후 원복하고 싶을 때

    ```
    git reflog  //원복하고 싶은 커밋 hash값을 확인 후
    git reset --hard 7ff69aa
    ```

    