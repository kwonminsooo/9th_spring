POST /api/reviews

Request Body

```java
{
  "userMissionId": "1",
  "storeId": "20",
  "rating": 5,
  "content": "음식이 맛있어요"
}
```

DTO

```java
public record ReviewCreateRequest(
        Long userMissionId,
        Long storeId,
        int rating,
        String content
) {}
```

Entity

UserMission

```java
@Entity
public class UserMission {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Mission mission;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member user;

    private boolean completed;

}
```

Review

```java
@Entity
public class Review {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member writer;

    @ManyToOne(fetch = FetchType.LAZY)
    private Store store;

    @ManyToOne(fetch = FetchType.LAZY)
    private UserMission userMission;

    private int rating;
    private String content;
    private LocalDateTime createdAt = LocalDateTime.now();
}

```

Repository

```java
public interface ReviewRepository extends JpaRepository<Review, Long> {}
public interface StoreRepository extends JpaRepository<Store, Long> {}
public interface MissionRepository extends JpaRepository<Mission, Long> {}
public interface UserMissionRepository extends JpaRepository<UserMission, Long> {}
public interface UserRepository extends JpaRepository<Member, Long> {}

```

Service

StoreReview

```java
@Service
@RequiredArgsConstructor
public class ReviewService {

    private final UserRepository userRepository;
    private final StoreRepository storeRepository;
    private final UserMissionRepository userMissionRepository;
    private final ReviewRepository reviewRepository;

    private static final Long USER_ID = 1;

    public Long createReview(ReviewCreateRequest req) {

        Member user = userRepository.findById(USER_ID)
                .orElseThrow(() -> new IllegalArgumentException("User Not Found"));

        Store store = storeRepository.findById(req.storeId())
                .orElseThrow(() -> new IllegalArgumentException("Store Not Found"));

        UserMission userMission = userMissionRepository.findById(req.userMissionId())
                .orElseThrow(() -> new IllegalArgumentException("UserMission Not Found"));

        Review review = new Review();
        review.setWriter(user);
        review.setStore(store);
        review.setUserMission(userMission);
        review.setRating(req.rating());
        review.setContent(req.content());

        Review saved = reviewRepository.save(review);

        return saved.getId();
    }
}

```

UserMission

```java
@Service
@RequiredArgsConstructor
public class MissionChallengeService {

    private final MissionRepository missionRepository;
    private final UserMissionRepository userMissionRepository;
    private final UserRepository userRepository;

    private static final Long USER_ID = 1;

    public Long challengeMission(Long missionId) {

        Member user = userRepository.findById(USER_ID)
                .orElseThrow(() -> new IllegalArgumentException("User Not Found"));

        Mission mission = missionRepository.findById(missionId)
                .orElseThrow(() -> new IllegalArgumentException("Mission Not Found"));

        UserMission userMission = new UserMission();
        userMission.setMission(mission);
        userMission.setUser(user);
        userMission.setCompleted(false);

        UserMission saved = userMissionRepository.save(userMission);
        return saved.getId();
    }
```

Controller

Review

```java
@RestController
@RequestMapping("/api/reviews")
@RequiredArgsConstructor
public class ReviewController {

    private final ReviewService reviewService;

    @PostMapping
    public ResponseEntity<?> createReview(@RequestBody ReviewCreateRequest req) {
        Long reviewId = reviewService.createReview(req);
        return ResponseEntity.ok("리뷰 등록 성공! id = " + reviewId);
    }
}
```

Mission

```java
@RestController
@RequestMapping("/api/missions")
@RequiredArgsConstructor
public class MissionChallengeController {

    private final MissionChallengeService missionChallengeService;

    @PostMapping("/{missionId}/challenge")
    public ResponseEntity<?> challenge(@PathVariable Long missionId) {
        Long id = missionChallengeService.challengeMission(missionId);
        return ResponseEntity.ok("미션 도전 시작! userMissionId = " + id);
    }
}

```