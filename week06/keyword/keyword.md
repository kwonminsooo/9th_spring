- QueryDSL에서 FetchJoin 하는 법

  사용방법

    ```json
    QMember m = QMember.member;
    QTeam t   = QTeam.team;
    
    List<Member> result = queryFactory
        .selectFrom(m)
        .join(m.team, t).fetchJoin()
        .where(m.status.eq(ACTIVE))
        .fetch();
    ```

  join(연관경로, 별칭).fetchJoin() 또는 leftJoin.fetchJoin()사용

- DTO 매핑 방식 (+DTO안에 DTO)

  1.Projections.constructor(생성자 매핑)

    ```json
    public record MemberDto(Long id, String name, String teamName) {}
    ```

  2.Projections.fields(필드명 매핑)

    ```json
    public class MemberDto {
      public Long id;
      public String name;
      public String teamName;
    }
    
    List<MemberDto> dtos = queryFactory
      .select(Projections.fields(MemberDto.class,
          m.id.as("id"),
          m.name.as("name"),
          t.name.as("teamName")))
      .from(m).leftJoin(m.team, t)
      .fetch();
    ```

  3.Projections.bean(setter 매핑)

    ```json
    public class MemberDto {
      private Long id; private String name; private String teamName;
    }
    var dtos = queryFactory
      .select(Projections.bean(MemberDto.class,
          m.id.as("id"),
          m.name.as("name"),
          t.name.as("teamName")))
      .from(m).leftJoin(m.team, t)
      .fetch();
    ```

  4.@QueryProjection(QDTO)

    ```json
    @Getter
    public class MemberDto {
      private final Long id;
      private final String name;
      private final String teamName;
    
      @QueryProjection
      public MemberDto(Long id, String name, String teamName) {
        this.id = id; this.name = name; this.teamName = teamName;
      }
    }
    // 쿼리
    var dtos = queryFactory
      .select(new QMemberDto(m.id, m.name, t.name))
      .from(m).leftJoin(m.team, t)
      .fetch();
    ```

- 커스텀 페이지네이션

  1.기본형(page)

  offset,limit으로 데이터를 가져오고 select(count())는 조인을 줄여서 따로 한번 더 날리는 방식

  2.Slice 방식

  limit(pageSize+1) 로 한 건 더 가져온후에 다음 있음/없음을 판단하는 방식

  3.커서 페이지네이션

  where(id<마지막 아이디) 와 같은 방식으로 다음페이지를 찾음

- transform - groupBy

  Post 1개에 comment 여러개 붙어 있는 것을 한번의 쿼리로 DTO에 담고 싶을때 사용하는 방식

    ```json
    import static com.querydsl.core.group.GroupBy.*;
    
    Map<Long, PostDto> result = queryFactory
        .from(post)
        .leftJoin(post.comments, comment)
        .transform(
            groupBy(post.id).as(
                Projections.constructor(PostDto.class,
                    post.id,
                    post.title,
                    list(Projections.constructor(CommentDto.class,
                        comment.id,
                        comment.content))
                )
            )
        );
    
    ```

  groupBy(post.id): 이 키를 기준으로 묶음

  as : 어떻게 DTO를 만들지 정의

- order by null

  보통의 DB의 경우 ORDER BY가 있으면 정렬용 작업을 하는데 그룹만 하고 결과를 보내고 싶을 경우는 정렬이 필요없기에 이경우에 order by null을 사용 이는 정렬 단계를 스킵해도 된다는 것을 DB에 알려주는 역할