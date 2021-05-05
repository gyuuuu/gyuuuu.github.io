---
title:  "multipart/form-data는 put으로 전송하면 안된다?"
excerpt: "multipart/form-data는 put으로 전송하면 안된다?"

categories:
  - devlog
tags:
  - http
  - put
  - multer
  - multipart/form-data
last_modified_at: 2021-05-06T00:10:28-00:00
---

이전에 사진을 입력받기 위해 multer 모듈을 이용해서 파일을 전송 받았다.

이제 그 후로 입력받은 사진에 대한 수정을 위한 route로 **PUT /image** 과 같은 형식으로 만들었다.

이전에 포스팅한 restful api의 원칙에 따라 수정에 대한 method는 당연히 PUT으로 한 것이었다

```tsx
@Put('/user/image')
@HttpCode(200)
@UseInterceptors(FileInterceptor('image', multerOptions))
async uploadImage(
  @UploadedFile() files: Express.Multer.File,
): Promise<CoreOutput> {
  return await this.userservice.uploadImage(files);
}
```

service 함수에서는 기존에 있던 이미지 파일을 삭제하고, multer가 저장한 새로운 파일에 대한 경로가  데이터베이스에 저장되게 하였다.

이렇게 한 후 postman으로 테스트를 해보았다.

![postman](/assets/images/postman.png)

> **??????**

> **아니 403 error???**

처음에는 nest 어플리케이션 상에서 미들웨어나 guard 부분을 잘못 적용시킨줄 알고 한참 찾아봤었다. 하지만 아무리 찾아봐도 그런 오류는 아니었다.

또한 이전에 만들었던 post요청에 대해서는 매우 정상적으로 잘 작동하고 있었기 때문에 나의 나의 당혹감은 더 커졌다.

확인해보니 GET, PUT, DELETE 메소드 모두 403 에러를 받았다.
심지어 컨트롤러에서 구현이 되지 않은 GET과 DELETE도 403이었다. **404가 아니라!!!!**

몇 시간동안 열심히 구글링 해본 결과 유사한 사례를 찾았다.

스프링 웹 MVC에서는 multipart/form-data에 대해 POST 메서드만 지원한다고 한다.

[multipart는 HTTP POST로만 전송해야 한다::Outsider's Dev Story](https://blog.outsider.ne.kr/1001)

스프링에 대한 지식이 없어서 해당 내용을 완벽히 이해하지는 못했지만 스프링에서 fileupload하는부분의 소스코드상에서 POST만 받는다고 하는 것같았다.

이와 관련해서 nest 에서도 multer나 fileupload 관련해서 POST 메소드만 선언된 게 있나 엄청 찾아보았지만 아직 찾지못했다... ~~없는 것일수도....?~~

이를 확인해보기 위해 디버깅 또한 해봤는데 생각보다 간단하고 유용해서 공유하려고 한다.

- pakage.json

```json
"scripts": {
    "prebuild": "rimraf dist",
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "pm2-runtime start ecosystem.config.js",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json --detectOpenHandles"
  },
```

pakage.json 파일을 살펴보면 start:debug가 있다. 이 커멘드를 실행하면 디버그 모드로 실행시킬 수 있는 것이다.

따라서 이렇게 실행하면.

```bash
npm run start:debug
```

![debug](/assets/images/debug1.png)

이런 콘솔이 찍히면서 디버그 모드가 시작된다.

사진에 디버거가 *ws://127.0.0.1:9229/3dd~~~* 에서 실행 중이라고 나오는데

여기로 접속하라는 말이 아니었다 ~~(처음에 계속 여기로 요청보냈음...)~~

크롬을 열고 *chrome://inspect* 로 들어가면 된다.

![debug](/assets/images/debug2.png)

그럼 이런 화면이 나오는데 조금만 기다리면

![debug](/assets/images/debug3.png)

target 이 나온다. 밑에 inspect 를 클릭하면

![debug](/assets/images/debug4.png)

이런 창이 생기고, 여기서 디버깅이 가능하다.

console 탭은 애플리케이션에서 console.log 부분이 나오는 곳이고

sorce 부분은 

![debug](/assets/images/debug5.png)

이렇게 소스코드를 볼수 있고, 라인을 클릭하면 파란색으로 보이는 것처럼 브레이크포인트를 설정할 수 있다.

이렇게 브레이크 포인트를 잡고 요청을 보내봤는데 결과는 403 동일했다.

혹시몰라 POST 에 대한 요청도 브레이크포인트 잡고 보내봤는데 이건 잘 멈추고 코드 흐름을 따라갈 수 있었다.

따라서 지금까지 결론

- GET, PUT, DELETE에 대해 컨트롤러단으로 오기 전에 어디에선가 막혀서 403에러를 리턴하는 것같다.
- 유력한 후보로는 Interceptor가 있다. 다음에 한번 자세히 알아봐야겠다.

이어서 그럼 어떤 식으로 수정을 해야할지 찾아보다가 이런 글을 발견했다.

rest를 만든 Roy T. Fielding이 직접 PUT을 어떻게 사용해야하는지 알려주었다.

> Roy T. Fielding added a comment - 07/Mar/13 06:25

PUT means the sent representation is the replacement value for the target resource. A server could certainly support that functionality using any container format, it wouldn't be "normal" to use a MIME multipart, nor is it expected to be supported by the file upload functionality defined for browsers in RFC1867.

If you want to PUT a package, I suggest defining a resource that can be represented by an efficient packaging format (like ZIP) and then using PUT on that resource to have the side-effect of updating the values of its subsidiary resources.

리소스에 packaging format 정의하라고 한다.

그러니까 PUT으로 사용하고 싶으면 **PUT /image/image-path.jpg** 와 같이 사용해야 한다고 말하는 것 같다.

일리있는 말이지만 현재 내 서버에서 이미지 리소스에 접근하는 것은 nginx가 담당하고 있기 때문에 해당 방법을 적용하기 힘들었다.

## 따라서 내가 해결한 방법은

그냥 POST로 구현했다.ㅎㅎ

이미 만들어놨던 POST 메소드에 로직으로 조금 수정했다.

이미지가 이미 존재하는 user가 새로운 이미지를 전송했다면 이건 수정에 대한 요청으로 보고, 기존 이미지를 지우고 새로운 이미지의 path를 리턴하는 방식이다.

이렇게 길고 길었던, 결말은 매우 단순한 multipart/form-data에 대한 수정작업이 마무리되었다.

## 느낀점

처음에 엉뚱한 곳부터 찾아가느라고 많이 헤맸지만 그래도 rest에 대해 조금 더 알게된것 같아서 나름 뿌듯했다.

아쉬운 점이 있다면 아직 403 error에 대한 정확한 원인을 찾지못했다는 점이다.

다음에 시간되면 Interceptor부분 다시 살펴보면서 다시 찾아보도록 해야겠다.

또한 혹시라도 이 글에서 잘못된 내용이 있다거나 에러의 원인을 안다면 이메일로 알려주면 정말 감사할것 같다. 

~~아직 댓글 설정을 안해놔서...~~