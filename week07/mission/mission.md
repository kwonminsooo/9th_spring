ApiResponse

```java
package com.example.umc9th.global.apiPayload;

import com.example.umc9th.global.apiPayload.code.BaseErrorCode;
import com.example.umc9th.global.apiPayload.code.BaseSuccessCode;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ApiResponse<T> {

    private boolean isSuccess;
    private String code;
    private String message;
    private T result;

    public static <T> ApiResponse<T> onSuccess(BaseSuccessCode code, T result) {
        return ApiResponse.<T>builder()
                .isSuccess(true)
                .code(code.getCode())
                .message(code.getMessage())
                .result(result)
                .build();
    }

    public static <T> ApiResponse<T> onFailure(BaseErrorCode code, T result) {
        return ApiResponse.<T>builder()
                .isSuccess(false)
                .code(code.getCode())
                .message(code.getMessage())
                .result(result)
                .build();
    }
}
```

BaseSuccessCode

```java
package com.example.umc9th.global.apiPayload.code;

public interface BaseSuccessCode {

    String getCode();
    String getMessage();
}
```

GeneralSuccessCode

```java
package com.example.umc9th.global.apiPayload.code;

import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public enum GeneralSuccessCode implements BaseSuccessCode {

    OK("COMMON200", "성공적으로 요청을 처리했습니다."),
    ;

    private final String code;
    private final String message;
}
```

```java
@RestController
@RequestMapping("/stores")
@RequiredArgsConstructor
public class StoreController {

    private final StoreService storeService;

 
    @GetMapping("/{storeId}")
    public ApiResponse<StoreResDTO.Detail> getStore(@PathVariable Long storeId) {

        StoreResDTO.Detail dto = storeService.getStoreDetail(storeId);

        return ApiResponse.onSuccess(
                GeneralSuccessCode.OK,
                dto
        );
    }

    @PostMapping
    public ApiResponse<StoreResDTO.Create> createStore(@RequestBody StoreReqDTO.Create request) {

        StoreResDTO.Create dto = storeService.createStore(request);

        return ApiResponse.onSuccess(
                GeneralSuccessCode.OK,
                dto
        );
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
    public ApiResponse<Page<MyReviewDto>> getMyReviews(
            @AuthenticationPrincipal CustomUser user,
            @RequestParam(required = false) Long storeId,
            @RequestParam(required = false) Integer rating,
            Pageable pageable
    ) {
        Long memberId = user.getId();

        Page<MyReviewDto> page = reviewQueryService.getMyReviews(memberId, storeId, rating, pageable);

        return ApiResponse.onSuccess(
                GeneralSuccessCode.OK,
                page
        );
    }
}
```