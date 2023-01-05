### Stream流根据对象的某个字段去重
#### 方法一 通过 Collectors.toCollection
```java
testList
  .stream()
  .collect(
  Collectors.toCollection(
    () -> new TreeSet<>(
      Comparator.comparing(tc -> tc.getName())
    )
  )
);
```
#### 方法二 通过collectingAndThen+toCollection
```java
testList
  .stream()
  .collect(
  Collectors.collectingAndThen(
    Collectors.toCollection(
      () -> new TreeSet<>(
        Comparator.comparing(tc -> tc.getName())
      )
    ),ArrayList::new
  )
);
```
