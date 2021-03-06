# tdd, cleancode, refactoring

* 1

    * 회사 일과 학습을 병행하기 쉽지 않다.
    * 쉽지 않은 일이기 때문에 아무나 할 수 없다.
    * 이번 5주의 교육 과정을 통해 개발자로서의 삶에 전환점을 만들 수 있는 계기가 되기를 바래 본다.

* 3

    * 여러분은 전문가다, 여러분의 코드에 자신감이 있어야 한다.

    * 본인이 짠 코드에 대해 개인적인 자부심을 느낄 정도의 리팩토링을 하다 보면 결국엔 그 과정이 실력으로 남는다.

    * **getter의 사용을 경계해라. 그리고 그 getter를 가진 쪽으로 메서드를 이동해라. 해당 객체로 메세지를 보내라. 그리고 결과를 받아서 처리해라.**

    * 객체들을 타고 타고 들어가서 실제 로직을 수행하기 때문에 bypass하는 메서드들이 많이 생긴다. 이상한게 아니다.

    * 상태를 가지는 객체에서 getter로 가지고 밖으로 나와서 로직을 구현하면, 그 객체를 사용하는 다른쪽에서 로직을 또 구현해야하고 중복 로직이 생기기 마련이다.

    * setter는 잊고 살자

    * **일급 콜렉션**을 사용할 때, 콜렉션을 랩핑했을 때 좋은점은?

        * 해당 객체를 재사용할때 유효성 검증을 다시 한번 하지 않아도 된다.(같은 콜렉션을 사용할 때 생기는 여러개의 중복 유효성 검증 제거)

    * 적절한 자료구조를 사용하면 유효성 검증 코드가 줄어들 수 있다.

    * LottoNumber를 자꾸 만들다 보면, new를 자꾸 해서 인스턴스 변수가 많이 생기면,

        * 매번 인스턴스를 만드는게 아니라, 만들어 놓고 재사용 해도 된다. Value Object이기 때문에 가능하다. 얼마든지 재사용해도 된다.

            ```java
            class LottoNumber {
                private static final Map<Integer, LottoNumber> lottoNos = new HashMap<>();
                
                static {
                    for (int i = 1; i < 46; i++) {
                        lottoNos.put(i, new LottoNumber(i));
                    }
                }
            
                private final int no;
            
                private LottoNumber(int no) {
                    if (no < 1 || no > 45) {
                        throw new IllegalArgumentException();
                    }
                    this.no = no;
                }
            
                static LottoNumber of(String value) {
                    if (Objects.isNull(value)) {
                        throw new IllegalArgumentException();
                    }
            
                    return of(Integer.parseInt(value.trim()));
                }
            
                static LottoNumber of(int value) {
                    ...
                }
            }
            ```


* 4
    * Lotto 피드백 + a
        * LottoNumber와 같이 값이 변경될 수 없는 객체를 VO라고 한다. DTO와 혼동 금지
        * LottoNumber를 재사용하기 위해 of를 여러개 추가한(정적 팩토리 메서드를 사용한) 이유
            * (생성자를 여러개 사용하지 않은 이유)
            * 생성자를 여러개 사용할 수도 있지만, 생성자는 메서드의 의도를 나타내는 네이밍을 할 수 없다.
    * 일급 Collections을 사용한다
        * collections을 primitive처럼 생각해서
        * collections을 컨트롤하는 열려있는 메서드에 대해 제한을 둬라.
        * 필요한 것만 오픈해라.
    * 상속 보다는 구성을 사용하라
        * 상속을 하게되면 부모클래스를 수정했을때, 자식클래스들이 모두 영향을 받게 된다.
