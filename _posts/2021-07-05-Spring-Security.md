---

layout: post

title: Spring Security

date: 2021-07-04 21:05:23 +0900

categories: Spring

---

스프링 시큐리티
-----------------

~~고백합니다.~~ 처음 서비스 개발을 진행했을 때 맡은 부분이 바로 로그인이였고, 이 시큐리티였다는 점이 제 개발 인생 중 가장 힘들었던 시간이였습니다.

그만큼 시큐리티에 대한 트라우마를 가지고 있으며 이겨내기 위해 이 포스팅을 적습니다.

일단 시큐리티를 공부하기 전 인증과 인가에 대한 개념을 간단히 알아보겠습니다.

## 인증과 인가

인증과 인가는 순차적으로 진행이 되어야 합니다. 인증되지 않는 사용자에게 인가를 제공할 수 없습니다. 그러므로 우리는 인증을 먼저 진행해야 합니다.

### 인증

인증이라는 단어는 쉽게 추측할 수 있습니다. **서비스에 접속하려는 사용자가 등록된 사용자 인지를 알아보는 것** 입니다. 이로써 인증을 거치게 되면 해당 사용자는 서비스에 등록되어 있는 사용자이므로 접근할 권한을 갖습니다.

이러한 인증을 서비스에 적용하면 **로그인**이라고 말할 수 있습니다.

### 인가

**인가는 리소스에 접근했을 때 해당 사용자가 접근하려는 리소스에 접근 할 권한이 있는지 알아보는 작업**이라고 할 수 있습니다.

이 작업이 필요한 이유는 간단한 예시로 알 수 있습니다. 만약 여러분이 게임을 하기위해 회원가입을 통해 사용자를 등록하고 로그인 과정을 거쳐 인증을 마쳤다고 생각해보겠습니다.

여러분은 리소스에 접근 할 권한을 얻었지만, 과연 운영자만이 사용가능 한 공지사항에 글을 올릴 수 있을까요?

답은 **아니다** 입니다. 쉽게 안된다는 것을 알 수 있죠.

이처럼 인증된 사용자에게도 모든 리소스에 접근 할 수 있는 것이 아닌 특정 리소스에만 접근 할 수 있도록 하는 제재가 있어야 합니다. 이 때 사용하는 것이 **권한** 입니다.

사용자마다 권한을 설정하여 권한에 맞게 리소스에 접근 할 수 있게 도와주는 것을 인가라고 말할 수 있습니다.

