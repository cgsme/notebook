# javax.validation 注解

## @Constraint

## @GroupSequence

## @OverrideAttribute

## @ReportAsSingleViolation

## @Valid

## 1. constraints 包

### 1.1 @AssertFalse

> 被注解的元素必须为false。

#### 1.1.1 @AssertFalse.List(AssertFalse[] value)

> 在相同元素上定义多个AssertFalse注解（不懂什么用处）

### 1.2 @AssertTrue

> 被注解的元素必须为true

#### 1.2.1 @AssertTrue.List(AssertTrue[] value)

> 在相同元素上定义多个AssertTrue注解（不懂什么用处）

### 1.3 @DecimalMax(String value)

> 被注解的元素必须为数字，且元素值必须小于等于指定的value值

### 1.4 @DecimalMin

> > 被注解的元素必须为数字，且元素值必须大于等于指定的value值

### 1.5 @Digits

> 被注解的元素必须为数字，且该数字在给定的范围内

### 1.6 @Email

> 被注解的元素必须为邮箱格式

### 1.7 @Future

> **未来**的某个**日期（date）**或**时间（time）**

### 1.8 @FutureOrPresent

> **当下**或这**未来**的某个**日期（date）**或**时间（time）**

### 1.9 @Max

### 1.10 @Min

### 1.11 @Negative

### 1.12 @NegativeOrZero

### 1.13 @NotBlank

### 1.14 @NotEmpty

### 1.15 @NotNull

> 被注解的元素不能为null

### 1.16 @Null

> 被注解的元素必须为null

### 1.17 @Past

### 1.18 @PastOrPresent

### 1.19 @Pattern(String regex)

> 被注解的字符序列必须匹配给定的正则表达式。

### 1.20 @Positive

> 被注解的元素必须是正数

### 1.21 @PositiveOrZero

> 被注解的元素必须是正数或者0

### 1.22 @Size

## 2. constrainvalidataion 包

### 2.1 @SupportedValidationTarget

## 3. executable

### 3.1 @ValidateOnExecution

## 4. groups 包

### 4.1 @ConvertGroup

## 5. valueextraction 包

### 5.1 @ExtractedValue

### 5.2 @UnwrapByDefault