* 5
    * QnA 피드백
        * 비즈니스로직은 어디에 구현하는 것이 맞을까?
            * Domain 객체에 로직을 추가하는 경우 안전한 코드가 되며, 단위 테스트도 더 쉬워진다.
            * 단위 테스트가 쉬운 이유는 Domain 객체의 경우 외부 DB나 API와 의존 관계를 가지지 않기 때문이다.
            * 상태를 가지는 도메인에 비즈니스로직을 구현해야 한다.
            * 서비스 로직에 비즈니스 로직을 구현해버리면 중복이 많이 생긴다. 계속해서 권한체크 등
                * 이렇게 하지 않으려면 상태를 가지는 객체인 도메인에 메세지를 전달해야 한다. 우리는 getter를 쓰지말고 상태를 가지는 객체에 메세지를 던지는 연습을 했지 않나.
            * 객체지향 적으로 코드를 짠다는 것은. 비즈니스 로직의 중복을 제거하는 일이다.
            * 메세지를 보내서 상태를 변경하지 않고 getter로 계속 가져와서 서비스단에서 짜다보면 테스트가 힘들어지고, 서비스가 뚱뚱해진다.
                * 서비스단에서 db에서 바로 조회해서 가져와서 확인하고 하는 로직이 들어가면, DB도 설치되어있어야하고, 테이블도 만들어져있어야 하고, 의존성이 많아진다.
                * 하지만, 도메인(엔티티)에 비즈니스로직을 만들면, DB에서 실제로 조회하는 코드가 없이 테스트가 가능하다. findById()로 가져와서 테스트 하는 대신, new Question()으로 만들어서 단위 테스트를 할 수가 있다.
            * 애플리케이션 개발할때 가장 중요한 것은 핵심 비즈니스 로직에 대한 단위 테스트이다. 반드시, 필요하다.
                * 그런데 이부분을 서비스로 빼게 되면 의존성이 많아져서 힘들어지고 그래서 안하게 된다.
                * Mock 프레임워크를 사용하지 않으면 더 힘들어진다.
            * junit만 사용하다가 mocito를 사용하면 엄청난 만족감을 느낀다.(DB없이 테스트가 가능하니까)
                * 근데 함정이다. 서비스에 비즈니스 로직을 서비스 레이어에 구현하고 있으니까 이게 만족도가 높아지는것이다.
                * 서비스에서 도메인 get하는 순간 의심해야 한다.
            * 의식적으로 서비스에 비즈니스 로직을 구현하지 않는다고 계속 생각하자. 그것만 해도 지금보다 훨 나을거다.
            * 서비스에서는 도메인에서 받은 결과를 가지고 다른 서비스와의 연결고리를 만드는 것이라고 생각하자.
                * 이렇게 되면 시간이 없어서 서비스 로직에 대한 테스트코드를 작성하지 못하더라도 덜 불안하지 않을까.
                * 물론 도메인 객체에 대한(핵심 비즈니스 로직에 대한) 단위 테스트는 필수이다.
        * 테스트의 우선순위는?
            * 물론 시간이 많다면 테스트를 다 하는 것이 좋다.
            * 하지만 그렇지 않을경우
            * 핵심 비즈니스 로직에 대한, 도메인에 대한 테스트는 필수이다.
            * 그다음 우선순위는 전체 레이어를 관통하는 atdd이다. 도메인 테스트보다 10배 정도 걸릴 수도 있지만 그래도 하는 편이 좋다. 이 안에서도 핵심 기능에 우선순위를 두고,
            * 그 다음 순위는 서비스 레이어의 테스트이다.
                * 서비스 레이어의 로직을 어떻게 하면 도메인 로직으로 옮길지 고민해보자
        * 서비스 레이어는 어떤 단위로 추가하는 것이 좋을까?
            * 밀접한 연관관계를 가지는 도메인에 대한 서비스 레이어를 만드는 것도 생각해볼만 하다.
                * 예를들면 QnA와 같이 1:N 관계를 가지는, 밀접한 관계를 가지는 서비스
            * 무조건 도메인 하나 추가되면 Controller - service  - repo 식의 정형화된 레이어를 추가하는 것에 대해 생각 해보자
        * 필드 각 값에 대한 테스트는 어떻게 하는 것이 좋을까?
            - 데이터를 추가 수정하는 경우 필드 값을 모두 테스트하고 싶다.
            - 필드 값 하나 하나를 테스트하려니 테스트 코드가 복잡해지고, 필드가 추가될 때마다 추가할 부분이 발생한다. 어떻게 하는 것이 좋을까?
            - title, contents 를 가지는 QuestionBody(VO)를 추가해서 거기에 Equals를 오버라이딩 하면?
            - update를 할때 QuestionBody를 받아서 해당 정보만 업데이트를 하면?
            - 정보가 추가될때 QuestionBody에 추가 해주면 된다. Question 도메인이 가벼워질 것이다.
            - 이렇게 빼다보면 바이패스 하는 메서드도 많아지고 일급 콜렉션 객체도 많아지는데 그거 두려워하지 마라, 객체지향 적으로 짜다보면 그렇게 된다.
        * Fixture(테스트를 위한 데이터)는 어떻게 생성하는 것이 좋을까?
            * 테스트를 진행하다보면 각 Test Case 별로 비슷한 데이터를 계속해서 생성해야 하는 상황이 발생한다.
            * 중복되는 부분도 많아 반복되는 것이 짜증난다. 어떻게 하는 것이 좋을까?
            * **Fixture를 쉽게 생성할 수 있는 방법을 찾는 것은 테스트를 유지하기 위해 정말 중요하다.**
        * atdd에서는 실제 repository를 접근해서 테스트하는 것은 좋지 않다.
            * restTemplate를 사용해서 테스트 해야한다. 
        * atdd 개발 프로세스의 핵심
            * 질문 삭제 실패 경우의 수가 4가지 있다고 가정하면
                * atdd의경우 질문 삭제 성공, 질문 삭제 실패의 2가지 경우만 구현해도 좋다.
                    * 실패의 경우의수 4개 모두 atdd 테스트 케이스를 만들려고 하지 마라.
                * Unit 테스트의 경우 질문 삭제 실패 4가지 모두 작성하는 것이 맞다.
        * jpa open in view 공부해보자.
            * boot에서 true인데,
            * 성능상 이슈있다. 디비 커넥션 물고있어서. 기본적으로 false로 설정.
            * @Transactional 범위 밖에서도 조회를 할 수 있게 한다.
                * findAll해서 가져와서 json으로 바꾸기 위해 @OneToMany Lazyloading 할때 
            * 이 얘기는 db랑 커넥션을 계속 물고있다는 이야기이다.
            * 서비스 메소드에서 트랜잭션 범위 내에서 request/response dto로 json으로 만들어서 리턴하는게 좋다. 이러면 엔티티 레이지로딩 문제도 발생하지 않는다.