인증과 인가의 방식에 대해서는 여러 방식이 있지만 스프링 시큐리티는 쿠키-세션을 이용한 방식으로 두 과정을 진행합니다. 자세한 내용은 [인증과 인가](https://lion2me.github.io/basic/2021/07/03/%EC%9D%B8%EC%A6%9D%EA%B3%BC-%EC%9D%B8%EA%B0%80.html) 포스팅을 보시면 이해하실 수 있을 것이라 생각됩니다.

## Spring Security

스프링 시큐리티는 위에서 말씀드린 내용과 같이 **쿠키-세션을 이용한 인증 방식으로 진행됩니다.**

이 방식에 대해 간략하게 설명을 드리면 사용자가 인증을 위한 정보인 아이디와 비밀번호를 입력하면 해당 정보를 서버에서 받은 뒤 등록 된 사용자임이 증명되면 SID라는 세션ID를 할당하고 사용자에게 전달합니다.

사용자는 해당 SID를 쿠키에 저장해놓고, 이 후 요청을 보낼 때 SID를 함께 보내면 서버는 SID에 매칭되는 유저의 정보를 기반으로 서비스를 제공하는 방식이라고 할 수 있습니다.

이 과정에서 SID가 유지되는 만료시간과 사용할때마다 달라지는 SID의 특성 상 안정성을 조금 보장할 수 있는 장점이 있습니다.

자 그러면 어떻게 시큐리티는 인증 과정을 진행할까요? 그리고 인가를 위해 어떤 방식을 사용하고 있을까요?

### 시큐리티의 동작

이 그림보다 좋은 그림을 보질 못해서 가져오도록 하겠습니다.

<img src="/public/img/시큐리티.png" width="600" height="400">

과정을 글로 표현해보겠습니다.

1. AuthenticationFilter에 Http Request가 입력으로 들어옵니다.

2. UsernamePasswordAuthenticationToken을 통해 토큰을 생성합니다.

3. AuthenticationManager에서 등록되어있는 Provider 중 해당 토큰의 정보로 True가 나오는 요소가 있는지 탐색합니다.

3-1. Provider는 UserDatailsService를 통해서 가지고 있는 유저의 아이디를 기반으로 해당 사용자 정보를 불러옵니다.

3-2. 불러온 유저의 정보 중 비밀번호가 토큰에 저장 된 비밀번호와 일치하는지 확인 과정을 거칩니다.

3-3. 일치한다면 가져온 토큰에 Authenticate 메서드를 통해 인증이 된 토큰임을 표시합니다.

4. AuthenticationFilter는 인증이 된 토큰이라면 SecurityContext 내의 세션에 저장합니다.

## 시큐리티의 필터(Filter)

<img src="/public/img/시큐리티필터.jpeg" width="600" height="400">

이 무서운 그림은 [hyozkim.log](https://velog.io/@sa833591/Spring-Security-5-Spring-Security-Filter-%EC%A0%81%EC%9A%A9) 페이지에서 가져온 그림입니다. 왼쪽에 보시면 시큐리티에서 사용하는 필터를 알 수 있습니다.

우리는 이 필터 중 몇 가지의 필터에 대해서 중점적으로 공부해보겠습니다.

### 1. SecurityContextPersistenceFilter

이 필터에 대한 내용은 [yaho1024님의 블로그](https://velog.io/@yaho1024/Spring-Security-SecurityContextPersistenceFilter-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)를 바탕으로 공부했습니다.

시큐리티의 동작을 나타내는 그림에서 확인해보면 SecurityContextHolder에 SecurityContext가 존재하고 그 안에 Authentication이 있는 것을 볼 수 있습니다.

간단하게 설명하면 **Authentication 객체를 ThreadLocal 방식으로 Thread 내에 정보를 저장하기 위해 SecurityContext에 담아 SecurityContextHolder에 담는 방식이라고 말씀드릴 수 있습니다.**

담긴 정보는 Thread가 기억하고 있는 정보이기 때문에 인증된 정보를 다시 불러올 수 있게 됩니다. 이 과정을 영속화(Persist)라고 표현하여서 SecurityContextPersistenceFilter라고 이름지은 것 같습니다.

### 2. (UsernamePassword)AuthenticationFilter

폼 방식을 이용하여 로그인이 진행되는 필터입니다. 동작의 그림에서 처음 HttpRequest를 받는 필터가 이 부분입니다.

Manager와 Provider는 간략하게 설명을 마쳤으니, 로그인이 성공할 때 실행되는 AuthenticationSuccessHandler와 실패할 때 실행되는 AuthenticationFailureHandler에 대해서 간략하게 말씀리겠습니다.

**로그인이 성공하면 메인화면으로 전환되거나 특정한 서비스가 동작해야합니다. 이 과정을 직접 설계할 수 있는 방법이 AuthenticationSuccessHandler 입니다.** 직접 Custom이 가능하기 때문에 구현 시 알아두면 좋을 것 같습니다.

~~저는 xml파일을 수정하여 설정했지만, 그게 트렌드가 아닌 것 같기에 설정은 다른 블로그를 참조하시면 됩니다.~~

또 **로그인이 실패하면 이벤트를 줄 수 있습니다. 그 방법이 AuthenticationFailureHandler 입니다.** 마찬가지로 Custom이 간단합니다.

### 3. LogoutFilter

적힌대로 로그아웃을 위한 필터입니다. 정확하게 말하면 **SecurityContextHolder에 저장되어 있는 Authentication 를 더이상 사용 할 수 없도록 없앱니다. 그 후 LogoutSuccessHandler를 실행시킵니다.**

세션을 없애는 이유는 ThreadLocal 방식을 사용 할 때는 객체를 제대로 제거해주어야 다음에 저장되는 내용이 잘못 된 주소를 참조하여 문제가 발생할 일이 없다고 합니다. 이건 추가적으로 공부한 내용입니다.

### 4. SessionManagementFilter

쿠키-세션을 사용하면 SID가 쿠키에도 저장이 되기 때문에 상시 브라우저가 해당 정보를 가지고 있을 경우 해킹의 위험성이 있습니다. 그렇기 때문에 SID가 해킹되어 해커가 SID를 이용해 서버에 요청을 보내더라도 방어할 방법이 필요합니다.

대표적으로 SID의 만료시간을 지정하여 시간이 지나면 해당 SID를 사용할 수 없도록 만드는 방법이 있습니다. ~~또한 세션 고정을 통한 공격을 방지해준다는데 이 부분은 추가적으로 공부하겠습니다..~~

그리고 현재 해당 SID를 이용하여 서비스에 인증 된 사용자가 있다면 다른 사용자가 접근 했을 때 허용할지 혹은 최대 허용 수를 지정하는 일이 필요합니다.

이러한 일을 SessionManagementFilter이 도와주는 일입니다.

### 5. ExceptionTranslationFilter & FilterSecurityInterceptor

Exception이 발생한 경우 어떤 방식으로 처리 할 것인지를 도와주는 필터입니다. 인가의 과정에서 에러가 발생 할 수 있습니다. *처음 인증을 할 때 문제가 발생할 경우는 비밀번호가 틀린 문제일 것이고 AuthenticationFailureHandler 에서 잡아 줄 것 같습니다.*

그러면 우리는 인가 과정에서 일어날 수 있는 일을 살펴봅시다.

**1. 리소스에 접근하려는데 SID가 만료되어서 인증되지 않는 사용자가 되었다.**

**2. 리소스에 접근 할 수 없는 권한으로 접근을 시도하였다.**

FilterSecurityInterceptor가 인가의 과정에서 위의 두 문제에 대해 각각 다음의 Exception을 내보냅니다.

**1번 문제에서는 AuthenticationException을 내보냅니다.**

**2번 문제에서는 AccessDeniedException를 내보냅니다.**

그리고 해당 정보를 ExceptionTranslationFilter는 각각 다른 방식으로 문제를 해결합니다.

AuthenticationException의 경우에는 AuthenticationEntryPoint을 동작시킵니다. 일반적으로는 다시 로그인을 하도록 유도하는 방향으로 설계한다고 합니다.

AccessDeniedException의 경우에는 현재 권한이 없다는 페이지를 리턴해주며, 이 부분은 자유롭게 Custom 할 수 있기 때문에 변경 할 수 있다고 합니다.

### 참고 페이지

<https://sjh836.tistory.com/165>

<https://velog.io/@sa833591/Spring-Security-5-Spring-Security-Filter-%EC%A0%81%EC%9A%A9>

<https://velog.io/@yaho1024/Spring-Security-SecurityContextPersistenceFilter-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0>

<https://velog.io/@devsh/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%84%B8%EC%85%98-Session-Management-%EC%BB%A4%EC%8A%A4%ED%84%B0%EB%A7%88%EC%9D%B4%EC%A7%95-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0>

<https://ncucu.me/130>
