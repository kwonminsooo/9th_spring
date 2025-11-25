공통

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface PageParam {
}

```

```java
public class InvalidPageException extends RuntimeException {
    public InvalidPageException(String message) {
        super(message);
    }
}

```

```java
@Component
public class PageParamArgumentResolver implements HandlerMethodArgumentResolver {

    private static final int DEFAULT_SIZE = 10;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(PageParam.class)
                && Pageable.class.isAssignableFrom(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) {

        String pageStr = webRequest.getParameter("page");
        int page = (pageStr == null) ? 1 : Integer.parseInt(pageStr);

        if (page <= 0) {
            throw new InvalidPageException("page 파라미터는 1 이상이어야 합니다. 입력값=" + page);
        }

        return PageRequest.of(page - 1, DEFAULT_SIZE);
    }
}
```

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final PageParamArgumentResolver pageParamArgumentResolver;

    public WebConfig(PageParamArgumentResolver resolver) {
        this.pageParamArgumentResolver = resolver;
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(pageParamArgumentResolver);
    }
}

```

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(InvalidPageException.class)
    public ResponseEntity<ApiErrorResponse> handleInvalidPage(InvalidPageException e) {
        ApiErrorResponse body = ApiErrorResponse.builder()
                .code("INVALID_PAGE")
                .message(e.getMessage())
                .build();
        return ResponseEntity.badRequest().body(body);
    }

}
```

```java
@Getter
@Builder
public class ApiErrorResponse {
    private String code;
    private String message;
}

```

공통 DTO

```java
@Getter
@Builder
public class MyReviewResponse {

    private Long reviewId;
    private Long storeId;
    private String storeName;
    private String content;
    private Double rating;
    private LocalDate createdDate;
}

```

```java
public enum MissionStatus {
    IN_PROGRESS,
    COMPLETED
}

```

```java
@Getter
@Builder
public class MissionResponse {

    private Long missionId;
    private Long storeId;
    private String storeName;
    private String title;
    private String description;
    private int rewardPoint;
    private MissionStatus status;
}

```

```java
@Component
public class ReviewConverter {

    public MyReviewResponse toMyReviewResponse(Review review) {
        return MyReviewResponse.builder()
                .reviewId(review.getId())
                .storeId(review.getStore().getId())
                .storeName(review.getStore().getName())
                .content(review.getContent())
                .rating((double) review.getRating())
                .createdDate(review.getCreatedAt().toLocalDate())
                .build();
    }

    public List<MyReviewResponse> toMyReviewResponses(List<Review> reviews) {
        return reviews.stream()
                .map(this::toMyReviewResponse)
                .toList();
    }
}

```

```java
@Component
public class MissionConverter {

    public MissionResponse toMissionResponse(Mission mission) {
        return MissionResponse.builder()
                .missionId(mission.getId())
                .storeId(mission.getStore().getId())
                .storeName(mission.getStore().getName())
                .title(mission.getTitle())
                .description(mission.getDescription())
                .rewardPoint(mission.getRewardPoint())
                .status(mission.getStatus())
                .build();
    }

    public List<MissionResponse> toMissionResponses(List<Mission> missions) {
        return missions.stream()
                .map(this::toMissionResponse)
                .toList();
    }
}

```

1.

```java
public interface ReviewRepository extends JpaRepository<Review, Long> {
    Page<Review> findByWriterId(Long memberId, Pageable pageable);
}
```

```java
@Service
@RequiredArgsConstructor
public class ReviewQueryService {

    private final ReviewRepository reviewRepository;
    private final ReviewConverter reviewConverter;

    private static final Long LOGIN_MEMBER_ID = 1L;

    public PageResponse<MyReviewResponse> getMyReviews(Pageable pageable) {
        Page<Review> page = reviewRepository.findByWriterId(LOGIN_MEMBER_ID, pageable);
        List<MyReviewResponse> content = reviewConverter.toMyReviewResponses(page.getContent());
        return PageResponse.<MyReviewResponse>builder()
                .content(content)
                .page(page.getNumber() + 1)
                .size(page.getSize())
                .totalElements(page.getTotalElements())
                .totalPages(page.getTotalPages())
                .build();
    }
}

