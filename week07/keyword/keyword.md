- RestContollerAdvice

  RestControllerAdvice는 스프링에서 REST API 컨트롤러들에서 발생하는 예외를 한 곳에서 모아서 처리할 때 쓰는 어노테이션이다. 원래라면 각 컨트롤러마다 예외 처리를 따로 해줘야 하는데, 이렇게 하면 코드가 지저분해지니까 공통으로 예외를 받아주는 클래스를 하나 두고 거기에 @RestControllerAdvice를 붙인다.

- lombok

  lombok은 자바에서 귀찮은 보일러플레이트 코드를 자동으로 만들어주는 라이브러리로 래스 하나 만들 때마다 getter, setter, toString, equals, hashCode, 생성자 등을 일일이 생성하기보다 어노테이션 몇 개로 끝내게 해주는 역할을 한다.

- dto 형식 public static VS record 비교하기

  DTO를 public static class 로 만드는 방식은 스프링에서 가장 흔히 보는 형태로, 컨트롤러나 서비스 안에 요청·응답 전용 클래스를 안쪽에 두는 방식으로  public static에서 static은 static을 붙여서 바깥 클래스의 인스턴스랑 상관없이 독립적으로 쓰게 해두는 거고, 또한 일반 클래스이기에 기본 생성자, getter/setter, validation 어노테이션 등을 자유롭게 붙일 수 있다는 장점이 있다.

  record

  record는 생성자, getter 역할의 메서드, equals, hashCode, toString을 자바가 자동으로 만들어주기 때문에 코드가 아주 짧고, 한 번 만들어진 뒤로 값이 바뀌지 않는 불변 객체로 컨트롤러에서 서비스 결과를 그대로 감싸서 리턴할 때, 구조가 단순한 응답을 보낼 때 자주 사용하는 방식이다.