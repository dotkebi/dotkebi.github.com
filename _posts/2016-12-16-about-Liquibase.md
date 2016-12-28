# Intro

[liquibase](http://www.liquibase.org/)란?

* Database migration tool이다. 유사한 툴로 [flyway](https://flywaydb.org/)등이 있다.
flyway를 언젠가 써볼 생각을 하고 있었는데, jHipster 안에 liquibase가 포함되어 있어서 반강제적으로 사용하게 되었다. 꽤 시행착오를 거쳤는데, 아직은 얼마나 유용한지는 모르겠다. 큰 산을 하나 넘은 기분. 그러나 아직 jHipster의 가이드가 적용된 liquibase에서 initial schema 정보는 어떻게 작성해야 하는가는 모르겠다.

## 특징

* Database schema에 생성된 테이블 정보를 바탕으로 xml 파일을 생성해준다.
ROR의 activeRecord에서 생성해주는 migrations의 느낌이 들긴한다. 다만, `Database schema에 생성된 테이블 정보를 바탕`으로 라는 말에 주의하자. Database first...에 가깝다.
JPA등으로 선언한 model 기반으로 생성하지 않더라.

* CSV로 작성된 데이터를 migration 시점에 넣어줄 수도 있다.

