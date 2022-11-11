---
layout: post
title: "자바 백엔드 기술 면접 대비하기 - 1편"
author: "sungjin seo"
date: 2022-10-31 09:52:00 +0900
categories: [Job, Study]
tags: [java, question]
comments: true
pin: false
---

## 자바의 모든 클래스는 Object 클래스를 상속받습니다. 그리고 Object클래스에는 equals() 와 hashCode() 라는 메소드가 선언되어 있습니다. 이 메소드들은 각각 어떤 역할일까요? 이 둘의 차이점은 무엇일까요?

<details>
<summary>내용확인</summary>
<div markdown="1">

> <strong>equals() 객체가 가지는 값이 같은지 비교하는 역할이며 hashCode()는 객체의 메모리 번지를 이용해서 해시코드를 만들어 리턴한다. 동일성비교를 하냐 동등성비교를 하느냐의 차이가 있다.</strong>

>><strong>동일성</strong> 비교는 == 비교다. 객체 인스턴스의 주소 값을 비교한다.
>> primitive data type의 경우 ==를 통해 값 비교가 가능하다.
>>
>><strong>동등성</strong> 비교는 equals() 메소드를 사용해서 객체 내부의 값을 비교한다.
>
> equals() 구현체
>``` java
>public boolean equals(Object anObject) {
>    if (this == anObject) {
>        return true;
>    }
>    if (anObject instanceof String) {
>        String aString = (String)anObject;
>        if (coder() == aString.coder()) {
>            return isLatin1() ? StringLatin1.equals(value, aString.value)
>                              : StringUTF16.equals(value, aString.value);
>        }
>    }
>    return false;
>}
>```
> hashCode() 구현체
>``` java
>public int hashCode() {
>    int h = hash;
>    if (h == 0 && value.length > 0) {
>        hash = h = isLatin1() ? StringLatin1.hashCode(value)
>                              : StringUTF16.hashCode(value);
>    }
>    return h;
>}
>```
>
> 재정의의 필요성
>> equals()와 hashcode()를 같이 재정의해야 하는 이유
>> 위의 그림과 같이  hashCode()를 재정의 하지 않으면 서로 다른 값 객체라도 해시값이 같을 수 있다.
>>equals()를 재정의 하지 않으면 hashCode()가 만든 해시 값을 이용하여 객체가 저장된 버킷을 찾을수는 있지만
>>해당 객체가 자신과 같은 객체인지 비교할 수 없기 때문에 null을 리턴하게 된다. 따라서 원하는 객체를 찾을수 없다.
>>이런 이유때문에 둘다 재정의해서 논리적 동등 객체일 경우 동일한 해시코드가 리턴되도록 해야한다.>
>
> 여기서부터는 잘 보지 않을 내용일거 같다
>
> [ Guaidlines to override hashCode() & equals() ]
>> hashCode와 equals를 생성하기 위해서는 같은 attribute를 이용하라.(e.g. Employee id)
>>
>> equals는 일관되어야 한다. 즉, 객체가 수정되지 않았다면 항상 결과가 동일해야 한다.
>>
>> a.equals(b) == true이면, a.hashCode() == b.hashCode() 역시 true여야 한다.
>>
>> 두 메소드는 항상 함께 오버라이드 되어야 한다.
>
> [ 라이브러리를 사용한 Override ]
>>  만약 ORM을 사용하고 있는 경우라면, hashCode와 equals를 오버라이드 하는 메소드 내부에서 Getter를 사용하기를 권장한다. 그 이유는 ORM에 의해 fields가 Lazy Loaded되어, getter를 부르기 전에는 사용이 불가능할 수 있기 때문이다.
>>  예를 들어 만약 Employee 클래스의 정보가 Lazy loaded 되었다면, id에 0이 할당되어 *e1.id == e2.id*가 0==0으로 처리될 수 있기 때문이다. 하지만 이것을 e1.getId() == e2.getId()로 수정한다면 ORM에 의해 id에 값이 할당된 후에 getId()가 호출가능하므로, 오작동을 멈출 수 있다.

아래의 예제처럼 객체의 값으로 동등성을 비교하려면 hashCode()를 오버라이딩해서 동일성을 유지하고 값을 비교해야한다.
```` java
import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;
public class Employee
{
	private Integer id;
	private String firstname;
	private String lastName;
	private String department;

	//Setters and Getters

