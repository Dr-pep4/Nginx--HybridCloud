# index.html, Nginx.conf


#### index.html
- awsoreo.shop 으로 라우팅된 웹 서비스에 접속시 메인화면 까지 동작
![ezgif com-video-to-gif](https://github.com/Dr-pep4/project/assets/102319207/aa46d5f4-140e-446e-a9ef-59b94fadf28f)

- 웹페이지 로드시 처음 배경을 흰색으로 하고 opacity가 0에서 1로 5초동안 서서히 증가하여 배경 이미지가 드러남        
        window.onload = function() {
            const fadeImage = document.querySelector('.fade-in-image');
            fadeImage.style.transition = 'opacity 5s';
            fadeImage.style.opacity = 1;


- 10초뒤 지정해놓은 엔드포인트 /main으로 이동
            setTimeout(function() {
                window.location.href = "/main";
            }, 10000);
        };


### Nginx.conf
- 
  
