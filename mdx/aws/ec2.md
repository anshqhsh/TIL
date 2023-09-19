### WARNING UNPROTECTED PRIVATE KEY FILE

- 키가 보호되지 않은 경우 발생하는 에러
  - 오픈되어 있는 키라 접속을 못한다고 에러가 납니다.
  - 키를 보호하려면 키의 권한을 변경해야 합니다.
  - 터미널에 아래 명령어를 해당 파일명을 추가하여 입력하여 권한 변경을 해줍니다.
  ```bash
    chmod 400 [키 이름]
  ```

![aws error1](https://taglog-image-uploader.s3.amazonaws.com/스크린샷 2023-09-19 오후 10.44.21.png)

> ### 참조
>
> https://velog.io/@rockwellvinca/AWS-SSH-%EC%A0%91%EC%86%8D-%EC%8B%9C-WARNING-UNPROTECTED-PRIVATE-KEY-FILE-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0
