---
title: "Spring에서 yaml파일 파싱하기"
last_modified_at: 2023-01-27
author: 이한솔
---

이번 포스팅에서는 Spring에서 yaml파일을 파싱하는 방법에 대해 알아보도록 하겠습니다.

---
> **@ConfigurationProperties**는 Spring에서 YAML, properties 파일에 정의된 프로퍼티들을 POJO에 매핑하여 Bean으로 만들 수 있게 해주는 어노테이션입니다.

<br>

## 사용방법
다음과 같은 yaml 파일을
```yaml
source-file:
    name: agent_log
    path: C:\apps\test\agent.log
```
prefix를 이용해 다음과 같이 POJO에 매핑하여 사용할 수 있습니다.
```java
@ConfigurationProperties(prefix = "source-file")
public class Property {
    private String name;
    private String path;
}
```
---
<br>

## 유연한 바인딩
프로퍼티 값을 객체에 바인딩 할 경우 필드를 Camel식으로 선언하고 키는 다양한 형식으로 선언하여 바인딩이 가능합니다.

예를 들어 다음과 같이 다양한 표기 방법으로 표현된 'classname'은
```yaml
component:
  className: com.epozen.acelletl.Filter
  classname: com.epozen.acelletl.Filter
  class_name: com.epozen.acelletl.Filter
  class-name: com.epozen.acelletl.Filter
  CLASS_NAME: com.epozen.acelletl.Filter
```
다음과 같이 카멜식인 'className'으로 모두 바인딩 가능합니다.
```java
@ConfigurationProperties(prefix = “component”)
public class Property {
    private String className;
}
```
---
<br>

## 형태별 바인딩
일반적인 바인딩의 경우 앞서 보여드린 예시처럼 구현하면 되고,<br>
List 형태의 경우는 다음과 같이 구현할 수 있습니다.
```yaml
component:  
  - class: com.epozen.acelletl.Filter
    from: agent_log
    name: alert_filter
  - class: com.epozen.acelletl.Appender
    from: alert_filter
    name: alert_appender
```
```java
@Getter
@Setter
@Component
@ConfigurationProperties
public class ComponentProperty {
	private List<Component> component;    
		
		@Getter    
		@Setter    
		public static class Component {        
			private String cLass;     
			private String from;      
			private String name;    
		}
	}
```

---

