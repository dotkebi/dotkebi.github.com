# Dagger2 : Dependency Injection

* [공식 홈](http://google.github.io/dagger/users-guide.html)

* 장점
    * 모든 instance 생성을 dagger2에 위임해버리면, activity나 application 생명 주기에 따라 instance를 관리해 줄 수 있음.
  
* 사용법
    * [번역](https://medium.com/@jason_kim/tasting-dagger-2-on-android-%EB%B2%88%EC%97%AD-632e727a7998#.4xp120zh2)
    * [예제](http://www.vogella.com/tutorials/Dagger/article.html)

---

## 간략한 구조

---

## 새로운 `activity`, `fragment` 등을 생성해서 객체 주입하는 시나리오

* 새 `activity`, `fragment` class 이름을 `ExampleActivity.java`라고 한다.  
* database에 접근하거나 할때, 필요한 `*Service`를 확인
* 멤버 변수에 정의. (`CustomerService`, `SaveService`를 사용한다고 가정)
* `ApplicationComponent.java`에 `void inject(ExampleActivity target);` 추가

**`BaseFA`로 정의된 `void inject(BaseFA target);`가 있어도 `void inject(ExampleActivity target);`를 만드는 이유**

* dagger2가 `Abstract`로 상속받아 만든 `class`에 별도로 정의된 `@Inject` 할 멤버변수는 제대로 인스턴스 주입이 되지 않는다  
* `ExampleActivity.java`에서 `inject` 실행

* `fragment`에서 `inject`도 동일한 방법으로 실행

---

    @Component(...)
    public interface ApplicationComponent {
       void inject(ExampleActivity target);
    }

    public class ExampleActivity extends BaseFA {
        @Inject
        CustomerService customerService;
         
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            ((MyApplication) getApplicationContext()).getApplicationComponent().inject(this);
        }
    }

---

## 인스턴스 하나만 주입하는 경우

    public class ExampleActivity extends BaseFA {
        private CustomerService customerService;
            
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            saveService = ((MyApplication) getApplicationContext()).getApplicationComponent().getSaveService();
        }
    
    }
    
* `ApplicationComponent.java`에 정의한 `SaveService getSaveService();`을 통해서 인스턴스가 주입됨.
