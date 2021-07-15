---
layout: customPost
title:  "MapStruct"
categories: 
  - Spring
  - MapStruct
---
- dto -> entity 또는 entity -> dto로 변환할 때 쓰이는 라이브러리.
- Mapper Interface를 구현하고 @Mapper를 Annotated하면 빌드시점에서 자동으로 implementation Class를 생성해줍니다. (intellij 기준으로 src/main/generated 아래 interface와 동일한 구조로 생성됨)

- Mapper Interface를 구현할 때 entity와 dto의 property name(변수명)이 다른 경우에는 @Mapping 애노테이션을 사용할 수 있고, 같으면 암묵적(자동)으로 맵핑 됩니다.

- @Mapping 애노테이션으로 지정된 property는 JavaBeans spectification에 만족해야 됩니다.

- 여러 Mapper Interface에서 반복되는 @Mapping은 Meta Annotation을 정의하여 사용할 수도 있습니다.

  ```java
  @Retention(RetentionPolicy.CLASS)
  @Mapping(target = "id", ignore = true)
  @Mapping(target = "creationDate", expression = "java(new java.util.Date())")
  @Mapping(target = "name", source = "groupName")
  public @interface ToEntity { }
  ```

  

- CDI(Context Denpency Injection)와 같은 종속성 주입 프레임워크(Spring framework 등)로 작업 주인 경우 DI를 통해 개체를 가져오는 것이 좋습니다. @Mapper 애노테이션의 componentModel 속성값을 사용할 수 있습니다.
  - default

  ```java
  @Mapper
  public interface PostalMapper {
  	PostalMapper INSTANCE = Mappers.getMapper(PostalMapper.class);
      
      D toDto(E e);
  }
  
  PostalMapper.INSTANCE.toDto(e);
  ```

  - cdi

  ```java
  @Mapper(componentModel = "cdi")
  public interface PostalMapper {
  	//...
  }
  
  @Inject
  ```

  - spring

  ```java
  @Mapper(componentModel = "spring")
  public interface PostalMapper {
  	//...
  }
  
  @Autowired
  ```

  - jsr330

  ```java
  @Mapper(componentModel = "jsr330")
  public interface PostalMapper {
  	//...
  }
  
  @Singleton
  ```




- Usage

  GenericMapper.java

  ```java
  public interface GenericMapper<D, E> {
      D toDto(E entity);
  
      /**
       * toEntity 메서드를 default로 정의한 이유는 Mapper의 구현체가 자동으로 생성될 때
       * toEntity가 추상 메서드일 경우 Entity의 기본 생성자를 통해 new하도록 되어있습니다. 
       * 만약 Entity에서 @NoArgsConstructor(access = AccessLevel.PROTECTED)를 사용하게 되면
       * MapperImpl은 Entity와 상속관계도 같은 패키지도 아니므로 오류가 발생합니다. 
       * 따라서 Entity의 생성자를 Protected로 설정하고 {@code dto -> entity}로 변환하려고 할 때는
       * entity 클래스에 @Builder를 작성하거나 해당 Mapper에서 toEntity 메서드를 오버라이드하고
       * Entity Factory 메서드를 통해 객체를 리턴하도록 해야 합니다.
       * @param dto dto 객체
       * @return 기본값은 null이며, 각 Mapper에서 Override하여 객체를 리턴함.
       */
      default E toEntity(D dto) {
          return null;
      }
  }
  ```

  PostalMaper.java
  
  ```java
  @Mapper(componentModel = "spring"
          , injectionStrategy = InjectionStrategy.CONSTRUCTOR
          , unmappedTargetPolicy = ReportingPolicy.IGNORE)
  public interface PostalMapper extends GenericMapper<PostalDto, PostalEntity> {
  
  }
  ```

  Service.java
  
  ```java
  	@Transactional(readOnly = true)
      public Page<PostalDto> findAllPostalHistory(PostalSearchDto searchDto) {
  
          return this.postalRepository.findAllPostalHistory(
                  searchDto.getPageRequest("saleSeq"),
                  searchDto)
                  .map(postalMapper::toDto);
      }
  ```
  
  