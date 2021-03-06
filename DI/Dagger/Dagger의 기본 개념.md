## 의존성 주입과 Dagger 라이브러리

### 의존성 주입

의존성 주입이란 **특정 객체의 인스턴스가 필요한 경우 이를 직접 생성하지 않고 외부에서 생성된 객체를 전달하는 기법**이다. 즉, 각 객체는 다른 객체의 생성에는 관여하지 않고, **객체를 필요로 하는 부분과 독립된 별도의 모듈이** 객체의 생성과 주입을 전담한다.

- **목적에 따라 동작을 변경하기 쉽다**. 필요한 객체를 외부에서 주입 받으므로, 상황에 따라 다른 동작을 하는 객체를 간편하게 생성할 수 있다.
- **생성한 객체를 쉽게 재사용할 수 있다.** 객체를 생성하는 작업을 특정 모듈에서 전담하므로, 객체를 생성하는 방법과 인스턴스를 효율적으로 관리할 수 있다.
- **객체를 생성하거나 사용할 때 발생하는 실수를 줄여준다**. 같은 역할을 하는 객체를 여러곳에서 생성하도록 코드를 작성한다면 생성하는 부분이 변경됐을때 작성한 곳을 찾아서 모두 변경해야 하지만, **의존성 주입을 사용하면 객체를 생성해주는 부분만 바꿔주면 된다.**
- Testable한 코드를 작성할 수 있다. 의존성 주입으로 인해 비지니스 로직에 집중하여 테스트 코드를 작성할 수 있다.



### 대거 라이브러리

스퀘어(square)에서 최초로 만든 라이브러리로, **자바 기반 프로젝트에서 의존성 주입**을 사용할 수 있도록 도와준다. 대거 라이브러리에서 **각 객체 간의 의존관계는** **어노테이션**으로 정의한다. 이처럼 의존 관계를 설정하고 검증하는 과정, 필요한 코드를 생성하는 과정이 모두 **빌드 단계**에서 일어나므로, 보다 견고한 어플리케이션을 만들 수 있다.

### 모듈

**모듈은 필요한 객체를 제공하는 역할을 한다.** 모듈은 클래스 단위로 구성되며 클래스 내에 **특정 객체를 반환하는 함수**를 정의하여 모듈에서 제공하는 객체를 정의한다.

- 모듈 클래스로 인식되게 하기 위해선 **@Module** 어노테이션을 붙여주고, 이 모듈에서 제공하는 객체를 정의하는 함수에는 **@Provider** 어노테이션을 붙인다. 
- 특정 객체를 생성할 때, 다른 객체가 필요한 경우 (**즉, 의존 관계에 있는 경우**) 객체를 생성하는 함수의 **매개변수로** 의존관계에 있는 객체를 추가한다. 

```kotlin
@Module
class BurgerModule {
  //햄버거 객체를 제공한다.
  //햄버거 객체를 만들기 위해 필요한 객체는 매개변수로 작성한다.
  @Provider
  fun provideBurger(bun: Bun, patty: Patty): Burfer{
    return Burger(bun, patty)
  }
  
  //빵 객체를 제공한다
  @Provider
  fun provideBun(): Bun {
    return WheatBun()
  }
  
  //패티 객체를 제공한다.
  @Provier
  fun providePatty(): Patty{
    return BeefPatty()
  }
}
```



### 컴포넌트 

**컴포넌트는 모듈에서 제공받은 객체를 조합하여 필요한곳에 주입(Inject)하는 역할을 한다.**

- 하나의 컴포넌트는 **여러 개의 모듈**을 조합할 수 있다.

- 목적에 따라 **분리된 여러 모듈로부터 필요한 객체를 받아 사용할 수 있다.**

- **@Component** 어노테이션을 붙인 **인터페이스** 로 선언하며, **@Component** 어노테이션의 **modules** **프로퍼티**를 통해 컴포넌트에 객체를 제공하는 모듈을 지정할 수 있다.

- 컴포넌트를 통해 **객체를 주입받을 대상은 함수의 파라미터로 선언하며, 값을 반환하지 않는다.** 

	```kotlin
	@Component(modules = arrayOf(BurgerModule::class))
	interface FastFoodComponent{
	  //store 클래스에 FastFoodComponent에서 제공하는 객체를 주입할 수 있도록 한다.
	  fun inject(store: Store)
	}
	```

- 컴포넌트와 모듈, 각 모듈에서 제공하는 객체 간의 의존관계를 그래프로 표시할 수 있으며, 이를 **객체 그래프** 라고 부른다.



