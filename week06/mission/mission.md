```java
@Entity
public class Review {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member writer;

    @ManyToOne(fetch = FetchType.LAZY)
    private Store store;

    private int rating;
    private String content;
    private LocalDateTime createdAt;
}

```

```java
public record MyReviewDto(
        Long reviewId,
        Long storeId,
        String storeName,
        int rating,
        String content,
        LocalDateTime createdAt
) {}

```

```java
public interface ReviewRepositoryCustom {
    Page<MyReviewDto> findMyReviews(Long memberId, Long storeId, Integer rating, Pageable pageable);
}

```

```java
@RequiredArgsConstructor
public class ReviewRepositoryImpl implements ReviewRepositoryCustom {

    private final JPAQueryFactory queryFactory;
    private final QReview review = QReview.review;
    private final QStore store = QStore.store;

    @Override
    public Page<MyReviewDto> findMyReviews(Long memberId, Long storeId, Integer rating, Pageable pageable) {

      
        BooleanBuilder builder = new BooleanBuilder();
        builder.and(review.writer.id.eq(memberId)); 

        if (storeId != null) {
            builder.and(review.store.id.eq(storeId));
        }
        if (rating != null) {
            builder.and(review.rating.eq(rating));
        }

   
        List<MyReviewDto> content = queryFactory
                .select(Projections.constructor(MyReviewDto.class,
                        review.id,
                        review.store.id,
                        review.store.name,
                        review.rating,
                        review.content,
                        review.createdAt
                ))
                .from(review)
                .leftJoin(review.store, store)
                .where(builder)
                .orderBy(review.createdAt.desc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

    
        long total = queryFactory
                .select(review.count())
                .from(review)
                .where(builder)
                .fetchOne();

        return new PageImpl<>(content, pageable, total);
    }
}

```

```java
public interface ReviewRepository extends JpaRepository<Review, Long>, ReviewRepositoryCustom {
}

```

```java
@Service
@RequiredArgsConstructor
public class ReviewQueryService {

    private final ReviewRepository reviewRepository;

    public Page<MyReviewDto> getMyReviews(Long memberId, Long storeId, Integer rating, Pageable pageable) {
        return reviewRepository.findMyReviews(memberId, storeId, rating, pageable);
    }
}

```

```java
@RestController
@RequestMapping("/api/my-reviews")
@RequiredArgsConstructor
public class MyReviewController {

    private final ReviewQueryService reviewQueryService;

    @GetMapping
    public Page<MyReviewDto> getMyReviews(
            @AuthenticationPrincipal CustomUser user, 
            @RequestParam(required = false) Long storeId,
            @RequestParam(required = false) Integer rating,
            Pageable pageable
    ) {
        Long memberId = user.getId();
        return reviewQueryService.getMyReviews(memberId, storeId, rating, pageable);
    }
}

```