	@Override
	public int hashCode()
	{
		final int PRIME = 31;
		return new HashCodeBuilder(getId()%2==0?getId()+1:getId(), PRIME).toHashCode();
	}

	@Override
	public boolean equals(Object o) {
	if (o == null)
	   return false;

	if (o == this)
	   return true;

	if (o.getClass() != getClass())
	   return false;

	Employee e = (Employee) o;

	return new EqualsBuilder().
			  append(getId(), e.getId()).
			  isEquals();
	}
}
````
</div>
</details>

## StringBuilder 와 StringBuffer 의 차이는 무엇일까요?
<details>
<summary>내용확인</summary>
<div markdown="1">

><strong>StringBuilder</strong>는 동기화를 지원하지 않는 <mark>비동기</mark>식인 반면, <strong>StringBuffer</strong>는 <mark>동기</mark>화를 지원하여 멀티 스레드 환경에서도 안전하게 동작할 수 있습니다.

>그 이유는 StringBuffer는 메서드에서 synchronized 키워드를 사용하기 때문인데요.
>java에서 synchronized 키워드는 여러개의 스레드가 한 개의 자원에 접근할려고 할 때, 현재 데이터를 사용하고 있는 스레드를 제외하고 나머지 스레드들이 데이터에 접근할 수 없도록 막는 역할을 수행합니다.
>
> String 을 사용해야 할 때
>>"String 은 <mark>불변성</mark>을 갖는다" 라는 특징이 존재합니다.
>>그렇기 때문에 우리는 변하지 않는 문자열을 자주 사용할 경우 String 타입을 사용하는 것이 성능면에서 유리할 것 입니다.
>
>StringBuilder 를 사용 해야 할 때
>>StringBuilder는 동기화를 지원하지 않는 반면, 속도면에선 StringBuffer 보다 성능이 좋습니다.
>>그렇기 때문에 우리는 <mark>단일 스레드</mark> 환경 과 문자열의 추가, 수정, 삭제 등이 빈번히 발생하는 경우 StringBuilder를 사용하는 것이 성능면에서 유리할 것입니다.
>
>StringBuffer 를 사용해야 할 때
>>StringBuffer는 동기화를 지원하여 멀티 스레드 환경에서도 안전하게 동작할 수 있습니다.
>>그렇기 때문에 우리는 <mark>멀티 스레드</mark> 환경 과 문자열의 추가, 수정, 삭제 등이 빈번히 발생하는 경우 StringBuffer를 사용하는 것이 성능면에서 유리할 것입니다.

</div>
</details>

## 로그처리시 System.out.println 메소드는 현업에서 절대 쓰지 말라고하는 메소드인데요. 그 이유가 무엇일까요?

<details>
<summary>내용확인</summary>
<div markdown="1">

> <strong>System.out.println()을 호출하게 되면 디스크 I/O 동기화 처리가 되기 때문에 전체적인 시스템의 성능이 저하 될 수 있고, System.out.println() 으로 디버그 처리한 부분을 일일이 주석처리, 해제하는 것은 개발 및 운영의 효율을 떨어트릴 수 있다.</strong>