```

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/reviews")
@Tag(name = "Review", description = "리뷰 관련 API")
public class ReviewController {

    private final ReviewQueryService reviewQueryService;

    @GetMapping("/me")
    @Operation(summary = "내가 작성한 리뷰 목록 조회",
               description = "로그인한 사용자가 작성한 리뷰를 10개씩 페이징 조회합니다.")
    public PageResponse<MyReviewResponse> getMyReviews(
            @Parameter(description = "1 이상의 페이지 번호", example = "1")
            @PageParam Pageable pageable) {

        return reviewQueryService.getMyReviews(pageable);
    }
```

2.

```java
public interface MissionRepository extends JpaRepository<Mission, Long> {

    Page<Mission> findByStoreId(Long storeId, Pageable pageable);
}

```

```java
@Service
@RequiredArgsConstructor
public class MissionQueryService {

    private final MissionRepository missionRepository;
    private final MissionConverter missionConverter;

    public PageResponse<MissionResponse> getStoreMissions(Long storeId, Pageable pageable) {
        Page<Mission> page = missionRepository.findByStoreId(storeId, pageable);
        List<MissionResponse> content = missionConverter.toMissionResponses(page.getContent());
        return PageResponse.<MissionResponse>builder()
                .content(content)
                .page(page.getNumber() + 1)
                .size(page.getSize())
                .totalElements(page.getTotalElements())
                .totalPages(page.getTotalPages())
                .build();
    }
}

```

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/stores")
@Tag(name = "StoreMission", description = "가게 미션 API")
public class StoreMissionController {

    private final MissionQueryService missionQueryService;

    @GetMapping("/{storeId}/missions")
    @Operation(summary = "가게의 미션 목록 조회",
               description = "특정 가게에 등록된 미션을 10개씩 페이징 조회합니다.")
    public PageResponse<MissionResponse> getStoreMissions(
            @Parameter(description = "가게 ID", example = "1")
            @PathVariable Long storeId,
            @Parameter(description = "1 이상의 페이지 번호", example = "1")
            @PageParam Pageable pageable) {

        return missionQueryService.getStoreMissions(storeId, pageable);
    }
}

```

3.

```java
public interface MissionRepository extends JpaRepository<Mission, Long> {

    Page<Mission> findByMemberIdAndStatus(Long memberId, MissionStatus status, Pageable pageable);
}
```

```java
@Service
@RequiredArgsConstructor
public class MyMissionService {

    private final MissionRepository missionRepository;
    private final MissionConverter missionConverter;

    private static final Long LOGIN_MEMBER_ID = 1L;

    public PageResponse<MissionResponse> getMyRunningMissions(Pageable pageable) {
        Page<Mission> page = missionRepository.findByMemberIdAndStatus(
                LOGIN_MEMBER_ID, MissionStatus.IN_PROGRESS, pageable);

        List<MissionResponse> content = missionConverter.toMissionResponses(page.getContent());
        return PageResponse.<MissionResponse>builder()
                .content(content)
                .page(page.getNumber() + 1)
                .size(page.getSize())
                .totalElements(page.getTotalElements())
                .totalPages(page.getTotalPages())
                .build();
    }
}
```

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/missions")
@Tag(name = "MyMission", description = "내 미션 API")
public class MyMissionController {

    private final MyMissionService myMissionService;

    @GetMapping("/me/running")
    @Operation(summary = "진행중인 내 미션 목록 조회",
               description = "로그인한 사용자의 진행중(IN_PROGRESS) 미션을 페이징 조회합니다.")
    public PageResponse<MissionResponse> getMyRunningMissions(
            @Parameter(description = "1 이상의 페이지 번호", example = "1")
            @PageParam Pageable pageable) {

        return myMissionService.getMyRunningMissions(pageable);
    }
}

```