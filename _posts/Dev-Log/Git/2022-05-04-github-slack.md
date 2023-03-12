---
title: "[Git] GitHub과 Slack 연동하기"
date: 2022-05-04 19:50:00 +0900
categories: [Dev-Log, Git]
tags: [git, collaboration, slack, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---

<br>

부스트캠프 대회 때마다 GitHub - Slack 연동 신청을 했었는데 정작 실제로 사용은 안했었다. ㅎㅎ 그렇게 마지막 대회가 되어서야 연동을 하게 되었는데 issue나 pull request 알림이 생각보다 굉장히 편리해서 이렇게 기록해둔다!

<br>

1. Slack의 상단 검색창에 `github`을 검색하거나 링크([slack.github.com](https://slack.github.com/))를 통해 Slack GitHub 앱을 추가한다.

   > 설치 과정에서 채널을 선택할 수 있다고 알고 있는데, 내가 설치할 때는 그런 화면이 나오지 않았다. 😢

2. GitHub을 추가할 채널명을 클릭한 후, `통합` - `앱 추가`를 통해 GitHub 앱을 추가한다.

    ![](/assets/img/posts/Dev-Log/Git/2022-05-04-github-slack.png){: width="60%" height="60%"}
   
3. Slack 내 입력창에 다음의 명령어를 입력하여 로그인을 진행한다.
   
   ```
   /github signin
   ```

4. 해당 채널의 입력창에 다음의 명령어를 입력한다.

   ```
   /github subscribe owner/repository [feature]
   ```

5. 연동을 해제하려면 다음의 명령어를 입력한다.

   ```
   /github unsubscribe owner/repository [feature]
   ```

6. 구독 가능한 항목(`feature`)은 다음과 같다. 다음의 것들 중 원하는 항목을 쭉 연달아서 `[feature]`에 입력하면 된다.
   
   - 디폴트로 활성화 되는 항목
     - `issues` - Opened or closed issues
     - `pulls` - New or merged pull requests
     - `statuses` - Statuses on pull requests
     - `commits` - New commits on the default branch (usually `master`)
     - `deployments` - Updated status on deployments
     - `public` - A repository switching from private to public
     - `releases` - Published releases

   - 추가 가능한 항목
     - `reviews` - Pull request reviews
     - `comments` - New comments on issues and pull requests
     - `branches` - Created or deleted branches
     - `commits:all` - All commits pushed to any branch