> 휘발된다
>> System.out.println() 은 로그가 표준 출력으로 출력된다.
>>
>>즉, 파일로 저장되지 않고 휘발된다는 의미이다. 로그는 에러가 발생한 상황을 기록하고, 추후 확인하여 문제를 진단하고, 재현하고, 고치기 위해 사용된다. 하지만 표준 출력으로 한번 출력되고 어디에도 저장되지 않으면 로그의 제 역할을 할 수 없다.
>>로그된 데이터는 실제로 기록되어야 한다. 하지만 System.out.println() 만으로는 불가능하다.
>
>에러 발생 시 추적할 수 있는 최소한의 정보가 남지 않는다
>>System.out.println() 은 인자로 전달한 문자열만을 출력한다.
>>
>>문제가 발생한 날짜, 시각 그리고 문제의 수준, 로그가 발생한 위치 등 최소한의 정보가 기록되지 않는다는 것 이다. 이런 제한적인 정보만으로는 문제를 해결하기 어려울 것 이다. 물론 이런 정보도 함께 인자로 전달한다면 충분히 에러와 장애를 추적할 수 있는 정보를 남길수야 있지만… 매번 그런 정보를 일일히 남기기엔 번거로울 것 이다.
>
> 로그 출력 레벨을 사용할 수 없다
>>로컬에서 개발할 때에는 디버깅을 위한 아주 상세한 정보가 출력되어 확인할 수 있어야한다.
>>
>>하지만, 프로덕션에서 동작하는 코드는 에러/장애가 발생할 때 문제를 진단할 수 있는 정보만을 남겨야한다. 개발시에만 사용되는 정보와 문제 상황에 대한 정보가 함께 로깅된다면 문제 해결을 위한 정작 중요한 정보를 얻기 힘들 뿐더러, 민감한 정보를 로그로 남길수도 있기 때문이다. 또한 의미없는 로그가 쌓여 서버 용량을 차지할 수도 있다.
>>따라서 로깅 라이브러리는 환경에 맞게(로컬 개발 환경, 개발 서버, 프로덕션 서버 등) 로그가 출력될 수 있도록 로그 출력 레벨이라는 기능을 제공한다. 많이 사용되는 Logback이라는 라이브러리에서는 TRACE, DEBUG, INFO, WARN, ERROR, FATAL 와 같은 레벨을 제공한다. 하지만 System.out.println() 은 이런 기능을 제공하지 않는다. 어떤 환경에서든 동일한 로그가 출력된다. 프로덕션에서 이런 로그를 제거하려면 코드를 일일히 제거하거나 주석처리하거나 별도의 조건문을 설정하는 등 번거로운 일들을 해야한다.
>>
> 성능저하의 원인이 될 수 있다
System.out.println() 의 구현을 한번 살펴보자.
> ```` java
>'/**
>  * Terminates the current line by writing the line separator string.  The
>  * line separator string is defined by the system property
>  * {@code line.separator}, and is not necessarily a single newline
>  * character ({@code '\n'}).
> */
> public void println() {
>     newLine();
> }
>````
>println() 은 newLine() 을 호출한다. newLine() 의 구현도 살펴보자.
> ```` java
> private void newLine() {
>     try {
>         synchronized (this) {
>             ensureOpen();
>             textOut.newLine();
>         // ...
>````
>synchronized 키워드가 붙어있다. 이때 newLine() 메소드는 임계영역(critical section)이 된다. 멀티 쓰레드 환경에서 A 쓰레드가 newLine() 메소드를 실행하면, 메소드는 잠기게 된다. 다른 쓰레드는 A 쓰레드가 모두 사용하고 잠금을 풀어준 뒤에서야 newLine() 메소드를 실행할 수 있다. 오버헤드가 발생하게 되는 것 이다.
>스프링을 실행하는 톰캣은 멀티 쓰레드로 동작한다. 요청이 오면 쓰레드 풀에서 쓰레드를 하나 가져와 요청을 처리한다. 그런데, System.out.println() 을 여러 쓰레드가 사용하면 그만큼 위에서 이야기한 오버헤드가 발생하고 처리가 느려질 것 이다. 따라서 실제 프로덕트의 코드에서는 System.out.println() 을 절대 사용해서는 안된다.
>
>>한 번 요청 시 5000명의 사용자를 요청하고, 처리 과정에서 응답시간이 20초 걸리는 사이트가 있는데, 원인을 알아보니 5000명의 정보를 다 System.out.println()으로 처리하고있던 것이다. 이는 System.out.println()을 줄임으로써 응답시간이 6초까지 줄었다. - 이상민, 자바 성능 튜닝이야기, 인사이트, 2013
</div>
</details>

## ArrayList 는 내부적으로 어떻게 구현되어있을까요?

<details>
<summary>내용확인</summary>
<div markdown="1">

> <strong>배열로 구성되어 있으며 배열의 사이즈는 동적으로 조정됩니다.</strong>

><strong>ArrayList</strong>
>ArrayList는 배열을 좀 더 편하게 쓸수있도록 Java에서 제공해주는 Class입니다.
><br>일반 배열과는 다르게 메모리가 가능한한 추가할 수 있고 삭제에 대해서도 해당 index를 비워두기만 하는게 아니라 재정렬해주는 기능을 기본으로 제공해주고 있습니다.
>
><strong>interface와 내부 변수 확인</strong>
>``` java
> public class ArrayList<E> extends AbstractList<E>
> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
> ```
> ArrayList는 AbstractList를 extends 받았고 List, RandomAccess, Cloneable, Serializable을 implements 받았습니다.
>
> RandomAccess는 index를 통해 직접 바로 접근 할수 있는 자료구조라는 의미입니다.
>
>그리고 아래가 ArrayList를 사용했을 때 실제로 데이터가 담기는 내부 변수입니다.
>
>``` java
>/**
> * The array buffer into which the elements of the ArrayList are stored.
> * The capacity of the ArrayList is the length of this array buffer. Any
> * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
> * will be expanded to DEFAULT_CAPACITY when the first element is added.
> */
>transient Object[] elementData; // non-private to simplify nested class access
>```
> 여기서 우리는 ArrayList는 우리가 아는 일반적인 배열로 구현되어있다는 것을 알 수 있습니다. 그리고 주석을 보면 해당 배열의 크기는 우리가 처음 add를 할 때 정해진다고 써있습니다
>
> 생성자
List<String> list = new ArrayList<>();
ArrayList를 사용하고 싶으면 보통 우리는 위와같이 선언하고 사용합니다. 이렇게 선언했을때 ArrayList에서 일어나는 내부로직을 알아보도록 하겠습니다.
>
>``` java
>/**
> * Constructs an empty list with an initial capacity of ten.
> */
>public ArrayList() {
>    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
>}
>```
>일반적으로 사용하는 별도의 파라미터가 없는 생성자입니다. Array에 값을 대입하는걸 알 수 있습니다. DEFAULTCAPACITY_EMPTY_ELEMENTDATA의 값은 비어있는 Array값입니다. 즉, size 0의 Array가 만들어질 것입니다.
>
>``` java
>/**
> * Constructs an empty list with the specified initial capacity.
> *
> * @param  initialCapacity  the initial capacity of the list
> * @throws IllegalArgumentException if the specified initial capacity
> *         is negative
> */
>public ArrayList(int initialCapacity) {
>    if (initialCapacity > 0) {
>        this.elementData = new Object[initialCapacity];
>    } else if (initialCapacity == 0) {
>        this.elementData = EMPTY_ELEMENTDATA;
>    } else {
>        throw new IllegalArgumentException("Illegal Capacity: "+
>    initialCapacity);
>    }
>}
>```
>instance를 생성할 때 생성자에 int형의 파라미터를 넘길 수 있습니다. 파라미터의 값에 따라서 바로 초기화 되는것을 알 수 있습니다.
>
><h3>add</h3>
>#add(Object)는 ArrayList의 제일 마지막에 값을 하나 추가하는 method입니다. 중간에 삽입하고 싶으시다면 #add(index, Object)를 사용하시면 됩니다. #add(Object)를 코드로 한번 알아보도록 하겠습니다.
>``` java
>public boolean add(E e) {
>    ensureCapacityInternal(size + 1);  // Increments modCount!!
>    elementData[size++] = e;
>    return true;
>}
>```
>실제 코드를 실행했을 때 발생하는 로직입니다. 가장먼저 내부 Object[]배열의 크기를 재산정합니다. 그리고 해당배열에 값을 입력 후 size값을 증가 그리고 return합니다.
>``` java
>private void ensureCapacityInternal(int minCapacity{
>    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
>}
>```
>현재 Object[] 배열과 size + 1 의 값을 이용하여 배열의 크기를 재산정(#calculateCapacity) 한 후 적용(#ensureExplicitCapacity)한다는 것을 알 수 있었습니다. 배열의 크기는 어떻게 재산정하는지 보도록 하겠습니다.
>
>``` java
>private static int calculateCapacity(Object[] elementData, int minCapacity) {
>    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
>        return Math.max(DEFAULT_CAPACITY, minCapacity);
>    }
>    return minCapacity;
>}
>```
>ArrayList가 현재 빈배열(초기화 상태)라고 하면 기본크기(DEFALUT_CAPACITY = 10)와 입력된 값 중 큰 값을 return하며 그게 아니라면 size + 1의 크기를 return합니다.
>
>``` java
>private void ensureExplicitCapacity(int minCapacity) {
>    modCount++;
>
>    // overflow-conscious code
>    if (minCapacity - elementData.length > 0)
>        grow(minCapacity);
>}
>```
>#calculateCapacity에서 return 받은 값을 minCapacity로 사용합니다. 그리고 해당 값이 Object[]의 크기보다 크다면 #grow라는 메서드를 호출하고 있습니다.
>
>``` java
>/**
>  * Increases the capacity to ensure that it can hold at least the
>  * number of elements specified by the minimum capacity argument.
>  *
>  * @param minCapacity the desired minimum capacity
> */
>private void grow(int minCapacity) {
>    // overflow-conscious code
>    int oldCapacity = elementData.length;
>    int newCapacity = oldCapacity + (oldCapacity >> 1);
>    if (newCapacity - minCapacity < 0)
>        newCapacity = minCapacity;
>    if (newCapacity - MAX_ARRAY_SIZE > 0)
>        newCapacity = hugeCapacity(minCapacity);
>     // minCapacity is usually close to size, so this is a win:
>        elementData = Arrays.copyOf(elementData, newCapacity);
>}
>```
>  로직을 보니 이 메서드가 실제로 배열의 크기를 재산정하고 기존(old)에 있던 정보를 새로운 배열(new)에 넣는 메서드입니다. 로직을 보면 newCapacity가 새로 만들어질 배열의 크기입니다. 이 값은 oldCapacity + (oldCapacity >> 1);입니다. 즉 기존 크기에서 50%의 크기를 더해서 새로운 크기를 산정하는 것입니다. 만약 해당크기가 minCapacity보다 작다면 minCapacity 값으로 재산정됩니다. 한번도 add하지 않았을때 10의 크기를 가지게 되는것입니다.
> 이런로직으로 ArrayList는 add 메서드가 호출될 때 크기를 재산정하고 실제 데이터가 들어가 있는 크기인 size index에 값을 넣은 후 size를 1증가 시킨후 return합니다.
>
>위의 로직에 따라 크기를 재산정 할때는 원래크기만큼 새로운 배열에 복사를 해야하므로 시간복잡도 O(n)을 가지며 추가할 때 O(1)을 가지게 되는것을 알 수 있었습니다. 크기를 overflow하지 않아 재산정을 하지 않을때는 O(1), 재산정이 필요하면 O(n)으로 정의할 수 있을 것입니다.
><h3>remove</h3>
>반대로 remove를 통해 들어있는 데이터를 제거하는 메서드를 한번 보도록 하겠습니다. remove는 E remove(int index)와 boolean remove(Object o)으로 index 기준으로 삭제와 value에 대한 삭제가 있습니다. 저희는 index기준의 삭제를 알아보도록 하겠습니다.
>
>``` java
>/**
>  * Removes the element at the specified position in this list.
>  * Shifts any subsequent elements to the left (subtracts one from their
>  * indices).
>  *
>  * @param index the index of the element to be removed
>  * @return the element that was removed from the list
>  * @throws IndexOutOfBoundsException {@inheritDoc}
> */
>public E remove(int index) {
>    rangeCheck(index);
>    modCount++;
>    E oldValue = elementData(index);
>
>    int numMoved = size - index - 1;
>    if (numMoved > 0)
>        System.arraycopy(elementData, index+1, elementData, index, numMoved);
>        elementData[--size] = null; // clear to let GC do its work
>    return oldValue;
>}
>```
>remove 메서드를 실행하면 가장 먼저 입력받은 index가 적절한 값인지 체크합니다. 그리고 삭제되는 Object의 값을 가져와서 변수에 담습니다. 그 후 arraycopy를 이용해 삭제되는 부분 + 1 ~ 마지막까지의 영역을 삭제되는 부분의 시작점을 기준으로 해서 옮깁니다. 그러면 삭제될 부분의 값은 다음 index의 값으로 겹쳐서 덮여 쓰여지게 됩니다. 그리고 size의 마지막 index는 size - 1의 값과 중복되기 때문에 null처리를 하여 GC가 삭제할 수 있도록 합니다. 그 후 임시 변수에 담아두었던 삭제된 값을 리턴합니다.
>``` java
>private void rangeCheck(int index) {
>    if (index >= size)
>        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
>}
>```
>index범위가 적절한지 체크하는 부분입니다. size보다 index값이 크거나 같으면 범위를 넘은것으로 Exception이 발생합니다.
>
>``` java
>E elementData(int index) {
>    return (E) elementData[index];
>}
>```
>배열의 값을 가져오는 메서드를 호출하여 oldValue에 넣는 메서드입니다.
>``` java showLineNumbers
>int numMoved = size - index - 1;
>if (numMoved > 0)
>    System.arraycopy(elementData, index+1, elementData, index, numMoved);
>elementData[--size] = null; // clear to let GC do its work
>
>return oldValue;
>
>```
>![img][arraylist_remove]
>
>중요한 부분이므로 이미지로 다시 설명드리겠습니다. 코드에 따르면 위와 같이 삭제되는 index의 다음부터 복사하여 삭제되는 index에 붙여 넣습니다. 그렇게 되면 6번과 7번 index가 중복이 일어납니다. 따라서 elementData[--size] = null; 코드를 통해 7번(마지막) index의 값을 null로 변경해 주는것입니다.
>삭제를 할때 우리는 index + 1에서 부터의 값을 index부터 시작하게끔 복사한다는 것을 알 수 있었습니다. 그때 O(n)의 시간복잡도를 가지며 마지막 값을 null로 변경해줍니다. 이때는 O(1)을 가지겠죠. 삭제에 대해서는 항상 시간복잡도 O(n)을 가진다는 것을 알 수 있습니다.
>
</div>
</details>

## 스레드는 왜 써야하는 것일까요?
<details>
<summary>내용확인</summary>
<div markdown="1">

><strong>⚡ 메모리 절약</strong><br>
>>OS마다 다르지만, 무슨 작업을 수행하려고 할 때 JVM은 적어도 32~64MB 물리 메모리 점유한다.
근데 스레드는 1MB 이내의 메모리만 점유한다. 그래서 스레드를 '경량 프로세스'라고도 부른다.

><strong>⚡ 프로세스 콘텍스트 스위칭(Context Switching)에 비해 오버헤드 절감</strong><br>
>>멀티 프로세스로 실행되는 작업을 멀티 스레드로 실행하게 되면 프로세스를 생성하여 자원을 할당하는 과정도 줄어들뿐더러 프로세스를 콘텍스트 스위칭(Context Switching)하는 것보다 오버헤드를 더 줄일 수 있게 된다.

><strong>⚡ 작업들 간 통신 비용 절감</strong>
>>프로세스 간의 통신 비용보다 하나의 프로세스 내에서 여러 스레드 간의 통신 비용이 훨씬 적으므로 작업들 간의 통신 부담을 줄일 수 있게 된다.
>
>![img][jvm_thread]
>
>NEW
>>스레드 객체는 생성됐지만 아직 스레드 대기열 큐에 올라가지 않았고 start() 되지 않은 상태이다.
>
>RUNNABLE
>>start()가 호출돼 실행 대기 중인 상태이다. run() 이 호출되면 running 상태가 된다.
>
>WAITING
>>일시정지 상태이며 다른 스레드의 통지(nofity)를 기다리는 상태이다.
>
>TIMED_WAITING
>>일시정지 상태이며 일정 시간 동안 기다리는 상태이다.
>
>TERMINATED
>>스레드 실행을 마치고 종료한다. run() 이 끝나면 소멸된다.
>
>BLOCK
>>일시정지 상태이며 사용하려는 객체의 모니터 락(monitor lock)이 풀리기를 기다리는 상태이다.
>
>참고로 자바, 스프링에서는 스레드를 내부적으로 관리해주면서 스레드 종료까지 시켜주진 않는다.
>이는 개발자가 직접 처리해야 하는 것이지 시스템에 맡기는 것이 아니다.
</div>
</details>

## 0이 들어있는 변수에 10개의 스레드가 동시에 접근해서 ++ 연산을 하면 우리 예상과 다르게 10이 나오지 않습니다. 왜 그럴까요?


## 자바에서 동시성과 관련된 예약어를 모두 말씀해주세요.
>Synchronized https://dev-cool.tistory.com/5
><br>Volatile https://dev-cool.tistory.com/30
><br>https://techblog.woowahan.com/2550/

## Blocking IO와 Non-Blocking IO 의 차이를 말씀해주세요.

## Serializable 은 무엇일까요?
<details>
<summary>내용확인</summary>
<div markdown="1">

>자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과
바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)을 아울러서 이야기합니다.
<br>시스템적으로 이야기하자면 JVM(Java Virtual Machine 이하 JVM)의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과
직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 같이 이야기합니다.

</div>
</details>

## ArrayList와 LinkedList의 차이는 무엇일까요?

[arraylist_remove]: https://github.com/sungjinseo/image-repository/blob/master/blog/2202-10-31-JAVA%EC%9D%B8%ED%84%B0%EB%B7%B0/arraylist_remove.png?raw=true
[jvm_thread]: https://github.com/sungjinseo/image-repository/blob/master/blog/2202-10-31-JAVA%EC%9D%B8%ED%84%B0%EB%B7%B0/jvm_thread.png?raw=true
