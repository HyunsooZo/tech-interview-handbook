---
icon: object-exclude
---

# Join

## Q. Inner Join과 Outer Join의 차이를 설명하고, 각각을 언제 사용하는지 예를 들어 설명해주세요.

**A.**\
Inner Join과 Outer Join은 테이블 간 데이터를 결합하는 방식으로, 반환되는 데이터 범위에서 차이가 있습니다.

* **Inner Join**: 두 테이블의 공통 키 값을 가진 행만 반환(교집합).
  * **사용 시기**: 공통 데이터만 필요한 경우. 예: 길드에 가입한 유저만 조회.
  *   **예시**:

      ```sql
      SELECT u.user_id, u.username, g.guild_name
      FROM users u
      JOIN guild_membership g ON u.user_id = g.user_id;
      ```

      결과: 길드에 가입한 유저만 반환.
* **Outer Join**: 한쪽 또는 양쪽 테이블의 모든 데이터를 포함(합집합). Left, Right, Full Outer Join으로 나뉩니다.
  * **Left Outer Join**: 왼쪽 테이블의 모든 행을 포함, 오른쪽 테이블에 없는 경우 NULL 반환.
  * **사용 시기**: 모든 유저와 가입 여부 확인. 예: 길드 미가입 유저도 포함.
  *   **예시**:

      ```sql
      SELECT u.user_id, u.username, g.guild_name
      FROM users u
      LEFT JOIN guild_membership g ON u.user_id = g.user_id;
      ```

      결과: 모든 유저 포함, 길드 미가입 유저는 `guild_name`이 NULL.

Inner Join은 정확한 매칭 데이터가 필요할 때, \
Outer Join은 전체 데이터와 부분 매칭을 포함해야 할 때 사용됩니다.

## Q. Self Join은 무엇이고, 어떤 상황에서 사용할 수 있을까요?

**A.**\
Self Join은 동일한 테이블을 자신과 조인하는 방식으로, 테이블 내 계층적 또는 상호 관계를 처리할 때 사용됩니다.

* **특징**: 테이블에 별칭(alias)을 부여해 두 개의 논리적 테이블로 취급.
* **사용 시기**: 같은 테이블 내 관련 데이터를 연결. 예: NPC 간 관계 조회.
*   **예시**: NPC 관계 테이블에서 NPC A와 B의 정보를 조회.

    ```sql
    SELECT a.name AS npc_a_name, b.name AS npc_b_name, r.relationship_type
    FROM npc_relationship r
    JOIN npc a ON r.npc_id = a.id
    JOIN npc b ON r.related_npc_id = b.id;
    ```

    결과: NPC A와 B의 이름 및 관계 유형 반환.

Self Join은 조직도(상사-부하), 네트워크(친구 관계) 등 자기 참조 데이터를 다룰 때 유용합니다.

## Q. 서브쿼리 대신 Join을 사용하면 어떤 장점이 있을까요?

**A.**\
서브쿼리는 쿼리 내에 쿼리를 중첩하는 방식이고, Join은 여러 테이블을 연결해 단일 쿼리로 처리합니다. \
Join은 서브쿼리 대비 성능과 가독성 측면에서 장점이 있습니다.

* **장점**:
  * **성능 향상**: 데이터베이스 옵티마이저가 Join을 최적화해 한 번에 데이터를 처리, 서브쿼리는 반복 실행으로 비효율적일 수 있음.
  * **가독성 개선**: 복잡한 서브쿼리보다 Join이 쿼리 구조를 명확히 표현.
  * **인덱스 활용**: Join은 인덱스를 효과적으로 사용해 검색 속도 향상.
*   **예시**: 캐릭터의 최고 등급 장비 조회.

    *   **서브쿼리**:

        ```sql
        SELECT c.id, c.name,
               (SELECT e.name FROM equipment e 
                WHERE e.character_id = c.id 
                ORDER BY e.grade DESC LIMIT 1) AS best_equipment
        FROM character c;
        ```
    *   **Join**:

        ```sql
        SELECT c.id, c.name, e.name AS best_equipment
        FROM character c
        JOIN equipment e ON c.id = e.character_id
        LEFT JOIN equipment e2 ON c.id = e2.character_id AND e.grade < e2.grade
        WHERE e2.character_id IS NULL;
        ```

    Join 방식은 서브쿼리 반복을 줄이고 인덱스를 활용해 성능이 우수합니다.