* 6
    * 넥스트스텝 멤버쉽
        * 목표
            * 멤버쉽에 참여하는 개발자에게 개발자로서의 삶 성장 이직 등 멘토링을 한다.
        * 이유
            * 어떻게 성장할 것인지 가이드가 없어 초보 개발자들이 겪는 어려움이 있음
            * 기본 역량은 좋은데 이력서, 자소서 작설 못하거나 면접을 못봐서 탈락
            * 기업이 원하는 인재상과 개발자가 생각하는 인재상에 괴리감을 가지는 개발자 많음
        * 이더 좋은 대우를 받을 수 있도록 만드는 것이 넥스트 스텝의 목표
        * 슬랙으로 커리어 이직 등 상담 신청
            * 자바지기님은 프로그래밍 위주
        * 추천 가능 기업
            * 우형
            * 카카오페이
            * 카카오 그라운드원
            * 쿠팡
            * 야놀자
        * 최소 기업 추천
            * 1,2,3,4 필수 볼링 제외
        * 멘토링 담당자
            * 신희송님
                * w사 근무중
            * *GRIT*이란? 그릿(*GRIT*)은 바로 끝까지 노력할 수 있는 마음의 근력이자, 모든 성취의 근원. 자기조절력, 자기동기력, 대인관계력
            * 이 자리에 서게된 동기
                * 쭉 보니까 면접에 떨어지는 패턴이 보이더라(실력이 있음에도 불구하고 떨어지는 패턴)