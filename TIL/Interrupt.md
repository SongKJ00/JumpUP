### 1. 인터럽트의 정의

-	특정한 사건(event)이 일어나면 CPU에게 알려 정해진 처리를 수행하도록 하는 것
-	CPU는 인터럽트 처리 루틴을 수행하고 기존에 수행하던 일을 계속하게 된다.<br>

### 2. 인터럽트 서비스 루틴

#### 2-1. 인터럽트 서비스 루틴이란?

-	인터럽트 발생 시 발생된 인터럽트에 대응하는 특정 기능을 처리하기 위해 미리 정해놓은 코드 루틴

#### 2-2. 인터럽트 서비스 루틴 점프방식

-	인터럽트가 발생하고 해당 인터럽트 소스의 위치를 알기 위한 절차

	-	\***벡터**라는 숫자가 CPU 코어에 보내지면 해당 소스를 찾아낼 수 있다.
	-	**벡터** : 발생한 인터럽트를 처리할 수 있는 서비스 루틴의 주소를 가지고 있는 공간

##### 2-2-1. 무조건 점프 방식

정해진 주소값으로 무조건 점프한다. 정해진 메모리 위치에 ISR 코드가 존재해야 한다.

##### 2-2-2. 인터럽트 벡터 테이블 참조 방식

\***인터럽트 벡터 테이블**에서 주소값을 얻어 점프한다.<br>* **인터럽트 벡터 테이블** : 인터럽트 벡터 값들이 모여있는 테이블

다음 내용은 [유명한 소장님의 글](http://avr.c1o.kr/)을 인용하여 정리한 것입니다.
### 3. 인터럽트의 종류

#### 3-1. 소프트웨어 인터럽트(내부 인터럽트)

##### 3-1-1. 소프트웨어 인터럽트란?

CPU 내부에서 발생한 소프트웨어 요청에 따라 발생되는 인터럽트

##### 3-1-2. AVR의 소프트웨어 인터럽트 - 타이머/카운터

-	Prescaler를 거친 클럭의 개수를 정해진 개수만큼 카운트 완료하면 CPU Core에 인터럽트를 보내는 방식
-	인터럽트가 CPU 내부에서 발생되기 때문에 내부 인터럽트라 한다.

##### 3-1-3. 리눅스의 소프트웨어 인터럽트

리눅스는 User Application이 동작하는 **User Level**과 OS가 동작하는 **Kernel Level**이 있다. 하지만 리눅스에서는 User Level과 Kernel Level은 서로 다른 CPU 동작 모드로 실행되기 때문에 User Level의 CPU 동작 모드에선 Kernel(OS)이 동작되는 메모리 영역으로 접근할 수가 없다.<br> 즉, User Application에서 직접 OS 내부의 디바이스 드라이버나 OS 내부 서비스 함수를 직접 사용 및 호출할 수가 없다. 하지만 인터럽트라는 방식으로 그것이 가능하다.<br> User Application에서 OS 내부로 해당 함수를 호출하는 과정에서 소프트웨어 인터럽트가 User Application가 호출한 시스템 함수를 Kernel에서 실행 가능하도록 한다.<br> 예를 들어, 코드 내에서 **open()** 이란 시스템 함수를 사용할 때 소프트웨어 인터럽트가 발생하게 되고 이때 CPU는 인터럽트 벡터 테이블에서 open() 호출에 관련된 인터럽트 벡터를 찾게 되고 인터럽트 벡터에 저장된 ISR 주소를 찾아가 시스템 콜 차원에서 sys_open() 함수를 실행하게 된다.<br> 따라서, 리눅스에서의 소프트웨어 인터럽트는 User Application이란 소프트웨어의 요청으로 발생된 인터럽트이기 때문에 내부 인터럽트라 말할 수 있다.<br><br>

#### 3-2. 하드웨어 인터럽트(외부 인터럽트)

##### 3-2-1. 하드웨어 인터럽트란?

CPU 외부에 있는 외부 디바이스(하드웨어)의 요청에 의해 발생되는 인터럽트.

##### 3-2-2. 폴링 방식

-	현재 외부 디바이스(하드웨어)를 주기적으로 관찰하며 원하는 신호 발생 시 미리 정해놓은 코드를 실행하는 방식

##### 3-2-3. 인터럽트 방식

-	평소에는 본래 태스크를 실행시키다가 하드웨어적으로 CPU 외부에서 특정 신호가 발생하면 ISR을 실행하는 방식

##### 3-2-4. 폴링 방식과 인터럽트 방식의 차이점

가장 중요한 차이점은 폴링 방식은 묶여있지만 인터럽트 방식은 묶여있지 않다는 것이다. 사실 이 차이점이 다라고 말할 수 있다. 폴링 방식은 절차적으로 프로그램을 실행하면서 폴링 방식의 코드를 만나면 그때그때마다 외부 디바이스에서 특정 신호를 보내왔는지 확인하게 된다. AVR에서는 주로 이런 코드를 폴링 방식이라 한다.<pre>while(PIND & 0xFF);</pre> 이 코드는 D 포트에 한 핀이라도 0이 들어오게 되면 다음 코드로 진행하게 되는 코드이다. while이 아닌 if else, switch case로 된 폴링 방식 코드도 있겠지만, 어찌 되었건 폴링 방식 코드가 나타날 때마다 외부 디바이스로부터 신호를 확인한다는 것이다. 이는 곧 외부 디바이스로부터 종속적이라는 것을 의미한다.<br><br> 인터럽트 방식은 외부 디바이스의 신호를 검사하는 코드가 없다. 단지 외부 디바이스에서 특정 신호를 보내왔을 때 어떤 행동을 할지 정해놓은 ISR 코드만 존재할 뿐이다. avr-gcc에서는 ISR을 다음과 같이 표현한다.<pre>ISR(벡터 이름)<br>\{<br> //수행할 내용 <br>\}</pre> 즉, 인터럽트 발생 시 **어떠한 행동을 하겠다**라는 것을 적어놓은 ISR 코드만 존재할 뿐, 외부 디바이스로부터 신호가 들어왔는지 검사하는 코드는 존재하지 않는다. 이는 곧 외부 디바이스로부터 종속적이지 않고 자유롭다는 뜻이다.<br> 같은 얘기일 수도 있지만 또 하나 폴링 방식과 인터럽트 방식이 크게 다른 점은 폴링 방식은 외부 디바이스로부터 신호가 발생한 것을 **폴링 방식 코드가 있는 부분**에서만 확인할 수 있지만, 인터럽트 방식은 프로그램 실행 중이라면 언제나(관련 레지스터가 제대로 세팅되어있을 때만) 외부 디바이스로부터 신호를 확인하여 ISR을 실행할 수 있다는 것이다.\`\`\`