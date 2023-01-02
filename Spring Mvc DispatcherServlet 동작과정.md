![115183554-f499ba80-a116-11eb-9592-5404d245e173](https://user-images.githubusercontent.com/60213850/210191086-4e4408fe-ae5b-43de-94b6-f9f12e02e7b9.png)


Spring MVC는 DispatcherServlet, View Resolver, Interceptor, Handler, View 등으로 구성되어있다.

이중에 DispatcherServlet이 가장 앞단의 front controller 역할을 하며 가장 핵심적인 역할을 한다.
** DispatcherServlet은 Java EE의 Servlet을 래핑한 클래스이다. 컨트롤러의 컨트롤러 같은 느낌으로 개발을 아주 편리하게 해준다.
  Request를 올바른 처리 핸들러에 위임하는 것 부터 Model과 View 처리, Controller 매핑 등 다양한 기능을 지원해준다. **
  
![img1 daumcdn](https://user-images.githubusercontent.com/60213850/210189647-9f3a0a7f-2a45-4431-b1f1-ce592f5981bd.png)

Front controller라고 표현된 DispatcherServlet이 받은 Request를 핸들러에 위임하고,
Controller에서 만들어진 model을 reponse에 알맞게 랜더링하고,
View Template을 랜더링하는 등 많은 작업을 수행한다.

DispatcherServlet이 어떻게 동작하는 과정은 다음과 같다.

1. 먼저 DispatcherServlet을 등록해야 한다. 

 먼저 DispatcherServlet을 사용하기 위해서는 당연히 어딘가에 등록을 해야할 것이다.
 요즘에는 잘 사용하지 않지만 web.xml을 사용했던 시절부터 쫓아가보자.

 어플리케이션 배포기술자(Deployment Descriptor) web.xml이라는 파일에 서버가 알아야 할 정보를 서술해 놓았었다.
 서블릿, 그리고 리스너와 필터와 같은 컴포넌트, error와 welcome페이지를 등록할 수 있다.

 일반적으로 여기에 DispatcherServlet을 url 패턴과 함께 등록하여 모든 url을 DispatcherServlet을 거치도록 설정을 해놓을 것이다.
 Servlet3.0이상에서부터 WebApplicationInitializer 구현 또는 AbstractAnnotationConfigDispatcherServletInitializer 상속으로
 web.xml에서 하던 설정을 java config로 설정할 수 있다.
 java config를 통해 설정하여 classpath내에 두면, SpringServletContainerInitializer가 서블릿을 초기화할 때 감지한다.

 WebApplicationInitializer든 web.xml이든 어떤 방법을 사용하든 DispatcherServlet이 등록되었을 것이다.

2. DispatcherServlet 초기화
 웹 컨테이너(supports servlet tomcat 같은)는 사용자의 요청에 대해 request와 response 객체를 생성한다.
 그리고 앞에서 살펴본 것 처럼 배포서술자를 통해 어떤 DispatcherServlet이 처리할지를 알아낸다.
 만일 해당 클래스가 한번도 실행된 적이 없다면, 새로 인스턴스를 생성하고 초기화를 한다.

 DispatcherServlet이 생성되고 초기화 될 때 initStrategies(ApplicationContext context) 메소드를 실행하여
 DispatcherServlet에 관련된 빈(Bean)들을 초기화 한다.
 앞에서 다양한 기능들을 지원해준다고 했는데, 그 다양한 기능을 전략 패턴을 통해 구현했다.
 생성된 전략 빈들을 DispatcherServlet에 주입하고, 주입된 전략 빈에 의해 기능이 수행된다.
 초기화 메소드에서 아래에서 설명할 전략 인터페이스의 빈이 존재하는지를 찾고,
 없으면 default 설정에 따라 빈을 생성하거나 아무 작업이 없을 수도 있다.
 (전략 인터페이스의 빈 생성은<mvc:annotation-driven/> 혹은@EnableMvc를 통해 웹 컨테이너가 initialize 되는 시점이다.)
 
 그럼 DispatcherServlet이 가지는 기능(전략 인터페이스)를 간단히 알아보자.

Spring MVC의 구체적인 동작과정은 다음과 같다.

  1. DispatcherServlet이 모든 웹 브라우저로부터의 요청을 받는다.
  2. DispatcherServlet은 HandlerMapping으로 주어진 request를 처리할 수 있는 Handler 객체를 가져온다.
  3. 가져온 Handler를 실행(invoke) 시킬 수 있는 HandlerAdater 객체를 가져온다.
  4. 만약 해당 Controller를 처리할 Handler객체에 적용할 interceptor가 존재한다면 모든 interceptor객체의 preHandle 메소드를 호출한다.
  5. HandlerAdapter객체를 통해 실제 컨트롤러의 메소드를 실행 후 ModelAndView를 얻는다.
  6. 만약 해당 Controller를 처리할 Handler객체에 적용할 interceptor가 존재한다면 모든 interceptor객체의 postHandler메소드를 호출한다.
  7. DispatcherServlet은 5번 과정에서 얻은 ModelAndView를 통해서 view name을 ViewResolver에게 전달하여 응답에 필요한 View객체를 얻어온다. 
  8. DispatcherServlet은 7번 과정에서 얻은 View객체에 5번 과정에서 얻은 ModelAndView의 Model을 파라미터로 넘겨주어 render메소드를 호출하여
     페이지 렌더링을 수행한다.
  9. DispatcherServlet은 랜더링 된 페이지를 response로 사용자에게 리턴한다.
