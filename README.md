# index.html, Nginx.conf


#### 접속 -> 메인화면
- awsoreo.shop 으로 라우팅된 Nginx에 접속시 메인화면 까지 동작
![ezgif com-video-to-gif](https://github.com/Dr-pep4/project/assets/102319207/aa46d5f4-140e-446e-a9ef-59b94fadf28f)

    
```
웹페이지 로드시 처음 배경을 흰색으로 하고 opacity가 0에서 1로 5초동안 서서히 증가하여 배경 이미지가 드러남


window.onload = function() {
    const fadeImage = document.querySelector('.fade-in-image');
    fadeImage.style.transition = 'opacity 5s';
    fadeImage.style.opacity = 1;
            
10초뒤 지정해놓은 엔드포인트 /main으로 이동


    setTimeout(function() {
        window.location.href = "/main";
        }, 10000);
};
```

<line>
    
### Nginx.conf

- 웹 서비스의 flow chart
<img width="600" alt="스크린샷 2023-09-19 오후 3 56 16" src="https://github.com/Dr-pep4/project/assets/102319207/3113bee3-1228-4bdf-8d75-d57c2bbf0ec1">

||페이지|endpoint|
|:-:|:-:|:-|
|frontend|메인 화면|/main|
|backend|관리자 로그인|/login|
|backend|응모자정보입력|/register|
|backend|관리 페이지|/admin_main|
|backend|경품 정보 입력|/enroll|


- proxy_pass에  upstream의 server URL을 지정하여 재사용시 upstream만을 수정할 수 있게 하여 재사용성 확보
```
upstream frontend {
    server jetty-public-front.apps.rosa-clu-07.q168.p1.openshiftapps.com;
    }



    # backend를 담당할 private cloud에서의 pod에 라우팅된 도메인을 backend로 지정

upstream backend {
    server jetty-private-backend.apps.team2-onprem.q168.p1.openshiftapps.com;
    }

 location /main {
             proxy_pass http://frontend/user_main.jsp;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_read_timeout 180s; 
        }

location /login{
             proxy_pass http://backend/admin_login.jsp;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
```
  
<line>
    
## Git Action
.github/workflows 디렉토리에서 '.yml' 파일로 설정

### 1. commit 감지 후 openshift에서 자동 배포

```
name: OpenShift Build

on:
  push:
    branches:
      - main  # 트리거할 브랜치

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repositoy
        uses: actions/checkout@v2
      
      - name: Set up OpenShift CLI
        uses: redhat-actions/oc-installer@v1
        with:
          openshift_server_url: ${{ secrets.OPENSHIFT_SERVER_URL }}
          openshift_token: ${{ secret.OPENSHIFT_TOKEN }}
          
          
     
      - name: Set up OpenShift Configuration
        run: |
          mkdir -p $HOME/.kube
          echo "$OPENSHIFT_CONFIG" > $HOME/.kube/config
        env:
          OPENSHIFT_CONFIG: ${{ secrets.OPENSHIFT_CONFIG }}

      - name: Log into OpenShift Cluster
        run: oc login ${{ secrets.OPENSHIFT_API_ENDPOINT }} --username ${{secrets.ROSA_ID}} --password ${{secrets.ROSA_PASSWORD}}

      - name: Deploy OpenShift Application
        run: oc start-build nginx --from-dir=$GITHUB_WORKSPACE
``` 
<line>
    
### 2. commit 감지 후 Webhook URL로 연동된 슬랙 채널에 알림
    
```
name: Slack Notification

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Set event timestamp
        id: event_timestamp
        run: echo "::set-output name=timestamp::$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
      
      - name: action-slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          author_name: You can get this as a pull request later
          fields: repo,commit,action,eventName,timestamp
          if_mention: failure, canceled
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        if: always()
```
<line>

