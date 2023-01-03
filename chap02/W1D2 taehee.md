# Chapter 2 : 의존성 역전하기   
앞에서 본 계층형 아키텍처의 문제점들 -> 대안은?     

<br/>
  
- [단일 책임 원칙](#단일-책임-원칙)
- [의존성 역전 원칙 ](#의존성-역전-원칙)  
- [클린 아키텍처](#클린-아키텍처)   
- [육각형 아키텍처(헥사고날 아키텍처)](#육각형-아키텍처-헥사고날-아키텍처)
- [마무리](#마무리)   

<br/>
   
## 단일 책임 원칙
- SRP(Single Responsibility Principle)    
  - 많이 알려진 설명 : 하나의 컴포넌트는 오로의 한가지 일만 해야 하고, 그것을 올바르게 수행해야 한다. 
  - 실제 정의 : 컴포넌트를` 변경하는 이유는 오직 하나뿐`이어야 한다.  

- 아키텍처에서의 의미 : 어떠한 다른 이유로 소프트웨어를 변경하더라도 이 컴포넌트는 안 변한다.
  - SRP를 따르다보니 `변경할 이유`가 컴포넌트 간의 `의존성 통해 전파`됨  
  - ex) A가 B를 의존 
    -> B 변경 시 A도 변경해야함     
    -> B는 의존하고 있는것이 없으니, 변경될 이유는 새로운 요구사항에 의해 B의 기능을 바꿔야 할 때뿐   

<br/>

- 변경에 대한 부수효과는 무섭다  

<br/>

## 의존성 역전 원칙   
- DIP(Dependency Inversion Principle)   
  - `코드상의 어떤 의존성이든 그 방향을 바꿀 수(역전시킬 수) 있다.`
  - 의존성의 양쪽코드를 모두 제어할 수 있을 때만 가능     
    - ex) third party library는 제어 불가해 -> 의존성 역전 불가

- 계층형 아키텍처에서 계층간 의존성 : 항상 다음 계층(아래방향)을 가리킴   
  
  -> `상위계층`들이 하위계층들에 비해 `변경할 이유가 더 많게`됨    
  
  - ex) 영속성계층(하위) 변경 -> 잠재적으로 `도메인계층(상위)도 변경`해야 하게됨        
    그렇지만, 도메인코드는 가장 중요한 코드인데? 그러한 이유로 변경하고싶지 않은데, 의존성을 제거해야 하나?    
    
    - NO! `DIP를 적용`하자   

<br/>

**의존성을 역전해보자**            

현재 `domain계층이 영속성계층에 의존`중         

![제목 없음](https://user-images.githubusercontent.com/103614357/210204445-a3f82a73-eed9-4b8f-9d20-3c3f6606070b.png)     

- `Entity를 Domain계층으로` 올림(Entity는 domain객체를 표현하고 domain 코드는 이 Entity들의 상태를 변경하는 일을 중심으로 하기 때문)       
  -> 영속성계층의 repository가 domain계층의 Entity에 의존하게 됨     
  -> 순환 의존성이 생김      
  => DIP 적용하기      
  
  - `domain계층` - `repository에 대한 interface` 만들기
  - `영속성계층` - `실제 repository 구현`하기      
  
<br/>

![제목 없음](https://user-images.githubusercontent.com/103614357/210204505-6833bf02-e87c-43a2-93e5-d6be61357b28.png)   

=> domain계층에 interface 도입 -> 의존성 역전    
-> `영속성계층이 domain계층에 의존`하게 됨    

<br/>

## 클린 아키텍처
- 로버트 C. 마틴이 `클린 아키텍처` 책에서 정립한 용어를 참고해서 정리하면   
  - 클린 아키텍처 : `domain 코드`가 `바깥으로 향하는 어떤 의존성도 없어야`함 의미     
    (대신 DIP로 모든 의존성이 domain코드 향하고 있음)        

=> 클린아키텍처에서 모든 의존성은 도메인 로직을 향해 `안쪽방향으로(코어로)` 향함    

![제목 없음](https://user-images.githubusercontent.com/103614357/210205600-ceb000c9-6153-4183-9957-6d3c64820aec.png)   

- 코어 : `Domain Entity`  
  - `Use Case`(앞에서 서비스라 불렀던, 단일 책임 갖기위해 세분화 되어있음) -> 뚱뚱한 서비스 문제 피할 수 있음    
- 코어 주변 : 비즈니스 규칙을 지원(영속성 or UI 제공하는) component들   

<br/>

=> `domain 코드` 입장 : 어떤 영속성 framework or UI framework 사용되는지 모름    
-> 특정 framework와 별개로 `독립적인 코드` = 비즈니스 규칙에 집중 가능      

- but 단점!?      
  - domain 계층이 외부계층(영속성 or UI)과 분리되어있어   
    -> application의 Entity에 대한 모델을 `각 계층에서 유지보수해야` 함     
    = 각 계층에서 각각 Entity 만들고, 데이터 주고 받을 때 `Entity를 서로 변환`해야함     
    
    => 그러나, 이건 바람직한 일! domain 코드를 framework에서 독립적으로 결합이 제거된 상태이니까     

<br/>

- domain model이 특정 framework에 결합된 예 
  - JPA에서 ORM이 관리하는 Entity는 인자 없는 기본 생성자 필수 -> 이를 domain model에 적용 시 결합될 수 밖에 없음  

<br/>

## 육각형 아키텍처(헥사고날 아키텍처)   
모호한 클린아키텍처의 원칙들을 구체적으로 만들어줌  

![제목 없음](https://user-images.githubusercontent.com/103614357/210206179-9205410f-384e-46eb-870d-0475f558c30a.png)


- 육각형 안 : `Domain Entity`, `Use Case`
- 육각형 바깥 : application과 상호작용하는 `adapter`들(web, database adapter 등)   
- application core와 adapter 간의 통신이 가능하려면 -> core가 각각의 `포트를 제공`해야함 => 포트와 어댑터 아키텍처라고도 불림  

=> 육각형에서 외부로 향하는 의존성이 없음(마틴이 클린 아키텍처에서 제시한 의존성 규칙이 그대로 적용)    
모든 의존성은 코어를 향함  

<br/>

## 마무리   
- domain code가 어떤 component code도 의존하고 있지 않다면      
  
  -> domain code `변경될 이유의 수`가 줄어듬  
  -> `유지보수성` 높아짐 & `비즈니스 문제에만 집중`되게 짤 수 있음        

<br/>