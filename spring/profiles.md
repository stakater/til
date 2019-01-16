## Is it possible to negate (!) a colleaction of spring profiles?

**Problem**
Is it possible to configure a bean in such a way that it wont be used by a group of profiles?
```
@Profile({"!" + SpringProfiles.SPRING_PROFILE_TEST, "!" + SpringProfiles.SPRING_PROFILE_LOCAL_TEST"})

or

@Profile("!dev, !qa, !local")

```

**Solution**

Source: https://stackoverflow.com/questions/43168881/can-i-negate-a-collection-of-spring-profiles

- It isn't possible but there is a workaround using
```

 @Conditional annotation. 

 ```
- Create conditional matchers

```
public abstract class ProfileCondition extends SpringBootCondition {
    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        if (matchProfiles(conditionContext.getEnvironment())) {
            return ConditionOutcome.match("A local profile has been found.");
        }
        return ConditionOutcome.noMatch("No local profiles found.");
    }

    protected abstract boolean matchProfiles(final Environment environment);
}


public class TestProfileCondition extends ProfileCondition {
    public boolean matchProfiles(final Environment environment) {
        return Arrays.stream(environment.getActiveProfiles()).anyMatch(prof -> {
            if(prof.equals(SpringProfiles.SPRING_PROFILE_TEST) || prof.equals(SpringProfiles.SPRING_PROFILE_LOCAL_TEST)){
                return false;
            }
            return true;
        });
    }
}

```

- In order to use it 
```
@Conditional(value = {TestProfileCondition.class})
public class MockImpl implements MyInterface {...}
```
