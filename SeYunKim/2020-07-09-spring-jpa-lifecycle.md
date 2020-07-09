> [예제코드](https://github.com/ksy90101/jpa-lifecycle-properties-ex)

![jpa-lifecycle-1](https://github.com/ksy90101/TIL/blob/master/spring/img/jpa-lifecycle-1.png?raw=ture)

![jpa-lifecycle-2](https://github.com/ksy90101/TIL/blob/master/spring/img/jpa-lifecycle-2.png?raw=ture)

- Spring JPA는 라이프사이클을 가지고 있다.
- 위의 사진 처럼 총 4가지의 생명주기를 가지고 있다.
- 테스트를 위해 Station Entity를 만든다.

```java
package jpa.lifecycle.ex.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Station {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	private String name;

	protected Station() {
	}

	public Station(final String name) {
		this.name = name;
	}

	public Long getId() {
		return id;
	}

	public String getName() {
		return name;
	}
}
```

```java
package jpa.lifecycle.ex.domain;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface StationRepository extends JpaRepository<Station, Long> {
}
```

### 생명 주기

- 비영속(New Transient)
    - 영속성 컨텍스트와 전혀 관계가 없는 상태로 객체만 생성된 상태이다.

```java
package jpa.lifecycle.ex.domain;

import static org.assertj.core.api.Assertions.*;

import javax.persistence.EntityManager;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

@DataJpaTest
class StationRepositoryTest {

	@Autowired
	private EntityManager entityManager;

	@Autowired
	private StationRepository stationRepository;

	@Test
	void newTransientTest() {
		Station station = new Station("잠실역");
		assertThat(entityManager.contains(station)).isFalse();
	}
}
```

- 영속(managed)
    - 영속성 컨텍스트에 저장된 상태로 Entity가 영속성 컨텍스트에 의해 관리되고 있는 상태이다.

```java
@Test
	void managedTest() {
		Station station = new Station("잠실역");
		stationRepository.save(station);
		stationRepository.flush();
		assertThat(entityManager.contains(station)).isTrue();
	}
```

- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태

```java
@Test
	void detachedTest() {
		Station station = new Station("잠실역");
		stationRepository.save(station);
		entityManager.detach(station);
		stationRepository.flush();
		assertThat(entityManager.contains(station)).isFalse();
		Station actual = entityManager.merge(station);
		assertThat(entityManager.contains(actual)).isTrue();
	}
```

- 이때, merge()를 하면 새로운 객체가 동등성을 보장하여 entityManger에 저장이 된다.
- 삭제(removed): 데이터 베이스와 영속성 컨텍스트 모두 삭제된 상태

    ```java
    @Test
    	void removedTest() {
    		Station station = new Station("잠실역");
    		stationRepository.save(station);
    		assertThat(stationRepository.findAll()).hasSize(1);
    		entityManager.remove(station);
    		assertThat(stationRepository.findAll()).hasSize(0);
    		stationRepository.flush();
    		Station station2 = new Station("잠실역");
    		entityManager.persist(station2);
    		assertThat(stationRepository.findAll()).hasSize(1);
    	}
    ```

    - 위의 테스트 코드에 대한 쿼리문을 확인하면 delete 쿼리문이 날라가는 것을 확인할 수 있다.

    ```java
    Hibernate: 
        
        drop table if exists station CASCADE 
    Hibernate: 
        
        create table station (
           id bigint generated by default as identity,
            name varchar(255),
            primary key (id)
        )
    Hibernate: 
        /* insert jpa.lifecycle.ex.domain.Station
            */ insert 
            into
                station
                (id, name) 
            values
                (null, ?)
    2020-07-09 18:30:15.426 TRACE 35604 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [잠실역]
    Hibernate: 
        /* select
            generatedAlias0 
        from
            Station as generatedAlias0 */ select
                station0_.id as id1_0_,
                station0_.name as name2_0_ 
            from
                station station0_
    2020-07-09 18:30:15.538 TRACE 35604 --- [    Test worker] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([id1_0_] : [BIGINT]) - [1]
    Hibernate: 
        /* delete jpa.lifecycle.ex.domain.Station */ delete 
            from
                station 
            where
                id=?
    2020-07-09 18:30:15.616 TRACE 35604 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
    Hibernate: 
        /* select
            generatedAlias0 
        from
            Station as generatedAlias0 */ select
                station0_.id as id1_0_,
                station0_.name as name2_0_ 
            from
                station station0_
    Hibernate: 
        /* load jpa.lifecycle.ex.domain.Station */ select
            station0_.id as id1_0_0_,
            station0_.name as name2_0_0_ 
        from
            station station0_ 
        where
            station0_.id=?
    2020-07-09 18:30:15.626 TRACE 35604 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
    Hibernate: 
        /* insert jpa.lifecycle.ex.domain.Station
            */ insert 
            into
                station
                (id, name) 
            values
                (null, ?)
    2020-07-09 18:30:15.627 TRACE 35604 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [잠실역]
    Hibernate: 
        /* select
            generatedAlias0 
        from
            Station as generatedAlias0 */ select
                station0_.id as id1_0_,
                station0_.name as name2_0_ 
            from
                station station0_
    2020-07-09 18:30:15.629 TRACE 35604 --- [    Test worker] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([id1_0_] : [BIGINT]) - [2]
    Hibernate: 
        
        drop table if exists station CASCADE
    ```