### 사용하는 곳

- **컴포넌트**를 통해 주입받는 객체는 **컴포넌트가 값을 주입하는 시점**에 초기화되므로 **@Inject lateinit var 로 선언한다**

- 컴포넌트에서 객체를 주입받는 클래스를 정의한 후 프로젝트를 **빌드**하면 **대거는 객체를 주입할 때 사용할 수 있는 컴포넌트의 코드를 생성해준다.** 규칙은 Dagger{ComponentName}이다.

	```kotlin
	class Store{
	  //@Inject 어노테이션을 사용하여 컴포넌트로부터 객체를 주입받는 것으로 표시한다.
	  @Inject lateinit var burger: Burger
	  
	  init{
	    //FastFoodComponent 를 기반으로 DaggerFoodComponent클래스를 대거가 생성한다.
	    DaggerFastFoodComponent.builder()
	    		.burgerModule(BurgerModule()) //BurgerModule의 객체를 컴포넌트에 전달한다.
	    		.build()
	    		.inject(store = this)
	  }
	}
	```

- 컴포넌트와 모듈을 조합하다 보면, 컴포넌트에 포함된 모듈에서는 인스턴스를 생성하여 제공할 수 없지만, **특정 객체를 만들기 위해서는 필요한 객체가 있다.** 대표적으로 **Application, Acitivity** 가 있다. (**시스템**에서 인스턴스를 생성한다.)

- 이 경우 **컴포넌트 빌더 인터페이스**를 사용하면 컴포넌트에 추가로 전달할 객체를 간편하게 정의할 수 있다.

- 컴포넌트 빌더 인터페이스는 **@Component.Builder** 로 표시하며 생성할 컴포넌트를 반환하는 **builder()** 함수를 반드시 포함해야 한다.

- 컴포넌트에 추가로 전달할 객체를 지정하려면, 해당 객체를 인자로 받고 -> 빌더 클래스를 반환하는 함수를 정의한 후 -> 해당 함수에 **@BindsInstance 어노테이션**을 추가해야 한다. 

	```kotlin
	@Component(modules = arrayOf(BurgerModule::class))
	interface FastFoodComponent {
	  @Component.Builder 
	  interface Builder {
	    //Application객체를 객체 그래프에 포함시킨다.
	    @BindsInstance
	    fun application(app: Application): Builder
	    //FastFoodComponent를 반환하는 build함수를 반드시 선언해야 한다.
	    fun build(): FastFoodComponent
	  }
	  
	  //사용자 정의 애플리케이션 클래스 FoodApp에 이 컴포넌트에서 제공하는 객체를 주입할 수 있도록 한다.
	  fun inject(app: FoodApp)
	}
	
	
	/** 다음은 변경된 FastFoodComponent를 사용하는 예시이다 */
	class FoodApp: Application {
	  //FastFoodComponent를 통해 객체를 주입받는다.
	  @Inject lateinit var burger: Burger
	  
	  override fun onCreate() {
	    super.onCreate()
	    
	    DaggerFastFoodComponent.builder()
	    		.application(app = this)
	    		.build()
	    		.inject(app = this)
	  }
	}
	```



### 액티비티를 객체 그래프에 추가하기

- 필요한 객체를 자체적으로 생성하는 대신, **대거를 통해 생성된 객체를 주입 받으려면 액티비티 역시 객체 그래프에 추가해야 한다.**
- 특정 액티비티를 객체 그래프에 추가하려면 새로운 모듈을 생성한 후, 이 모듈에 객체 그래프에 추가할 액티비티를 정의하면 된다. 이 때 **@ContributesAndroidInjector 어노테이션**을 추가 선언한다. 
- 컴포넌트에서는 앞서 작성했던 모듈과 더불어 대거의 안드로이드 지원 모듈인 **AndroidSupportInjectionModule**을 컴포넌트 모듈로 추가하며, AndroidSupportInjectionMoudle을 사용하기 위해 컴포넌트는 **AndroidInjector 인터페이스를 상속한다.** 
- AndroidInjector 인터페이스의 타입 인자로 넣어줄때는 **DaggerApplication을 상속받는 클래스**를 넣어준다.
- DaggerApplication을 상속하면 **applicationInjector() 함수**를 구현해야 한다. 이를 구현하려면 작성한 컴포넌트의 구현체인 Dagger<컴포넌트> 를 사용해야 한다. 이는 대거가 생성해주는 클래스이므로 프로젝트를 한 번 빌드해주면 자동 생성이 된다. 
