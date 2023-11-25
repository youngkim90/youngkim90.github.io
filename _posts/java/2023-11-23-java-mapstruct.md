---
title: MapStruct를 활용한 엔티티, DTO 매핑
date: 2022-08-20 00:00:00 +0900
categories: [ Programming, Java ]
tags: [ java, mapstruct, mapper ]
image:
  path: https://www.devopsschool.com/blog/wp-content/uploads/2022/03/java_logo_icon_168609.png
  content: false
---

**MapStruct**는 자바 언어를 활용한 코드 생성 기반의 객체 매핑용 라이브러리이다.  
어노테이션 등을 활용해 객체 간의 매핑 작업을 편리하게 수행할 수 있도록 지원하고, 주로 **DTO**(Data Transfer Object)와 **엔티티**(Entity) 등의 객체를 변환하기 위한 용도로
사용된다.  
반복적이고 번거로운 매핑 코드를 빌드 시 자동으로 생성하여 개발자가 코드를 간결하게 작성할 수 있도록 도움을 주기 때문에 자주 사용된다.

## 1. 환경설정

mapstrut을 사용하기 위해서는 다음과 같은 의존성을 추가해야 한다.

Gradle

``` gradle
// lombok
implementation 'org.projectlombok:lombok:1.18.22'
annotationProcessor 'org.projectlombok:lombok:1.18.22'
annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'
// mapstruct
implementation 'org.mapstruct:mapstruct:1.4.2.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
```

Maven

``` maven
<org.mapstruct.version>1.5.5.Final</org.mapstruct.version>
// mapstruct
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct</artifactId>
  <version>${org.mapstruct.version}</version>
</dependency>

// plugin/annotationProcessorPaths
<annotationProcessorPaths>
  <path>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>${org.mapstruct.version}</version>
  </path>
  <path>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok-mapstruct-binding</artifactId>
    <version>0.2.0</version>
  </path>
</annotationProcessorPaths>
```

## 2. 매핑 DTO 및 Mapper 생성

``` java
@Builder
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private int id;

  @Column
  @Comment("고객명")
  private String name;

  @Column
  @Comment("고객 이메일")
  private String email;
}

@Builder
@Getter
@AllArgsConstructor
public class MemberRequestDTO {

  private String name;

  private String email;
}

@Builder
@AllArgsConstructor
public class MemberResultDTO {

  private int id;
  
  private String name;

  private String email;
}
```

예시를 위해 `Member`엔티티 와 엔티티가 변환할 or 변환될 DTO 클래스를 생성하였다.  
이제 **mapstruct**를 사용하여 엔티티와 DTO를 매핑시켜 줄 Mapper 인터페이스를 생성해보자.

``` java
@Mapper(
  unmappedTargetPolicy = ReportingPolicy.IGNORE, // 매핑되지 않은 필드 무시
  nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE // null 값 무시
)
public interface MemberMapper {
  MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);
  
  Member toMember(MemberRequestDTO request);

  MemberResultDTO toMemberResultDTO(Member member);
}
```

위와 같이 `MemberMapper` 인터페이스에 `@Mapper` 어노테이션을 추가해주면 **compile** 시에 엔티티 <-> DTO를 매핑시켜주는 코드가 **annotationPath**(
target/generated-sources/annotations/..) 하위에 생성된다.
`@Mapper` 어노테이션에는 다양한 옵션을 설정할 수 있는데, 자세한 내용은
[mapstruct 공식문서](https://mapstruct.org/documentation/stable/reference/html/#mapper-configuration)를 참고하면 된다.
기본적으로 매핑되지 않은 즉, 엔티티와 DTO 간의 이름이 다른 필드들은 매핑하지 않는 설정과 필드 값이 null 인 필드는 매핑하지 않는 설정만 추가하였다.

이제 `MemberMapper`를 사용하여 **MemberRequestDTO -> Member**, **Member -> MemberResultDTO** 객체로의 매핑을 시켜보자.

## 3. Mapper를 사용한 매핑

위에서 `MamberMapper` 인터페이스를 잘 생성했다면 compile 후에 annotationPath에 `MamberMapperImpl` 자바파일이 생성되었을 것이다.

``` java
public class MemberMapperImpl implements MemberMapper {

    @Override
    public Member toMember(MemberRequestDTO request) {
        if ( request == null ) {
            return null;
        }

        Member.MemberBuilder member = Member.builder();

        member.name( request.getName() );
        member.email( request.getEmail() );

        return member.build();
    }

    @Override
    public MemberResultDTO toMemberResultDTO(Member member) {
        if ( member == null ) {
            return null;
        }

        MemberResultDTO.MemberResultDTOBuilder memberResultDTO = MemberResultDTO.builder();

        memberResultDTO.id( member.getId() );
        memberResultDTO.name( member.getName() );
        memberResultDTO.email( member.getEmail() );

        return memberResultDTO.build();
    }
}
```

이처럼 mapstruct 기능을 활용하면 위와 같이 엔티티와 DTO를 매핑시켜주는 코드를 자동으로 생성할 수 있다.  
이제 `MemberMapper`를 사용하여 엔티티와 DTO를 매핑시켜보자.

``` java
  public MemberResultDTO getMember(MemberRequestDTO request) {
    MemberMapper mapper = MemberMapper.INSTANCE; // mapper 인스턴스 생성
    Member member = memberRepository.save(mapper.toMember(request));
    
    return mapper.toMemberResultDTO(member);
  }
```

변환이 필요한 곳에서 `MemberMapper` 인터페이스를 호출하여 **MemberRequestDTO -> Member**로 생성한 후에 jpa로 insert 후에 **Member ->
MemberResultDTO**로 변환하여 반환하는 작업이 간단하게 수행된다.  
이 후로는 필요 시 Mapper 인터페이스와 변환할 메소드만 정의해주어 사용하면 되므로 코드량도 줄일 수 있고 매핑도 간단해진다.

물론 mapstruct를 사용하지 않고 jackson 라이브러리의 **ObjectMapper**를 사용하여 변환할 수도 있지만, mapstruct에 비해 코드가 깔끔하지 않고,
객체간 변환이 많아질 수록 관리가 복잡해질 수 있다. 예시 코드처럼 엔티티와 요청/응답DTO 객체간 매핑이 많고 반드시 변환이 필요한 경우
엔티티 별로 Mapper 인터페이스를 생성하고 필요한 변환 메소드를 정의하여 사용하면 추후 사용 및 관리적인 측면에서도 유용하게 사용할 수 있다.