Join은 대규모 데이터나 복잡한 관계 처리 시 서브쿼리보다 효율적입니다.

## Q. 조인을 사용할 때 N+1 문제가 발생하는 이유와 해결 방법은 무엇인가요?

**A.**\
N+1 문제는 ORM(예: JPA) 사용 시 발생하는 쿼리 비효율성으로, \
부모 엔티티 1개를 조회한 후 연관된 자식 엔티티를 N개 각각 조회해 총 N+1번 쿼리가 실행되는 현상입니다.

* **발생 이유**:
  * 지연 로딩(Lazy Loading) 설정에서 부모 엔티티 조회(1번) 후, 각 자식 엔티티를 개별 쿼리로 조회(N번).
  *   예: 캐릭터 목록 조회 후 각 캐릭터의 장비를 별도 쿼리로 요청.

      ```sql
      SELECT * FROM character; -- 1번
      SELECT * FROM equipment WHERE character_id = 1; -- N번
      SELECT * FROM equipment WHERE character_id = 2;
      ...
      ```
* **해결 방법**:
  *   **Fetch Join**: JPA에서 `@Query` 또는 `fetch` 키워드로 연관 데이터를 한 번에 조회.

      ```java
      @Query("SELECT c FROM Character c JOIN FETCH c.equipment")
      List<Character> findAllWithEquipment();
      ```
  * **Eager Loading**: 연관 데이터를 즉시 로딩하도록 설정(과도한 로딩 주의).
  * **Batch Fetch**: 여러 엔티티를 배치로 조회(예: `IN` 쿼리 사용).
  *   **SQL Join**: ORM 대신 직접 Join 쿼리 작성으로 단일 쿼리 실행.

      ```sql
      SELECT c.*, e.* FROM character c JOIN equipment e ON c.id = e.character_id;
      ```

N+1 문제를 해결하면 쿼리 수를 줄여 성능을 크게 개선할 수 있습니다.

## Q. 조인하는 컬럼에 인덱스를 걸었을 때와 걸지 않았을 때 차이가 있나요?

**A.**\
조인 조건 컬럼에 인덱스를 설정하면 쿼리 성능이 크게 향상됩니다.

* **인덱스 없을 때**:
  * 데이터베이스는 전체 행을 스캔(Nested Loop Join)하거나 비효율적인 방식으로 조인.
  * 대규모 데이터에서 성능 저하 심각(예: O(n²) 복잡도).
* **인덱스 있을 때**:
  * 데이터베이스 옵티마이저가 효율적인 조인 알고리즘(예: Hash Join, Merge Join)을 선택.
    * **Hash Join**: 한 테이블의 조인 키로 해시 테이블 생성, 다른 테이블을 빠르게 매칭. \
      인덱스 없어도 메모리 충분 시 유효.
    * **Merge Join**: 정렬된 테이블을 병합, 인덱스로 정렬 상태 활용 시 매우 빠름.
  * 검색 속도 향상(O(log n) 또는 O(n)), 특히 대규모 데이터에서 효과적.
*   **예시**:

    ```sql
    SELECT u.user_id, u.username, g.guild_name
    FROM users u JOIN guild_membership g ON u.user_id = g.user_id;
    ```

    `g.user_id`에 인덱스가 있으면 조인 시 빠르게 매칭, 없으면 전체 스캔.
* **결론**: 조인 컬럼(특히 외래키 또는 자주 사용되는 키)에 인덱스를 설정하면 \
  옵티마이저가 효율적인 실행 계획을 수립해 쿼리 성능을 최적화합니다. \
  단, 인덱스 유지 비용(쓰기 성능 약간 저하)을 고려해야 합니다.
