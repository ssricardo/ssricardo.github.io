---
layout: page
title:  "Junit and Bean Validation"
date:   2017-11-06 18:00:00 -0200
categories: java junit validation
---

# Junit and Bean Validation

Sometimes you need to test something which uses *Bean Validation*. For instance, you are using JPA and your entities use Bean Validation. In this case, if you have any implementation of jsr-303 (like hibernate validation), it will run automatically on JPA calls.  

In the above case, the test can throw `javax.validation.ConstraintViolationException`.

## Handling JSR-303 Constraint Exceptions

When a *ConstraintViolationException* has been thrown in a test case, the test will fail (unless it's excepting that exception). In this case, Junit will show the exception (`javax.validation.ConstraintViolationException` in this case) and the stack trace.  
The problem is that in cases like this, the stack trace will not show intern data from this exception. I will print only the exception itself. Thus, you aren't able to get the actual validation problems. Without knowing the exact violations, you would need to try guessing it. 

### Junit Rule

A generic way to solve this, is using a Junit Rule that catches this specific exception e prints its violations.

Use `ConstraintViolationRule` for this goal.  As any Junit Rule, basically you need to add and annotate it in your test classes:  


```
    @Rule
    public ConstraintViolationRule constraintRule = ConstraintViolationRule.getInstance();
```

The above code will make *ConstraintViolationRule* catch and print the violations for tests in the same class.  If you prefer a different behavior instead of only printing it, just modify it in *ConstraintViolationRule* class.

This utility is available in [ConstraintViolationRule](https://github.com/ssricardo/lab/blob/master/java/ConstraintViolationRule.java).
