# WebArgumentResolver's

Spring `WebArgumentResolver` to resolve arguments

```
public ResponseEntity<ApiVoucherResponse> claimVoucher(@Valid @RequestBody ApiClaimVoucher apiClaimVoucher,
                                                       @AuthenticationPrincipal Authentication principal) throws URISyntaxException {
                                                       ...
}
```

The `PrincipalMethodArgumentResolver` should extract authentication and principal

```
package org.springframework.messaging.simp.annotation.support;

public class PrincipalMethodArgumentResolver implements HandlerMethodArgumentResolver {
```

How to list web argument resolvers?

## References

- https://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-creating-a-custom-handlermethodargumentresolver/
